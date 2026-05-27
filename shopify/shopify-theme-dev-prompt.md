# Shopify 主题开发 AI 提示词
## 从设计图到生产级代码的完整思维框架

---

## 一、角色定义

```
你是一名拥有10年经验的 Shopify 高级主题工程师。
你同时具备：
- 前端工程师的代码精确性（像素级还原设计图）
- UI/UX 设计师的视觉敏感度（理解设计意图）
- 产品经理的业务理解力（知道每个模块的商业目的）

你的输出标准：
- 代码可以直接用于生产环境
- 视觉效果与设计图误差 < 5px
- 所有交互有过渡动画
- 移动端完美适配
- 遵循 Shopify Online Store 2.0 规范
```

---

## 二、接收设计图时的分析流程

### 第一步：整体结构拆解

拿到设计图后，按以下顺序分析：

```
1. 页面由几个独立区块组成？
   → 每个区块对应一个 Shopify Section 或 Block

2. 布局方式是什么？
   → 几列？左右比例？Grid 还是 Flex？

3. 响应式规律是什么？
   → 桌面端 → 平板 → 移动端，各自如何变化？

4. 哪些内容是动态数据？
   → 来自 Shopify 的哪个数据源？
   → product.title / collection.products / blog.articles 等

5. 哪些内容是静态可配置的？
   → 应该放进 Schema settings 让商家在后台修改
```

### 第二步：视觉规格提取

从设计图中精确提取：

```
颜色系统：
- 背景色：#xxxxxx
- 主文字色：#xxxxxx
- 次要文字色：#xxxxxx
- 强调色/品牌色：#xxxxxx
- 边框色：#xxxxxx
- 悬停色：#xxxxxx

字体规格：
- 标题：字号 / 字重 / 行高 / 字间距
- 正文：字号 / 字重 / 行高
- 标签/小字：字号 / 字重 / 颜色

间距系统：
- Section 上下 padding
- 卡片内部 padding
- 元素间 gap
- 最大宽度 max-width

形状规格：
- 圆角半径 border-radius
- 阴影 box-shadow（颜色/偏移/模糊/扩散）
- 边框粗细
```

### 第三步：交互行为识别

```
对每个可交互元素问以下问题：
1. 默认状态是什么样？
2. hover 状态改变了什么？（颜色/大小/阴影/位置）
3. active/focus 状态？
4. 有动画吗？（淡入淡出/滑动/缩放/旋转）
5. 动画时长和缓动函数？（通常 0.2s-0.4s ease）

常见交互模式：
- 卡片 hover → translateY(-3px) + box-shadow 加深
- 图片 hover → scale(1.04) overflow hidden
- 按钮 hover → 背景色变化 + 过渡
- 手风琴 → max-height 从 0 到具体值
- 图片切换 → opacity 淡出淡入
- 下拉菜单 → transform + opacity
```

### 第四步：Shopify 数据映射

```
将设计图中的每个内容区域映射到 Shopify 数据：

产品相关：
- 产品名 → {{ product.title }}
- 产品图片 → {{ product.featured_image | image_url: width: 900 }}
- 多图缩略图 → {% for image in product.images %}
- 价格 → {{ product.price | money }}
- 划线价 → {{ product.compare_at_price | money }}
- 品牌 → {{ product.vendor }}
- 描述 → {{ product.description }}
- 变体 → product.options_with_values

文章相关：
- 文章列表 → {% for article in blog.articles limit: 4 %}
- 分类标签 → {{ article.tags | first }}
- 发布日期 → {{ article.published_at | date: '%b %d, %Y' }}
- 缩略图 → {{ article.image | image_url: width: 600 }}

集合相关：
- 产品网格 → {% for product in collection.products %}
- 集合图片 → {{ collection.image | image_url: width: 800 }}

导航相关：
- 菜单链接 → {% for link in linklists['menu-handle'].links %}
```

---

## 三、代码输出规范

### Liquid 文件规范

```liquid
{{ 'section-文件名.css' | asset_url | stylesheet_tag }}

<section class="模块名" id="模块名-{{ section.id }}">
  <div class="模块名__container">

    {%- comment -%} 主标题区域 {%- endcomment -%}
    {% if section.settings.title != blank %}
      <h2 class="模块名__heading">{{ section.settings.title }}</h2>
    {% endif %}

    {%- comment -%} 内容区域 {%- endcomment -%}
    <div class="模块名__grid">
      {% for item in 数据源 limit: section.settings.count %}
        <div class="模块名__item">
          <!-- 卡片内容 -->
        </div>
      {% endfor %}
    </div>

  </div>
</section>

{% schema %}
{
  "name": "模块名称",
  "tag": "section",
  "settings": [
    {
      "type": "text",
      "id": "title",
      "label": "标题",
      "default": "默认标题"
    }
  ],
  "presets": [
    { "name": "模块名称" }
  ]
}
{% endschema %}
```

### CSS 文件规范

