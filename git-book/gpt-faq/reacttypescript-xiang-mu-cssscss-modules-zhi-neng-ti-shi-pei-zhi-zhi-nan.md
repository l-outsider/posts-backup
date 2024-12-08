# React-TypeScript 项目 CSS/SCSS Modules 智能提示配置指南

## 问题描述

在 React + TypeScript 项目中使用 CSS Modules 时，我们希望：

1. 获得类名的智能提示
2. 使用BEM 命名规范时，支持点语法访问样式类名
3. 类名不被哈希化，方便用户覆盖样式

## 类名的智能提示

在编写组件时，需要频繁切换文件来查看和复制类名，影响开发效率。

### 1. 安装依赖

```bash
npm install -D typescript-plugin-css-modules sass
```

### 2. TypeScript 配置

在 `tsconfig.json` 或 `tsconfig.app.json` 中添加插件配置：

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "typescript-plugin-css-modules"
      }
    ]
  }
}
```

### 3. VS Code 配置

1. 创建 `.vscode/settings.json`（可尝试，我没配置这一步依然可以）：

```json
{
  // 使用项目本地的 Typescript 版本
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  // 启用 CSS Modules 插件
  "typescript.tsserver.pluginPaths": ["typescript-plugin-css-modules"]
}
```

2. 切换到工作区 TypeScript 版本（<mark style="color:red;">重要，一定要选择工作区的Typescript版本</mark>）：

```
1. 按 Cmd/Ctrl + Shift + P
2. 输入: "TypeScript: Select TypeScript Version"
3. 选择 "Use Workspace Version"
4. 重启 TS Server
```

### 4. 使用示例

```scss
// Button.module.scss
.container {
  color: red;
}

.primary {
  background: blue;
}
```

```tsx
// Button.tsx
import styles from './Button.module.scss'

const Button = () => (
  // 输入 styles. 时会提示 container 和 primary
  <button className={styles.container}>
    Click me
  </button>
)
```

## 支持点语法访问样式类名

### 1. 问题描述

BEM 命名规范使用连字符和下划线（如 `button__icon--large`），但这种命名方式不能直接用点语法访问。

### 2. 解决方案

在 Vite 配置中启用驼峰命名转换：

```typescript
// vite.config.ts
export default defineConfig({
  css: {
    modules: {
      localsConvention: 'camelCase'  // 关键配置
    }
  }
});
```

### 3. 效果展示

```scss
// Button.module.scss
.button__icon--large {
  font-size: 20px;
}
```

```tsx
// Button.tsx
import styles from './Button.module.scss';

// 可以使用驼峰形式访问
<span className={styles.buttonIconLarge}></span>
```

## 类名不被哈希化 方便全局覆盖样式

### 1. 问题描述

组件库的用户需要能够方便地覆盖组件样式，但 CSS Modules 默认会给类名添加哈希值。

### 2. 解决方案

配置 Vite 不添加哈希值：

```typescript
// vite.config.ts
export default defineConfig({
  css: {
    modules: {
      generateScopedName: '[local]'  // 保持原始类名
    }
  }
});
```

### 3. 效果展示

```scss
// 组件库的样式
.button {
  background: blue;
}

// 用户的样式
.button {
  background: red;  // 可以直接覆盖
}
```

### 4. 完整配置示例

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  css: {
    modules: {
      localsConvention: 'camelCase',  // 支持驼峰命名访问
      generateScopedName: '[local]'    // 不添加哈希
    }
  }
});
```

## 补充说明

### 1. BEM 命名转换规则

原始 BEM 类名 -> 驼峰命名：

* `button` -> `button`
* `button__icon` -> `buttonIcon`
* `button--primary` -> `buttonPrimary`
* `button__icon--large` -> `buttonIconLarge`

### 2. 智能提示优化

如果使用 VSCode，可以安装以下插件增强体验：

* CSS Modules
* SCSS IntelliSense

### 3. 如果使用的 Builder 没有内置 CSS Modules 的类型声明

需要自行声明模块文件

1. 使用全局声明文件

创建 `src/types/scss.d.ts`：

```typescript
declare module '*.module.scss' {
  const classes: { [key: string]: string };
  export default classes;
}
```

2. 使用 npm 包 typed-scss-modules 自动生成类型声明（webpack使用css-modules-typescript-loader）

## 常见问题

1. **类型声明冲突**
   * Vite 已经提供了 `*.module.scss` 的类型声明
   * 不需要创建 `scss.d.ts` 文件
   * 如遇到类型冲突，优先使用 Vite 的类型声明
2. **没有智能提示**
   * 确认 VS Code 使用的是工作区 TypeScript 版本
   * 尝试重启 TS Server
   * 检查 `tsconfig.json` 中的插件配置是否正确
3. **类型提示不准确**
   * 确保已安装 `typescript-plugin-css-modules`
   * 检查 SCSS 文件是否以 `.module.scss` 结尾
   * 重启 VS Code
4. **新项目配置**
   * 建议将 `.vscode/settings.json` 提交到版本控制
   * 在项目文档中说明 VS Code TypeScript 版本设置步骤
