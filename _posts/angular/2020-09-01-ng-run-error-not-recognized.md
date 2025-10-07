---
title: "the term 'ng' is not recognized as an internal or external command, operable program or batch file."
date: 2020-09-01 17:30:00 +0900
comments: true
categories: angular
tags: [error]
---


## Issue Description

When attempting to install the Angular CLI on Windows using the following command:

```
> npm i -g @angular/cli
```

Executing the `ng` command results in the following error:

```
'ng' is not recognized as an internal or external command, operable program or batch file.
```

This indicates that the system cannot locate the Angular CLI executable.

## Resolution

This issue typically arises from an incorrect configuration of environment variables. The following steps outline the solution:

### Modifying Environment Variables

1. Open the **Environment Variables** settings on your system.
2. Locate the `Path` variable within the System variables section and open it for editing.
3. Add the following path to the list of variables: `%AppData%\npm`.
4. Ensure that the `%AppData%\npm` entry is positioned *above* the entry for Node.js in the list. Alternatively, move it to the very top of the list. This ensures that the system searches the npm global package directory before the Node.js directory.

### Reinstalling Angular CLI

1. Uninstall the existing Angular CLI installation:

```
npm uninstall -g @angular/cli
```

1. Clear the npm cache:

```
npm cache clean --force
```

The `--force` flag is mandatory for this command. Without it, the execution will be canceled, and npm will suggest running `npm cache verify` instead. Using `--force` bypasses this verification and forcibly clears the cache.

1. Reinstall the Angular CLI globally:

```
npm install -g @angular/cli
```

## References

- [the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet](https://stackoverflow.com/questions/44958847/the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet/44958882)
- [the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet-in-angular](https://stackoverflow.com/questions/59545882/the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet-in-angular)