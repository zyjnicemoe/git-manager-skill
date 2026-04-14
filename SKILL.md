---
name: git-manager
description: >
  This skill handles Git operations: batch cloning GitHub/GitLab/Gitea/Bitbucket/Azure DevOps repos,
  migrating GitHub repos to Gitea with mirror sync, batch pulling local repos,
  and full-featured single-repo operations (merge, rebase, reflog, bisect, worktree, LFS, etc.).
  Also covers Gitea organization creation and mirror sync triggering.
  Use whenever the user asks about Git clone, sync, batch operations, or repo management.
author: zhuyijun
homepage: https://zyjblogs.cn
---

# Git Manager

> Git 仓库管理技能，支持五大平台批量操作和 Gitea 运维。

## 核心脚本

| 脚本 | 行数 | 功能 |
|------|------|------|
| `scripts/batch_clone.py` | ~900 | 五平台批量克隆 + **Gitea 运维**（迁移/组织创建/镜像同步触发） |
| `scripts/batch_pull.py` | ~265 | 批量更新本地仓库，支持并发/rebase/stash |
| `scripts/git_ops.py` | ~1360 | 单仓库完整操作，27 个子命令 |
| `scripts/git_lfs.py` | ~273 | Git LFS 跟踪管理、迁移、扫描 |

**执行方式：**
```bash
python scripts/batch_clone.py [args]
python scripts/batch_pull.py  [args]
python scripts/git_ops.py     [args]
python scripts/git_lfs.py      [args]
```

**依赖：** Python 3.8+，纯标准库，零额外依赖。

---

## 高频场景速查

### 批量克隆（batch_clone.py）

```bash
# GitHub org
python scripts/batch_clone.py --platform github --type org --id my-org --output ./repos

# GitLab group（含子组）
python scripts/batch_clone.py --platform gitlab --host https://gitlab.com \
  --type group --id 123456 --token $GITLAB_TOKEN --output ./repos

# GitHub → Gitea 迁移（启用镜像同步）
python scripts/batch_clone.py --platform gitea --host https://gitea.com \
  --migrate --src https://github.com/user/repo --name myrepo \
  --token $GITEA_TOKEN

# 创建 Gitea 组织（需管理员 token）
python scripts/batch_clone.py --platform gitea --host https://gitea.com \
  --create-org myorg --desc "Team" --token $GITEA_ADMIN_TOKEN

# 触发 Gitea 镜像同步
python scripts/batch_clone.py --platform gitea --host https://gitea.com \
  --sync --owner myuser --repo myrepo --token $GITEA_TOKEN

# 并发 + LFS + 过滤
python scripts/batch_clone.py --platform github --type org --id my-org \
  --output ./repos --workers 4 --lfs --filter api --limit 10
```

**参数说明：**
- `--platform`: `github` / `gitlab` / `gitea` / `bitbucket` / `azure`
- `--type`: `org` / `group` / `user` / `project`（按平台选填）
- `--token`: 支持 `GITHUB_TOKEN` / `GITLAB_TOKEN` / `GITEA_TOKEN` 环境变量
- `--workers N`: 并发克隆（默认 1，4-8 显著加速）
- `--format json`: JSON 输出便于程序化处理
- `--dry-run`: 仅预览不克隆
- `--archived`: 包含已归档仓库
- `--recursive / --no-recursive`: GitLab 子组控制（默认开启）

### 批量更新（batch_pull.py）

```bash
python scripts/batch_pull.py ./repos --rebase --stash --workers 4
python scripts/batch_pull.py ./repos --dry-run        # 预览
```

### 单仓库操作（git_ops.py）

```bash
# 克隆
python scripts/git_ops.py clone <URL> [dest] [-b branch] [--depth N] [--lfs]

# 暂存 + 提交
python scripts/git_ops.py add    <path> -A && \
python scripts/git_ops.py commit <path> -m "msg"

# 合并 / 衍合
python scripts/git_ops.py merge  <path> <branch> --no-ff
python scripts/git_ops.py rebase <path> --branch main

# 查看
python scripts/git_ops.py diff  <path> --staged
python scripts/git_ops.py log   <path> --oneline --graph -n 20

# 进阶
python scripts/git_ops.py stash      <path> --save "wip"
python scripts/git_ops.py reflog      <path> -n 30      # 找回丢失提交
python scripts/git_ops.py worktree    <path> --add ../feat -b feature
python scripts/git_ops.py bisect      <path> --start HEAD v1.0.0
python scripts/git_ops.py cherry-pick <path> abc1234
python scripts/git_ops.py grep        <path> "TODO" -n
```

### Git LFS（git_lfs.py）

```bash
python scripts/git_lfs.py ./repo --install
python scripts/git_lfs.py ./repo --track "*.psd" "*.zip" "*.pdf"
python scripts/git_lfs.py ./repo migrate --pattern "*.zip" --to lfs
```

---

## 平台 ID 类型

| 平台 | `--type` | `--id` 示例 |
|------|---------|-------------|
| GitHub | `org` / `user` / `project` | `my-org` / `johndoe` / `owner/repo` |
| GitLab | `group` / `user` / `project` | `123456` / `johndoe` / `123` |
| Gitea | `org` / `user` | `myorg` / `johndoe` |
| Bitbucket | `workspace` | `my-workspace` |
| Azure | `project` | 项目名（需 `--org` 指定组织） |

---

## 用户意图 → 脚本映射

| 用户说 | 脚本 | 关键参数 |
|--------|------|---------|
| 克隆 GitHub org 所有仓库 | `batch_clone.py` | `--platform github --type org --id <org>` |
| 克隆 GitLab group | `batch_clone.py` | `--platform gitlab --type group --id <id>` |
| GitHub → Gitea 迁移+镜像同步 | `batch_clone.py` | `--migrate --src <url> --name <repo>` |
| 在 Gitea 创建组织 | `batch_clone.py` | `--create-org <name>` |
| 触发 Gitea 镜像同步 | `batch_clone.py` | `--sync --owner <u> --repo <r>` |
| 更新本地所有仓库 | `batch_pull.py` | `./repos` |
| 合并 / rebase 分支 | `git_ops.py` | `merge / rebase <path> <branch>` |
| 找回误删的提交 | `git_ops.py` | `reflog <path>` |
| 多分支同时工作 | `git_ops.py` | `worktree <path> --add <dir> -b <branch>` |
| 二分定位 bug | `git_ops.py` | `bisect <path> --start HEAD v1.0.0` |
| 搜索仓库文本 | `git_ops.py` | `grep <path> "pattern"` |
| 管理 LFS 跟踪 | `git_lfs.py` | `--track "*.psd"` |

---

## 注意事项

1. **Token 安全**：使用环境变量而非命令行参数暴露 token
2. **Rate Limit**：GitHub 未认证仅 60次/小时，批量操作务必加 `--token`
3. **rebase 风险**：已推送的公共分支禁止 rebase
4. **冲突处理**：merge/rebase 遇冲突后 `git add` + `git merge/rebase --continue`
5. **GitLab group_id**：Group → Settings → General 查看数字 ID
6. **Gitea pull mirror**：需服务器管理员在 `app.ini` 设置 `PULL_REQUEST_PUSH_MIRRORS=true`
7. **LFS 依赖**：目标机器需先安装 `git-lfs`

---

## 参考资料

完整示例见 `examples.md`，包含 13 个覆盖所有平台和 Gitea 运维的完整场景。
