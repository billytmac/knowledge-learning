不是必须，但**建议依赖数组里包含所有在 effect 内部用到的响应式值**。

---

### 什么是响应式值

```ts
const [count, setCount] = useState(0)  // ✅ 响应式
const value = useMemo(() => ..., [])   // ✅ 响应式
const name = props.name                // ✅ 响应式

const REF_VALUE = 'hello'             // ❌ 非响应式（模块级常量）
const ref = useRef(0)                 // ❌ ref.current 非响应式
```

---

### 哪些可以不放进依赖

```ts
// 1. 稳定的函数（React 保证不变）
const [count, setCount] = useState(0)
useEffect(() => {
  setCount(c => c + 1) // setCount 不需要放依赖
}, [])

// 2. useRef 的值
const ref = useRef(0)
useEffect(() => {
  ref.current = 1 // ref 不需要放依赖
}, [])

// 3. 模块级常量
const MAX = 100
useEffect(() => {
  console.log(MAX) // 不需要放依赖
}, [])
```

---

### 如果漏写依赖会怎样

```ts
const [count, setCount] = useState(0)

useEffect(() => {
  console.log(count) // 用到了 count
}, [])              // 但没写进依赖 → 闭包陷阱，拿到旧值
```

---

**一句话总结：** 组件内部定义的、会随渲染变化的值（state、props、派生值）都应该放进依赖，稳定不变的（常量、ref、setState）可以省略。eslint-plugin-react-hooks 的 `exhaustive-deps` 规则会自动帮你检查。


取决于这个变量**在哪里定义**、**会不会随渲染变化**。

---

### 组件内部定义的普通变量

```ts
function MyComponent() {
  const [count, setCount] = useState(0)
  
  const double = count * 2  // 组件内普通变量，依赖 count
  
  useEffect(() => {
    console.log(double) // 必须放依赖，每次渲染都可能不同
  }, [double])
}
```

---

### 组件外部定义的常量

```ts
const OPTIONS = ['a', 'b', 'c']  // 组件外，永远不变

function MyComponent() {
  useEffect(() => {
    console.log(OPTIONS) // 不需要放依赖
  }, [])
}
```

---

### 组件内部定义的对象/数组（陷阱！）

```ts
function MyComponent() {
  const options = { page: 1 }  // 每次渲染都是新引用！
  
  useEffect(() => {
    fetchData(options)
  }, [options]) // ⚠️ 会无限触发，因为每次渲染 options 都是新对象
}
```

解决：
```ts
// 方案1：useMemo 稳定引用
const options = useMemo(() => ({ page: 1 }), [])

// 方案2：移到组件外
const OPTIONS = { page: 1 }

// 方案3：放进 useEffect 内部
useEffect(() => {
  const options = { page: 1 }
  fetchData(options)
}, [])
```

---

**判断规则：**

| 变量类型 | 是否放依赖 |
|---------|-----------|
| 组件内 state/props 派生的变量 | ✅ 要放 |
| 每次渲染新建的对象/数组 | ⚠️ 要用 useMemo 稳定后再放 |
| 组件外的常量 | ❌ 不用放 |
| useRef 的值 | ❌ 不用放 |

好处就三个，都是在防真实 bug：

1. 防止“闭包旧值”  
`useEffect/useCallback` 里读到旧的 state/props/函数，行为和你预期不一致。

2. 让副作用和数据变化保持同步  
依赖变了就重跑，不会漏跑；避免“界面更新了但副作用没更新”。

3. 降低维护成本  
别人改了变量来源后，lint 会提醒你补依赖，减少隐性回归。  

简言之：`exhaustive-deps` 是用“提前报错”换“线上少出诡异问题”。