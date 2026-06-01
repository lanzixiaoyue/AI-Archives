# Cursor × Shopify App 开发教程

## 适合场景

App 开发主要用于操作 Shopify 后台数据，例如：

* 查询产品数量
* 读取产品信息
* 修改产品价格
* 创建页面
* 修改导航菜单
* 查询订单
* 读取客户数据
* 使用 Admin API 处理业务逻辑

简单理解：

> 只要你想让 AI 或程序读写 Shopify 后台数据，就需要走 App 开发路线。

---

## 一、App 开发和主题开发的区别

| 对比项    | Theme 开发                  | App 开发                  |
| ------ | ------------------------- | ----------------------- |
| 主要作用   | 改前台页面 UI                  | 操作后台数据                  |
| 常见文件   | sections、assets、templates | shopify.app.toml、app 目录 |
| 技术重点   | Liquid、CSS、JS             | Admin API、GraphQL、权限    |
| 是否操作数据 | 一般不操作                     | 可以操作产品、订单、客户            |
| 适合场景   | 落地页、产品页、Section           | 自动化、数据查询、业务系统           |

重要认知：

> Shopify AI Toolkit 插件本身不能直接操作店铺数据。
> 如果要查询产品、订单、客户等数据，需要通过 App 获取权限和 Access Token。

---

## 二、前置准备

### 1. 检查 Node.js 环境

```bash
node -v
npm -v
```

建议 Node.js 版本大于等于 18。

---

### 2. 安装 Shopify CLI

```bash
npm install -g @shopify/cli
```

验证：

```bash
shopify version
```

---

### 3. 登录 Shopify

```bash
shopify auth login
```

按终端提示完成浏览器授权。

---

## 三、创建 Shopify App 项目

创建项目目录：

```bash
mkdir shopify-app
cd shopify-app
```

初始化 Shopify App：

```bash
npm init @shopify/app@latest
```

CLI 会询问你选择什么类型的项目。

推荐选择官方推荐的 React Router / Remix 类型。

如果提示类似：

```text
Build a React Router app
```

可以选择这个。

---

## 四、启动 App 开发环境

进入 App 项目目录：

```bash
cd your-app-name
```

启动开发：

```bash
shopify app dev
```

这个命令会自动：

* 启动本地开发服务器
* 创建本地 tunnel
* 打开 Shopify 授权页面
* 将 App 安装到店铺
* 进入 Shopify Admin 测试 App

---

## 五、创建 Shopify 后台自定义应用

进入 Shopify 后台：

```text
Shopify 后台
→ 设置
→ 应用和销售渠道
→ 开发应用
→ 创建应用
```

创建后需要配置 Admin API 权限。

---

## 六、配置 Admin API 权限

根据你要做的事情选择权限。

建议最小化授权，不要乱开权限。

| 权限                      | 用途      | 建议        |
| ----------------------- | ------- | --------- |
| Products Read           | 读取产品    | 常用        |
| Products Write          | 修改产品    | 需要时再开     |
| Orders Read             | 读取订单    | 需要订单数据时开  |
| Customers Read          | 读取客户    | 谨慎开启      |
| Pages Read / Write      | 读取或创建页面 | 需要页面自动化时开 |
| Navigation Read / Write | 修改菜单导航  | 需要菜单自动化时开 |

例如，如果你只是想查询产品数量，只需要：

```text
Products Read
```

---

## 七、安装 App 并获取 Client ID / Client Secret

配置权限后点击：

```text
保存
→ 安装应用
```

然后可以看到：

* Client ID
* Client Secret

注意：

> Client Secret 不要发给别人，也不要提交到 Git 仓库。

---

## 八、通过 Client ID 和 Client Secret 换取 Access Token

使用 curl 请求：

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

成功后会返回：

```json
{
  "access_token": "shpat_xxxxxxxxxxxxxxxx",
  "scope": "read_products"
}
```

这个 `access_token` 就可以用于 Admin API 请求。

---

## 九、配置环境变量

不要把 Token 写死在代码里。

可以在项目根目录创建 `.env` 文件：

```env
SHOPIFY_STORE=your-store.myshopify.com
SHOPIFY_ADMIN_ACCESS_TOKEN=shpat_xxxxxxxxxxxxxxxx
SHOPIFY_API_VERSION=2026-04
```

`.env` 文件不要提交到 Git。

可以在 `.gitignore` 里确认有：

```text
.env
```

---

## 十、使用 Admin GraphQL API 查询产品数量

GraphQL Query 示例：

```graphql
query {
  productsCount {
    count
  }
}
```

也可以使用新版 Products 连接查询方式：

```graphql
query {
  products(first: 1) {
    edges {
      node {
        id
        title
      }
    }
  }
}
```

Cursor 提示词可以这样写：

```text
使用当前 Shopify App 的 Admin GraphQL API 查询当前店铺产品总数。

要求：
- 只读操作
- 不修改任何数据
- 输出完整 GraphQL Query
- 输出 API 版本
- 输出当前授权 Scopes
- 输出查询结果
```

---

## 十一、App 开发日常工作流

```text
创建 App 项目
↓
配置 Admin API 权限
↓
安装 App 到店铺
↓
获取 Client ID / Client Secret
↓
换取 Access Token
↓
配置环境变量
↓
使用 Admin API 查询或修改数据
```

---

## 十二、App 开发提示词模板

### 只读查询类

```text
使用当前 Shopify App 的 Admin GraphQL API，
查询【具体数据】。

要求：
- API 版本：2026-04
- 只读操作
- 不修改任何数据
- 输出完整 GraphQL Query
- 输出请求结果
- 如果权限不足，说明需要开启哪个 scope
```

---

### 写入修改类

```text
使用当前 Shopify App 的 Admin GraphQL API，
修改【具体数据】。

要求：
- API 版本：2026-04
- 这是写操作，请先说明需要哪些权限
- 输出完整 GraphQL Mutation
- 输出变量 Variables
- 输出操作结果
- 不要操作无关数据
```

---

## 十三、常见问题

### 1. 找不到 Access Token

现在 Shopify 不建议直接在后台复制 Token。

更标准的方式是：

```text
Client ID + Client Secret
→ OAuth / Client Credentials
→ 换取 Access Token
```

---

### 2. 权限不足

如果 API 返回权限错误，需要回到 App 后台增加对应 scope。

例如：

* 读取产品需要 `read_products`
* 修改产品需要 `write_products`
* 读取订单需要 `read_orders`
* 修改页面需要 `write_pages`

修改权限后需要重新部署或重新安装 App。

---

### 3. 不知道该选哪些权限

原则：

> 用什么开什么，不用的不打开。

比如只是查询产品数量：

```text
只开 Products Read
```

不要随便开启 Customers Write、Financial reports 等高敏感权限。

---

### 4. App 和主题项目是不是一个目录？

不是。

建议分开：

```text
Documents/
├── shopify-theme/   主题开发项目
└── shopify-app/     App 开发项目
```

这样不会混乱。

---

## 十四、App 开发总结

App 开发的核心不是写页面，而是：

```text
权限
Access Token
Admin API
GraphQL
数据操作
```

如果你只是做页面 UI，不需要 App。

如果你要让 Cursor 查询产品、订单、客户、页面、导航等后台数据，就需要 App。
