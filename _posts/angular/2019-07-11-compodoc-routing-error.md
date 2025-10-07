---
title: "Fail to Understand New Routing Structure"
date: 2019-07-11 13:44:00 +0900
comments: true
categories: angular
tags: [compodoc, error]
---


When attempting documentation with Compodoc, an issue arises where it fails to recognize the routing method introduced in Angular 8 ( `import().then(m => m.Module)`).

Routes parsing error, maybe a trailing comma or an external variable, trying to fix that later after sources scanning.

Currently, there is no direct solution. Until this issue is resolved, you can either bypass the routing structure visualization during documentation generation or revert the code to the previous routing structure for documentation purposes and then switch back to the new routing method.

The official Usage menu on the website provides a list of options. Using the `--disableRoutesGraph` option allows you to ignore routing diagram generation.

```bash
npx compodoc -p src/tsconfig.app.json --disableRoutesGraph
```

## Update 2021.07.12

The Compodoc update has resolved the above issue.

## Reference Site

[Compodoc Guides](https://compodoc.app/guides/getting-started.html)