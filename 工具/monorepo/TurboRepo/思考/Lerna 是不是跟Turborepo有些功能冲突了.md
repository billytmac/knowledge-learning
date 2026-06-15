没有冲突，两者几乎**完全不重叠**，分工很清晰：

## 各自管什么

| 功能 | Lerna | Turborepo |
|------|-------|-----------|
| 版本号管理 | ✅ | ❌ |
| npm 发布 | ✅ | ❌ |
| Changelog 生成 | ✅ | ❌ |
| 任务编排（build/test/lint） | ⚠️ 有但很弱 | ✅ 核心功能 |
| 增量构建缓存 | ❌ | ✅ |
| 远程缓存 | ❌ | ✅ |
| 并行任务执行 | 基础支持 | ✅ 智能调度 |

---

## 唯一有点重叠的地方

Lerna 有个 `lerna run` 命令，可以跨包执行脚本：

```bash
lerna run build   # 在所有包里执行 npm run build
lerna run test
```

Turbo 的 `turbo run` 也做同样的事，但更强：
- 自动分析依赖拓扑，保证 `core` 先于 `ui` 构建
- 有缓存，没变更的包直接跳过
- 并行度更智能

---

## 所以实际组合方式

**现代推荐用法：**
```
Turborepo   →  负责 run（构建/测试/lint 的编排和缓存）
Lerna       →  负责 version + publish（版本和发布）
```

两者可以直接共存，`lerna.json` 和 `turbo.json` 放在同一个项目根目录下互不干扰。

Lerna 官方文档也明确说：**把任务执行交给 Turbo，Lerna 只做发布**，这是推荐的搭配。