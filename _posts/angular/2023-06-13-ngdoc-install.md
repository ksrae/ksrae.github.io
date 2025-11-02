---
title: "ngDoc - Install"
date: 2023-06-13 16:12:00 +0900
comments: true
categories: angular
tags: [document, ngdoc]
---

# Streamlining Angular Project Documentation with ngDoc

## Introduction

For Angular projects requiring documentation resembling Google's Material Design Components library or Microsoft's Fluent 2, ngDoc offers a robust solution.<br/><br/>

This library enables the inclusion of interactive examples alongside their corresponding code snippets. Furthermore, its Markdown-based documentation approach facilitates a smooth and efficient documentation process.<br/><br/>

This blog post guides you through the installation and configuration of ngDoc within your Angular project. With just a few simple steps, you can have ngDoc up and running, automating the necessary configurations.<br/><br/>

We'll cover the basic installation process and methods for incorporating different features. A more in-depth exploration will follow in a future post.

## Installing ngDoc

To begin, install ngDoc in your Angular project. Open your terminal and execute the following command:

```
ng add @ng-doc/add
```

This command automatically installs and configures ngDoc within your project.

### Important Note

`ng add @ng-doc/add` must be executed from the directory where you want to configure ngDoc. For example, if you're working on documentation files in `src/docs`, navigate to that folder before running the installation command.

### Manual Installation (Optional)

For more granular control over the installation process or in case of issues with the automatic installation, refer to the official ngDoc documentation for manual installation instructions. This resource provides detailed steps to guide you through a successful ngDoc installation.<br/>

You can find the manual installation guide on the NgDoc website: https://ng-doc.com/docs/getting-started/installation.

## Entities

Entities represent the fundamental building blocks of an ngDoc setup. They typically consist of `category` and `page` components. Non-standalone configurations require a `module`.<br/>

The `module` serves a similar role to modules in Angular versions prior to 14. Components external to ngDoc that are to be included within a `page` must be defined in the `module`'s `declarations`. Note that, unlike traditional Angular modules, `page` components do not need to be declared or imported.

### Category

A `category` functions similarly to a folder in a file system. It is responsible for grouping similar documents and serves no other purpose.<br/>

Create a category using the following command:

```
ng g @ng-doc/builder:category "[category name]"
```

This will generate an `ng-doc.category.ts` file similar to this:

```tsx
import {NgDocCategory} from '@ng-doc/core';

export const MyAwesomeCategory: NgDocCategory = {
  title: 'MyAwesomeCategory',
};

export default MyAwesomeCategory;
```

### Page

A `page` primarily utilizes Markdown files but offers easy support for additional features through an `ng-doc.page.ts` file.<br/>

Create a page using the following command:

```
ng g @ng-doc/builder:page "[page name]"
```

This will generate `index.md` and `ng-doc.page.ts` files. The main documentation is written in the `index.md` file, while adding components and other functionalities is handled in `page.ts`.<br/>

For non-standalone setups, the `-m` parameter should be included when adding a page:

```
ng g @ng-doc/builder:page "[page name]" -m
```

In this case, in addition to the two files above, an `ng-doc.module.ts` file is generated:

```tsx
import {NgModule} from '@angular/core';
import {CommonModule} from '@angular/common';

@NgModule({
	imports: [CommonModule],
	// Declare you demo components here
	declarations: [],
})
export class PageModule {}
```

In this scenario, the above module must be imported into `ng-doc.page.ts`, as shown below:

```tsx
import {NgDocPage} from '@ng-doc/core';
import {PageModule} from './ng-doc.module';

const Page: NgDocPage = {
	title: `demo`,
	mdFile: './index.md',
  imports: [PageModule],
};

export default Page;
```