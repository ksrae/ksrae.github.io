---
title: "Deploy Angular Universal Project from GitHub to Heroku"
date: 2019-07-29 12:26:00 +0900
comments: true
categories: angular
tags: [heroku, deploy]
---

Let's explore how to deploy an Angular Universal project uploaded to GitHub to Heroku.

While the deployment process itself isn't overly complex, I'll address some areas where clear explanations were lacking, which prompted me to consolidate this guide.

## Project Creation

- First, sign up for Heroku.
- Click "New" to create a new project.
- Select your newly created project.

## Dyno Formation Verification

- Navigate to the "Overview" tab.
- Examine the Dyno formation. Note that the free version typically defaults to `npm start`.

```
This app is using free dynos

web npm start ON
```

## Deploy Configuration

- Go to the "Deploy" tab.
- In the "Deployment Method" section, choose "GitHub."
- Under "App connected to Github," select the repository you intend to deploy.
- Choose between "Automatic deploys" and "Manual deploy." If you're comfortable with Git, "Manual" might be preferable. Otherwise, "Automatic" allows you to select a branch for automatic deployments. I've chosen "Automatic" for this demonstration.

## `package.json` Modification

Since modifying Dynos is restricted in the free Heroku tier, we'll adjust the `scripts` section within the `package.json` file. If Dyno modifications were possible, you could directly specify the execution commands in the Dyno itself, rendering this section unnecessary.

Here's a typical `package.json` structure:

```json
{
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e",
    "compile:server": "webpack --config webpack.server.config.js --progress --colors",
    "serve:ssr": "node dist/server",
    "build:ssr": "npm run build:client-and-server-bundles && npm run compile:server",
    "build:client-and-server-bundles": "ng build --prod && ng run ssr01:server:production --bundleDependencies all"
  }
}
```

### `start` Modification

Modify the `start` script to enable Heroku to execute `server.js`. Typically, Angular Universal projects generate a `server.js` file within the `dist` folder after the build process.  Therefore, set the value to `dist/server`.

```json
"start": "node dist/server"
```

### `heroku-postbuild` Addition

Add a `heroku-postbuild` entry. Heroku checks for both `heroku-postbuild` and `build` during the build process. While you could modify the `build` script, we'll add `heroku-postbuild` and remove the `build` script for this example.

```json
"heroku-postbuild": "npm run build:ssr"
```

This executes the Angular Universal build command.

### `deploy` Addition (Optional)

This wasn't necessary for my project, but I've included it as some resources describe its usage. It's likely applicable when manually triggering the deploy command.

```json
"deploy": "npm run build:ssr && git subtree push --prefix dist heroku master"
```

### Modified `package.json`

```json
{
  "scripts": {
    "ng": "ng",
    "start": "node dist/server",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e",
    "compile:server": "webpack --config webpack.server.config.js --progress --colors",
    "serve:ssr": "node dist/server",
    "build:ssr": "npm run build:client-and-server-bundles && npm run compile:server",
    "build:client-and-server-bundles": "ng build --prod && ng run searchword:server:production --bundleDependencies all",
    "deploy": "npm run build:ssr && git subtree push --prefix dist heroku master",
    "heroku-postbuild": "npm run build:ssr"
  }
}
```

## Upload / Build Execution

- If using "Automatic deploys," the build will start automatically when you push changes to the selected branch.
- If using "Manual deploy," the build starts when you click the "Deploy" button.

You can monitor the build progress in the "Deploy" tab or the "Overview" tab.

## Execution Verification

Click the "Open App" button in the upper right corner to view your deployed application.

## Log Access and CLI Execution

If errors occur, you can examine the logs or execute commands directly from the console.

- Click "More" in the upper right corner.
- Select "View logs" to examine the application logs.
- Select "Run console" to test commands directly.

## Heroku Free Services Shutdown

Note: Heroku's free services were phased out starting in August 2022. Information in this guide pertaining to the free tier may no longer be applicable.

## Reference
- [How to Deploy Angular Application to Heroku - Olutunmbi Banto - Medium](https://medium.com/@hellotunmbi/how-to-deploy-angular-application-to-heroku-1d56e09c5147)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Universal-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EB%A5%BC-Github%EC%97%90%EC%84%9C-Heroku%EB%A1%9C-%EB%B0%B0%ED%8F%AC)
<br/>