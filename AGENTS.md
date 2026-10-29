# 给 AI 编程助手的约定

- 用简体中文回复，进度说明也一样；提交信息用英文。
- 开发、测试和代码约定见 [CONTRIBUTING.md](CONTRIBUTING.md)，这里不重复。要点：`bun run check` 必须通过；
  验证用 `bun run dev:fake`（本地假 Kmoe），不要用真实账号触发下载。
- 对 Kmoe 友好：批量请求每 10 秒一个页面，不加并发。请求太快会被限制整个 IP。
- 新的默认值只影响之后，不迁移、不改写用户已有的数据和设置。
- 发版：改 `package.json` 的版本、写 `CHANGELOG.md`（含底部的对比链接）、更新
  `.github/ISSUE_TEMPLATE/bug_report.yml` 里的版本示例；合到 `main` 后打 `vX.Y.Z` 注释标签并推送。
  Release 工作流先跑检查，再构建并推送 GHCR 镜像；NAS 上在 Portainer 里重新拉取镜像部署。
- 这是公开仓库：推送和打标签都会对外发布，先确认再做；不要提交密钥、内网地址或本地评审文件。

## 本地 fork 的分支约定

（这一节只存在于 `local` 分支，不要带进任何 PR。）

- `main`：上游的镜像，不放自己的提交。同步：
  `git fetch upstream && git switch main && git reset --hard upstream/main && git push --force-with-lease fork main`。
- `fix/<主题>`：一个 bug 一个分支，从 `upstream/main` 切，用来开 PR；合并或被拒之后删掉。
- `local`：自用的集成分支 = 上游 + 还没被合并的修复。上游长期不维护时，`local` 就是这个项目最新的可用版本；
  镜像从 `local` 打 tag 构建。

每次提交放进哪个分支：

- 准备提给上游的修复 → `fix/<主题>`，再合进 `local` 供本地运行。
- 只给自己用的改动（部署方式、本文件的这一节、个人配置）→ 只提交到 `local`，不要放进 `fix/*`。
- 上游拒绝或放弃的 PR → 从 `local` 上去掉（`git revert`），不让已废弃的改动留在 `local` 里。

**PR 一律先给我看草稿，我同意后才提交**（这一条不分分支，任何时候都适用）：标题、正文全文、要提交的 diff 概要
先发出来，等我说可以，才运行 `gh pr create`。不要自己决定开 PR，也不要把草稿改成 ready for review、关闭或重开 PR。
仓库是公开的，PR 一旦提交就对外可见；我不同意时，草稿只能留在本地。

同步与冲突：`fix/*` 与上游同步用 `git rebase upstream/main`（PR 历史保持干净，force-push 用 `--force-with-lease`）；
`local` 与上游同步用 `git merge upstream/main`，不 force-push。

发布与构建：在 `local` 上打 `vX.Y.Z-jerry.N` 标签并推送，fork 的 Actions 会先跑 `bun run check`，
再构建 amd64 / arm64 镜像推到 `ghcr.io/jerrylf7/kmoe-sync`；NAS 的 compose 用具体版本号，不用 `:latest`。
