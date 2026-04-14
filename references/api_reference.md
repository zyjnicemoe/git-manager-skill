# API 参考

## 认证方式

| 平台 | Header 格式 | Token 类型 |
|------|------------|-----------|
| GitHub | `Authorization: Bearer <token>` | Fine-grained PAT / Classic PAT |
| GitLab | `Authorization: Bearer <token>` + `PRIVATE-TOKEN: <token>` | Personal Access Token / Group Access Token |
| Gitea | `Authorization: token <token>` | Access Token |
| Bitbucket | `Authorization: Bearer <token>` | OAuth / App Password |
| Azure DevOps | `Authorization: Basic base64(":PAT")` | Personal Access Token |

---

## GitHub API

### 基础端点

| 操作 | 端点 |
|------|------|
| 用户仓库列表 | `GET /users/{username}/repos?per_page=100` |
| Organization 仓库 | `GET /orgs/{org}/repos?per_page=100` |
| 单仓库信息 | `GET /repos/{owner}/{repo}` |
| 版本 | `GET /api/v3/version` |

### Token 创建

1. GitHub → Settings → Developer settings → Personal access tokens → Generate new token
2. 勾选 `repo`（私有仓库需要）
3. 复制 Token（只显示一次）

### Rate Limit

| 类型 | 限制 |
|------|------|
| 未认证 | 60 次/小时 |
| 已认证 | 5000 次/小时 |

---

## GitLab API

### 基础端点

| 操作 | 端点 |
|------|------|
| Group 项目列表 | `GET /api/v4/groups/{id}/projects?include_subgroups=true` |
| 用户项目列表 | `GET /api/v4/users/{id}/projects` |
| 单项目信息 | `GET /api/v4/projects/{id}` |

### Token 创建

1. GitLab → User Settings → Access Tokens
2. 勾选 `api` / `read_api` scope
3. 复制 Token

---

## Gitea API

### 基础端点

| 操作 | 端点 |
|------|------|
| 用户信息 | `GET /api/v1/user` |
| 用户仓库列表 | `GET /api/v1/users/{username}/repos` |
| 组织仓库列表 | `GET /api/v1/orgs/{org}/repos` |
| 单仓库信息 | `GET /api/v1/repos/{owner}/{repo}` |
| 创建组织 | `POST /api/v1/admin/users/{uid}/orgs` |
| 迁移仓库 | `POST /api/v1/repos/migrate` |
| 触发镜像同步 | `POST /api/v1/repos/{owner}/{repo}/mirror_sync` |
| 实例版本 | `GET /api/v1/version` |

### 迁移请求体

```json
{
  "clone_addr": "https://github.com/user/repo.git",
  "uid": 1,
  "repo_name": "repo",
  "repo_owner": "myorg",
  "mirror": true,
  "private": false,
  "description": "Mirror from GitHub"
}
```

### 常见错误

| 错误信息 | 原因 | 解决方案 |
|---------|------|---------|
| `administrator has disabled the creation of new pull mirrors` | pull mirror 被禁用 | 联系 Gitea 管理员在 `app.ini` 开启 |
| `repo already exist` | 仓库已存在 | 跳过或删除后重试 |
| 403/422 | token 权限不足 | 检查 token 是否包含 repo 和 admin 权限 |

### Token 创建

1. Gitea → Settings → Applications → Create Token
2. 名称任意，勾选对应权限（repo、admin 等）
3. 复制 Token

---

## Bitbucket API

### 基础端点

| 操作 | 端点 |
|------|------|
| Workspace 仓库 | `GET /2.0/repositories/{workspace}?pagelen=100` |

### 认证

- **OAuth 2.0**：推荐，用于生产环境
- **App Passwords**：适用于脚本（Settings → App passwords）

### Token 创建

1. Bitbucket → Personal Settings → App passwords
2. 创建并设置仓库读取权限
3. 使用 `Authorization: Basic base64("username:app_password")`

---

## Azure DevOps API

### 基础端点

| 操作 | 端点 |
|------|------|
| 项目仓库列表 | `GET /{org}/{project}/_apis/git/repositories?api-version=6.0` |

### 认证

- **Basic Auth**：`base64(":{PAT}")`
- PAT 需要 `Code: Read` 权限

### Token 创建

1. Azure DevOps → User Settings → Personal Access Tokens
2. 创建 Token，范围选择 `Code: Read`
3. 使用 `Authorization: Basic base64(":{token}")`

---

## 分页处理

```python
# 通用分页模式
def paginate(base_url, token, page_param="page", per_page=100):
    results, page = [], 1
    while True:
        url = f"{base_url}?{page_param}={page}&per_page={per_page}"
        data = http_get(url, token)
        if not data:
            break
        items = data.get("data", data)  # Gitea 用 data 字段
        if not items:
            break
        results.extend(items)
        if len(items) < per_page:
            break
        page += 1
    return results
```

---

## 通用 HTTP 请求模板

```python
import urllib.request, json, base64

def http_request(method, url, token=None, auth_type="bearer", data=None):
    headers = {"Accept": "application/json", "User-Agent": "git-manager-skill/2.0"}

    if token:
        if auth_type == "basic":
            headers["Authorization"] = f"Basic {base64.b64encode(f':{token}'.encode()).decode()}"
        elif auth_type == "gitea_token":
            headers["Authorization"] = f"token {token}"
        elif auth_type == "gitlab":
            headers["Authorization"] = f"Bearer {token}"
            headers["PRIVATE-TOKEN"] = token
        else:
            headers["Authorization"] = f"Bearer {token}"

    body = json.dumps(data).encode() if data else None
    if body:
        headers["Content-Type"] = "application/json"

    req = urllib.request.Request(url, data=body, method=method, headers=headers)
    try:
        with urllib.request.urlopen(req, timeout=30) as resp:
            raw = resp.read().decode(errors="replace")
            return resp.status, json.loads(raw) if raw else {}
    except urllib.error.HTTPError as e:
        body = e.read().decode(errors="replace")
        return e.code, json.loads(body) if body.startswith("{") else body
    except Exception as e:
        return -1, {"error": str(e)}
```
