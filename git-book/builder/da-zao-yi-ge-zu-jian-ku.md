---
cover: >-
  https://images.unsplash.com/photo-1728343071206-06359c948426?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHJhbmRvbXx8fHx8fHx8fDE3Mjk4NDQzNjl8&ixlib=rb-4.0.3&q=85
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 打造一个组件库

在做toB场景下、或者在前端团队内部，我们更多时候是提供一个 UI Components Library 提供给客户或者团队内部使用，因此如何做出一个好用的、现代化的Component Library是一个值得在前端团队内永恒讨论的话题。

最近在做跨团队的React组件库，那么本次我将把我的经验分享出来。



## 1 初始化项目

这里目前我强烈推荐[使用Vite创建项目](https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project)，因为Vite支持库模式，适合构建组件库类型的项目。

接下来的内容将以typescript-react模板为例。

```bash
pnpm create vite <project-name> --template react-ts
```

下面是该命令生成的结果（2024-10-29版本）：

```
lib-component
├──public/**
├──src/**
├──.gitignore
├──eslint.config.js
├──index.html
├──package.json
├──README.md
├──tsconfig.app.json
├──tsconfig.json
├──tsconfig.node.json
└──vite.config.ts
```

接下来修改项目以便于我们开发一个组件库。

我会首先删除 public 文件夹，以及清空 src 文件夹(除了`vite-env.d.ts`)。

## 2  修改 eslint 配置

如果有需求，我建议删除eslint相关的内容并重新自己配置，因为vite自带的eslint使用的是最新的`eslint@9`的版本，很多常用的eslint-plugin或extend其实并没有支持最新的eslint flat config形式。

例如，我们项目内部有统一的eslint-config配置，因此我会移除现有的eslint并重新安装`eslint@8`并配置我们内部的shareable config。

```javascript
const path = require('path');

module.exports = {
  root: true,
  extends: [
    <extend your content>,
  ],
  parserOptions: {
    project: [
      `${__dirname}/tsconfig.json`,
      `${__dirname}/tsconfig.app.json`,
      `${__dirname}/tsconfig.node.json`,
    ],
  },
  settings: {
    'import/resolver': {
      alias: {
        map: [['@', path.resolve(__dirname, './src')]],
        extensions: ['.js', '.jsx', '.ts', '.d.ts', '.tsx'],
      },
    },
  },
  ignorePatterns: [
    'node_modules',
    'dist',
    'public',
  ],
};
```

在配置eslint配置时要注意，由于很多很多大型项目都使用了`typescript-eslint` 以保证`@typescript-eslint/parser`可以正确读取并应用当前tsconfig的配置。因此我们要配置`parserOptions.project`以在vite环境下生效，[更多参考资料](https://typescript-eslint.io/packages/parser/#project)。

在这里`parserOptions.project`会默认读取`tsconfig.json` 文件，因此对于vite脚手架创建的项目需要进行手动配置。

大概的说，[`tsconfig.*`](#user-content-fn-1)[^1] 中会包含`include`属性，表明了该 tsconfig 所涵盖的文件范围，而对于这些文件，eslint parser会读取这些 tsconfig 配置并进行校验。

除此之外，例如使用了`eslint-import-resolver-typescript`来增强导入能力，也需要进行[额外的配置](https://www.npmjs.com/package/eslint-import-resolver-typescript#configuration)。

## 3 编写组件代码

现在我们来设计项目结构，在组件库中，最常见的就是 src/components ，因此先编写一个 Button 组件和一个 Input组件。

```markdown
lib-component
├── src/
│   └── components/
│       ├── Button/
│       │   ├── Button.tsx
│       │   ├── Button.module.scss
│       │   └── index.ts
│       └── Input/
│           ├── Input.tsx
│           ├── Input.module.scss
│           └── index.ts
```

好了，现在已经有了一个组件库应该有的雏形，那么我希望构建的结果可以支持es按需导入，并且实现JavaScript和css的分割

## 4 配置构建器&#x20;



[^1]: 
