# React memo 使用场景

## 🎯 核心原则

> **memo 是性能优化工具，不是默认选项！**  
> **只在有明确性能问题时使用，过度使用反而降低性能**

---

## ✅ 应该使用 memo 的场景

### 1. **纯展示组件 + 频繁重渲染的父组件**

```typescript
// ❌ 父组件每次状态变化，UserCard 都会重渲染
function Dashboard() {
  const [count, setCount] = useState(0); // 频繁变化
  const user = { name: 'Alice', age: 25 }; // 不变
  
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      <UserCard user={user} /> {/* 每次都重渲染 ❌ */}
    </>
  );
}

// ✅ 使用 memo 避免不必要的重渲染
const UserCard = memo(({ user }: { user: User }) => {
  console.log('UserCard 渲染');
  return <div>{user.name}</div>;
});
```

---

### 2. **渲染成本高的组件**

```typescript
// 复杂的图表组件
const ExpensiveChart = memo(({ data }: { data: number[] }) => {
  // 复杂计算
  const processedData = data.map(/* 耗时操作 */);
  
  return (
    <canvas>
      {/* 渲染 1000+ DOM 节点 */}
    </canvas>
  );
});

// 大列表中的单项
const ListItem = memo(({ item }: { item: Item }) => {
  return (
    <div className="complex-layout">
      {/* 复杂的 UI 结构 */}
    </div>
  );
});

function List({ items }: { items: Item[] }) {
  return items.map(item => <ListItem key={item.id} item={item} />);
}
```

---

### 3. **列表渲染中的子项**

```typescript
// ❌ 滚动时整个列表都重渲染
function TodoList() {
  const [todos, setTodos] = useState([...]);
  const [filter, setFilter] = useState('all'); // 频繁切换
  
  return todos.map(todo => <TodoItem todo={todo} />);
}

// ✅ 只重渲染变化的项
const TodoItem = memo(({ todo }: { todo: Todo }) => {
  return <div>{todo.text}</div>;
});
```

---

### 4. **接收回调函数作为 props 的组件**

```typescript
function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ 每次渲染创建新函数，memo 失效
  const handleClick = () => console.log('click');
  
  // ✅ 配合 useCallback 使用
  const handleClick = useCallback(() => {
    console.log('click');
  }, []);
  
  return <Button onClick={handleClick} />;
}

const Button = memo(({ onClick }: { onClick: () => void }) => {
  console.log('Button 渲染');
  return <button onClick={onClick}>点击</button>;
});
```

---

### 5. **第三方库组件的包装**

```typescript
// 包装昂贵的第三方组件
const MemoizedReactQuill = memo(ReactQuill);
const MemoizedEChartsComponent = memo(EChartsComponent);

function Editor() {
  const [content, setContent] = useState('');
  const [otherState, setOtherState] = useState(0);
  
  return (
    <>
      <button onClick={() => setOtherState(prev => prev + 1)}>
        其他操作
      </button>
      {/* otherState 变化时不会重渲染编辑器 */}
      <MemoizedReactQuill value={content} onChange={setContent} />
    </>
  );
}
```

---

## ❌ 不应该使用 memo 的场景

### 1. **几乎每次都会重渲染的组件**

```typescript
// ❌ count 一直变，memo 无意义
const Counter = memo(({ count }: { count: number }) => {
  return <div>{count}</div>;
});

function App() {
  const [count, setCount] = useState(0);
  return <Counter count={count} />; // 每次都传新值
}
```

---

### 2. **props 是对象/数组/函数且未 memo 化**

```typescript
function Parent() {
  const user = { name: 'Alice' }; // 每次渲染都是新对象
  
  // ❌ memo 失效，因为 user 引用每次都变
  return <UserCard user={user} />;
}

const UserCard = memo(({ user }) => <div>{user.name}</div>);

// ✅ 正确做法
function Parent() {
  const user = useMemo(() => ({ name: 'Alice' }), []);
  return <UserCard user={user} />;
}
```

---

### 3. **简单组件 (渲染成本低)**

