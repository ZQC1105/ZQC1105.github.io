## http://localhost:3000/api/swagger#/issue/issueCreateComment // 创建评论

要获取改动文件中的数据，您需要结合 Gitea API 进行多次调用。以下是完整的解决方案：

## 方法1：获取完整文件内容（推荐）

### 步骤1：获取 PR 改动的文件列表

```bash
# 调用 API 获取改动的文件列表
curl -X GET "http://localhost:3000/api/v1/repos/Chen/Test/pulls/5/files" \
  -H "Authorization: token YOUR_ACCESS_TOKEN"
```

返回示例：

```json
[
  {
    "filename": "src/main.py",
    "status": "modified", // 或 "added", "removed", "renamed"
    "additions": 10,
    "deletions": 5,
    "changes": 15,
    "contents_url": "http://localhost:3000/api/v1/repos/Chen/Test/contents/src/main.py?ref=3a3d672e8e2935ca02946c70e2eddf1ac5d6d31b"
  }
]
```

### 步骤2：获取文件的具体内容

```bash
# 使用 contents_url 获取文件内容
curl -X GET "http://localhost:3000/api/v1/repos/Chen/Test/contents/src/main.py?ref=3a3d672e8e2935ca02946c70e2eddf1ac5d6d31b" \
  -H "Authorization: token YOUR_ACCESS_TOKEN"
```

返回示例：

```json
{
  "name": "main.py",
  "path": "src/main.py",
  "content": "IyEvdXNyL2Jpbi9lbnYgcHl0aG9uMwoKcHJpbnQoIkhlbGxvIFdvcmxkIikK...",  # Base64 编码的内容
  "encoding": "base64",
  "type": "file",
  "sha": "3a3d672e8e2935ca02946c70e2eddf1ac5d6d31b"
}
```

### 步骤3：解码 Base64 内容

```python
import base64
import requests

# 获取文件内容
response = requests.get(
    "http://localhost:3000/api/v1/repos/Chen/Test/contents/src/main.py",
    params={"ref": "3a3d672e8e2935ca02946c70e2eddf1ac5d6d31b"},
    headers={"Authorization": "token YOUR_ACCESS_TOKEN"}
)

file_data = response.json()
content = base64.b64decode(file_data["content"]).decode("utf-8")
print(content)
```

## 方法2：获取 diff/patch 格式的改动内容

如果您只需要查看改动的部分，而不是完整的文件内容：

### 使用 diff_url

```bash
# 获取 diff 格式的改动
curl -X GET "http://localhost:3000/Chen/Test/pulls/5.diff" \
  -H "Authorization: token YOUR_ACCESS_TOKEN"
```

返回示例：

```diff
diff --git a/src/main.py b/src/main.py
index b34496d..3a3d672 100644
--- a/src/main.py
+++ b/src/main.py
@@ -1,3 +1,5 @@
 #!/usr/bin/env python3

-print("Hello World")
+print("Hello World")
+print("This is a new line")
+print("Another new line")
```

### 使用 patch_url

```bash
# 获取 patch 格式的改动
curl -X GET "http://localhost:3000/Chen/Test/pulls/5.patch" \
  -H "Authorization: token YOUR_ACCESS_TOKEN"
```

## 方法3：完整的 Python 脚本示例

以下是一个完整的脚本，接收 webhook 后获取改动的文件内容：

```python
import base64
import requests
import json

def get_pr_file_contents(gitea_url, repo_owner, repo_name, pr_number, token):
    """获取 PR 中所有改动文件的内容"""

    headers = {"Authorization": f"token {token}"}

    # 1. 获取 PR 改动的文件列表
    files_url = f"{gitea_url}/api/v1/repos/{repo_owner}/{repo_name}/pulls/{pr_number}/files"
    files_response = requests.get(files_url, headers=headers)
    files = files_response.json()

    file_contents = {}

    for file_info in files:
        filename = file_info["filename"]
        status = file_info["status"]

        print(f"处理文件: {filename} (状态: {status})")

        # 2. 获取文件内容
        # 使用 PR 合并后的 commit SHA
        contents_url = f"{gitea_url}/api/v1/repos/{repo_owner}/{repo_name}/contents/{filename}"
        params = {"ref": "3a3d672e8e2935ca02946c70e2eddf1ac5d6d31b"}  # 使用 PR 的 merge commit

        contents_response = requests.get(contents_url, headers=headers, params=params)

        if contents_response.status_code == 200:
            file_data = contents_response.json()
            if file_data.get("encoding") == "base64":
                content = base64.b64decode(file_data["content"]).decode("utf-8")
                file_contents[filename] = {
                    "status": status,
                    "content": content
                }
        else:
            print(f"无法获取 {filename} 的内容: {contents_response.status_code}")

    return file_contents

def get_pr_diff(gitea_url, repo_owner, repo_name, pr_number, token):
    """获取 PR 的 diff 内容"""

    diff_url = f"{gitea_url}/{repo_owner}/{repo_name}/pulls/{pr_number}.diff"
    response = requests.get(diff_url, headers={"Authorization": f"token {token}"})

    return response.text

# 使用示例
if __name__ == "__main__":
    # 配置信息
    GITEA_URL = "http://localhost:3000"
    TOKEN = "your_access_token_here"

    # 从 webhook 数据中提取的信息
    repo_owner = "Chen"
    repo_name = "Test"
    pr_number = 5

    # 获取所有改动文件的内容
    files_content = get_pr_file_contents(GITEA_URL, repo_owner, repo_name, pr_number, TOKEN)

    # 打印文件内容
    for filename, data in files_content.items():
        print(f"\n=== {filename} (状态: {data['status']}) ===")
        print(data['content'])

    # 或者获取 diff
    diff = get_pr_diff(GITEA_URL, repo_owner, repo_name, pr_number, TOKEN)
    print(f"\n=== Diff 内容 ===\n{diff}")
```

