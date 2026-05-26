# Codex Shopify Workflow

## 1. Project Overview

### 项目目标

本项目的目标是搭建一套可用于 Shopify 主题开发的 Codex 工作流，使 Codex 能够通过 Shopify Dev MCP Server 理解 Shopify 官方开发上下文，并配合 Shopify CLI 完成主题拉取、本地预览、开发、测试和推送。

### 为什么使用 Codex + Shopify

Codex 适合承担代码阅读、修改、问题定位、文档整理和自动化命令执行等工作。Shopify 主题开发涉及 Liquid、JSON templates、sections、snippets、assets、schema 配置以及 Shopify CLI 命令，使用 Codex 可以提升以下效率：

- 快速理解 Dawn Theme 或现有主题结构。
- 根据需求生成或修改 sections、snippets、templates。
- 协助执行 Shopify CLI 命令并分析报错。
- 通过 MCP 获取更贴近 Shopify 官方文档和开发规范的上下文。

### 最终实现效果

预期最终效果：

- Codex 已配置 Shopify Dev MCP Server。
- 本机已安装 Shopify CLI。
- 可通过 Shopify CLI 登录 Shopify 账号。
- 可拉取 Dawn Theme 或指定 Shopify store 的主题代码。
- 可在本地运行 `shopify theme dev` 预览和调试主题。
- 可将修改推送到未发布主题进行测试，确认后再发布。

当前已完成：

- Node.js 可用，版本为 `v24.16.0`。
- npm 可用，版本为 `11.13.0`。
- Shopify CLI 已安装，版本为 `4.0.0`。
- Codex 已添加 `shopify-dev-mcp` 配置。

## 2. Architecture

```text
Codex
  ↓
MCP
  ↓
Shopify CLI
  ↓
Dawn Theme
  ↓
Shopify Store
```

说明：

- Codex：负责代码理解、修改、命令执行和开发协作。
- MCP：通过 Shopify Dev MCP Server 提供 Shopify 开发上下文。
- Shopify CLI：负责认证、主题拉取、本地开发、预览和推送。
- Dawn Theme：Shopify 官方参考主题，也可作为主题开发基础。
- Shopify Store：最终承载和测试主题的店铺，例如 `marstek-ess.myshopify.com`。

## 3. Environment Setup

### Node.js

#### 安装方式

当前环境已经安装 Node.js 和 npm，未在本次流程中重新安装 Node.js。

验证命令：

```bash
node -v
npm -v
```

当前结果：

```bash
node: v24.16.0
npm: 11.13.0
```

#### 版本要求

Shopify CLI 和 Shopify Dev MCP Server 依赖 Node.js 运行环境。根据 Shopify 相关工具链的一般要求，建议使用 Node.js 18 或更高版本。

当前 `v24.16.0` 满足要求。

#### 遇到的问题

Node.js 本身未发现版本问题。

安装 npm 全局包时遇到两个问题：

- 沙箱网络环境无法解析 `registry.npmjs.org`。
- 默认全局 npm 目录 `/usr/local/lib/node_modules` 当前用户无写入权限。

#### 解决方案

网络问题通过授权联网后解决。

权限问题通过将 npm 全局安装目录切换到用户目录解决：

```bash
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
```

并将用户级 npm bin 目录加入 zsh PATH：

