---
title: "the term 'ng' is not recognized as an internal or external command, operable program or batch file."
date: 2020-09-01 17:30:00 +0900
comments: true
categories: angular
tags: [error]
---

[한국어(Korean) Page](https://velog.io/@ksrae/ng-%EC%9A%A9%EC%96%B4%EA%B0%80-cmdlet-%ED%95%A8%EC%88%98-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%ED%8C%8C%EC%9D%BC-%EB%98%90%EB%8A%94-%EC%8B%A4%ED%96%89%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8A%94-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EC%9D%B4%EB%A6%84%EC%9C%BC%EB%A1%9C-%EC%9D%B8%EC%8B%9D%EB%90%98%EC%A7%80-%EC%95%8A%EC%8A%B5%EB%8B%88%EB%8B%A4)
<br/>

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