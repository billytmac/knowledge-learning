因为**性能**，深比较代价太高。

---

### 如果用深比较会怎样

```ts
const state = {
  users: [
    { id: 1, name: 'Alice', roles: [...], settings: { theme: 'dark', ... } },
    { id: 2, name: 'Bob',   roles: [...], settings: { ... } },
    // ... 1000 条
  ]
}
```

每次 setState 都要递归比较所有嵌套层级，**组件一多，性能直接崩**。

---

### Object.is 的优势

```ts
// 一次比较，O(1) 复杂度
Object.is(oldState, newState)

// 深比较，O(n) 甚至更高
_.isEqual(oldState, newState) // 要递归遍历所有层级
```

---

### React 的设计哲学

React 把**判断数据是否变化的责任交给开发者**：

> "你返回新引用，我就认为变了。你自己控制什么时候该变。"

```ts
// 没变 → 返回原引用
setState(oldObj)

// 变了 → 返回新引用
setState({ ...oldObj, name: 'new' })
```

这样 React 只需要做廉价的引用比较，性能可预测可控。

---

### 类比

就像 Git 的 commit hash，不需要比较所有文件内容，只看 hash 变没变，变了就认为有更新，简单高效。