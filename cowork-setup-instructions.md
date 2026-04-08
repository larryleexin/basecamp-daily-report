# Cowork 安装指南 — Basecamp Daily Report

由于 Cowork 运行在沙盒 VM 中，无法安装 CLI 工具或访问本机 Keychain。
请按以下步骤在 Cowork 中使用本插件。

---

## 方法 1：Cowork Project Instructions（推荐，最简单）

### Step 1：获取 Basecamp API Token

在你的 **本机终端**（不是 Cowork）运行：

```bash
security find-generic-password -s basecamp -w \
  | cut -d: -f2 \
  | base64 -d \
  | python3 -c "import sys,json; print(json.loads(sys.stdin.read())['access_token'])"
```

> 如果你还没安装过 Basecamp CLI，先安装并认证：
> ```bash
> # 从 GitHub Releases 下载（macOS arm64 为例）
> curl -fsSL $(curl -fsSL https://api.github.com/repos/basecamp/basecamp-cli/releases/latest | grep browser_download_url | grep darwin_arm64 | head -1 | cut -d'"' -f4) -o /tmp/bc.tar.gz && tar -xzf /tmp/bc.tar.gz -C ~/.local/bin/ basecamp && chmod +x ~/.local/bin/basecamp
> ~/.local/bin/basecamp auth login
> ```

### Step 2：在 Cowork 中创建项目

1. 打开 Claude Desktop → **Cowork** 标签页
2. 创建一个新项目（或打开已有项目）
3. 点击项目 **设置 / Custom Instructions**
4. 将下面 `cowork-project-instructions.md` 的完整内容粘贴进去
5. 将 `YOUR_TOKEN_HERE` 替换为 Step 1 获取的 Token
6. 将 `YOUR_ACCOUNT_ID` 替换为你的 Basecamp Account ID（Genimex: `3609478`）

### Step 3：使用

在 Cowork 对话中直接说：
- "生成今日 Basecamp 项目摘要"
- "Generate daily Basecamp summary for 3M projects"
- "拉取最近 7 天的项目活动总结"

---

## 方法 2：上传插件 zip

1. 下载 `basecamp-daily-report-v1.0.2.zip`
2. Cowork → 自定义 → 浏览插件 → 上传 zip
3. 在插件设置中填入 `Basecamp API Token` 和 `Account ID`

---

## Token 有效期

- Token 有效期约 **14 天**，到期后需重新获取
- 重新获取方法：在本机终端运行 `basecamp auth login` 后重复 Step 1
- Refresh Token 有效期 **10 年**，CLI 会自动续期

---

## 安全提示

- Token 等同于你的 Basecamp 登录凭证，不要分享给他人
- 每个同事应使用自己的 Token
- 用完后删除 `basecamp-api-token.txt` 文件
