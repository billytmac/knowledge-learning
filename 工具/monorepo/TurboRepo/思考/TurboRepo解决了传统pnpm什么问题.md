## TurboRepo 解决的问题

### 传统 pnpm workspace 的痛点

在 monorepo 里，你通常需要按顺序执行任务：

```bash
pnpm --filter @app/core build
pnpm --filter @app/utils build  # 依赖 core，必须等 core 先完成
pnpm --filter @app/ui build     # 依赖 utils，必须等 utils 先完成
```

包一多，这个过程就会很慢，而且每次都要**从头跑一遍**。

---

### TurboRepo 解决的三个核心问题

#### 1. 没有并行执行 — 任务串行很慢

pnpm 本身不理解任务之间的依赖关系，默认串行执行。

```
pnpm：  core ──→ utils ──→ ui ──→ ...   （一个个来）

Turbo： core ──→ utils ──→ ui
              ↘ button ──→ ...          （没有依赖关系的并行跑）
```

Turbo 会分析依赖图，**没有依赖关系的包同时执行**，充分利用 CPU 多核。

---

#### 2. 没有缓存 — 没改的包也要重新跑

这是最大的痛点。你只改了 `ui` 包，但每次 CI 还是要把所有包的 `build`、`lint`、`test` 全跑一遍。

```
传统方式：
  改了 ui → core 没变也要重新 build ← 纯浪费时间

Turbo：
  改了 ui → core 没变 → 直接用上次的缓存结果 ✅
```

Turbo 会对每个任务的**输入（源码 + 依赖）做哈希**，输入没变就直接复用输出。

---

#### 3. 缓存只在本地 — 换台机器或 CI 又从头来

本地缓存解决了开发机的问题，但 CI 每次都是全新环境，缓存失效。

Turbo 支持**远程缓存（Remote Cache）**，把缓存推到云端：

```
同事 A 在本地跑了 build，缓存上传到云端
同事 B / CI 拉到同样的代码 → 直接复用 A 的缓存结果，0 秒完成
```

---

### 对比总结

| 问题 | pnpm workspace | TurboRepo |
|------|---------------|-----------|
| 任务执行 | 串行 | 并行（依赖图调度） |
| 重复构建 | 每次全量跑 | 输入未变直接用缓存 |
| CI 缓存 | 无 | 远程缓存，团队共享 |
| 任务编排 | 需手动控制顺序 | 自动分析依赖图 |

---

### 一句话总结

> pnpm 解决的是"装包"，TurboRepo 解决的是"**跑任务要快**"。两者互补，现代 monorepo 通常两个都用。



----------------------

Turbo 一般就是拿来编排 monorepo 里的“跨包任务”，尤其是这些最常见：

- `build`
- `dev`
- `lint`
- `typecheck`
- `test`
- `storybook`
- `e2e`
- `format` 或 `fmt:check`
- 代码生成任务，比如 `gen`、`codegen`

它最适合的场景不是“单个项目里跑一个命令”，而是：

- 有多个 app / package
- 这些包之间有依赖关系
- 你希望只跑受影响的任务
- 你希望缓存结果，加快二次执行

比如你的仓库就很适合用 Turbo 去编排：

- `web` 的 `build`
- `ui` 包的 `lint`
- 未来如果 `ui` 加了 `typecheck` / `build`，也能一起串起来

一个典型思路是：

- `build`：构建所有需要发布或运行的包
- `typecheck`：跑所有包的 TS 检查
- `lint`：跑所有包的规范检查
- `test`：跑单测
- `dev`：启动主应用开发环境
- `build` 依赖 `^build`
- `typecheck` 依赖 `^typecheck`
- `lint` 依赖 `^lint`

这里的 `^` 很关键，意思是先执行依赖包的同名任务。

Turbo 的价值主要不是“替你发明任务”，而是把各包已有的脚本统一调度起来，所以前提通常是每个包先有一致的脚本名，比如都叫：

- `build`
- `lint`
- `typecheck`
- `test`

一句话说，Turbo 一般用来做这件事：  
“把 monorepo 里各 package 的同名任务，按依赖关系、增量范围和缓存机制统一跑起来。”

如果你想，我下一步可以直接结合你这个仓库，给你列一个“最适合当前项目的 Turbo 编排清单”。