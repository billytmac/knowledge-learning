这是 Next.js 官方提供的**自动升级工具**。

---

### 拆解每个部分

**`npx @next/codemod`** — 运行 Next.js 官方的代码迁移工具

**`upgrade`** — 执行升级操作

**`canary`** — 升级到 canary 版本（每日构建的最新版，比 latest 更新但不稳定）

---

### 它会做什么

1. 把你项目的 `next` 版本升级到指定版本
2. 自动扫描你的代码，找出**不兼容的写法**
3. 自动修改代码，适配新版本的 API

---

### 常见的版本目标

```bash
npx @next/codemod upgrade latest   # 升级到最新稳定版
npx @next/codemod upgrade canary   # 升级到最新预览版
npx @next/codemod upgrade 15       # 升级到指定大版本
```

---

### 什么是 canary 版本

```
stable(latest)  → 正式版，生产环境用
canary          → 每日构建，包含最新特性但可能不稳定
```

---

### 什么时候用这个命令

- 跨大版本升级 Next.js（比如 14 → 15）
- 不想手动一个个改不兼容的 API
- 想体验最新特性但不想手动迁移

一般**生产项目不建议升级到 canary**，用 `latest` 更稳妥。