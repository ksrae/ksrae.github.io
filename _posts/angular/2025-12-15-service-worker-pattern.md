---
title: Service Worker Patterns in Angular - Caching & Push Notifications
date: 2025-12-15 12:00:00 +0900
comments: true
categories: angular
tags: [serviceworker]
---

This post explores advanced techniques to level up your PWA: implementing **Dynamic Caching Strategies** and handling **Push Notifications**.

## Dynamic Caching Strategies

While static caching serves assets well, dynamic content requires more sophisticated approaches to balance performance and data accuracy. Here are three common strategies you can implement directly in your Service Worker script.

### 1. Cache First Strategy

This strategy is ideal for non-critical resources or assets that rarely change (e.g., images, fonts). It checks the cache first; if the resource exists, it's served immediately. The network is only used as a fallback.

```jsx
// sw.js (Service Worker)
const CACHE_NAME = 'asset-cache-v1';

self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    (async () => {
      // 1. Try to find the request in the cache
      const cachedResponse = await caches.match(event.request);
      if (cachedResponse) {
        return cachedResponse;
      }

      // 2. If not in cache, fetch from network
      return fetch(event.request);
    })()
  );
});
```

### 2. Network First Strategy

For critical data where freshness is paramount (e.g., account balance, live scores), use Network First. It attempts to fetch from the server. If the request fails (offline), it falls back to the cache.

```jsx
// sw.js
self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    (async () => {
      try {
        // 1. Try to fetch from the network first
        const networkResponse = await fetch(event.request);
        return networkResponse;
      } catch (error) {
        // 2. If network fails (offline), fallback to cache
        console.warn('Network request failed, serving from cache.', error);
        return caches.match(event.request);
      }
    })()
  );
});
```

### 3. Stale-While-Revalidate Strategy

This is often the best of both worlds for content feeds. It serves the cached version immediately for speed but updates the cache in the background from the network for the next visit.

```jsx
// sw.js
const DYNAMIC_CACHE = 'dynamic-data-v1';

self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    (async () => {
      const cachedResponse = await caches.match(event.request);

      // 1. Fetch from network and update cache in the background
      const networkFetch = fetch(event.request).then(async (res) => {
        const cache = await caches.open(DYNAMIC_CACHE);
        cache.put(event.request, res.clone()); // Update cache
        return res;
      });

      // 2. Return cached response immediately if available, else wait for network
      return cachedResponse || networkFetch;
    })()
  );
});
```

## Handling Push Notifications

Service Workers enable us to send notifications to users even when the browser tab is closed. In Angular, instead of using the raw `window` API, we can leverage the `SwPush` service provided by `@angular/service-worker` for a cleaner implementation.

### Step 1: Subscribing to Push Notifications (Angular Component)

We use `SwPush` to request a subscription. This handles the permission prompt automatically if not already granted.

```jsx
// notification.component.ts
import { Component, inject } from '@angular/core';
import { SwPush } from '@angular/service-worker';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-notification',
  standalone: true,
  template: `
    <button (click)="subscribeToNotifications()" [disabled]="!isEnabled">
      Enable Notifications
    </button>
  `
})
export class NotificationComponent {
  private swPush = inject(SwPush);
  private http = inject(HttpClient);

  readonly VAPID_PUBLIC_KEY = 'YOUR_VAPID_PUBLIC_KEY'; // From your backend/VAPID generator
  
  get isEnabled() {
    return this.swPush.isEnabled;
  }

  subscribeToNotifications() {
    this.swPush.requestSubscription({
      serverPublicKey: this.VAPID_PUBLIC_KEY
    })
    .then(sub => {
      // Send the subscription object to the backend
      this.http.post('/api/notifications/subscribe', sub).subscribe({
        next: () => console.log('Successfully subscribed to notifications!'),
        error: err => console.error('Could not subscribe', err)
      });
    })
    .catch(err => console.error('Could not subscribe to notifications', err));
  }
}
```

### Step 2: Handling Incoming Push Notifications (Service Worker)

Finally, listen for the `push` event in the Service Worker to display the notification.

```jsx
// sw.js
self.addEventListener('push', (event: PushEvent) => {
  if (!event.data) return;

  const payload = event.data.json();
  const title = payload.title || 'New Notification';
  const options = {
    body: payload.body,
    icon: '/assets/icons/icon-192x192.png',
    badge: '/assets/icons/badge-72x72.png',
    data: payload.url // Store URL to open on click
  };

  event.waitUntil(
    self.registration.showNotification(title, options)
  );
});
```

## Conclusion

By implementing dynamic caching strategies like Stale-While-Revalidate, we can significantly improve the perceived performance of our Angular applications. Furthermore, utilizing Angular's `SwPush` service simplifies the complexity of managing Push API subscriptions.<br/><br/>

Mastering these Service Worker patterns transforms a standard web app into a robust PWA.


## Reference
[Service Workers in Angular: Advanced Techniques (Part 2)](https://levelup.gitconnected.com/service-workers-in-angular-advanced-techniques-part-2-e359436625cb)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Service-Worker-%EC%8B%AC%ED%99%94-%EC%BA%90%EC%8B%B1-%EC%A0%84%EB%9E%B5%EA%B3%BC-%ED%91%B8%EC%8B%9C-%EC%95%8C%EB%A6%BC-%EA%B5%AC%ED%98%84)
<br/>