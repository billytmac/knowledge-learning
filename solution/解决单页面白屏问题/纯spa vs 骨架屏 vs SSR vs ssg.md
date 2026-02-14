## SPA 的白屏问题

传统 SPA（如纯 React/Vue）的加载过程：

```html
<!-- 用户首次访问看到的 HTML -->
<html>
  <body>
    <div id="root"></div>  <!-- 空白！ -->
    <script src="bundle.js"></script>
  </body>
</html>
```

**问题流程：**
1. 浏览器下载 HTML（几乎是空的）
2. 下载并解析 JS bundle（可能很大，几百 KB 到几 MB）
3. JS 执行，React/Vue 渲染页面
4. 可能还要发起 API 请求获取数据
5. 最终渲染出内容

**白屏时间 = 步骤 1-5 的总耗时**，在慢网络下可能长达几秒。

## 预渲染的解决方案

### SSR（服务端渲染）

```javascript
// Next.js SSR
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}

export default function Page({ data }) {
  return <div>{data.title}</div>;
}
```

**返回的 HTML：**
```html
<html>
  <body>
    <div>实际的内容已经在这里了！</div>
    <script src="bundle.js"></script>
  </body>
</html>
```

用户立即看到内容，JS 加载后"水合"（hydration）使页面可交互。

### SSG（静态生成）

```javascript
// 构建时预渲染
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data } };
}
```

构建时就生成好带内容的 HTML，CDN 直接返回，速度更快。

## 其他解决白屏的方法对比

### 1. 骨架屏（Skeleton Screen）

```jsx
// 不是真正的内容，但比白屏好
function Page() {
  const { data, isLoading } = useSWR('/api/data');
  
  if (isLoading) {
    return <Skeleton />; // 显示占位符
  }
  
  return <Content data={data} />;
}
```

**优点：** 简单，不需要服务端
**缺点：** SEO 不友好，仍需等待 JS 加载

### 2. Loading 状态

```jsx
<div id="root">
  <div class="loading">加载中...</div>
</div>
```

**优点：** 最简单
**缺点：** 体验一般，SEO 不友好

### 3. 代码分割 + 懒加载

```javascript
const HeavyComponent = lazy(() => import('./Heavy'));

// 减少首次 bundle 大小，加快初始渲染
```

**优点：** 减少白屏时间
**缺点：** 不能完全消除白屏

### 4. 预渲染（Prerendering）工具

```javascript
// 使用 react-snap 等工具
// 爬取 SPA 的所有路由，生成静态 HTML
```

**优点：** 不需要改造成 SSR
**缺点：** 只适合静态内容，动态内容仍有问题

## 预渲染的优势

| 方法 | 首屏速度 | SEO | 动态内容 | 实现难度 |
|------|---------|-----|---------|---------|
| 纯 SPA | ❌ 慢 | ❌ 差 | ✅ 好 | ✅ 简单 |
| 骨架屏 | ⚠️ 一般 | ❌ 差 | ✅ 好 | ✅ 简单 |
| SSG 预渲染 | ✅ 快 | ✅ 好 | ⚠️ 需 ISR | ⚠️ 中等 |
| SSR 预渲染 | ✅ 快 | ✅ 好 | ✅ 好 | ❌ 复杂 |

## 实际效果对比

```
纯 SPA 白屏时间：
HTML (50ms) + JS下载 (1s) + JS执行 (500ms) + API (300ms) = 1.85s

SSR 预渲染：
HTML (200ms) + 立即显示内容 → JS水合 (1s后可交互)
首屏：200ms，可交互：1.2s

SSG 预渲染：
HTML (50ms, 来自CDN) + 立即显示内容 → JS水合 (1s后可交互)  
首屏：50ms，可交互：1s
```

所以**预渲染是目前解决白屏问题最有效的方案**，特别是结合 Next.js 这类框架时。如果你的应用需要好的首屏体验和 SEO，预渲染几乎是必选项。