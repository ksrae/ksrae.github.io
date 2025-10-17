---
title: "Angular Devtool in Chrome DevTools"
date: 2022-05-02 14:02:00 +0900
comments: true
categories: angular
tags: [chrome, tool]
---

This article explores how to add and utilize the Angular tab within Chrome's Developer Tools (DevTools).

## Adding the Angular Tab to Chrome DevTools

Follow these steps to integrate the Angular tab into your Chrome DevTools:

- Navigate to the [Chrome Web Store](https://chrome.google.com/webstore/category/extensions) and search for "Angular DevTools." Alternatively, click on the [Angular DevTools](https://chrome.google.com/webstore/detail/angular-devtools/ienfalfjdbdpebioblfackkekamfmbnh) link directly.
- Click "Install" to complete the installation process. Upon successful installation, the button label will change to "Remove from Chrome."
- After opening a new Chrome window or restarting Chrome, you will find the Angular tab integrated within your Chrome DevTools panel.

## Using Angular DevTools

Let's examine the prerequisites and usage of Angular DevTools.

### Prerequisites

Angular DevTools is specifically designed to analyze applications developed using the Angular framework. Note the following requirements for proper functionality:

- It exclusively supports Angular-based applications.
- It necessitates Angular version 9 or later with Ivy enabled.
- Analysis is limited to development (or dev) environments. Projects built for production are not supported.

### Usage Instructions

The left panel displays a component hierarchy. Clicking on a component reveals its corresponding template. The right panel showcases the component's properties:

- Input and Output variables
- Component variables
- Variables from services utilized by the component

A key advantage of this tool is its ability to inspect properties linked to services, as well as state management variables. This allows developers to directly observe the current values of these variables.<br/>

Another significant benefit is real-time value reflection. For example, if a variable stores values that change every second, Angular DevTools reflects these changes instantaneously.<br/>

By readily visualizing the variables associated with each component, developers can significantly reduce their reliance on console-based debugging.<br/>

For comprehensive information, refer to the official Angular documentation.

## Resources

- [Angular DevTools Overview](https://angular.io/guide/devtools)