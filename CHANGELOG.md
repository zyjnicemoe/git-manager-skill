# Changelog

All notable changes to `git-manager` skill are documented here.

## [2.4.0] - 2026-04-15

### Changed
- **`SKILL.md`**: 全面重构，从 420 行压缩至 ~250 行
  - 删除与 `README.md` 重复的命令示例、注意事项等冗余内容
  - 聚焦 AI 工作流指导（用户意图 → 脚本映射表）
  - 修复不存在文件的错误引用
- **`references/`**: 重建目录结构
  - 新建 `references/api_reference.md`：完整 API 端点 + 认证方式 + 通用请求模板
  - `references/examples.md`：与根目录 examples.md 保持同步
- **`marketplace.json`**: 更新描述，补充 Gitea / Bitbucket / Azure DevOps / mirror-sync 等标签

### Evaluated (No Change)
- **GitHub/GitLab 运维模块**：评估后决定不实现
  - GitHub CLI (`gh`) / GitLab CLI (`glab`) 生态成熟，自建价值低
  - batch_clone 已覆盖高频需求（克隆/迁移/列表）
  - 平台 API 差异大，维护成本高
- **脚本拆分**：暂不拆分，`git_ops.py` (1358行) 虽大但结构清晰，命令分区合理
  - 拆分阈值设为 2000 行，未来按命令类别拆分至 `git_ops/commands/`
- **公共代码模块化**：`subprocess` 调用模式重复不足以支撑独立模块

---

## [2.3.0] - 2026-04-15

### Added
- **Gitea 迁移功能**：新增 `--migrate` 命令，支持将外部仓库迁移到 Gitea
  - 自动启用镜像同步（`mirror: true`）
  - 自动获取用户 uid，无需手动指定
  - 新增参数: `--migrate`、`--src`（源地址）、`--name`（目标仓库名）、`--owner`、`--private`、`--desc`
- **Gitea 组织创建**：新增 `--create-org` 命令（需管理员权限）
- **Gitea 镜像同步触发**：新增 `--sync` 命令，手动触发已有仓库的镜像同步
- **HTTP 辅助重构**：`http_request()` 替换原 `http_get()`，统一处理 GET/POST/PATCH，支持 4 种认证格式
- **Gitea API 探测**：新增 `gitea_get_current_user()`、`gitea_get_repo()`、`gitea_check_mirror_available()`
- **迁移健壮性**：自动检测仓库是否存在、镜像状态、pull mirror 是否被禁用

---

## [2.2.0] - 2026-04-15

### Added
- **Bitbucket 支持**：`--platform bitbucket`，支持按 workspace 批量克隆
- **Azure DevOps 支持**：`--platform azure`，支持按 Project 批量克隆

---

## [2.1.0] - 2026-04-14

### Added
- **`git_ops.py`**: 新增 `reflog`、`describe`、`worktree`、`grep`、`cherry-pick`、`revert`、`bisect` 等 7 个高级命令

---

## [2.0.0] - 2026-04-14

### Added
- **`batch_clone.py`**: `--workers` 并发、`--format json`、`--recursive` 子组、Token 环境变量支持
- **`batch_pull.py`**: `--dry-run` 预览模式
- **`git_lfs.py`**: 10 个子命令覆盖 LFS 全生命周期
- **`git_ops.py`**: 扩展至 18 个子命令
- **`README.md` / `README_en.md`**: 中英双语文档

---

## [1.0.0] - 2026-04-14

### Added
- **`batch_clone.py`**: 批量克隆 GitHub / GitLab / Gitea
- **`batch_pull.py`**: 批量拉取/更新本地仓库
- **`git_ops.py`**: 单仓库 Git 操作基础命令