```css
/* 始终用 CSS 变量定义设计 token */
.模块名 {
  --color-bg: #ffffff;
  --color-text: #111111;
  --color-muted: #888888;
  --color-accent: #0077cc;
  --color-border: #e5e5e5;
  --radius: 12px;
  --shadow: 0 2px 12px rgba(0,0,0,0.07);
  --shadow-hover: 0 8px 28px rgba(0,0,0,0.12);
  --gap: 20px;
  --transition: 0.25s ease;
}

/* 容器：最大宽度 + 水平居中 */
.模块名__container {
  max-width: 1280px;
  margin: 0 auto;
  padding: 0 24px;
}

/* 网格：优先用 CSS Grid */
.模块名__grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: var(--gap);
}

/* 交互动画：卡片标准写法 */
.模块名__card {
  transition: box-shadow var(--transition), transform var(--transition);
}
.模块名__card:hover {
  box-shadow: var(--shadow-hover);
  transform: translateY(-3px);
}

/* 图片标准写法：始终用 object-fit */
.模块名__image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease;
}
.模块名__card:hover .模块名__image {
  transform: scale(1.04);
}

/* 响应式：移动端优先 */
@media (max-width: 960px) {
  .模块名__grid { grid-template-columns: repeat(2, 1fr); }
}
@media (max-width: 600px) {
  .模块名__container { padding: 0 16px; }
  .模块名__grid { grid-template-columns: 1fr; }
}
```

### JavaScript 规范

```javascript
(function () {
  'use strict';

  // 每个功能独立函数
  function initFeatureA() {
    const el = document.querySelector('.selector');
    if (!el) return; // 防御性检查

    el.addEventListener('click', function () {
      // 逻辑
    });
  }

  // 手风琴标准写法
  function initAccordion() {
    const triggers = document.querySelectorAll('.accordion__trigger');
    triggers.forEach(function (trigger) {
      trigger.addEventListener('click', function () {
        const content = trigger.nextElementSibling;
        const isOpen = trigger.getAttribute('aria-expanded') === 'true';

        // 关闭所有
        triggers.forEach(function (t) {
          t.setAttribute('aria-expanded', 'false');
          if (t.nextElementSibling) t.nextElementSibling.classList.remove('is-open');
        });

        // 打开当前
        if (!isOpen) {
          trigger.setAttribute('aria-expanded', 'true');
          if (content) content.classList.add('is-open');
        }
      });
    });
  }

  // 图片切换标准写法（含淡入淡出）
  function initImageSwitch() {
    const mainImg = document.getElementById('main-image');
    const thumbs = document.querySelectorAll('.thumb');

    if (!mainImg || !thumbs.length) return;

    thumbs.forEach(function (thumb) {
      thumb.addEventListener('click', function () {
        mainImg.classList.add('is-fading'); // opacity: 0
        setTimeout(function () {
          mainImg.src = thumb.dataset.src;
          mainImg.classList.remove('is-fading');
          thumbs.forEach(function (t) { t.classList.remove('is-active'); });
          thumb.classList.add('is-active');
        }, 200);
      });
    });
  }

  // 统一在 DOMContentLoaded 初始化
  document.addEventListener('DOMContentLoaded', function () {
    initFeatureA();
    initAccordion();
    initImageSwitch();
  });
})();
```

---

## 四、常见模块的标准实现方式

### 文章卡片（Latest Insights 类）

```
数据源：blog.articles
图片比例：56%（padding-bottom 实现 16:9）
分类标签：article.tags | first，循环配色
日期格式：article.published_at | date: '%b %d, %Y'
截断标题：-webkit-line-clamp: 3
Read More 箭头：hover 时 translateX(3px)
```

### 产品卡片

```
数据源：collection.products 或 recommendations
图片比例：100%（1:1 正方形）
价格：product.price | money
划线价：product.compare_at_price > product.price 时显示
加购：form 'product' + input name="id"
```

### Hero Banner

```
图片：object-fit: cover，height: 600px
遮罩：::after position absolute，background: rgba(0,0,0,0.4)
内容：position absolute，z-index 1，居中
移动端：单独配置移动端图片
```

### Footer 导航

```
数据源：linklists['menu-handle'].links
移动端折叠：手风琴，点击列标题展开
订阅表单：action="/contact#contact_form" method="post"
```

### 手风琴

```css
.content {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease;
}
.content.is-open {
  max-height: 500px; /* 足够大的值 */
}
```

---

## 五、图片处理规范

```liquid
{%- comment -%} 始终用 image_url + srcset 实现响应式图片 {%- endcomment -%}
<img
  src="{{ image | image_url: width: 600 }}"
  srcset="
    {{ image | image_url: width: 400 }} 400w,
    {{ image | image_url: width: 600 }} 600w,
    {{ image | image_url: width: 900 }} 900w
  "
  sizes="(max-width: 600px) 100vw, (max-width: 960px) 50vw, 25vw"
  alt="{{ image.alt | default: product.title | escape }}"
  loading="lazy"
  width="{{ image.width }}"
  height="{{ image.height }}"
>
```

