# Cursor × Shopify 主题开发教程

## 适合场景

主题开发主要用于开发 Shopify 店铺前台页面，例如：

* 首页模块
* 产品页模块
* 落地页 Section
* 自定义样式
* 图片、按钮、卡片、Tab、轮播等页面 UI

简单理解：

> 只要是改店铺前台页面样式和布局，基本都走 Theme 开发路线。

---

## 一、前置准备

### 1. 检查 Node.js 环境

```bash
node -v
npm -v
```

Node.js 建议使用 18 以上版本。

如果版本太低，去 Node.js 官网下载 LTS 版本。

---

### 2. 安装 Shopify CLI

```bash
npm install -g @shopify/cli
```

验证是否安装成功：

```bash
shopify version
```

---

### 3. 安装 Cursor 的 Shopify AI Toolkit 插件

在 Cursor 插件市场安装 Shopify AI Toolkit。

这个插件主要作用是：

* 查询 Shopify 官方文档
* 生成 Liquid 代码
* 生成 Section 代码
* 检查 Shopify 相关 API / Schema 写法

注意：

> Shopify AI Toolkit 不能直接操作店铺数据，它主要是帮助写代码和查文档。

---

### 4. 配置 Shopify Dev MCP Server

推荐使用 Cursor 一键安装 Shopify Dev MCP。

也可以手动配置：

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

配置完成后，完全退出 Cursor，然后重新打开。

---

### 5. 登录 Shopify 店铺

```bash
shopify auth login
```

终端会给你一个验证码，浏览器会打开 Shopify 授权页面。

登录账号并授权即可。

---

## 二、拉取 Shopify 主题到本地

### 1. 创建本地主题目录

```bash
mkdir ~/Documents/shopify-theme
cd ~/Documents/shopify-theme
```

---

### 2. 拉取线上主题

```bash
shopify theme pull --store your-store.myshopify.com
```

执行后会列出店铺里的主题。

选择带 `[live]` 的正式主题，回车下载。

---

### 3. 本地主题目录结构

下载完成后，你会看到类似结构：

```text
shopify-theme/
├── assets/          CSS / JS / 图片资源
├── config/          主题配置
├── layout/          页面布局模板
├── locales/         多语言翻译
├── sections/        自定义 Section
├── snippets/        可复用代码片段
└── templates/       页面模板
```

重点目录：

```text
sections/  写自定义模块
assets/    写 CSS / JS / 图片资源
snippets/  写可复用代码
templates/ 控制页面使用哪些模块
```

---

## 三、用 Cursor 打开主题项目

进入主题目录后执行：

```bash
code .
```

这样会用 Cursor 打开整个 Shopify 主题项目。

---

## 四、配置 `.cursorrules`

在主题根目录创建 `.cursorrules` 文件。

内容可以写：

```text
你是一名资深 Shopify 主题开发专家，正在开发 Online Store 2.0 主题。

严格遵守以下规则：

- Section 文件放 sections/ 目录，扩展名 .liquid
- CSS 文件放 assets/ 目录，命名为 section-文件名.css
- 在 section 顶部用 stylesheet_tag 引入对应 CSS
- 所有 Section 必须包含完整 {% schema %} 块
- 图片使用 image_url filter + img_tag 处理
- 不使用任何前端框架，只用原生 Liquid + 原生 JS
- 响应式优先，移动端优先设计
- 每个 Section 的 Schema 必须包含 name、tag、settings、presets
```

这个文件的作用是告诉 Cursor：

> 以后帮我写 Shopify 代码时，必须按照这些规范来。

---

## 五、启动本地预览

执行：

```bash
shopify theme dev --store your-store.myshopify.com
```

成功后终端会输出预览链接：

```text
Preview your theme: https://your-store.myshopify.com/?preview_theme_id=xxxxx
```

打开这个链接后，本地修改代码，浏览器会实时刷新。

这不会影响线上正式主题。

---

## 六、用 Cursor 开发 Section

在 Cursor Composer 里输入需求，例如：

```text
我在开发 Shopify Online Store 2.0 主题。

请帮我创建一个“产品特色”Section，要求：

- 左右交替布局
- 图片在左，文字在右，第二个模块反转
- 每个模块包含图片、标签、标题、描述、按钮链接
- 移动端上下堆叠
- 后台最多可配置 4 个特色模块

输出文件：
- sections/product-features.liquid
- assets/section-product-features.css
```

---

## 七、推送代码到 Shopify

### 方式一：推送为草稿主题

不影响线上正式主题：

```bash
shopify theme push --store your-store.myshopify.com --unpublished
```

也可以只推送某几个文件：

```bash
shopify theme push --store your-store.myshopify.com --unpublished \
  --only sections/your-section.liquid \
  --only assets/section-your-section.css
```

---

### 方式二：推送到正式主题

确认没问题后再执行：

```bash
shopify theme push --store your-store.myshopify.com
```

---

## 八、主题开发日常工作流

```text
第一次：
shopify theme pull
↓
Cursor 打开主题目录
↓
shopify theme dev
↓
浏览器预览

日常开发：
Cursor 写代码
↓
保存文件
↓
浏览器预览
↓
确认无误
↓
theme push 上线
```

---

## 九、常见问题

### 1. `shopify theme dev` 报 502

可能是网络或 Shopify 临时问题。

解决方法：

```bash
shopify theme push --store your-store.myshopify.com --unpublished
```

先推送为草稿主题预览。

---

### 2. `shopify auth login` 报错

如果提示 `Nonexistent flag: --store`，说明新版 CLI 不支持：

```bash
shopify auth login --store your-store.myshopify.com
```

直接执行：

```bash
shopify auth login
```

---

### 3. MCP 连接失败

如果是 Mac，不要配置 Windows 的 `cmd`。

Mac 推荐：

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

如果 Cursor 找不到 npx，可以查路径：

```bash
which npx
```

然后改成绝对路径。

---

## 十、主题开发提示词模板

```text
我在开发 Shopify Online Store 2.0 主题，店铺是 your-store.myshopify.com。

请帮我创建【Section 名称】，具体要求：

布局：【描述布局结构】
内容：【描述每个模块的内容】
样式：【颜色、字体、间距、圆角等】
移动端：【响应式要求】
后台可配置项：【列出 settings 和 blocks】

输出文件：
- sections/section-名称.liquid
- assets/section-section-名称.css

要求：
- 符合 Shopify theme-check
- 不写死文字、图片、颜色、链接
- 使用 schema settings 和 blocks 管理内容
- CSS 单独放 assets
```
