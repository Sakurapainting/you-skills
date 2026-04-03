---
name: repo-sync-dev-main
description: "Create a PR from dev to main based on actual changes, merge it, then sync and push dev/main so both branches are aligned safely."
---

# Repo Sync Dev Main

用于对自己有权限的 `origin` 仓库执行 `dev -> main` 的自我 PR 同步流程：基于实际改动生成 PR 标题与描述、创建并合并 PR、再把 `dev` 与 `main` 同步到一致状态。

## 何时使用

仅使用 /repo-sync-dev-main 时

本技能默认目标是用户自己的 `origin` 仓库，且用户对创建 PR、合并 PR、推送分支有权限。

## 输入格式

默认约定：
- 远端：`origin`
- 来源分支：`dev`
- 目标分支：`main`
- 默认 merge 策略：`merge commit`

可接受的附加输入：
- 自定义 source / target branch
- 自定义 remote
- 指定 merge 策略：`merge` / `squash` / `rebase`
- 用户补充本次发布重点或 PR 备注

如果用户没有特别说明，始终按 `origin/dev -> origin/main` 处理。

## 依赖要求

- 必须安装并已登录 GitHub CLI：`gh auth status`
- 当前目录必须是目标仓库的 git 根目录或其子目录
- 远端上必须存在目标分支与来源分支
- 用户必须具备创建 PR、合并 PR、推送分支的权限
- 仓库若启用了 branch protection，必须遵守，不可绕过

## 标准流程

1. 预检查
- 先执行：
  - `gh auth status`
  - `git status --short`
  - `git fetch origin --prune`
  - `git branch -r --list origin/dev origin/main`
  - `git rev-list --left-right --count origin/main...origin/dev`
- 必须确认：
  - working tree 干净
  - `origin/dev` 与 `origin/main` 都存在
  - `dev` 相比 `main` 确实有待同步改动
- 如果 `dev` 与 `main` 没有差异，直接说明无需创建 PR，并停止。

2. 生成 PR 标题和描述
- 必须基于真实 git 输出，优先使用：
  - `git log --oneline origin/main..origin/dev`
  - `git diff --stat origin/main...origin/dev`
  - `git diff --name-only origin/main...origin/dev`
- 标题要求：
  - 反映本次 `dev -> main` 的主要改动主题
  - 简短明确，不使用空泛标题如 `sync dev to main`
  - 如果改动较杂，可使用偏中性的 release/sync 标题，但仍要体现主要模块或收益
- PR body 建议采用以下结构：

```markdown
## Summary
- 

## Included changes
- 

## Validation
- 
```

- 不允许凭空编造变更说明；证据不足时要明确说明不确定性。

3. 检查是否已有现成 PR
- 先检查：
  - `gh pr list --base main --head dev --state open --json number,title,url,headRefName,baseRefName`
- 如果已存在 open 的 `dev -> main` PR：
  - 优先复用该 PR
  - 不重复创建新 PR
  - 报告已有 PR 的编号和链接

4. 创建 PR
- 只有确认不存在 open PR 时，才创建：
  - `gh pr create --base main --head dev --title "<generated title>" --body "<generated body>"`
- 标题和描述必须来源于当前实际 diff / commit，而不是固定模板。

5. 检查 PR 状态与可合并性
- 创建后或复用现有 PR 后，必须检查：
  - `gh pr view <PR> --json number,title,url,state,mergeable,reviewDecision,statusCheckRollup,headRefName,baseRefName`
  - `gh pr checks <PR>`
- 如果存在以下任一情况，则停止：
  - merge conflict
  - required checks 失败
  - 无 merge 权限
  - PR 已关闭或状态异常
- 停止时明确列出阻塞项和建议下一步动作。

6. 合并 PR
- 默认使用 merge commit：
  - `gh pr merge <PR> --merge`
- 可选策略：
  - `gh pr merge <PR> --squash`
  - `gh pr merge <PR> --rebase`
- 只有在用户明确要求或仓库策略要求时才使用 `squash` / `rebase`。
- 不允许绕过保护规则，不允许使用强制或破坏性参数。

7. merge 后同步 `dev` 与 `main`
- PR 合并成功后，继续执行：
  - `git fetch origin --prune`
  - `git checkout main`
  - `git pull --ff-only origin main`
  - `git checkout dev`
  - 优先执行 `git merge --ff-only origin/main`
  - `git push origin dev`
- 最终验证：
  - `git fetch origin --prune`
  - `git rev-parse origin/main`
  - `git rev-parse origin/dev`
- 目标结果：
  - `origin/main` 与 `origin/dev` 指向同一提交

8. 结果汇报
- 输出应包含：
  - 是否创建了新 PR，或复用了已有 PR
  - PR 标题
  - PR 链接
  - merge 是否成功
  - `origin/dev` 与 `origin/main` 是否已对齐
- 如果任一步骤因保护规则、冲突或权限问题中止，要明确写出阻塞原因。

## 执行约束

- 所有 PR 标题与描述必须基于实际 `git` / `gh` 输出生成。
- 不允许编造变更原因、测试结果或合并状态。
- 不允许在 dirty working tree 下继续执行切分支、merge 或 push。
- 不允许 force push。
- 不允许绕过 branch protection、required checks 或权限控制。
- 如果已存在 open 的 `dev -> main` PR，不要重复创建。
- 如果 merge 或 checks 存在阻塞，不要继续执行 post-merge sync。
- 如果远端状态在执行过程中发生变化，必须重新 `fetch` 并重新判断。
- 如果 `git merge --ff-only origin/main` 失败，说明 `dev` 已发生新的分叉，必须停止并汇报，不要擅自改用危险命令修复。
- 默认 remote/source/target 为 `origin` / `dev` / `main`，除非用户明确指定其他值。
- 这是一个会影响远端仓库的工作流；执行创建 PR、合并 PR、push 前必须明确告诉用户将要执行的动作，并在缺少明确授权时先征求确认。

## 推荐输出结构

```markdown
# Dev -> Main 同步结果

## 1. 基本信息
- 仓库：
- 远端：
- 来源分支：
- 目标分支：
- merge 策略：

## 2. PR 信息
- 是否复用现有 PR：
- PR：
- 标题：
- 链接：

## 3. 执行结果
- PR 创建：成功 / 跳过 / 失败
- PR 合并：成功 / 跳过 / 失败
- 分支同步：成功 / 跳过 / 失败

## 4. 最终状态
- `origin/main`：
- `origin/dev`：
- 是否一致：是 / 否

## 5. 阻塞项或后续动作
- 
```

## 失败兜底

当出现以下情况时，必须停止并给出最小修复建议：

1. `gh` 不可用或未登录
- 提示检查：
  - `gh --version`
  - `gh auth status`
  - `gh auth login`

2. 当前目录不是 git 仓库
- 明确提示需要在目标仓库目录下重新执行。

3. 缺少 `origin/dev` 或 `origin/main`
- 明确提示缺失的分支名，并要求用户提供正确分支。

4. working tree 不干净
- 明确提示先提交或 stash 本地改动，再重新执行。

5. 已存在 open 的 `dev -> main` PR
- 报告该 PR 链接与编号，并继续基于该 PR 判断是否可合并；不要重复创建。

6. PR 有冲突、checks 失败或无权限
- 列出具体阻塞项。
- 不执行 merge，也不执行后续 branch sync。

7. merge 后 `dev` 无法安全 fast-forward 到 `origin/main`
- 明确提示 `dev` 已出现新的分叉或远端状态已变化。
- 不要使用 force push 或破坏性修复手段。
- 向用户汇报当前状态并请求下一步决策。
