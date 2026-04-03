# repo-sync-dev-main skill

Claude 版本。

用于对自己有权限的仓库执行 `dev -> main` 的自我 PR 同步流程：

- 基于实际改动生成 PR 标题和描述
- 创建或复用 `dev -> main` PR
- 合并 PR
- 把 `dev` 与 `main` 同步到一致状态

最小使用示例：

- 帮我把 dev 合到 main，并同步 dev/main
- 为这个仓库创建一个 dev -> main 的 PR，标题和描述根据实际改动生成，然后合并
- 做一次自己的仓库 release sync：dev -> main -> merge -> sync dev

建议前置检查：

```bash
gh --version
gh auth status
git status --short
```

默认策略：

- remote：`origin`
- source：`dev`
- target：`main`
- merge strategy：`merge`

说明：

- 默认优先 `merge commit`，便于 merge 后把 `dev` fast-forward 到 `main`
- 如果仓库策略要求，也可以显式改成 `squash` 或 `rebase`
- 如果有 open 的 `dev -> main` PR，应优先复用，不重复创建
- 这是有远端副作用的工作流，创建 PR、合并 PR、push 前应先向用户确认
