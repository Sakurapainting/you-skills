# You Skills

[中文说明](README.zh-CN.md)

A collection of reusable AI assistant skills and workflow assets.

## Overview

This repository hosts vendor-specific skill packages and supporting documentation designed to improve developer workflows.

## Features

- Reusable skill packages for specific engineering tasks.
- Clear vendor-specific instruction files per skill.
- A scalable `vendor/skill` directory layout.

## Project Structure

- `copilot/pr-analyze/`: GitHub Copilot version of the pull request analysis skill.
- `claude/pr-analyze/`: Claude version of the pull request analysis skill.

## Getting Started

1. Clone the repository.
2. Open the folder in your editor.
3. Place Copilot skill files under `~/.copilot/skills/<skill-name>/` and keep `SKILL.md` as the main instruction file.
4. Place Claude skill files under `~/.claude/skills/<skill-name>/` and keep `CLAUDE.md` as the main instruction file.
5. If you use GitHub Copilot, read `copilot/pr-analyze/SKILL.md`.
6. If you use Claude, read `claude/pr-analyze/CLAUDE.md`.
7. Adjust or extend skill content based on your team needs.

## Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch: `git checkout -b feat/your-change`.
3. Keep changes focused and document intent clearly.
4. Keep vendor-specific conventions inside their own folders.
5. Commit with meaningful messages.
6. Open a pull request with:
   - A concise summary of what changed
   - Why the change is needed
   - Any validation notes

## Roadmap

- Add more domain-focused skills.
- Improve examples and troubleshooting guidance.
- Standardize contribution templates.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
