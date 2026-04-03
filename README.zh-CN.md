# You Skills

[English](README.md)

这是一个可复用的 AI 助手技能与工作流资源集合。

## 项目简介

本仓库用于存放按厂商区分的技能包及配套文档，帮助团队在不同 AI 助手场景下复用工程工作流。

## 主要特性

- 面向特定工程任务的可复用技能包。
- 每个技能提供清晰的厂商专属说明文件。
- 采用可扩展的 `vendor/skill` 目录结构。

## 目录结构

- `copilot/pr-analyze/`：GitHub Copilot 版本的 Pull Request 分析技能。
- `claude/pr-analyze/`：Claude 版本的 Pull Request 分析技能。
- `claude/repo-sync-dev-main/`：Claude 版本的自有仓库 `dev -> main` 自我 PR 同步技能。

## 快速开始

1. 克隆仓库。
2. 使用编辑器打开仓库目录。
3. Copilot skill 文件应放在 `~/.copilot/skills/<skill-name>/` 下，并以 `SKILL.md` 作为主说明文件。
4. Claude skill 文件应放在 `~/.claude/skills/<skill-name>/` 下，并以 `SKILL.md` 作为主说明文件，同时补充 `name` / `description` 头信息。
5. 如果你使用 GitHub Copilot，请阅读 `copilot/pr-analyze/SKILL.md`。
6. 如果你使用 Claude，请根据场景阅读 `claude/pr-analyze/SKILL.md` 或 `claude/repo-sync-dev-main/SKILL.md`。
7. 根据团队规范调整或扩展技能内容。

## 贡献指南

欢迎贡献。

1. Fork 本仓库。
2. 创建功能分支：`git checkout -b feat/your-change`。
3. 保持改动聚焦，并清晰说明修改意图。
4. 厂商专属规范应保留在各自目录中，不要混写。
5. 使用明确的提交信息进行提交。
6. 提交 Pull Request，并附上：
   - 本次改动摘要
   - 改动动机
   - 验证说明（如有）

## 规划

- 增加更多面向场景的技能包。
- 补充示例与常见问题排查文档。
- 统一贡献模板与协作约定。

## 许可证

本项目采用 MIT License，详情见 [LICENSE](LICENSE)。