```typescript
// ❌ 不需要 memo，重渲染成本比 memo 检查还低
const SimpleText = memo(({ text }: { text: string }) => {
  return <span>{text}</span>;
});

// ✅ 直接渲染更高效
const SimpleText = ({ text }: { text: string }) => {
  return <span>{text}</span>;
};
```

---

### 4. **组件内部有 useState/useContext**

```typescript
// ❌ 组件内部状态变化，memo 无法阻止重渲染
const Counter = memo(() => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>{count}</button>
    </div>
  );
});
```

---

## 🔍 memo 工作原理

```typescript
// React.memo 默认行为：浅比较 props
const MyComponent = memo((props) => {
  return <div>{props.name}</div>;
});

// 等价于
function MyComponent(props) {
  return <div>{props.name}</div>;
}

// React 内部：
if (shallowEqual(prevProps, nextProps)) {
  return previousResult; // 跳过渲染
} else {
  return <MyComponent {...nextProps} />; // 重新渲染
}
```

---

## 🛠️ 自定义比较函数

```typescript
const UserCard = memo(
  ({ user }: { user: User }) => {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // 返回 true = 不重渲染
    // 返回 false = 重新渲染
    return prevProps.user.id === nextProps.user.id;
  }
);
```

---

## 📊 memo + useCallback + useMemo 配合

```typescript
function Parent() {
  const [count, setCount] = useState(0);
  
  // ✅ 正确的组合使用
  const user = useMemo(() => ({ 
    name: 'Alice', 
    age: 25 
  }), []);
  
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      <MemoizedComponent user={user} onClick={handleClick} />
    </>
  );
}

const MemoizedComponent = memo(({ user, onClick }) => {
  return <div onClick={onClick}>{user.name}</div>;
});
```

---

## ⚠️ 常见陷阱

### 陷阱 1: 传递内联对象/数组
```typescript
// ❌ memo 失效
<MemoComponent config={{ theme: 'dark' }} />

// ✅ 提取到外部或 useMemo
const config = useMemo(() => ({ theme: 'dark' }), []);
<MemoComponent config={config} />
```

### 陷阱 2: 传递内联函数
```typescript
// ❌ memo 失效
<MemoComponent onClick={() => console.log('click')} />

// ✅ 使用 useCallback
const handleClick = useCallback(() => console.log('click'), []);
<MemoComponent onClick={handleClick} />
```

### 陷阱 3: 传递 children
```typescript
// ❌ memo 失效，children 每次都是新的 React 元素
<MemoComponent>
  <div>内容</div>
</MemoComponent>

// ✅ 将 children 也 memo 化或提取
const content = useMemo(() => <div>内容</div>, []);
<MemoComponent>{content}</MemoComponent>
```

---

## 🎯 决策流程图

```
需要使用 memo 吗？
    ↓
父组件频繁重渲染？ 
    ↓ 是
props 经常保持不变？
    ↓ 是
渲染成本高？(复杂计算/大量 DOM)
    ↓ 是
props 是原始值或已 memo 化的对象/函数？
    ↓ 是
✅ 使用 memo

任何一步是 "否" → ❌ 不使用 memo
```

---

## 📝 最佳实践总结

| 场景 | 是否使用 memo |
|------|--------------|
| 列表项组件 | ✅ 推荐 |
| 复杂图表/地图组件 | ✅ 推荐 |
| 第三方库组件包装 | ✅ 推荐 |
| 纯展示组件(props 稳定) | ✅ 可选 |
| 简单文本/图标组件 | ❌ 不推荐 |
| props 频繁变化 | ❌ 无意义 |
| 有内部 state/context | ❌ 无效果 |
| 未优化的 props(对象/函数) | ❌ 会失效 |

---

## 💡 记忆口诀

> **"贵重物品才需要保险箱"**
> 
> - **贵** = 渲染成本高
> - **重** = props 稳定/不常变
> - **物品** = 纯展示组件
> - **保险箱** = memo

**不要过度优化！先测量性能，再决定是否使用 memo。**