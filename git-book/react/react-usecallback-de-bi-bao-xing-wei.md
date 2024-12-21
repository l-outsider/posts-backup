# React useCallback的闭包行为

## 在 react 中，如果不把 state 放入 useCallback 的依赖数组，callback 函数永远拿到的是初始值

让我用一个例子来说明：

```tsx
function Example() {
  const [count, setCount] = useState(0);

  // ❌ 错误示范：callback 中的 count 永远是 0
  const handleClick = useCallback(() => {
    console.log(count);     // 永远输出 0
    setCount(count + 1);    // 永远是 0 + 1
  }, []); // 空依赖数组

  // ✅ 正确做法 1：将 count 加入依赖
  const handleClick = useCallback(() => {
    console.log(count);     // 输出当前的 count
    setCount(count + 1);    // 正确累加
  }, [count]);

  // ✅ 正确做法 2：使用函数式更新，不需要依赖 count
  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // 空依赖数组也没问题

  return <button onClick={handleClick}>{count}</button>;
}
```

关键点：

1. 闭包会捕获创建时的值
2. 没有依赖的 useCallback 只在组件首次渲染时创建一次
3. 如果需要访问最新的 state，要么：
   * 将 state 加入依赖数组
   * 使用函数式更新（推荐）

useEffect\useLayoutEffect 与 useCallback、useMemo 不同：

```tsx
export default function App() {
  const [s1, setS1] = useState(0);
  const [s2, setS2] = useState(100);

  useEffect(() => {
    console.log(s1, s2); // ✅ 能够正确的打印 s2
  }, [s1]);

  function click() {
    setS1((prev) => prev + 1);
    setS2((prev) => prev - 1);
  }

  return (
    <div className="App">
      <button onClick={click}>click</button>
    </div>
  );
}
```
