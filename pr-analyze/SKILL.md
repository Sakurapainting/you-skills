---
name: pr-analyze
description: "Comprehensive GitHub PR analysis workflow for VS Code Copilot. Use when user provides a PR number or URL and asks to analyze purpose, impact, CI status, related PRs, and review risks."
---

# PR Analyze

用于在 VS Code Copilot 中对 GitHub Pull Request 做结构化分析，并输出中文结论。

## 何时使用

当用户出现以下意图时自动使用本技能：
- 帮我分析 PR
- 这个 PR 是干什么的
- 看下这个 PR 的影响范围
- 评估这个 PR 能不能合并
- 用户直接给出 PR 链接或 `owner/repo#123` 形式

## 输入格式

支持以下输入：
- PR 编号：`123`（当前目录是目标仓库）
- PR URL：`https://github.com/owner/repo/pull/123`
- 简写：`owner/repo#123`

## 依赖要求

- 必须安装并已登录 GitHub CLI：`gh auth status`
- 当前目录建议是 git 仓库根目录

## 标准流程

1. 识别仓库与 PR
- 如果是 URL 或 `owner/repo#123`，直接提取 owner/repo 与编号。
- 如果只有编号，先用 git remote 推断仓库。

2. 拉取 PR 基础信息
- `gh pr view <PR> --repo <owner/repo> --json title,body,state,author,baseRefName,headRefName,files,additions,deletions,commits,mergeable,labels`
- `gh pr diff <PR> --repo <owner/repo>`

3. 生成功能摘要
- 用 1-3 句话说明 PR 目标、改动面、主要收益。
- 如果 PR 关联 Issue（Closes/Fixes/Resolves），提取并记录。

4. 兼容性与风险评估
- 从以下维度评估：
  - API/协议兼容性
  - 配置与环境变量
  - 数据结构/存储
  - 依赖升级与许可证风险
  - 安全与权限边界
- 结论用 `低/中/高` 风险等级。

5. 查重与关联 PR
- 根据关键词搜索历史 PR：
  - `gh pr list --repo <owner/repo> --state all --search "<keyword>" --json number,title,state,author`
- 标注关系类型：重复/互补/依赖/冲突/无关。

6. CI 与可合并性检查
- `gh pr checks <PR> --repo <owner/repo>`
- 若失败，列出失败项、可能原因、建议修复顺序。

7. 代码评审重点（轻量）
- 优先找：
  - 行为回归风险
  - 边界条件遗漏
  - 错误处理与日志缺失
  - 测试覆盖不足
- 输出 `阻塞问题` 与 `建议优化` 两类。

8. 输出结构化报告
- 使用以下模板输出，必须中文：

```markdown
# PR 分析报告

## 1. 基本信息
- 仓库：
- PR：
- 标题：
- 作者：
- 状态：
- 目标分支 / 来源分支：

## 2. 这个 PR 做了什么
-

## 3. 改动范围
- 文件数 / 增删行：
- 主要模块：

## 4. 兼容性与风险
| 维度 | 结论 | 说明 |
|---|---|---|
| API | 低/中/高 | |
| 配置 | 低/中/高 | |
| 数据 | 低/中/高 | |
| 依赖 | 低/中/高 | |
| 安全 | 低/中/高 | |

## 5. 关联项
- 关联 Issue：
- 相关 PR：

## 6. CI 与可合并性
- CI 状态：
- 可合并性：

## 7. 评审发现
### 阻塞问题
1.

### 建议优化
1.

## 8. 结论
- 建议：可合并 / 修复后合并 / 暂不合并
- 需要行动：
```

## 执行约束

- 优先基于命令输出给结论，不要臆测。
- 如果信息不足，明确写证据不足，并给出下一条应执行命令。
- 不要输出与 PR 无关的大段背景知识。
- 每次分析必须同时输出到对话和 Markdown 文件（不可省略落盘）。

## 报告落盘（必需）

每次执行分析都必须保存一份 Markdown 报告，规则如下：

1. 目录规则
- 默认目录：`$HOME/.copilot/skills/pr-analyze/reports`
- 如果目录不存在，先创建目录。

2. 文件命名
- 文件名格式：`pr-{repo_slug}-{pr_number}-{YYYYMMDD-HHMMSS}.md`

3. 执行步骤
- 写入 Markdown 文件。
- 最后回显“报告已保存到: <绝对路径>”。

## 失败兜底

当 `gh` 不可用或未登录时：
1. 明确提示缺失条件（CLI 或鉴权）。
2. 给出最小修复步骤：
   - `gh --version`
   - `gh auth login`
3. 请求用户提供 PR URL 后重试。
