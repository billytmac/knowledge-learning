`lerna changed` 是用来**预览哪些包在下次发布时会有版本变更**的命令。

---

**作用：**

在执行 `lerna version` 或 `lerna publish` 之前，先看看哪些包有改动：

```bash
$ lerna changed

pkg-a
pkg-b

Found 2 packages ready to publish
```

---

**判断逻辑和 `lerna version` 完全一样：**

```
找到上一个 git tag
        ↓
扫描 tag 之后有代码变动的包
        ↓
列出这些包的名称
```

---

**常用参数：**

```bash
lerna changed          # 列出有变动的包名
lerna changed -l       # 同时显示当前版本号
lerna changed --json   # 以 JSON 格式输出
```

输出示例：
```bash
$ lerna changed -l

pkg-a  v1.0.0
pkg-b  v2.3.1

Found 2 packages ready to publish
```

---

**典型使用场景：**

发版前先跑一下确认范围，避免 `lerna version` 执行后才发现升错了包：

```bash
lerna changed   # 先确认
lerna version   # 再发版
```

本质上就是 `lerna version` 的**dry run 预览版**，不会实际修改任何东西。

准确说是**从上一个 tag 到当前 HEAD 之间的所有 commit**，HEAD 就是你最新的提交。

```
pkg-a@1.0.0 (上一个tag)
      ↓
   commit A  (fix: 修复登录)
   commit B  (feat: 新增导出)
   commit C  (docs: 更新文档)
      ↓
   HEAD (当前最新提交)
```

`lerna changed` 扫描的就是这个区间内所有有改动的包。

---

**所以不是"只比较 tag"，而是：**

- 起点 = 上一个 tag
- 终点 = 当前 HEAD（最新提交）
- 中间所有 commit 都会被扫描

---

**特殊情况：**

```bash
# 如果从没打过 tag
# Lerna 会扫描整个 git 历史的所有 commit
```

这也是为什么第一次执行 `lerna version` 前最好先手动打一个初始 tag，避免把历史所有 commit 都算进去。