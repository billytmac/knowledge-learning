有必要，但要看你们更在意“提交速度”还是“构建兜底”。

如果是我来落这个仓库，我通常会这样建议：

1. `build` 前至少加 `typecheck`
2. `lint` 不一定强绑到 `build`，更适合放到 CI 或单独命令
3. 用 Turbo 来编排这些任务，收益是明显的

原因很实际。

`typecheck` 更像“能不能正确编译和运行”的底线检查。很多 TS 问题不会被 `next build` 完整替代，尤其 monorepo 里包之间依赖变多之后，单独跑 `tsc --noEmit` 更稳定，也更适合作为构建前置。所以 `build` 前加 `typecheck`，我觉得是值得的。

`lint` 则偏“代码规范 + 一部分潜在问题提醒”。它当然也能拦住 bug，但很多 lint 规则不是构建阻断级别。如果每次 `build` 都先 lint，大仓库里会明显拖慢速度，开发体验容易变差。所以更常见的做法是：

- 本地开发时：按需跑 `lint`
- 提交前或 CI：统一跑 `lint`
- `build` 前：优先只卡 `typecheck`

借助 Turbo 的好处主要有这几个：

- 自动按依赖顺序执行，比如先跑依赖包再跑应用
- 增量执行，只跑受影响的 package/task
- 缓存结果，没变的任务可以直接复用
- 一个命令统一编排 monorepo，不用自己手写很多串联脚本

对你这个仓库来说，Turbo 最适合做的是把流程统一起来，比如：

- `turbo run typecheck`
- `turbo run lint`
- `turbo run build`

以及让 `build` 依赖 `typecheck`，而不是依赖 `lint`。这样通常平衡最好。

我给你的落地建议是：

- 推荐方案：`build` 依赖 `typecheck`
- `lint` 保持独立，放到 CI 或手动执行
- 如果你们团队很重视质量门禁，再考虑让 CI 里同时卡 `lint + typecheck + build`

如果你愿意，我可以直接帮你把 `turbo.json` 和各 package 脚本整理成这一套，尽量最小改动。