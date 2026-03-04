`forwardRef` 常见有两种语境:
**在react19已被废弃掉，在react19作为props引入**
1. **React 里的 `forwardRef` 是什么**
- 用来把父组件传进来的 `ref`，转发到子组件内部某个 DOM 或组件上。
- 因为函数组件默认拿不到 `ref`，要用 `forwardRef` 包一下。

```tsx
// 第一个是 ref 类型
// 第二个是 props 类型（可省略，让 TS 推断）
// render 只的是render function
forwardRef<RefType, PropsType>(render)

// 使用 forwardRef() 让组件接收 ref 并将其传递给子组件：
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
const Input = React.forwardRef<HTMLInputElement, { label: string }>(
  ({ label }, ref) => (
    <label>
      {label}
      <input ref={ref} />
    </label>
  )
);
```

2. **React 使用场景**
- 父组件需要直接操作子组件里的 DOM（`focus()`、`scrollIntoView()`、测量尺寸）。
- 封装通用组件库（如输入框、弹窗）时，仍保留外部 `ref` 能力。
- 配合 `useImperativeHandle` 暴露受控方法（而不是暴露整个 DOM）。

---

如果你说的是 **NestJS 的 `forwardRef`**，它是另一回事：用于解决模块/服务之间的**循环依赖**。  
你要的话我可以再给你 NestJS 版本示例。