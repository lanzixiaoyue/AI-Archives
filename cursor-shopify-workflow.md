Cursor 接 Shopify 完整步骤
Cursor 有两种方式，推荐优先用 Plugin（最简单，自动更新），MCP 作为补充。

方式一：Plugin 安装（官方推荐）
Cursor 的 Plugin 是官方推荐的安装方式，会自动更新，始终保持最新能力。 shopify
第一步：去 Cursor Marketplace 安装
直接点这个链接在浏览器打开：
https://cursor.com/marketplace/shopify
页面上有一个 Sign In To Add 按钮，登录你的 Cursor 账号后点击即可一键安装 Shopify 插件。 cursor
####
这个插件包含：

搜索 Shopify 文档
生成和验证 GraphQL、Liquid、UI Extension 代码
Admin API / Storefront API / Functions 全套支持

第二步：重启 Cursor
安装完成后关闭并重新打开 Cursor，插件自动生效。
第三步：验证插件
在 Cursor 的 Chat 里输入：
@shopify 帮我查询 Admin GraphQL API 中获取产品列表的写法
能得到准确回答说明插件已激活。

第二步：重启 Cursor
安装完成后关闭并重新打开 Cursor，插件自动生效。
第三步：验证插件
在 Cursor 的 Chat 里输入：
@shopify 帮我查询 Admin GraphQL API 中获取产品列表的写法
能得到准确回答说明插件已激活。

方式二：Dev MCP Server（进阶，可与 Plugin 同时使用）
Dev MCP Server 让 Cursor 的 AI 能实时搜索 Shopify 文档、探索 API schema，支持 Admin GraphQL API、Storefront API、Liquid、Functions、POS UI Extensions 等全套 API。服务器在本地运行，不需要额外认证。 shopify
第一步：确认 Node.js ≥ 18
输入node -v
第二步：在 Cursor 里添加 MCP Server
方法 A — 一键自动添加（最快）：
浏览器打开这个链接，Cursor 会自动弹出配置界面：
https://cursor.com/en/install-mcp?name=shopify-dev-mcp&config=eyJjb21tYW5kIjoibnB4IC15IEBzaG9waWZ5L2Rldi1tY3BAbGF0ZXN0In0%3D
方法 B — 手动添加：
打开 Cursor，进入 Cursor → Settings → Cursor Settings → Tools and integrations → New MCP server，粘贴以下配置： shopify
json{
  "mcpServers": {
    "shopify-dev-mcp": {
      "command": "npx",
      "args": ["-y", "@shopify/dev-mcp@latest"]
    }
  }
}
Windows 用户如果出现连接错误，改用：
json{
  "mcpServers": {
    "shopify-dev-mcp": {
      "command": "cmd",
      "args": ["/k", "npx", "-y", "@shopify/dev-mcp@latest"]
    }
  }
}
第三步：保存并重启 Cursor
配置保存后，完全关闭 Cursor 再重新打开。

第三部分：安装 Shopify CLI（操作店铺必须）
光有 AI 工具还不够，要真正推送主题、执行店铺操作需要 Shopify CLI。
在 Cursor 终端（Ctrl+``  ``）执行：
输入npm install -g @shopify/cli
shopify version  # 验证安装成功
登录你的店铺：
输入shopify auth login 
终端会输出设备码 To run this command, log in to Shopify.
User verification code: VNQL-QBBZ
浏览器弹出授权页 → 输出设备码-登录 → 点允许。

连接成功后，在 Cursor Chat 里可以这样问：

Shopify CLI 登录店铺 + MCP Server 连接。

Cursor 连接店铺开发自定义 Section 完整步骤
整体架构：
Cursor（AI 写代码）
    ↓
Shopify CLI（连接店铺、同步文件）
    ↓
你的 Shopify 店铺主题

第一步：安装 Shopify CLI
在 Cursor 终端（Ctrl+``  ``）执行：
输入npm install -g @shopify/cli
验证安装：
shopify version

第二步：把店铺主题拉到本地
2-1 创建工作目录
输入mkdir my-shopify-theme
cd my-shopify-theme
2-2 拉取线上主题
shopify theme pull --store your-store.myshopify.com
第一次运行会弹出浏览器要求登录授权，登录你的 Shopify 账号点允许。之后会列出你店铺里所有主题，选择你要编辑的那个（一般是 [live] 标注的那个）。
拉取完成后本地目录结构如下：
my-shopify-theme/
├── assets/
├── config/
├── layout/
├── locales/
├── sections/        ← 自定义 section 放这里
├── snippets/
└── templates/
2-3 用 Cursor 打开这个目录
输入code .
或者 Cursor 菜单 → File → Open Folder → 选择 my-shopify-theme

第三步：启动实时预览（热更新）
在终端执行：
输入shopify theme dev --store your-store.myshopify.com
这会启动一个本地预览服务器，终端会输出一个预览链接，例如：
Preview your theme: https://your-store.myshopify.com/?preview_theme_id=xxxxx
此时你在本地修改任何文件，浏览器会实时刷新预览效果，不影响线上正式主题。

第四步：让 Cursor AI 帮你开发 Section
现在在 Cursor 的 Chat 或 Composer（Ctrl+I）里，直接用自然语言描述你要做的 Section。
示例指令
开发新 Section：
我正在开发 Shopify 主题的自定义 Section。
请帮我在 sections/ 目录下创建一个"产品推荐"Section，要求：
- 顶部有可配置的标题
- 显示 4 个产品卡片（网格布局）
- 每个卡片显示产品图片、名称、价格
- 响应式，移动端变为 2 列
- 包含完整的 Schema 设置
基于已有文件修改：
帮我修改 sections/latest-insights.liquid，
在卡片底部加一个"分享"按钮，点击复制文章链接
Cursor 会自动：

在 sections/ 目录下创建 .liquid 文件
在 assets/ 目录下创建对应的 .css 文件
生成完整的 Schema JSON


第五步：推送到线上店铺
开发完成，确认预览效果没问题后，推送到正式主题：
输入# 推送所有改动
shopify theme push --store your-store.myshopify.com

# 只推送特定文件（更安全）
shopify theme push --store your-store.myshopify.com --only sections/your-section.liquid --only assets/your-section.css

完整工作流总结
第一次使用：
shopify theme pull  →  Cursor 打开目录  →  shopify theme dev（预览）

日常开发循环：
Cursor AI 写代码  →  保存文件  →  浏览器预览  →  满意后 shopify theme push

实用技巧
让 Cursor 更了解你的项目，在项目根目录创建 .cursorrules 文件：
你是一名 Shopify 主题开发专家。
- 本项目是 Shopify Online Store 2.0 主题
- Section 文件放在 sections/ 目录，CSS 放在 assets/
- 所有 Section 必须包含完整的 {% schema %} 配置
- 使用 Liquid 语法，不使用 Vue/React
- 图片使用 Shopify 的 image_url filter 处理响应式
- 遵循 Shopify 主题开发规范
这样每次问 Cursor 问题，它都会自动按照 Shopify 规范来写代码，不需要每次重复说明。