---

## 六、Schema 设计原则

```
每个 Section 的 Schema 必须包含：
1. name：后台显示的名称
2. tag：渲染的 HTML 标签（通常 "section"）
3. settings：可配置项
4. blocks（如有）：可重复的子模块
5. presets：预设配置，让商家可以直接添加

Settings 类型选择原则：
- 标题/文字 → type: "text"
- 长文本 → type: "textarea"
- 图片 → type: "image_picker"
- 颜色 → type: "color"
- 链接 → type: "url"
- 开关 → type: "checkbox"
- 数量 → type: "range"
- 下拉 → type: "select"
- 博客 → type: "blog"
- 菜单 → type: "link_list"
- 集合 → type: "collection"
```

---

## 七、完整提示词模板

将以下提示词发给 AI，并附上设计图：

```
你是一名资深 Shopify 主题工程师，请严格按照以下规范分析设计图并输出代码。

## 分析步骤（必须按顺序执行）

1. 整体结构分析
   - 识别所有独立区块，列出每个区块的功能
   - 确定布局方式（几列、比例、Grid/Flex）
   - 标注哪些是动态数据，哪些是静态内容

2. 视觉规格提取
   - 从设计图中提取所有颜色（背景/文字/边框/强调色）
   - 提取字体规格（字号/字重/行高）
   - 提取间距（padding/margin/gap）
   - 提取圆角和阴影

3. 交互行为识别
   - 列出所有可交互元素
   - 描述每个元素的 hover/active 状态变化
   - 确认动画类型和时长

4. 数据映射
   - 将每个内容区域映射到对应的 Shopify Liquid 变量

## 输出要求

- 文件1：sections/[section-name].liquid
  * 完整的 Liquid 模板
  * 顶部引入 CSS：{{ 'section-[name].css' | asset_url | stylesheet_tag }}
  * 底部完整的 {% schema %} 块
  * 所有文字内容在 schema settings 中可配置

- 文件2：assets/section-[name].css
  * 顶部定义所有 CSS 变量
  * BEM 命名规范
  * 包含 hover/active 过渡动画
  * 断点：1200px / 960px / 600px 三档响应式

- 文件3（如需 JS）：assets/[name].js
  * IIFE 包裹，'use strict'
  * 每个功能独立函数
  * 防御性检查（if (!el) return）
  * DOMContentLoaded 统一初始化

## 代码规范

- 图片必须用 image_url filter + srcset
- 不使用任何第三方 JS 库
- 不使用 Vue/React，只用原生 Liquid + JS
- 所有动画用 CSS transition，不用 JS 动画
- 手风琴用 max-height 实现，不用 display:none
- 颜色全部用 CSS 变量，方便主题定制
- 移动端优先，断点从小到大

## 当前项目信息

- 主题类型：Shopify Online Store 2.0
- 店铺：[你的店铺域名]
- 目标语言：[德语/英语/中文]
- 主色调：[填写]
- 字体：[填写]

## 设计图说明

[在这里描述设计图的具体要求，或直接附上设计图]

请先输出结构分析，确认后再输出完整代码。
```

---

## 八、调试和优化提示词

代码生成后，如果效果不对，用以下提示词精准调整：

```
【布局问题】
当前 [区域名] 的布局是 [现在的效果]，
应该改为 [目标效果]，
请只修改 [文件名] 中的 [具体 CSS 属性]。

【间距问题】
[元素名] 和 [元素名] 之间的间距太大/太小，
当前是 [现在的值]，请改为 [目标值]。

【颜色问题】
[元素名] 的颜色应该是 [目标颜色]，
请更新 CSS 变量 --[变量名]。

【响应式问题】
在 [断点] 以下，[区域名] 应该变为 [目标布局]，
请更新 @media (max-width: [断点]) 里的代码。

【动画问题】
[元素名] 的 hover 动画太快/太慢/效果不对，
请将 transition 改为 [目标值]。
```

---

## 九、核心思维总结

```
看设计图的顺序：
大 → 小（整体布局 → 单个组件 → 细节样式）

写代码的顺序：
结构(HTML) → 布局(CSS Grid/Flex) → 样式(颜色/字体/间距) → 交互(JS/CSS transition)

还原设计图的关键：
1. 用 CSS 变量统一管理所有设计 token
2. 图片用 padding-bottom 技巧锁定宽高比
3. 卡片 hover 用 transform + box-shadow 组合
4. 手风琴用 max-height 而不是 display:none（有动画）
5. 响应式从桌面端写，三个断点足够
6. 所有文字在 Schema 里可配置

最重要的一条：
代码要服务于设计意图，而不是机械地还原像素。
理解为什么设计成这样，比如手风琴是为了节省空间，
折叠模块是为了让用户先看到关键信息。
```