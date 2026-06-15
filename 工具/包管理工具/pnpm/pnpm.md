# 到指定目录路径
pnpm --filter <package-name> 
pnpm --dir <package-name> 
pnpm -C <package-name> 
## 举例
pnpm --filter my-app add lodash
pnpm --dir ./packages/my-app install
pnpm -C my-app add lodash
# 在monorepo安装某个本地包
pnpm -C <package-name>@workspace:*

这是 **pnpm** 特有的 workspace 协议，用于 monorepo 中引用本地包。

## `workspace:*`

```json
"my-package": "workspace:*"
```

- 引用本地 workspace 中的包
- `*` 表示**不限制版本**，始终使用本地版本
- 发布时会被替换为该包的**实际版本号**（如 `1.2.3`）

## `workspace:^`

```json
"my-package": "workspace:^"
```

- 同样引用本地包
- `^` 表示发布时替换为**带 `^` 的版本号**（如 `^1.2.3`），允许小版本升级

---

## 对比总结

| 协议 | 开发时 | 发布后替换为 |
|------|--------|------------|
| `workspace:*` | 本地包 | `1.2.3`（精确版本） |
| `workspace:^` | 本地包 | `^1.2.3`（兼容版本） |
| `workspace:~` | 本地包 | `~1.2.3`（补丁版本） |

补充：
`^` 和 `~` 都是语义化版本范围，但松紧不同。`^1.2.3` 允许 `1.x` 范围内的更新（也就是 `>=1.2.3 <2.0.0`），`~1.2.3` 只允许 `1.2.x` 的补丁更新（也就是 `>=1.2.3 <1.3.0`）。如果主版本是 `0`，`^0.2.3` 会更严格（只允许到 `<0.3.0`），因为 `0` 代表不稳定版本。

**开发阶段**三者效果一样，区别只体现在**将包发布到 npm 时**，pnpm 会自动将 `workspace:` 协议替换为对应格式的真实版本号。

如果不需要发布，通常用 `workspace:*` 最简单直接。