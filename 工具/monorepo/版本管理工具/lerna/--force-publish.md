## `--force-publish` 用法

### 基本语法

```bash
# 强制所有包都发布新版本
lerna version --force-publish

# 只强制指定的包
lerna version --force-publish=package-a,package-b

# 用通配符
lerna version --force-publish=@myorg/*
```

---

### 具体行为

正常情况下，Lerna 只 bump 有 git 变更的包：
```
utils  v1.0.0 → v1.0.1  ✅ 有变更
core   v1.0.0            ❌ 没变更，跳过
ui     v1.0.0            ❌ 没变更，跳过
```

加了 `--force-publish` 后，没有变更的包也会被 bump（至少 patch）：
```
utils  v1.0.0 → v1.0.1  ✅
core   v1.0.0 → v1.0.1  ✅ 强制
ui     v1.0.0 → v1.0.1  ✅ 强制
```

---

### 常见使用场景

**1. 首次发布或重置基准**
```bash
# 项目初始化，所有包统一发一次
lerna version 1.0.0 --force-publish
```

**2. 基础包变更，强制下游跟进**
```bash
# core 改了，强制所有依赖它的包也发版
lerna version --force-publish=ui,dashboard
```

**3. Fixed 模式下保持版本一致**

Fixed 模式（所有包同一版本号）时，`--force-publish` 几乎是标配，否则没变更的包版本号会落后：
```bash
# lerna.json
{
  "version": "1.2.0",  # fixed 模式
}

lerna version --force-publish  # 保证所有包版本号同步
```

---

### 配置到 `lerna.json` 避免每次手写

```json
{
  "version": "independent",
  "command": {
    "version": {
      "forcePublish": true
    }
  }
}
```

这样直接运行 `lerna version` 就默认带上这个行为了。