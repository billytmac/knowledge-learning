这是个常见的 monorepo 版本管理选择问题，两者定位有些不同，我来帮你梳理一下关键差异：

**Changesets** 更适合大多数现代项目，特别是：
- 需要精细控制每个包版本变更说明的场景
- 团队协作，PR 驱动的发版流程
- 不需要复杂任务编排，只关注版本管理和发布
- 与 pnpm workspaces / Turborepo 搭配使用

**Lerna** 更适合：
- 已有 Lerna 的老项目（迁移成本高）
- 需要任务运行（`lerna run`）+ 版本管理一体化
- 习惯 conventional commits 自动生成 changelog 的团队

**核心对比：**

| | Changesets | Lerna |
|---|---|---|
| 维护状态 | 活跃（Atlassian 维护） | 活跃（Nx 接管后复苏） |
| 版本策略 | 手动写 changeset 文件 | conventional commits 自动推断 |
| Changelog | 自动生成，质量高 | 自动生成 |
| CI 集成 | GitHub Bot 很好用 | 一般 |
| 学习曲线 | 低 | 中 |
| 任务编排 | ❌ 需配合 Turborepo/Nx | ✅ 内置 |

**我的建议：** 新项目直接选 **Changesets + pnpm workspaces**，如果还需要任务编排就加 **Turborepo**。这套组合是目前社区主流，Vercel、Radix UI、tRPC 等知名项目都在用。

