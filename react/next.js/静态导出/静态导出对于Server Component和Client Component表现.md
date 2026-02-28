## 静态导出对于 Server Component 和 Client Component的表现

配置是 `output: 'export'`（静态导出），Next.js 在构建时会把 **所有组件都预渲染为 HTML** — 不管是 Server Component 还是 Client Component：

| 模式                               | Server Component                        | Client Component                     |
| ---------------------------------- | --------------------------------------- | ------------------------------------ |
| 有 Node.js 服务器                  | 每次请求时服务端渲染，JS 不发送到客户端 | 预渲染 HTML + 发送 JS 用于 hydration |
| **`output: 'export'`（你的情况）** | **构建时渲染为 HTML，JS 不发送**        | **构建时也渲染为 HTML，JS 也发送**   |

所以在 `output: 'export'` 模式下，**HTML 输出本来就是一样的**，因为两种组件都在构建时生成了 HTML。


## 具体解释

好问题！这里的"JS 发不发送"指的是**组件的 JS 代码是否会打包到客户端**。

### Server Component（不发送 JS）

```typescript
// page.tsx - Server Component（无 "use client"）
export default function Page() {
  const rules = ['规则1', '规则2', '规则3']
  
  return (
    <div>
      {rules.map(rule => <p key={rule}>{rule}</p>)}
    </div>
  )
}
```

**构建后：**
```html
<!-- HTML 里有内容 -->
<div>
  <p>规则1</p>
  <p>规则2</p>
  <p>规则3</p>
</div>

<!-- ❌ 但这段 map 逻辑的 JS 代码不会打包到客户端 -->
<!-- ✅ 客户端 bundle.js 里没有这个组件的代码 -->
```

**结果：**
- HTML 文件：包含渲染好的内容（较大）
- JS 文件：不包含这个组件（较小）
- 总体积：**更小**

---

### Client Component（发送 JS）

```typescript
// page.tsx
"use client" // 👈 变成客户端组件

export default function Page() {
  const rules = ['规则1', '规则2', '规则3']
  
  return (
    <div>
      {rules.map(rule => <p key={rule}>{rule}</p>)}
    </div>
  )
}
```

**构建后：**
```html
<!-- HTML 里也有内容（因为是 output: 'export'） -->
<div>
  <p>规则1</p>
  <p>规则2</p>
  <p>规则3</p>
</div>

<!-- ✅ 但这段 map 逻辑的 JS 代码会打包到客户端 -->
<script src="/_next/static/chunks/page-abc123.js"></script>
```

**page-abc123.js 里包含：**
```javascript
// 这些代码会被打包进 JS
function Page() {
  const rules = ['规则1', '规则2', '规则3'];
  return /*#__PURE__*/ (0, jsx_runtime.jsx)("div", {
    children: rules.map(rule => /*#__PURE__*/ (0, jsx_runtime.jsx)("p", {
      children: rule
    }, rule))
  });
}
```

**结果：**
- HTML 文件：包含渲染好的内容
- JS 文件：**也包含这个组件的完整代码**
- 总体积：**更大**（重复了）

---

## 直观对比

### Server Component 的产物
```
out/
├── index.html (50KB) ← 包含所有规则的 HTML
└── _next/static/
    └── chunks/
        └── main.js (100KB) ← 只有 React 运行时，没有你的组件代码
```

### Client Component 的产物
```
out/
├── index.html (50KB) ← 包含所有规则的 HTML
└── _next/static/
    └── chunks/
        ├── main.js (100KB)
        └── page.js (30KB) ← 额外！包含你的组件代码和数据
```

---

## 为什么会重复？

因为 Client Component 需要：

1. **Hydration（水合）**：让 HTML 变成可交互的
   ```javascript
   // JS 需要重新执行一遍，绑定事件监听器
   ReactDOM.hydrateRoot(document.getElementById('root'), <Page />)
   ```

2. **后续交互**：用户点击、输入时需要重新渲染
   ```typescript
   "use client"
   export default function Page() {
     const [show, setShow] = useState(true)
     // 这个逻辑必须在客户端，所以 JS 必须发送
     return show ? <div>...</div> : null
   }
   ```

---

## 总结

| | HTML 里有内容 | JS 里有组件代码 | 总体积 |
|---|---|---|---|
| **Server Component** | ✅ 有 | ❌ 没有 | 小 |
| **Client Component (output: 'export')** | ✅ 有 | ✅ 也有 | 大（重复） |

**"JS 发不发送"** = **组件代码是否打包到客户端 bundle 里**

你的拆分方式让静态内容不打包 JS，只有交互部分打包，这是最佳实践！🎯

服务端组件是否可以写js交互逻辑
不可以