## 方法4：在 webhook 处理脚本中集成

```python
from flask import Flask, request, jsonify
import requests
import base64
import os

app = Flask(__name__)

GITEA_URL = "http://localhost:3000"
GITEA_TOKEN = os.environ.get("GITEA_TOKEN", "your_token_here")

@app.route('/webhook', methods=['POST'])
def handle_webhook():
    data = request.json

    # 只处理 PR 相关事件
    if "pull_request" in data:
        action = data.get("action")
        pr = data["pull_request"]
        repo = data["repository"]

        print(f"收到 PR #{pr['number']} {action} 事件")

        # 获取改动的文件列表
        headers = {"Authorization": f"token {GITEA_TOKEN}"}
        files_url = f"{GITEA_URL}/api/v1/repos/{repo['full_name']}/pulls/{pr['number']}/files"

        files_response = requests.get(files_url, headers=headers)

        if files_response.status_code == 200:
            files = files_response.json()

            for file_info in files:
                filename = file_info["filename"]
                print(f"文件 {filename} 被修改了")

                # 根据文件类型处理
                if filename.endswith('.json'):
                    # 处理 JSON 文件
                    process_json_file(repo['full_name'], filename, pr['merge_commit_sha'])
                elif filename.endswith('.py'):
                    # 处理 Python 文件
                    process_python_file(repo['full_name'], filename, pr['merge_commit_sha'])
                # ... 其他文件类型

        return jsonify({"status": "success"}), 200

    return jsonify({"status": "ignored"}), 200

def get_file_content(repo_full_name, filename, ref):
    """获取文件内容"""
    url = f"{GITEA_URL}/api/v1/repos/{repo_full_name}/contents/{filename}"
    response = requests.get(url, params={"ref": ref},
                           headers={"Authorization": f"token {GITEA_TOKEN}"})

    if response.status_code == 200:
        file_data = response.json()
        if file_data.get("encoding") == "base64":
            return base64.b64decode(file_data["content"]).decode("utf-8")
    return None

def process_json_file(repo, filename, ref):
    """处理 JSON 文件"""
    content = get_file_content(repo, filename, ref)
    if content:
        try:
            json_data = json.loads(content)
            print(f"JSON 数据: {json_data}")
            # 在这里处理您的 JSON 数据
        except json.JSONDecodeError:
            print(f"文件 {filename} 不是有效的 JSON")

def process_python_file(repo, filename, ref):
    """处理 Python 文件"""
    content = get_file_content(repo, filename, ref)
    if content:
        print(f"Python 文件内容长度: {len(content)} 字符")
        # 在这里处理您的 Python 代码

if __name__ == '__main__':
    app.run(port=5000)
```

## 注意事项

1. **Token 权限**：需要使用有权限访问仓库的 Gitea token
2. **Ref 参数**：在获取文件内容时，需要指定正确的 `ref`（分支、commit SHA 或 tag）
3. **文件状态**：`status` 字段可以是 `added`、`modified`、`removed` 或 `renamed`
4. **编码处理**：API 返回的文件内容是 Base64 编码的，需要解码

## 获取 Gitea Token

1. 登录 Gitea
2. 点击右上角头像 -> 设置 -> 应用
3. 生成新的 Token，选择必要的权限（至少需要 `repo` 权限）

这样您就可以获取改动文件中的具体数据并进行处理了。
