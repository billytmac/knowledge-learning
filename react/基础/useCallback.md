## `useCallback` 的使用场景

### ✅ **需要使用的情况**

#### 1. **传递给优化过的子组件**
```typescript
// 子组件使用了 React.memo
const ChildComponent = React.memo(({ onClick }) => {
  return <button onClick={onClick}>Click</button>
})

function Parent() {
  // ✅ 需要 useCallback，否则每次渲染都创建新函数，导致 memo 失效
  const handleClick = useCallback(() => {
    console.log('clicked')
  }, [])
  
  return <ChildComponent onClick={handleClick} />
}
```

#### 2. **作为其他 Hook 的依赖项**
```typescript
function Component({ id }) {
  // ✅ fetchData 被用作 useEffect 的依赖
  const fetchData = useCallback(async () => {
    const data = await api.get(id)
    setData(data)
  }, [id])
  
  useEffect(() => {
    fetchData()
  }, [fetchData]) // 如果不用 useCallback，会无限循环
}
```

#### 3. **传递给第三方库或性能敏感的组件**
```typescript
// ✅ 虚拟列表等性能敏感场景
<VirtualList
  renderItem={useCallback((item) => <Item data={item} />, [])}
/>
```

---

### ❌ **不需要使用的情况**

#### 1. **普通事件处理器**
```typescript
function Component() {
  // ❌ 没必要，普通按钮点击不需要优化
  const handleClick = useCallback(() => {
    console.log('clicked')
  }, [])
  
  // ✅ 直接定义即可
  const handleClick = () => {
    console.log('clicked')
  }
  
  return <button onClick={handleClick}>Click</button>
}
```

#### 2. **组件内部使用的函数**
```typescript
function Component() {
  // ❌ 只在组件内部用，不需要缓存
  const processData = useCallback((data) => {
    return data.map(x => x * 2)
  }, [])
  
  // ✅ 直接定义
  const processData = (data) => {
    return data.map(x => x * 2)
  }
  
  const result = processData(someData)
}
```

#### 3. **依赖项频繁变化**
```typescript
function Component({ filter, sortBy, searchTerm }) {
  // ❌ 依赖项太多且经常变，缓存没意义
  const handleFilter = useCallback(() => {
    return data.filter(filter).sort(sortBy).search(searchTerm)
  }, [filter, sortBy, searchTerm, data])
  
  // ✅ 直接定义更清晰
  const handleFilter = () => {
    return data.filter(filter).sort(sortBy).search(searchTerm)
  }
}
```

---

## 核心原则

> **过早优化是万恶之源** - Donald Knuth

- **默认不用** `useCallback`
- **只在遇到实际性能问题时**才添加
- **明确知道为什么需要时**才使用

大多数情况下，函数重新创建的成本**远低于** `useCallback` 本身的成本。