# Changelog

All notable changes to `git-manager` skill are documented here.

## [2.0.0] - 2026-04-14

### Added
- **`batch_clone.py`** - 新增 `--workers N` 并发克隆支持，显著加速大批量仓库克隆
- **`batch_clone.py`** - 新增 `--format json` JSON 格式输出，便于程序化调用和集成
- **`batch_clone.py`** - 新增 `--recursive / --no-recursive` GitLab 子组包含控制（默认开启）
- **`batch_clone.py`** - Token 支持环境变量 `GITHUB_TOKEN` / `GITLAB_TOKEN` / `GITEA_TOKEN`，避免命令行暴露敏感信息
- **`batch_clone.py`** - 新增启动信息显示：Token 状态、并发线程数
- **`batch_pull.py`** - 新增 `--dry-run` 预览模式，安全查看即将更新的仓库列表
- **`git_lfs.py`** - 新增 10 个子命令：`install`, `track`, `untrack`, `fetch`, `pull`, `push`, `ls-files`, `ls-tracks`, `scan`, `status`, `migrate`
- **`git_ops.py`** - 扩展至 18 个子命令，新增：`add`, `commit`, `reset`, `stash`, `diff`, `log`, `show`, `blame`, `tag`, `remote`, `clean`, `gc`, `lfs`
- **`README.md`** / **`README_en.md`** - 新增中英双语使用文档
- **`marketplace.json`** - 新增 skills.sh 发现元数据

### Fixed
- **`git_ops.py`** - 修复 pull `--rebase` 参数构造死代码（删除无效的 `cmd[1]` 赋值）
- **`git_lfs.py`** - 修复 track 命令传参错误（`git lfs track add` → `git lfs track`）
- **`batch_clone.py`** - Token 获取逻辑统一使用显式参数或环境变量，避免 None 比较问题

### Changed
- **`batch_clone.py`** - 重构 `batch_clone()` 函数，支持并发和 JSON 输出双模式
- **`SKILL.md`** - 同步新增参数说明和使用示例，补充用户场景映射表

---

## [1.0.0] - 2026-04-14

### Added
- **`batch_clone.py`** - 批量克隆 GitHub / GitLab / Gitea 仓库
- **`batch_pull.py`** - 批量拉取/更新本地 Git 仓库
- **`git_ops.py`** - 单仓库 Git 操作基础命令
- **`references/api_reference.md`** - 各平台 API 端点速查
- **`references/examples.md`** - 完整使用示例
