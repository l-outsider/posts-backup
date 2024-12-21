# React 组件树的更新与执行顺序

## 执行顺序

在 react 组件树中，各个组件的函数体、useEffect、useLayoutEffect 的执行顺序：

```tsx
// Parent.tsx
function Parent() {
  console.log('1. Parent 函数体执行');

  useLayoutEffect(() => {
    console.log('4. Parent useLayoutEffect 执行');
    return () => console.log('Parent useLayoutEffect cleanup');
  });

  useEffect(() => {
    console.log('6. Parent useEffect 执行');
    return () => console.log('Parent useEffect cleanup');
  });

  return <Child />;
}

// Child.tsx
function Child() {
  console.log('2. Child 函数体执行');

  useLayoutEffect(() => {
    console.log('5. Child useLayoutEffect 执行');
    return () => console.log('Child useLayoutEffect cleanup');
  });

  useEffect(() => {
    console.log('7. Child useEffect 执行');
    return () => console.log('Child useEffect cleanup');
  });

  return <GrandChild />;
}

// GrandChild.tsx
function GrandChild() {
  console.log('3. GrandChild 函数体执行');

  useLayoutEffect(() => {
    console.log('6. GrandChild useLayoutEffect 执行');
    return () => console.log('GrandChild useLayoutEffect cleanup');
  });

  useEffect(() => {
    console.log('8. GrandChild useEffect 执行');
    return () => console.log('GrandChild useEffect cleanup');
  });

  return null;
}
```

执行顺序：

1. 先执行所有组件的函数体（自上而下）
   1. Parent 函数体
   2. Child 函数体
   3. GrandChild 函数体
2. 然后执行所有 useLayoutEffect（自上而下）
   1. Parent useLayoutEffect
   2. Child useLayoutEffect
   3. GrandChild useLayoutEffect
3. 最后执行所有 useEffect（自上而下）
   1. Parent useEffect
   2. Child useEffect
   3. GrandChild useEffect

卸载时的清理顺序（自下而上）：

1. GrandChild cleanup
2. Child cleanup
3. Parent cleanup

关键点：

1. 函数体在每次渲染时都会执行
2. useLayoutEffect 在 DOM 变更后同步执行
3. useEffect 在渲染完成后异步执行
4. cleanup 函数执行顺序与挂载顺序相反

## 即使没有发生 rerender 函数体也会执行

父组件渲染时，子组件的函数体也会执行（即使子组件的 props 和 state 没有变化）。

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  console.log('Parent 函数体执行');
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        count: {count}
      </button>
      <Child />
    </>
  );
}

function Child() {
  console.log('Child 函数体执行'); // 每次 Parent 更新，这里都会执行
  return <div>Child</div>;
}
```

可以使用 memo 嵌套，使子组件不执行：

```tsx
const Child = memo(() => {
  console.log("Child 函数体执行"); // 这里不会再执行
  return <div>Child</div>;
});
```
