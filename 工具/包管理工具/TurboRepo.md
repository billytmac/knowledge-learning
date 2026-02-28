# 什么是 Turborepo
Turborepo 是一个高性能的构建系统，专为 JavaScript 和 TypeScript 的 Monorepo 项目设计。它提供了一种高效管理和构建项目中多个包的方式，通过缓存先前构建和测试的结果来显著减少重复工作的需要，从而加快开发和持续集成的流程。Turborepo 旨在提高大型 Monorepo 项目的构建效率，特别是在复杂的项目中，它可以处理依赖关系、执行任务、并确保构建的正确性和效率。


```
{
  "$schema": "https://turbo.build/schema.json",
  "ui": "tui",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["$TURBO_DEFAULT$", ".env*"],
      "outputs": [".next/**", "!.next/cache/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "check-types": {
      "dependsOn": ["^check-types"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

这段是 `TurboRepo` 的任务编排配置，作用是告诉 turbo 在 monorepo 里怎么跑、怎么缓存、任务之间怎么依赖。

你这份配置的含义：

- `"$schema"`  
  指向 turbo 配置的 JSON schema，主要给编辑器做提示和校验。

- `"ui": "tui"`  
  用 turbo 的终端交互界面（TUI）显示任务状态。

- `"tasks.build"`  
  - `"dependsOn": ["^build"]`：先跑当前包依赖包的 `build`，再跑自己。`^` 表示“上游依赖包”。
  - `"inputs": ["$TURBO_DEFAULT$", ".env*"]`：缓存计算时，除了默认输入，还把 `.env*` 也算进去；环境变量变化会触发重跑。
  - `"outputs": [".next/**", "!.next/cache/**"]`：声明产物目录。turbo会缓存这些输出，但排除 `.next/cache`。

- `"tasks.lint"`  
  - `"dependsOn": ["^lint"]`：先 lint 依赖包，再 lint 当前包。

- `"tasks.check-types"`  
  - `"dependsOn": ["^check-types"]`：先检查依赖包类型，再检查当前包。  
  注意：你的包脚本里如果叫 `typecheck`，这里就要叫 `typecheck`，不然 `turbo run check-types` 会找不到对应 script。

- `"tasks.dev"`  
  - `"cache": false`：开发任务不走缓存。
  - `"persistent": true`：这是长期运行任务（如 `next dev`），turbo 不会等它自动退出。

Turbo 怎么用（常见）：

1. 在根目录安装并配置 `turbo.json`。  
2. 每个子包 `package.json` 里定义同名脚本（如 `build`、`lint`、`dev`）。  
3. 用命令运行：  
   - `pnpm turbo run build`  
   - `pnpm turbo run lint`  
   - `pnpm turbo run dev --filter=web`（只跑某个包）  

核心价值：并行执行 + 依赖顺序 + 增量缓存（本地/远程），让 monorepo 更快。