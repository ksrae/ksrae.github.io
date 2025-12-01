---
title: "angular project on electron"
date: 2019-09-16 10:19:00 +0900
comments: true
categories: angular
tags: [electron, error]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Electron%EC%97%90%EC%84%9C-%EC%89%BD%EA%B2%8C-Angular-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EB%A1%9C%EB%93%9C)
<br/>

## Setting up Electron

Electron can be easily installed from npm.

```
npm i electron@latest
```

## Building the Angular Project

To run an Angular project within Electron, you first need to build the project to generate the `dist` folder.

```
ng build
```

## Creating `main.js`

Electron executes a JavaScript file that loads the HTML. Therefore, you need to create a `main.js` file. Here's an example of what `main.js` should look like:

```jsx
const {app, BrowserWindow} = require('electron')
const url = require("url");
const path = require("path");

let mainWindow

function createWindow () {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  mainWindow.loadURL(/*url or file name*/)

  mainWindow.on('closed', function () {
    mainWindow = null
  })
}

app.on('ready', createWindow)

app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

app.on('activate', function () {
  if (mainWindow === null) createWindow()
})

```

## Loading the Project

You can either load the Angular project from a local file or a server address. This is configured in the `loadURL` method. For local files, specify the accurate path and the HTML file to execute.

```jsx

  // Running from online
  mainWindow.loadURL('http://127.0.0.1:4000')

  // Running from offline
  mainWindow.loadURL(
    url.format({
      pathname: path.join(__dirname, `/dist/index.html`),
      protocol: "file:",
      slashes: true
    })
  );

```

## Modifying `package.json`

Modify the `package.json` configuration to enable easy execution.

```json
  "scripts": {
	...
	"start:electron": "ng build --prod --base-href ./ && electron .",
    "electron": "electron .",
  }
```

Two scripts are created: `start:electron` to execute the Angular build and Electron in one step, and `electron` to execute Electron separately. Creating only one script is also fine. The `prod` and `base-href` settings are added, but you can remove them if necessary.

Executing the script will now show the app running as a desktop application.

## Troubleshooting

> If the app doesn't run despite the above settings, verify the following in the generated `index.html` file within the build folder:
1) Is `base href` set to `"./"`?
The default is `/`, so change it to `"./"`.
> 

>

> 2) Is the `type` of the script set to `"text/javascript"`?
The default is `type="module"`, so change it to `"text/javascript"`.
> 

## References

- https://www.sitepoint.com/build-a-desktop-application-with-electron-and-angular/
- [Build Angular Desktop Apps With Electron \| AngularFirebase](https://angularfirebase.com/lessons/desktop-apps-with-electron-and-angular/)