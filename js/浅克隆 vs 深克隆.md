
---

### 浅克隆 vs 深克隆

针对的是**复制对象**时的行为。

```ts
const obj = { a: 1, b: { c: 2 } }

// 浅克隆 — 只复制第一层，嵌套对象还是同一个引用
const shallow = { ...obj }
shallow.b.c = 999
console.log(obj.b.c) // 999，原对象被影响了！

// 深克隆 — 完整复制所有层级，完全独立
const deep = JSON.parse(JSON.stringify(obj))
deep.b.c = 999
console.log(obj.b.c) // 2，原对象不受影响
```

---

### 浅比较 vs 深比较

针对的是**判断两个对象是否相等**时的行为。

```ts
const a = { x: { y: 1 } }
const b = { x: { y: 1 } }

// 浅比较 — 只比较第一层引用
a === b           // false，引用不同
a.x === b.x       // false，引用不同

//React 的浅比较底层用的就是 Object.is，但只比较第一层。


// 深比较 — 递归比较所有层级的值
_.isEqual(a, b)   // true，值完全相同
```

---

### 两者的关系

| | 操作对象 | 关注点 |
|--|---------|-------|
| 浅/深克隆 | 复制 | 复制几层 |
| 浅/深比较 | 比较 | 比较几层 |

本质上都是在处理同一个问题：**嵌套引用类型的处理深度**。

---

### 在 React 中的体现

```ts
// React.memo 用的是浅比较
// props 引用没变就不重新渲染
const Comp = React.memo(({ obj }) => <div />)

// 如果传入每次新建的对象，浅比较认为变了，会重新渲染
<Comp obj={{ x: 1 }} /> // ⚠️ 每次都是新引用，浅比较 = 不等
```