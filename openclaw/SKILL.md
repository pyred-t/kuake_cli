---
name: kuake
description: 夸克网盘文件管理 CLI 工具集成
metadata: {"openclaw":{"emoji":"📦","requires":{"env":["KUAKE_COOKIE"]},"os":["win32","linux","darwin"],"homepage":"https://github.com/zhangjingwei/kuake_sdk","primaryEnv":"KUAKE_COOKIE"}}
---

# 夸克网盘文件管理技能

本技能提供夸克网盘（Quark Cloud Drive）的完整文件管理能力，包括文件上传、下载、列表、分享等功能。

## 前置要求

### 1. 安装 kuake CLI 工具

- 从 [GitHub Releases](https://github.com/zhangjingwei/kuake_sdk/releases) 下载对应平台的二进制文件
- 或从源码编译：`./build.sh`

**推荐安装位置：**
- Windows: `%USERPROFILE%\bin\kuake.exe`
- Linux/macOS: `~/bin/kuake`

**配置方式（三选一）：**

**方式 1：添加到 PATH（推荐）**
- 将 kuake 所在目录添加到系统 PATH
- 可直接使用 `kuake` 命令

**方式 2：通过环境变量指定路径**
在 `~/.openclaw/openclaw.json` 中配置：

```json
{
  "skills": {
    "entries": {
      "kuake": {
        "enabled": true,
        "apiKey": "your_cookie_value_here",
        "env": {
          "KUAKE_PATH": "C:\\Users\\<用户名>\\bin\\kuake.exe"
        }
      }
    }
  }
}
```

**方式 3：使用完整路径**
在命令中使用完整路径（不推荐，但可用）

### 2. 配置认证信息

kuake 支持环境变量 KUAKE_COOKIE（推荐，OpenClaw 标准配置）**

在 `~/.openclaw/openclaw.json` 中配置：

```json
{
  "skills": {
    "entries": {
      "kuake": {
        "enabled": true,
        "apiKey": "your_cookie_value_here"
      }
    }
  }
}
```

## 前置检查（每次使用前建议执行）

```bash
# 检查 kuake 命令是否可用
# Windows
where.exe kuake 2>nul || echo "kuake 命令未找到，请确保已安装并添加到 PATH"

# Linux/macOS
which kuake 2>/dev/null || echo "kuake 命令未找到，请确保已安装并添加到 PATH"

# 验证版本
kuake version || echo "kuake 命令执行失败，请检查安装和认证配置"

# 检查环境变量（可选）
# Windows PowerShell
if (-not $env:KUAKE_COOKIE) { Write-Host "提示：KUAKE_COOKIE 环境变量未设置，可使用 -cookies 参数或 config.json" }

# Linux/macOS
if [ -z "$KUAKE_COOKIE" ]; then echo "提示：KUAKE_COOKIE 环境变量未设置，可使用 -cookies 参数或 config.json"; fi
```

## 命令调用方式

**Fallback 逻辑（按优先级）：**

执行命令时，按以下顺序尝试：

1. **环境变量 KUAKE_PATH**（如果设置了完整路径）
   ```bash
   # Windows PowerShell
   if ($env:KUAKE_PATH) { & $env:KUAKE_PATH user } else { kuake user }
   
   # Linux/macOS
   ${KUAKE_PATH:-kuake} user
   ```

2. **PATH 中的 kuake 命令**（如果已添加到 PATH）
   ```bash
   kuake user
   ```

3. **完整路径 fallback**（Windows 默认位置）
   ```bash
   # Windows
   C:\Users\<用户名>\bin\kuake.exe user
   
   # Linux/macOS
   ~/bin/kuake user
   ```

**在 OpenClaw 中使用时，优先检查 `KUAKE_PATH` 环境变量，如果不存在则使用 `kuake` 命令，最后 fallback 到完整路径。**

## 可用命令

### 1. 获取用户信息

当用户询问夸克网盘账号信息、容量、会员状态时，使用：

```bash
# 优先使用 KUAKE_PATH 环境变量，否则使用 kuake 命令
# Windows PowerShell
if ($env:KUAKE_PATH) { & $env:KUAKE_PATH user } else { kuake user }

# Linux/macOS
${KUAKE_PATH:-kuake} user

# 如果已在 PATH 中，可直接使用
kuake user
```

**输出示例：**
```json
{
  "success": true,
  "code": "OK",
  "message": "操作成功",
  "data": {
    "nickname": "用户名",
    "capacity": XXX,
    "used": XXXX
  }
}
```

### 2. 列出目录内容

当用户需要查看网盘中的文件列表时，使用：

```bash
# 列出根目录
kuake list "/"

# 列出指定目录
kuake list "/documents"

# 流式输出（用于管道模式）
kuake list "/" --stream
```

**参数说明：**
- `path`: 目录路径，根目录使用 `"/"`
- `--stream`: 可选，输出流式 JSON（每行一个文件对象）

**输出示例：**
```json
{
  "success": true,
  "code": "OK",
  "data": {
    "list": [
      {
        "fid": "file_id",
        "file_name": "example.txt",
        "path": "/example.txt",
        "size": 1024,
        "dir": false,
        "ctime": 1234567890,
        "mtime": 1234567890
      }
    ]
  }
}
```

### 3. 获取文件信息

当用户需要查看文件详细信息时，使用：

```bash
kuake info "/file.txt"
```

**参数说明：**
- `path`: 文件或文件夹路径（必须用引号包裹）

### 4. 下载文件

当用户需要下载文件时，使用：

```bash
# 仅获取下载链接
kuake download "/file.txt"

# 下载到当前目录
kuake download "/file.txt" .

# 下载到指定路径
kuake download "/file.txt" ./downloads/file.txt
```

**参数说明：**
- `path`: 文件路径（必须用引号包裹）
- `dest`: 可选，目标路径。如果提供则下载到本地，否则仅返回下载链接

### 5. 上传文件

当用户需要上传文件到网盘时，使用：

```bash
# 基本上传
kuake upload "local_file.txt" "/remote_file.txt"

# 指定并行度上传
kuake upload "local_file.txt" "/remote_file.txt" --max_upload_parallel 8

# 使用上传策略
kuake upload "local_file.txt" "/remote_file.txt" --policy skip
```

**参数说明：**
- `file`: 本地文件路径（必须用引号包裹）
- `dest`: 目标路径（必须用引号包裹）
- `--max_upload_parallel N`: 可选，并行上传分片数（1-16，默认 4）
- `--policy`: 可选，上传策略（skip=跳过已存在，overwrite=覆盖，rsync=同步）

**注意：**
- 上传进度输出到 stderr，不影响 JSON 解析
- 所有路径参数必须用引号包裹

### 6. 创建文件夹

当用户需要创建新文件夹时，使用：

```bash
kuake create "folder_name" "/"
```

**参数说明：**
- `name`: 文件夹名称（必须用引号包裹）
- `pdir`: 父目录路径，根目录使用 `"/"`（必须用引号包裹）

### 7. 移动文件/文件夹

当用户需要移动文件或文件夹时，使用：

```bash
kuake move "/source/file.txt" "/destination/"
```

**参数说明：**
- `src`: 源路径（必须用引号包裹）
- `dest`: 目标路径（必须用引号包裹）

### 8. 复制文件/文件夹

当用户需要复制文件或文件夹时，使用：

```bash
kuake copy "/source/file.txt" "/destination/"
```

**参数说明：**
- `src`: 源路径（必须用引号包裹）
- `dest`: 目标路径（必须用引号包裹）

### 9. 重命名文件/文件夹

当用户需要重命名文件或文件夹时，使用：

```bash
kuake rename "/old_name.txt" "new_name.txt"
```

**参数说明：**
- `path`: 文件或文件夹路径（必须用引号包裹）
- `newName`: 新名称（必须用引号包裹）

### 10. 删除文件/文件夹

当用户需要删除文件或文件夹时，使用：

```bash
kuake delete "/file.txt"
```

**参数说明：**
- `path`: 文件或文件夹路径（必须用引号包裹）

**注意：** 删除目录会递归删除所有子文件和子目录，请谨慎操作。

### 11. 创建分享链接

当用户需要创建文件分享链接时，使用：

```bash
# 创建7天有效期的分享（不需要提取码）
kuake share "/file.txt" 7 "false"

# 创建30天有效期的分享（需要提取码）
kuake share "/file.txt" 30 "true"
```

**参数说明：**
- `path`: 文件或文件夹路径（必须用引号包裹）
- `days`: 有效期天数，`0`=永久，`1`=1天，`7`=7天，`30`=30天
- `passcode`: `"true"`=需要提取码，`"false"`=不需要提取码

**输出示例：**
```json
{
  "success": true,
  "code": "OK",
  "data": {
    "share_url": "https://pan.quark.cn/s/xxx",
    "pwd_id": "share_id",
    "passcode": "1234",
    "expires_at": 1234567890000
  }
}
```

### 12. 取消分享

当用户需要取消文件分享时，使用：

```bash
# 通过 share_id 取消
kuake share-delete "fdd8bfd93f21491ab80122538bec310d"

# 通过文件路径取消
kuake share-delete "/file.txt"

# 同时取消多个分享
kuake share-delete "share_id1" "share_id2" "/file.txt"
```

**参数说明：**
- `share_id_or_path`: share_id 或文件路径（必须用引号包裹），支持多个

### 13. 获取分享列表

当用户需要查看自己的分享列表时，使用：

```bash
# 使用默认参数
kuake share-list

# 指定分页和排序
kuake share-list 1 50 "created_at" "desc"
```

**参数说明：**
- `page`: 页码（默认 1）
- `size`: 每页数量（默认 50）
- `orderField`: 排序字段（默认 "created_at"）
- `orderType`: 排序方式，"asc" 或 "desc"（默认 "desc"）

### 14. 转存分享文件

当用户需要转存他人分享的文件到自己的网盘时，使用：

```bash
# 转存到根目录
kuake share-save "https://pan.quark.cn/s/xxx"

# 指定提取码和目标目录
kuake share-save "https://pan.quark.cn/s/xxx" "1234" "/folder"
```

**参数说明：**
- `share_link`: 分享链接（如 `https://pan.quark.cn/s/xxx`），会自动提取 pwd_id
- `passcode`: 提取码（可选），如果分享链接中包含提取码会自动提取
- `dest_dir`: 目标目录（可选，默认 `"/"`），可以是路径或 FID

## 使用注意事项

1. **路径参数必须用引号包裹**
   - 所有路径参数必须用引号包裹，例如：`"/file.txt"`、`"/folder/"`

2. **配置文件位置**
   - 默认配置文件：`config.json`（当前目录）
   - 可通过 `-c` 或 `--config` 参数指定配置文件路径
   - 可通过 `-cookies` 参数直接传递 Cookie（绕过配置文件）

3. **输出格式**
   - 所有命令结果以 JSON 格式输出到 stdout
   - 上传进度、帮助信息输出到 stderr
   - 退出码：`0`=成功，`1`=失败

4. **管道模式**
   - `list` 命令支持 `--stream` 选项，输出流式 JSON
   - `delete`、`info`、`download` 命令支持从 stdin 读取 JSON 输入
   - 自动检测 stdin，有数据时自动进入管道模式

5. **错误处理**
   - 所有错误信息包含在 JSON 响应的 `message` 字段中
   - 可通过 `success` 字段判断操作是否成功
   - 错误码在 `code` 字段中

## 使用示例

### 示例 1：查看用户信息和根目录文件

```bash
# 获取用户信息
kuake user

# 列出根目录
kuake list "/"
```

### 示例 2：上传文件并创建分享

```bash
# 上传文件
kuake upload "local_file.txt" "/remote_file.txt"

# 创建分享链接
kuake share "/remote_file.txt" 7 "false"
```

### 示例 3：批量操作（管道模式）

```bash
# 列出大文件并删除
kuake list "/" --stream | jq -r 'select(.size > 1000000) | .path' | kuake delete

# 列出文件并获取信息
kuake list "/documents" --stream | kuake info
```

### 示例 4：使用 Cookie 参数

```bash
# 直接使用 Cookie（无需配置文件）
kuake -cookies "your_cookie_value" user
kuake -cookies "your_cookie_value" list "/"
```

## 技能使用指南

当用户在 OpenClaw 中请求以下操作时，使用本技能：

- **查看网盘信息**：使用 `kuake user` 获取用户信息和容量
- **浏览文件**：使用 `kuake list` 列出目录内容
- **上传文件**：使用 `kuake upload` 上传本地文件
- **下载文件**：使用 `kuake download` 下载文件或获取链接
- **管理文件**：使用 `move`、`copy`、`rename`、`delete` 等命令
- **分享文件**：使用 `share`、`share-delete`、`share-list` 管理分享
- **转存分享**：使用 `share-save` 转存他人分享的文件

**重要提示：**
- 执行命令前，确保 kuake CLI 已正确安装并在 PATH 中
- 确保已配置有效的 Cookie 或配置文件
- 所有路径参数必须用引号包裹
- 敏感操作（如删除）请确认后再执行

## 常见问题

| 现象 | 可能原因 | 解决方案 |
|------|----------|----------|
| `kuake: command not found` | PATH 未包含 kuake 可执行文件目录 | Windows: `$env:Path += ";$env:USERPROFILE\bin"`<br>Linux/macOS: `export PATH="$HOME/bin:$PATH"`<br>或重新打开终端 |
| `Failed to initialize client` | Cookie 配置错误或未配置 | 检查 `KUAKE_COOKIE` 环境变量、`config.json` 文件或使用 `-cookies` 参数 |
| 技能状态显示 `✗ missing` | OpenClaw 启动时 PATH 不包含 kuake，或执行环境未继承 PATH | 1. 在 `openclaw.json` 的 `skills.entries.kuake.env` 中设置 `PATH`<br>2. 重启 OpenClaw 会话<br>3. 重新加载技能 |
| `at least one access token is required` | 未配置任何认证信息 | 配置环境变量、配置文件或使用 `-cookies` 参数 |

**故障排查步骤：**

1. **验证 kuake 命令可用性**
   ```bash
   # Windows
   where.exe kuake
   kuake version
   
   # Linux/macOS
   which kuake
   kuake version
   ```

2. **检查认证配置**
   ```bash
   # 检查环境变量
   echo $env:KUAKE_COOKIE  # Windows PowerShell
   echo $KUAKE_COOKIE      # Linux/macOS
   
   # 检查配置文件
   Test-Path config.json   # Windows
   ls config.json          # Linux/macOS
   ```

3. **验证技能状态**
   ```bash
   openclaw skills info kuake
   ```

4. **如果问题仍然存在**
   - 检查 OpenClaw 日志获取详细错误信息
   - 确保已重新编译 kuake（如果从源码安装）
   - 确保 OpenClaw 版本支持环境变量注入
