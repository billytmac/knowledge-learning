## useMemo 是什么

`useMemo` 是 React 的一个 Hook，用于**缓存计算结果**，只有在依赖项发生变化时才重新计算。

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

它接收两个参数：一个计算函数，和一个依赖数组。React 会在首次渲染时执行计算函数，之后只有依赖项变化时才重新执行。

---

## 使用场景

**1. 避免昂贵的重复计算**

```javascript
const sortedList = useMemo(() => {
  return bigList.sort((a, b) => a.value - b.value); // 数据量大时很耗时
}, [bigList]);
```

没有 useMemo 的话，每次组件重渲染都会重新排序，即使 `bigList` 没变。

**2. 避免子组件不必要的重渲染**

```javascript
const filters = useMemo(() => ({ type: 'active', page }), [page]);

// 配合 React.memo 使用
return <ExpensiveChild filters={filters} />;
```

如果不用 useMemo，每次渲染都会创建新的对象引用，导致子组件总是重渲染。

**3. 作为其他 Hook 的稳定依赖**

```javascript
const options = useMemo(() => ({ userId, role }), [userId, role]);

useEffect(() => {
  fetchUserData(options);
}, [options]); // options 引用稳定，不会触发多余的 effect
```

---

## 和 useCallback 的区别

| | useMemo | useCallback |
|---|---|---|
| 缓存的是 | **计算结果**（值） | **函数本身** |
| 返回 | `fn()` 的返回值 | `fn` 函数引用 |
| 典型用途 | 缓存数组、对象、计算值 | 缓存事件处理函数 |

```javascript
// useMemo 缓存值
const total = useMemo(() => items.reduce((sum, i) => sum + i.price, 0), [items]);

// useCallback 缓存函数
const handleClick = useCallback(() => doSomething(id), [id]);
```

---

## 什么时候不该用

useMemo 本身也有开销，**不要滥用**。以下情况不需要：

- 计算逻辑很简单（加减法、简单过滤）
- 组件本身很少重渲染
- 依赖项几乎每次都会变（缓存没有意义）

> 核心原则：**先写出可读的代码，只在发现性能问题时再引入 useMemo。**

# 个对象有必要一定用useMemo？

对象用 `useMemo` 的意义在于**稳定引用**，但只有在这个对象的引用稳定性"被消费"时才有价值。

**不需要 useMemo 的情况：**

```javascript
// 这个对象只在组件内部自己用，没必要
const style = { color: 'red', fontSize: 14 };
return <div style={style}>hello</div>;
```

div 是原生元素，不会因为 style 引用变化而产生性能问题，加 useMemo 纯属多此一举。

**真正需要的情况只有两种：**

```javascript
// 1. 传给用 React.memo 包裹的子组件
const config = useMemo(() => ({ type, page }), [type, page]);
return <MemoizedChild config={config} />;

// 2. 作为 useEffect / useCallback 等的依赖项
const query = useMemo(() => ({ keyword, page }), [keyword, page]);
useEffect(() => {
  fetchData(query);
}, [query]);
```

这两种场景之外，对象加不加 `useMemo` 效果几乎一样。React 重渲染本身的开销通常远大于重新创建一个普通对象的开销，过早优化反而让代码变复杂。