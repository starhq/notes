
# git提交信息规范化

```shell
npm install -g commitizen cz-conventional-changelog

echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc

# 后续用cz来替代git commit
```

# 校验git提示信息是否符合规范

```shell
# 先初始化git仓库,再执行如下操作
bun add -d @commitlint/{cli,config-conventional} husky

npx husky init

echo 'npx --no-install commitlint --edit "$1"' > .husky/commit-msg

echo 'export default { extends: ["@commitlint/config-conventional"] };' > commitlint.config.js
```