```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

该配置已写入：

```bash
~/.zshrc
```

新终端会自动生效；当前终端可执行：

```bash
source ~/.zshrc
```

### Shopify CLI

#### 安装命令

安装命令：

```bash
npm install -g @shopify/cli
```

当前安装结果：

```bash
shopify version
# 4.0.0
```

如果当前 shell 找不到 `shopify`，可使用完整路径：

```bash
/Users/apple/.npm-global/bin/shopify version
```

#### 登录方式

当前 Shopify CLI `4.0.0` 的登录命令为：

```bash
shopify auth login
```

注意：当前版本的 `shopify auth login` 不支持 `--store` 参数。以下命令会失败：

```bash
shopify auth login --store marstek-ess.myshopify.com
```

已验证报错：

```text
Nonexistent flag: --store
```

正确流程是先登录 Shopify 账号：

```bash
shopify auth login
```

CLI 会输出设备验证码和授权链接，例如：

```text
User verification code: XXXX-XXXX
Open this link to start the auth process: https://accounts.shopify.com/activate-with-code...
```

需要在浏览器中打开链接并完成授权。验证码有有效期，过期后需要重新执行登录命令生成新验证码。

#### Theme Pull

登录完成后，可拉取主题代码。典型命令如下：

```bash
shopify theme pull --store marstek-ess.myshopify.com
```

如果需要指定主题 ID：

```bash
shopify theme pull --store marstek-ess.myshopify.com --theme THEME_ID
```

如果是第一次操作某个 store，CLI 可能会要求选择主题或完成额外授权。

### MCP Setup

#### config.toml 配置

Codex MCP 配置文件位置：

```bash
~/.codex/config.toml
```

已添加配置：

```toml
[mcp_servers.shopify-dev-mcp]
command = "npx"
args = ["-y", "@shopify/dev-mcp@latest"]
```

可通过以下命令检查 Codex MCP 配置：

```bash
codex mcp list
codex mcp get shopify-dev-mcp
```

当前已验证 Codex 能识别该 MCP server：

```text
Name             Command  Args                        Status
shopify-dev-mcp  npx      -y @shopify/dev-mcp@latest  enabled
```

#### MCP 启动方式

Codex 会在需要时通过 stdio transport 启动 MCP server：

```bash
npx -y @shopify/dev-mcp@latest
```

手动执行该命令时，如果没有报错并持续运行，通常表示 MCP server 已启动并在等待 MCP client 连接。

配置修改后，当前 Codex 会话可能不会热加载新 MCP server。建议重启 Codex 会话后再使用。

#### 官方文档链接

Shopify AI toolkit 文档：

```text
https://shopify.dev/docs/apps/build/ai-toolkit
```

相关小节：

```text
https://shopify.dev/docs/apps/build/ai-toolkit#install-with-the-dev-mcp-server
```

## 4. Theme Development Workflow

### shopify theme dev

进入主题项目目录后运行：

```bash
shopify theme dev --store marstek-ess.myshopify.com
```

该命令会启动本地开发服务器，并生成可预览的本地链接或 Shopify 预览链接。

### 本地预览

`shopify theme dev` 启动后，可以在浏览器中预览主题改动。常见用途：

- 检查 Liquid 渲染结果。
- 调试 section schema 配置。
- 检查 CSS、JS、图片资源是否正常加载。
- 验证移动端和桌面端布局。

### section 开发

Shopify Online Store 2.0 主题通常通过 sections 和 blocks 组织页面能力。常见开发位置：

```text
sections/
snippets/
templates/
assets/
config/
locales/
```

开发 section 时通常需要关注：

- Liquid HTML 结构。
- `{% schema %}` 配置。
- blocks 配置。
- settings 类型和默认值。
- CSS class 命名和响应式样式。
- 与 Dawn Theme 原有 snippets、global settings 的兼容性。

### theme push

将本地主题推送到 Shopify：

```bash
shopify theme push --store marstek-ess.myshopify.com
```

建议优先推送到未发布主题，避免直接影响线上店铺。

指定主题 ID：

```bash
shopify theme push --store marstek-ess.myshopify.com --theme THEME_ID
```

### 未发布主题测试

推荐流程：

1. 从 Shopify 后台复制 Dawn Theme 或当前线上主题，创建一个未发布主题。
2. 使用 `shopify theme pull` 拉取该未发布主题。
3. 本地通过 `shopify theme dev` 开发和预览。
4. 使用 `shopify theme push` 推送到未发布主题。
5. 在 Shopify 后台或预览链接中验收。
6. 确认无问题后再发布。

## 5. Problems & Solutions

### Problem 1

#### 现象

执行：

```bash
npm install -g @shopify/cli
```

第一次失败，报错：

```text
getaddrinfo ENOTFOUND registry.npmjs.org
```

#### 原因分析

当前命令运行环境的网络访问受限，无法解析或访问 npm registry。

#### 解决方案

使用授权联网方式重新执行安装命令：

```bash
npm install -g @shopify/cli
```

#### 最终结果

网络问题解决，但随后暴露出全局 npm 目录权限问题，见 Problem 2。

### Problem 2

#### 现象

授权联网后再次安装 Shopify CLI，报错：

```text
EACCES: permission denied, mkdir '/usr/local/lib/node_modules/@shopify'
```

#### 原因分析

默认 npm 全局安装目录位于：

```bash
/usr/local/lib/node_modules
```

当前用户没有该目录的写入权限。

#### 解决方案

将 npm 全局安装 prefix 改到用户目录：

```bash
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
```

重新安装：

```bash
npm install -g @shopify/cli
```

将用户级 npm bin 加入 PATH：

```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

#### 最终结果

Shopify CLI 安装成功：

```bash
/Users/apple/.npm-global/bin/shopify version
# 4.0.0
```

### Problem 3

#### 现象

执行：

```bash
shopify auth login --store marstek-ess.myshopify.com
```

报错：

```text
Nonexistent flag: --store
```

#### 原因分析

当前 Shopify CLI `4.0.0` 中，`shopify auth login` 只支持 `--alias`，不支持 `--store`。

#### 解决方案

使用正确登录命令：

```bash
shopify auth login
```

store 参数应在 theme 相关命令中使用，例如：

```bash
shopify theme pull --store marstek-ess.myshopify.com
shopify theme dev --store marstek-ess.myshopify.com
shopify theme push --store marstek-ess.myshopify.com
```

#### 最终结果

已确认正确登录命令。一次登录流程已启动，但验证码过期，需要重新执行 `shopify auth login` 并及时在浏览器中完成授权。

### Problem 4

#### 现象

执行：

```bash
shopify auth login
```

在沙箱环境中失败：

```text
EPERM: operation not permitted, mkdir '/Users/apple/Library/Preferences/shopify-cli-kit-nodejs'
```

#### 原因分析

Shopify CLI 登录时需要写入 macOS 用户偏好目录，但当前命令沙箱不允许写入该目录。

#### 解决方案

使用授权方式重新执行：

```bash
/Users/apple/.npm-global/bin/shopify auth login
```

#### 最终结果

CLI 成功生成设备验证码和授权链接，但验证码过期。需要重新运行登录命令并在有效期内完成授权。

## 6. Useful Commands

```bash
# Node.js / npm
node -v
npm -v
npm config get prefix

# Shopify CLI
shopify version
shopify auth login

# If PATH is not loaded yet
/Users/apple/.npm-global/bin/shopify version
/Users/apple/.npm-global/bin/shopify auth login

# Theme workflow
shopify theme pull --store marstek-ess.myshopify.com
shopify theme dev --store marstek-ess.myshopify.com
shopify theme push --store marstek-ess.myshopify.com

# Optional: specify theme ID
shopify theme pull --store marstek-ess.myshopify.com --theme THEME_ID
shopify theme push --store marstek-ess.myshopify.com --theme THEME_ID

# Codex MCP
codex mcp list
codex mcp get shopify-dev-mcp
npx -y @shopify/dev-mcp@latest

# Reload zsh PATH after modifying ~/.zshrc
source ~/.zshrc
```

