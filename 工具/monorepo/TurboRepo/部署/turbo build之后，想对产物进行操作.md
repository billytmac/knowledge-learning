明白了，你的问题是：**Turbo 构建完之后，如何知道哪些包的产物发生了变化，从而只部署那些包**。

---

### Turbo 提供了 `--filter` 和输出摘要

```bash
turbo build --output-logs=errors-only --summarize
```

`--summarize` 会生成一份 `.turbo/runs/*.json` 报告，里面记录了哪些包是**真正重新构建的**，哪些是**走了缓存的**：

```json
{
  "tasks": [
    { "taskId": "apps/web#build", "cache": { "status": "MISS" } },  ← 重新构建了
    { "taskId": "apps/admin#build", "cache": { "status": "HIT" } }  ← 走缓存，没变
  ]
}
```

---

### 基于这个做部署判断

```bash
turbo build --summarize

# 读取 summary，找出 cache MISS 的包
REBUILT=$(cat .turbo/runs/*.json | jq -r '
  .tasks[] | select(.cache.status == "MISS") | .taskId
')

echo "$REBUILT" | grep "apps/web" && echo "部署 web..."
echo "$REBUILT" | grep "apps/admin" && echo "部署 admin..."
```

---

### 更简单的方案

直接用 `turbo run deploy`，在每个包的 `package.json` 里定义 `deploy` 脚本，让 Turbo 管理依赖和缓存：

```json
// apps/web/package.json
{
  "scripts": {
    "build": "next build",
    "deploy": "你的部署命令"
  }
}
```

```json
// turbo.json
{
  "pipeline": {
    "deploy": {
      "dependsOn": ["build"],  ← build 完才部署
      "cache": false           ← 部署不需要缓存
    }
  }
}
```

```bash
turbo run deploy  # 自动只部署有变动的包
```

这样 Turbo 会自动处理"build 有变动才 deploy"的逻辑，不需要手动写检测脚本。