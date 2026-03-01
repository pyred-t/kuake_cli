# OpenClaw Skill 部署指南

## 前置要求

### 1. 设置 kuake 命令

```powershell
# 创建工具目录并复制文件
$BinDir = "$env:USERPROFILE\bin"
New-Item -ItemType Directory -Force -Path $BinDir
Copy-Item "dist\kuake-v1.3.9-windows-amd64.exe" "$BinDir\kuake.exe" -Force

# 添加到用户 PATH
$currentPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($currentPath -notlike "*$BinDir*") {
    $newPath = $currentPath
    if ($newPath -and -not $newPath.EndsWith(';')) { $newPath += ';' }
    $newPath += $BinDir
    [Environment]::SetEnvironmentVariable("Path", $newPath, "User")
}

# 验证（需要重新打开 PowerShell 窗口）
kuake version
```

### 2. PowerShell 执行策略

如果运行 `openclaw` 时出现执行策略错误：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 3. 配置认证信息

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

**如何获取 Cookie：** 登录夸克网盘，按 F12 打开开发者工具，在 Network 标签页中复制任意请求的 Cookie 值。

**如果 OpenClaw 找不到 kuake 命令，有两种解决方案：**

**方案 1：通过 KUAKE_PATH 环境变量指定完整路径（推荐）**

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

**方案 2：在 PATH 中添加 kuake 目录**

```json
{
  "skills": {
    "entries": {
      "kuake": {
        "enabled": true,
        "apiKey": "your_cookie_value_here",
        "env": {
          "PATH": "C:\\Users\\<用户名>\\bin;C:\\Windows\\system32;C:\\Windows;C:\\Program Files\\PowerShell\\7"
        }
      }
    }
  }
}
```

**注意：** 将 `<用户名>` 替换为实际用户名（如 `zhang`）。推荐使用方案 1，更简洁且不依赖 PATH。

## 部署步骤

```powershell
mkdir -Force $env:USERPROFILE\.openclaw\skills\kuake
Copy-Item openclaw\SKILL.md $env:USERPROFILE\.openclaw\skills\kuake\
```

## 验证

```powershell
openclaw skills list
openclaw skills info kuake
```

如果技能状态为 `✓ ready`，说明部署成功。
