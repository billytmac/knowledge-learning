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

**开发阶段**三者效果一样，区别只体现在**将包发布到 npm 时**，pnpm 会自动将 `workspace:` 协议替换为对应格式的真实版本号。

如果不需要发布，通常用 `workspace:*` 最简单直接。