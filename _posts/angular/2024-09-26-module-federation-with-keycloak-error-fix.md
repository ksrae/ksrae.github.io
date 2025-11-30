---
title: "Error Fix for Keycloak with Angular Module Federation"
date: 2024-09-26 13:59:00 +0900
comments: true
categories: angular
tags: [standalone, keycloak, module-federation]
---

Today, we will define what Keycloak and Module Federation are and explore how to resolve encountered errors.

# Keycloak

Keycloak is an integrated authentication system and an open-source software that provides Identity and Access Management (IAM). It primarily supports Single Sign-On (SSO), preventing users from undergoing repetitive login processes across multiple applications.

- **User Management:** Keycloak supports various authentication methods and offers centralized user management. This allows users to access multiple services with a single account.
- **Enhanced Security:** It supports modern security protocols such as OAuth2 and OpenID Connect, securely handling authentication between applications and services.
- **Easy Customization:** Keycloak provides a flexible solution for developers, allowing customization based on its out-of-the-box features.
- **Open Source:** As an open-source project, Keycloak benefits from community support and allows direct source code modification if needed.

# Module Federation

Module Federation, a feature of Webpack 5, facilitates the easy implementation of micro-frontend architectures. It allows sharing and dynamic loading of configurations between multiple independent application modules.

- **Independent Build and Deployment:** Each module can be developed and deployed independently, ensuring that changes to specific features do not affect the entire application. This enables a more efficient CI/CD pipeline.
- **Reusable Common Code:** Module Federation allows sharing common libraries or components between applications, reducing redundant code and providing a consistent user experience. For example, a design system shared across multiple applications can be easily built.
- **Real-Time Updates:** When a common component is updated, applications can automatically use the latest version at runtime without redeployment, offering significant convenience in version management.
- **Easy Scalability:** Each module operates independently, allowing the system to be easily scaled as needed. This reduces the impact on the overall application's performance, even as specific features or services increase.

# Resolving Errors in Angular 17

## Environment

- Angular with standalone (17.0.1)
- Angular module federation (17.0.1)
- nx (@nx/js 17.1.3)
- keycloak-angular (15.0.0)
- keycloak-js (22.0.5)

Encountering the "WEBPACK_IMPORTED_MODULE_5 is not a constructor" error during the integration of Keycloak in an Angular 17 environment can stem from various factors. A systematic approach to inspection and correction is essential. The following outlines the causes of this issue along with effective solutions.

## Version Compatibility Check

First, ensure that the versions of libraries such as `keycloak-angular` and `keycloak-js` used with Angular 17 are compatible. Using incompatible library versions can lead to unintended behavior or errors. Follow these steps to verify:

### Check Currently Installed Package Versions

Use the command `npm list keycloak-angular keycloak-js` to check the installed versions. Verify that these versions are compatible with Angular 17. Refer to the official Angular documentation or the GitHub page of the packages to find recommended versions.

### Update Dependencies

If the versions are incompatible, update to appropriate versions using commands like `npm install keycloak-angular@<desired_version>` or `npm install keycloak-js@<desired_version>`.

## `angular.json` File Configuration

Verify that the build settings in the `angular.json` file are correctly configured. The `allowedCommonJsDependencies` property, in particular, allows the use of CommonJS modules and should be configured as follows:

```json
"allowedCommonJsDependencies": [
  "base64-js",
  "js-sha256",
  "keycloak-js"
]
```

This setting explicitly allows the CommonJS modules used by the Angular application, helping to minimize potential conflicts in module federation.

## Webpack Configuration Check

Specific Keycloak-related settings are required. For Module Federation purposes, it is crucial to add Keycloak to the shared and skipped lists. Adjust the Webpack configuration file as follows:

```jsx
const sharedConfig = {
  singleton: false,
  strictVersion: false,
  requiredVersion: 'auto'
};

const skipList = [
  '@angular-architects/module-federation',
  'tslib',
  'zone.js',
  'keycloak-js'
];

shareAll(sharedConfig, skipList);
```

This configuration ensures that Keycloak's dependencies are not handled by Module Federation, preventing conflicts between modules and precluding potential runtime errors.

## Debugging and Additional Logging

If the problem persists, adding additional logging to the part where the error message occurs is an important method to identify the root cause of the problem. For example, you can separate each step into a helper function when initializing Keycloak to check which step is causing the error:

```jsx
const initKeycloak = async () => {
  try {
    const keycloak = new Keycloak();
    await keycloak.init({
      onLoad: 'login-required',
      checkLoginIframe: false,
    });
    console.log('Keycloak initialized successfully');
  } catch (error) {
    console.error('Error initializing Keycloak: ', error);
  }
};
```

Adding logging in this format is very helpful in diagnosing the exact location and reason for the error.

## Conclusion

Through these steps, the "WEBPACK_IMPORTED_MODULE_5 is not a constructor" issue can be systematically resolved. Based on each check item and adjustment, effectively integrate the Keycloak and Angular environments to build a stable application.

# References

[MFE WEBPACK_IMPORTED_MODULE_5 is not a constructor #465](https://github.com/mauriciovigolo/keycloak-angular/issues/465)