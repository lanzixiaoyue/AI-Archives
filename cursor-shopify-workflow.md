# Cursor × Shopify 开发完整指南

> 本文档覆盖两条开发路线：**主题开发（Theme）** 和 **App 开发**，适合零基础到进阶的 Shopify 开发者。

---

## 目录

1. [整体架构](#整体架构)
2. [前置准备](#前置准备)
3. [主题开发路线](#主题开发路线)
4. [App 开发路线](#app-开发路线)
5. [Cursor 提示词规范](#cursor-提示词规范)
6. [常见问题](#常见问题)

---

## 整体架构

Cursor 连接 Shopify 的完整链路如下：

```
Claude / Cursor / Codex
        ↓
Shopify AI Toolkit（插件层：文档查询 + 代码生成）
        ↓
MCP Server（协议层：实时 API Schema 查询）
        ↓
Shopify CLI（执行层：主题推送 / App 部署）
        ↓
Shopify Store / Admin API
```

### 两条开发路线对比

| 路线 | 适用场景 | 核心文件 | 能做什么 |
|---|---|---|---|
| **Theme 开发** | 页面 UI、Section、样式 | `sections/` `assets/` `templates/` | 开发页面布局、自定义模块 |
| **App 开发** | 数据操作、业务逻辑 | `shopify.app.toml` | 读写产品、订单、客户数据 |

> ⚠️ 重要认知：Shopify AI Toolkit 插件**只提供文档和代码生成能力**，无法直接操作店铺数据。要操作店铺数据，必须走 App 路线获取 Access Token。

---

## 前置准备

### 1. 环境要求

```bash
node -v   # 必须 >= 18
npm -v    # 确认可用
```

Node.js 不满足版本要求请前往 [nodejs.org](https://nodejs.org) 下载 LTS 版本。

### 2. 安装 Shopify CLI

```bash
npm install -g @shopify/cli
shopify version   # 验证安装成功
```

### 3. 安装 Shopify AI Toolkit 插件（Cursor）

浏览器打开以下链接，登录 Cursor 账号后一键安装：

```
https://cursor.com/marketplace/shopify
```

插件能力：
- 搜索 Shopify 官方文档
- 生成和验证 GraphQL、Liquid、UI Extension 代码
- 支持 Admin API / Storefront API / Functions 全套

### 4. 配置 Dev MCP Server（可选，增强 AI 能力）

**方法 A：一键安装（推荐）**

浏览器打开以下链接，Cursor 自动弹出配置界面：

```
https://cursor.com/en/install-mcp?name=shopify-dev-mcp&config=eyJjb21tYW5kIjoibnB4IC15IEBzaG9waWZ5L2Rldi1tY3BAbGF0ZXN0In0%3D
```

**方法 B：手动配置**

打开 `Cursor → Settings → Tools and integrations → New MCP server`，粘贴以下配置：

```json
{
  "mcpServers": {
    "shopify-dev-mcp": {
      "command": "npx",
      "args": ["-y", "@shopify/dev-mcp@latest"]
    }
  }
}
```

> Windows 用户若出现连接错误，将 `command` 改为 `"cmd"`，`args` 改为 `["/k", "npx", "-y", "@shopify/dev-mcp@latest"]`。

配置完成后**完全退出 Cursor 重新打开**生效。

### 5. 登录店铺

```bash
shopify auth login
```

终端会输出一个设备验证码，例如：

```
To run this command, log in to Shopify.
User verification code: VNQL-QBBZ
```

浏览器弹出授权页 → 输入验证码 → 登录账号 → 点允许。

---

## 主题开发路线

适用于：开发页面 UI、自定义 Section、修改主题样式。

### 第一步：拉取主题到本地

```bash
# 创建工作目录（推荐放在文稿目录）
mkdir ~/Documents/shopify-theme
cd ~/Documents/shopify-theme

# 拉取线上主题
shopify theme pull --store your-store.myshopify.com
```

执行后会列出店铺里所有主题，用方向键选择带 `[live]` 标注的正式主题，回车开始下载。

下载完成后本地目录结构：

```
shopify-theme/
├── assets/          ← CSS / JS / 图片资源
├── config/          ← 主题配置
├── layout/          ← 页面布局模板
├── locales/         ← 多语言翻译
├── sections/        ← 自定义 Section（主要开发目录）
├── snippets/        ← 可复用代码片段
└── templates/       ← 页面模板（控制哪个页面用哪个 Section）
```

用 Cursor 打开目录：

```bash
code .
```

### 第二步：配置 `.cursorrules`（重要）

在主题根目录创建 `.cursorrules` 文件，让 Cursor AI 始终按照 Shopify 规范写代码：

```
你是一名资深 Shopify 主题开发专家，正在开发 Online Store 2.0 主题。

严格遵守以下规则：
- Section 文件放 sections/ 目录，扩展名 .liquid
- CSS 文件放 assets/ 目录，命名为 section-文件名.css
- 在 section 顶部用 stylesheet_tag 引入对应 CSS
- 所有 Section 必须包含完整 {% schema %} 块
- 图片使用 image_url filter + img_tag 处理
- 不使用任何前端框架，只用原生 Liquid + 原生 JS
- 响应式优先，移动端优先设计
- 每个 Section 的 Schema 必须包含：name、tag、settings、presets
```

### 第三步：启动本地预览

```bash
shopify theme dev --store your-store.myshopify.com
```

成功后终端输出预览链接：

```
Preview your theme: https://your-store.myshopify.com/?preview_theme_id=xxxxx
```

在浏览器打开此链接，本地修改文件后浏览器**实时刷新**，不影响线上正式主题。

> 如果 `theme dev` 不稳定（502 报错），改用推送草稿主题的方式预览（见下方）。

### 第四步：用 Cursor 开发 Section

在 Cursor Composer（`Cmd+I`）里用自然语言描述需求，例如：

```
我在开发 Shopify Online Store 2.0 主题。

请帮我创建一个"产品特色"Section，要求：
- 左右交替布局：图片在左文字在右，第二个反转
- 每个特色包含：图片、标签、标题、描述、链接
- 移动端变为上下堆叠
- 后台可配置最多 4 个特色模块

输出：
- sections/product-features.liquid
- assets/section-product-features.css
```

### 第五步：推送到店铺

**推送为草稿预览（不影响线上）：**

```bash
shopify theme push --store your-store.myshopify.com --unpublished \
  --only sections/your-section.liquid \
  --only assets/section-your-section.css
```

**推送到正式主题：**

```bash
shopify theme push --store your-store.myshopify.com
```

### 主题开发工作流总结

```
第一次：
shopify theme pull → Cursor 打开目录 → shopify theme dev

日常循环：
Cursor AI 写代码 → 保存文件 → 浏览器预览 → 满意后 theme push 上线
```

---

## App 开发路线

适用于：通过 Admin API 读写产品、订单、客户等店铺数据。

> App 开发和 Theme 开发是**两套完全独立的体系**，分别对应不同的项目目录和工作流。

### 第一步：创建 App 项目

```bash
mkdir shopify-app
cd shopify-app
npm init @shopify/app@latest
```

CLI 会询问框架选择，推荐选择官方推荐的 **Remix**：

```
Build:
> Remix   ← 选这个
```

### 第二步：创建并安装 App（获取 Access Token）

**2026 年最新方式（重要）：**

2026 年 1 月起，Shopify 已停止在店铺后台直接生成 Access Token 的旧方式，现在必须通过以下流程：

**在 Shopify 后台创建自定义应用：**

```
Shopify 后台 → 设置 → 应用和销售渠道 → 开发应用 → 创建应用
```

**配置所需权限（最小化原则）：**

| 权限 | 用途 | 是否勾选 |
|---|---|---|
| `Online store pages` Read and write | 创建/修改页面 | ✅ 按需 |
| `Navigation` Read and write | 创建/修改导航菜单 | ✅ 按需 |
| `Products` Read | 读取产品 | ✅ 按需 |
| `Orders` Read | 读取订单 | ✅ 按需 |
| `Products` Write | 修改产品 | ⚠️ 谨慎勾选 |
| `Customers` Write | 修改客户数据 | ❌ 非必要不勾 |
| `Financial reports` | 财务数据 | ❌ 不勾 |

配置完成后点「保存」→「安装应用」，获取 **Client ID** 和 **Client Secret**。

**通过 Client Credentials 换取 Access Token：**

```bash
curl -X POST \
  https://your-store.myshopify.com/admin/oauth/access_token \
  -H "Content-Type: application/json" \
  -d '{
    "client_id": "你的ClientID",
    "client_secret": "你的ClientSecret",
    "grant_type": "client_credentials"
  }'
```

成功返回：

```json
{
  "access_token": "shpat_xxxxxxxxxxxxxxxx",
  "scope": "write_pages,read_pages,write_navigation"
}
```

> ⚠️ Access Token 请妥善保管，不要提交到 Git 仓库。

### 第三步：启动 App

```bash
shopify app dev
```

CLI 会自动：
- 创建本地隧道（tunnel）
- 安装 App 到店铺
- 启动本地开发服务器
- 打开浏览器进入 Admin 界面

### 第四步：配置 Admin API 权限

进入 Shopify Partners 后台：

```
Apps → 你的 App → Configuration → Admin API access scopes
```

按需添加权限，修改后执行：

```bash
shopify app deploy
```

重新部署使权限生效。

### 第五步：Cursor 操作店铺数据

将 Access Token 配置到 MCP，然后在 Cursor Chat 里：

```
使用 Admin GraphQL API 查询当前店铺的产品总数。
只读操作，不修改任何数据。

输出：
- 产品数量
- 使用的 API 版本
- 当前授权 Scopes
- 完整 GraphQL Query
```

### App 开发工作流总结

```
创建 App → 配置权限 → shopify app dev → 获取 Token
                                              ↓
                              Cursor 通过 Admin API 操作店铺数据
```

---

## Cursor 提示词规范

### 主题开发提示词模板

```
我在开发 Shopify Online Store 2.0 主题，店铺是 your-store.myshopify.com。

请帮我创建【Section 名称】，具体要求：

布局：[描述布局结构]
内容：[描述每个模块的内容]
样式：[颜色、字体等]
移动端：[响应式要求]
后台可配置项：[列出可配置的内容]

输出文件：
- sections/section-名称.liquid
- assets/section-section-名称.css
```

### App 数据操作提示词模板

```
使用当前 Shopify App 的 Admin GraphQL API，
查询/创建/修改 [具体数据]。

要求：
- API 版本：2026-04
- [只读操作 / 写操作]
- 输出完整的 GraphQL Query/Mutation
- 输出操作结果
```

---

## 常见问题

### Q：`shopify theme dev` 报 502 错误

原因：Shopify 服务器临时问题或网络不稳定。

解决：改用推送草稿主题方式预览：

```bash
shopify theme push --store your-store.myshopify.com --unpublished
```

### Q：MCP Server 连接失败（`spawn cmd ENOENT`）

原因：Mac 环境下配置了 Windows 的 `cmd` 命令。

解决：打开 `~/.cursor/mcp.json`，将 `command` 改为 `"npx"`。

### Q：MCP 连接失败（`Connection closed`）

原因：`npx` 路径未加入系统 PATH，Cursor 找不到命令。

解决：用 `which npx` 获取完整路径，在 `mcp.json` 中使用绝对路径：

```json
{
  "mcpServers": {
    "shopify-dev-mcp": {
      "command": "/usr/local/bin/npx",
      "args": ["-y", "@shopify/dev-mcp@latest"]
    }
  }
}
```

### Q：`shopify auth login` 报 `Nonexistent flag: --store`

原因：新版 CLI 的 `auth login` 不支持 `--store` 参数。

解决：直接执行 `shopify auth login`，不带任何参数。

### Q：2026 年找不到 Access Token 在哪里复制

原因：Shopify 2026 年 1 月起废弃了直接复制 Token 的方式。

解决：现在需要通过 Client ID + Client Secret 走 OAuth 流程换取 Token，详见 [App 开发路线 - 第二步](#第二步创建并安装-app获取-access-token)。

---

## 参考资源

| 资源 | 链接 |
|---|---|
| Shopify 官方文档 | https://shopify.dev/docs |
| Admin GraphQL API | https://shopify.dev/docs/api/admin-graphql |
| Shopify CLI 文档 | https://shopify.dev/docs/api/shopify-cli |
| Access Token 获取 | https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/generate-app-access-tokens-admin |
| Cursor MCP 配置 | https://cursor.com/en/install-mcp |
| Dev MCP Server | https://shopify.dev/docs/apps/build/devmcp |
