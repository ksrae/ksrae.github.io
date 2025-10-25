---
title: "How to handle browser close (unload event)"
date: 2022-12-01 10:47:00 +0900
comments: true
categories: angular
tags: [unload, beforeunload]
---

> This article explores how to handle browser close events, providing examples and a structured overview of the relevant events.
> 

<br/>

# Beforeunload & Unload Events

When a browser is closed, two events are triggered: `beforeunload` and `unload`. Let's examine each of these events in detail.

<br/>

## beforeunload

The `beforeunload` event is fired before the document unloads. At this point, the document is still visible and the event can be cancelled.

When this event occurs, a confirmation event is triggered, allowing you to either proceed with the unload or cancel it.

Note that to trigger the confirmation event, you must implement `preventDefault()` or `return false`.

<br/>

## unload

The `unload` event fires after the `beforeunload` event. It is generally triggered when the document is being unloaded. Key characteristics include:

- All resources still exist.
- Resources exist but are no longer visible to the user.
- UI events (e.g., `open`, `alert`, `confirm`) do not function.
- The `unload` event cannot be cancelled under any circumstances.

<br/>

# Angular Example

In Angular, you can handle these events using the `@HostListener` decorator.

<br/>

## beforeunload

### Handling in a Separate Function

```tsx
  @HostListener('window:beforeunload', ['$event'])
  beforeunload(e: any) {
    return false;
  }
```

### Handling in OnDestroy

```tsx
  @HostListener('window:beforeunload')
  ngOnDestroy() {
    console.log('destroy');
    return false;
  }
```

<br/>

# Avoiding the unload Event

It is generally recommended to avoid using the `unload` event due to the following reasons:

- On mobile, if a user switches to another app and then closes the browser through the app manager, the `unload` event may not be triggered.
- The `bfcache` is not triggered after an `unload` event.
- Some pages with `unload` events will not have `bfcache` applied, resulting in poor performance.
- Some browsers disable the `unload` event due to `bfcache` issues.

<br/>

# Why Alert, Confirm Are Not Working?

According to HTML specifications, `window.alert`, `window.confirm`, and `window.prompt` events are ignored within `unload` handlers.

Furthermore, most modern browsers have enhanced security measures in place, and you can no longer customize the browser's confirmation message.

<br/>

# Why Events Might Not Fire

The `beforeunload` or `unload` events should be triggered under the following circumstances:

- Closing the browser
- Closing the tab
- Refreshing the tab (using F5, the browser's refresh button, or script-initiated refresh)
- Navigating back or forward (using the browser's buttons or script-initiated navigation)

However, you might find that these events do not always work as expected during implementation and testing.

This is because modern browsers will immediately close a tab or browser without firing the `beforeunload` or `unload` events if there has been no user activity on the page.

Therefore, to ensure these events are triggered, users must interact with the page (even just dragging the screen) before closing it.

<br/>

# Alternative Events

## visibilitychange

The `visibilitychange` event detects and reports all changes in the browser's visibility state.

However, it can only detect minimization (i.e., hidden) and visibility changes. It cannot detect when the browser is closed. Therefore, this event is not suitable as a direct replacement for `unload`.

```tsx
  @HostListener('document:visibilitychange', ['$event'])
  visibilitychanges() {
    console.log(document.visibilityState);
    return false;
  }
```

<br/>

## pagehide

The `pagehide` event is a recommended alternative to the `unload` event.

The primary reason for this recommendation is the behavior of the `bfcache`. Using `pagehide` does not affect `bfcache` and, therefore, does not negatively impact page performance.

Many developers recommend this event as an alternative. The `pageshow` event is the recommended alternative to the `load` event.

Note that this event, as an alternative to `unload`, cannot be cancelled.

```tsx
  @HostListener('window:pagehide', ['$event'])
  pageHide(e: any) {
    e.preventDefault();
  }
```

<br/>

# Using beforeunload and pagehide Together

Typically, when the `beforeunload` event is triggered, the `pagehide` event is not.

However, during a refresh, the `pagehide` event is triggered *after* the `beforeunload` event (similar to how the `unload` event behaves).

Therefore, you can use these events together if you have logic that should only be executed during a refresh.

```tsx
  @HostListener('window:pagehide', ['$event'])
  pageHide(e: any) {
    console.log('pagehide');
  }

  @HostListener('window:beforeunload', ['$event'])
  beforeunload(e: any) {
    console.log('beforeunload')
    return false;
  }
```

<br/>

# Reference

[unload event](https://developer.mozilla.org/en-US/docs/Web/API/Window/unload_event)

[bfcache](https://web.dev/bfcache/#only-add-beforeunload-listeners-conditionally)

[custom confirmation message is not working](https://stackoverflow.com/questions/38879742/is-it-possible-to-display-a-custom-message-in-the-beforeunload-popup)