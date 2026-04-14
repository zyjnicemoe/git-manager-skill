# git-manager Skill - 完整示例

## 场景一：克隆团队 GitHub Organization 下所有仓库

```bash
python scripts/batch_clone.py \
  --platform github \
  --type org \
  --id my-team-org \
  --output ./team-repos \
  --token $GITHUB_TOKEN \
  --workers 4 \
  --lfs
```

## 场景二：克隆 GitLab Group 及其子组所有项目

```bash
export GITLAB_TOKEN=glpat-xxxxxxxxxxxx
python scripts/batch_clone.py \
  --platform gitlab \
  --host https://gitlab.com \
  --type group \
  --id my-group \
  --output ./gitlab-projects \
  --recursive \
  --workers 8
```

## 场景三：克隆 Bitbucket Workspace 下所有仓库

```bash
python scripts/batch_clone.py \
  --platform bitbucket \
  --type workspace \
  --id my-workspace \
  --output ./bitbucket-repos \
  --token $BB_TOKEN \
  --workers 4
```

## 场景四：Azure DevOps 项目批量克隆

```bash
python scripts/batch_clone.py \
  --platform azure \
  --org my-company \
  --project "Dev Team" \
  --type project \
  --id "Dev Team" \
  --output ./azure-repos \
  --token $AZURE_PAT \
  --workers 4
```

## 场景五：从 GitHub 迁移仓库到 Gitea（启用镜像同步）

```bash
# 迁移单个仓库，自动启用镜像
python scripts/batch_clone.py \
  --platform gitea \
  --host https://gitea.com \
  --migrate \
  --src https://github.com/myuser/myproject \
  --name myproject \
  --token $GITEA_TOKEN

# 迁移到指定组织
python scripts/batch_clone.py \
  --platform gitea \
  --host https://gitea.com \
  --migrate \
  --src https://github.com/myuser/myproject \
  --name myproject \
  --owner myorg \
  --private \
  --token $GITEA_TOKEN
```

## 场景六：创建 Gitea 组织

```bash
# 需要管理员权限的 token
python scripts/batch_clone.py \
  --platform gitea \
  --host https://gitea.com \
  --create-org skills \
  --desc "AI Skills Collection" \
  --token $GITEA_ADMIN_TOKEN
```

## 场景七：触发 Gitea 仓库镜像同步

```bash
python scripts/batch_clone.py \
  --platform gitea \
  --host https://gitea.com \
  --sync \
  --owner myuser \
  --repo myproject \
  --token $GITEA_TOKEN
```

## 场景八：批量更新本地所有仓库

```bash
# 标准 pull（快进）
python scripts/batch_pull.py ./all-repos

# rebase 方式，有修改时自动 stash
python scripts/batch_pull.py ./all-repos --rebase --stash --workers 4

# 只查看要更新的仓库（不执行）
python scripts/batch_pull.py ./all-repos --dry-run
```

## 场景九：找回误删的提交

```bash
# 查看 reflog 找到丢失的提交
python scripts/git_ops.py reflog ./my-repo -n 30

# 找到目标 commit 后，创建分支恢复
python scripts/git_ops.py checkout ./my-repo -b recovered-branch <commit-hash>
```

## 场景十：多分支同时工作

```bash
# 查看现有工作树
python scripts/git_ops.py worktree ./my-repo --list

# 为新功能添加工作树
python scripts/git_ops.py worktree ./my-repo ../feature-auth -b auth-feature

# 开发完成后删除工作树
python scripts/git_ops.py worktree ./my-repo --remove ../feature-auth
```

## 场景十一：二分定位 bug

```bash
# 启动 bisect，指定已知正常和异常的提交
python scripts/git_ops.py bisect ./my-repo --start HEAD v1.0.0

# 手动标记
python scripts/git_ops.py bisect ./my-repo --good   # 当前正常
python scripts/git_ops.py bisect ./my-repo --bad    # 当前有问题

# 或自动运行测试
python scripts/git_ops.py bisect ./my-repo --start HEAD v1.0.0 --run "make test"

# 重置
python scripts/git_ops.py bisect ./my-repo --reset
```

## 场景十二：批量管理 Git LFS

```bash
# 为目录下所有仓库初始化 LFS
Get-ChildItem -Recurse -Directory | ForEach-Object {
    python scripts/git_lfs.py $_.FullName --install 2>$null
}

# 迁移历史大文件到 LFS
python scripts/git_lfs.py ./my-repo migrate --pattern "*.zip" --to lfs
python scripts/git_lfs.py ./my-repo migrate --pattern "*.psd" --to lfs
```

## 场景十三：Gitea 私有实例完整迁移流程

```bash
# 1. 探测实例版本
curl https://gitea.example.com/api/v1/version

# 2. 获取当前用户 uid
curl -H "Authorization: token $GITEA_TOKEN" \
     https://gitea.example.com/api/v1/user

# 3. 创建组织（需 admin token）
python scripts/batch_clone.py \
  --platform gitea \
  --host https://gitea.example.com \
  --create-org myteam \
  --desc "My Team" \
  --token $GITEA_ADMIN_TOKEN

# 4. 迁移仓库到组织
python scripts/batch_clone.py \
  --platform gitea \
  --host https://gitea.example.com \
  --migrate \
  --src https://github.com/myuser/repo1 \
  --name repo1 \
  --owner myteam \
  --token $GITEA_TOKEN

# 5. 触发同步
python scripts/batch_clone.py \
  --platform gitea \
  --host https://gitea.example.com \
  --sync \
  --owner myteam \
  --repo repo1 \
  --token $GITEA_TOKEN
```
