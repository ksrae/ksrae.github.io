---
title: "Make Matenenance or Offline Notice using PWA"
date: 2019-07-08 16:50:00 +0900
comments: true
categories: angular
tags: [pwa, maintenance, offline]
---



How to Implement Maintenance or Offline Notices Using a PWA

## Objective

- Implement an easy way to display maintenance messages within a PWA.
- Ensure the app remains functional while informing users about their offline status.

## Maintenance Mode Implementation

To quickly display maintenance messages, we use a `.json` file that is read upon application load. This approach allows for dynamic updates without requiring a full application redeployment.

## Challenges

When developing a PWA, `.json` files can be cached, leading to two primary issues:

- **Delayed Updates:** Updates to the `.json` file might not be immediately reflected, potentially causing users to experience outdated maintenance information.
- **Offline Data Integrity:** Even in offline mode, the cached `.json` file will be read, preventing the display of specific offline messages.

## Solution

Prevent the `.json` file from being cached by excluding it from the `ngsw.json` configuration.

- **Initial Load Strategy:**
- Read the `.json` file during the initial module load.
- On success, store the result in a variable or subject for later use.
- On failure (e.g., network error), handle the exception in a catch event.
- **Component Integration:**
- Subscribe to the variable or subject within the component's `onInit` lifecycle hook to react to changes in the maintenance status.

## Outcome

- Maintenance messages are applied immediately, independent of application updates.
- Custom offline messages and network troubleshooting instructions can be displayed.

## Source Code

```json
// maintenance.json
{
    "maintenance": false
}
```

```json
// ngsw.json (maintenance.json is excluded from caching)
{
  "configVersion": 1,
  "index": "/index.html",
  "assetGroups": [
    ...
    {
      "name": "assets",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "urls": [
        "/assets/icons/icon-128x128.png",
      ],
      "patterns": []
    }
  ],
  "dataGroups": [],
  "hashTable": {
    "/assets/icons/icon-128x128.png": "dae3b6ed49bdaf4327b92531d4b5b4a5d30c7532",
    ...
  },
  "navigationUrls": [
    ...
  ]
}
```

```tsx
// app.module.ts

export function getConfig(config: AppService) {
  return async () => await config.getMaintenance();
}
...
  providers: [
    HttpClient,
    {
      provide: APP_INITIALIZER,
      useFactory: getConfig,
      deps: [AppService],
      multi: true
    }
  ]
```

```tsx
// app.service.ts

    getMaintenance() {
        return new Promise((resolve, reject) => {
            this.http.get(`${window.location.origin}/assets/maintenance.json`, { headers: new HttpHeaders({ timeout: `${3000}`})})
            .toPromise().then(item => {
                this.config = item['maintenance'];
                resolve();
            }).catch(err => {
                this.offline = true;
                resolve();
            });
        });
    }
```

```tsx
// app.component.ts

  ngOnInit() {
    const maintenance = this.appService.config;
    const offline = this.appService.offline;
    if (maintenance) {
      alert('MAINTENANCE');
    }
    if (offline) {
      alert('offline');
    }
    
    if (this.swUpdate.isEnabled) {
      ...
    }
```

## Application

Now, to initiate maintenance, simply update the `"maintenance"` value in `maintenance.json` to `true` and deploy only that file.

When the application is offline, you can display a message indicating the offline status or instructions for checking the network connection.

You could also enhance the `"maintenance"` property by adding a time range. The client could then process the maintenance value only during the specified time window.