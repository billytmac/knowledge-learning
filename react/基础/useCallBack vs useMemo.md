核心区别一句话：

- `useMemo`：**缓存“计算结果”**
- `useCallback`：**缓存“函数本身”**

```tsx
const value = useMemo(() => expensiveCalc(a, b), [a, b]); // value 是结果
const fn = useCallback(() => doSomething(a, b), [a, b]);  // fn 是函数
```

常见使用场景：

1. `useMemo`
- 计算开销大，不想每次 render 都重新算
- 给子组件传对象/数组，避免每次新引用导致不必要渲染

2. `useCallback`
- 把回调函数传给 `React.memo` 的子组件，避免函数引用变化导致子组件重渲染
- 作为 `useEffect` 依赖时，避免函数每次变导致 effect 反复执行

关系：
- `useCallback(fn, deps)` 本质上可看作 `useMemo(() => fn, deps)` 的语义化写法。