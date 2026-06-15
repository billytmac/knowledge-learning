执行 version:bump 后，Lerna 会自动：

分析每个包自上次 tag 以来的 commit
根据 commit 类型（fix/feat/breaking）自动决定版本号
更新各 package.json 的 version
在每个包目录生成 CHANGELOG.md
创建 git commit + git tag
推送到远程
注意：由于你的包都是 "private": true，第一次运行时可能需要加 --force-publish 参数。如果后续需要发布到 npm，去掉 private 字段即可配合 lerna publish 使用。