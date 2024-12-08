# react组件库打包时 external 为什么只需要 react 和 react/jsx-runtime?

这是因为 Vite/Rollup 的**依赖分析机制是基于实际的导入语句**（import statements）工作的。

1\. JSX 转换后的实际代码：

```tsx
// 你写的 JSX 代码
function App() {
  return <div>Hello</div>;
}

// 经过编译后实际变成（使用新的 JSX Transform）
import { jsx } from 'react/jsx-runtime';

function App() {
  return jsx('div', { children: 'Hello' });
}
```

2\. 依赖分析：

```tsx
// Vite/Rollup 看到的是这些实际的导入
import React from 'react';              // ← 需要 external
import { jsx } from 'react/jsx-runtime'; // ← 也需要 external
```

如果不在 external 中声明 react/jsx-runtime：

* 这个模块可能会被打包进你的库
* 可能导致运行时错误
* 破坏与宿主应用的 React 实例的一致性

所以：

{% code title="vite.config.ts" %}
```tsx
export default defineConfig({
  build: {
    rollupOptions: {
      external: [
        'react',
        'react/jsx-runtime',  // 必须显式声明
        'react-dom',
      ]
    }
  }
})
```
{% endcode %}

***

## 继续提问：为什么 external 中可以不声明 'react-dom' ? （是否声明对构建结果无变化）

啊！这涉及到 Rollup 的依赖分析机制。让我用具体例子解释：

1. 实际的代码导入情况：

```tsx
// React 相关导入
import React from 'react'                    // 直接导入
import { jsx } from 'react/jsx-runtime'      // 直接导入（由编译器生成）

// ReactDOM 相关导入
import ReactDOM from 'react-dom'             // ← 看这里
```

2. package.json 的依赖声明：

```json
{
  "dependencies": {
    "react-dom": {
      "peerDependencies": {
        "react": "^18.0.0"  // react-dom 的 package.json 声明了对 react 的依赖
      }
    }
  }
}
```

3. Rollup 的依赖分析逻辑：

```jsx
// 当你声明 'react' 为 external 时
external: ['react']

// Rollup 会：
1. 标记 'react' 为 external
2. 检查所有依赖 'react' 的包
3. 自动将这些包也标记为 external
   → 因为 react-dom 依赖 react，所以它会被自动 external
```

4. 而 react/jsx-runtime 的情况不同：

```tsx
// 它是一个具体的文件路径导入
import { jsx } from 'react/jsx-runtime'

// 不会被自动 external，因为：
1. 它是一个具体的文件路径
2. 不是一个独立的包
3. Rollup 无法通过依赖树推断它应该被 external
```

所以最终配置：

{% code title="vite.config.ts" %}
```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      external: [
        'react',              // 必须：直接导入
        'react/jsx-runtime',  // 必须：具体路径导入
        // 'react-dom'        // 可选：会被自动 external
      ]
    }
  }
})
```
{% endcode %}

这就是为什么：

* react/jsx-runtime 必须显式声明
* react-dom 可以被自动推断为 external

不过，为了代码的清晰性和可维护性，显式声明 react-dom 也是个好习惯！
