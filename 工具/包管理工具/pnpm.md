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