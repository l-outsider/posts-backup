# useEventCallback

`useEventCallback` 的主要好处是解决了两个问题：

1. 函数依赖问题：

```typescript
function Component() {
  const [count, setCount] = useState(0);

  // 🔴 问题：handleClick 的作用是使 count 变化
  // 因此 handleClick 必须依赖 count 并每次重新创建
  // 否则 count 永远是 1
  const handleClick = useCallback(() => {
    console.log(count);
    setCount(count + 1);
  }, [count]);

  // 添加事件监听也需要把 handleClick 作为依赖
  useEffect(() => {
    document.addEventListener('click', handleClick);
    return () => document.removeEventListener('click', handleClick);
  }, [handleClick]); // 每次 count 变化都要重新绑定事件
}
```

使用 `useEventCallback` 后：

```tsx
function Component() {
  const [count, setCount] = useState(0);

  // ✅ handleClick 永远是同一个引用，但始终能访问最新的 count
  const handleClick = useEventCallback(() => {
    console.log(count);
  });

  useEffect(() => {
    document.addEventListener('click', handleClick);
    return () => document.removeEventListener('click', handleClick);
  }, []); // 依赖项为空，不需要重新绑定
}
```

2. MUI 中的实际应用例子：

```typescript
// MUI Button 组件
function Button(props) {
  const { onClick, disabled } = props;
  
  // 即使 props 变化，rippleRef 也能访问最新值，且函数引用保持稳定
  const handleRippleStart = useEventCallback((e) => {
    if (disabled) return;
    rippleRef.current?.start(e);
    onClick?.(e);
  });

  useEffect(() => {
    // 不需要把 handleRippleStart 加入依赖
    someEventListener(handleRippleStart);
  }, []);
}

// MUI TextField 组件
function TextField() {
  const [value, setValue] = useState('');
  
  // 处理复杂的输入逻辑，但不会因为 value 变化而重新创建
  const handleChange = useEventCallback((e) => {
    let newValue = e.target.value;
    // 复杂的值处理逻辑...
    setValue(newValue);
  });

  return <input onChange={handleChange} value={value} />;
}
```

主要优势：

1. 函数引用永远稳定，减少不必要的重渲染
2. 始终能访问到最新的 props/state
3. 减少 useEffect 的依赖项
4. 避免了闭包陷阱

实际上，React 团队也意识到这个需求，正在开发官方的 `useEvent` Hook，原理与 `useEventCallback` 类似。

* [useEvent RFC](https://github.com/reactjs/rfcs/blob/useevent/text/0000-use-event.md)
* [MUI Button 源码](https://github.com/mui/material-ui/blob/master/packages/mui-material/src/ButtonBase/ButtonBase.js)

## 实现

使用 `useEventCallback` 自定义 hook（推荐）：

```tsx
// hooks/useEventCallback.ts
import { useCallback, useRef } from 'react';

export function useEventCallback<T extends (...args: any[]) => any>(fn: T): T {
  const fnRef = useRef(fn);
  fnRef.current = fn;

  return useCallback((...args: Parameters<T>): ReturnType<T> => {
    return fnRef.current.apply(null, args);
  }, []) as T;
}

// 使用示例
function Component() {
  // 2. effects/hooks
  useEffect(() => {
    handleScroll();
  }, []); // 不需要依赖 handleScroll

  // 3. handlers (可以放在后面了)
  const handleScroll = useEventCallback(() => {
    // ...
  });

  return <div />;
}
```

useEventCallback 这种模式在很多流行库中都有使用（如 MUI），是一个经过实践检验的解决方案。如果项目中大量使用这种模式，可以考虑把这个 hook 添加到项目的公共 hooks 中。参考：

* [https://github.com/mui/material-ui/blob/master/packages/mui-utils/src/useEventCallback/useEventCallback.ts](https://github.com/mui/material-ui/blob/master/packages/mui-utils/src/useEventCallback/useEventCallback.ts)
* [React useEvent RFC](https://github.com/reactjs/rfcs/blob/useevent/text/0000-use-event.md)
