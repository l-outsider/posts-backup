# CSS/SCSS Modules 智能提示配置指南

## 安装依赖

```bash
npm install -D typescript-plugin-css-modules sass
```

## TypeScript 配置

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

## VS Code 配置

1. 创建 `.vscode/settings.json`（可忽略，我没配置这一步依然可以）：

```json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

2. 切换到工作区 TypeScript 版本（<mark style="color:red;">重要，一定要选择工作区的Typescript版本</mark>）：

```
1. 按 Cmd/Ctrl + Shift + P
2. 输入: "TypeScript: Select TypeScript Version"
3. 选择 "Use Workspace Version"
4. 重启 TS Server
```

## 使用示例

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

## 验证配置

创建测试文件验证配置是否生效：

```scss
// Test.module.scss
.test {
  color: red;
}
```

```tsx
// Test.tsx
import styles from './Test.module.scss'

// 输入 styles. 应该出现 .test 提示
const Test = () => <div className={styles.}>
```

如果看到类型提示，说明配置成功。
