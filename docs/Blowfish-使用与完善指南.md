# Blowfish 主题使用与完善指南

本文档基于 [Blowfish 官方文档](https://blowfish.page/docs/) 整理，用于在 Hugo 项目中使用 Blowfish 主题并逐步完善个人主页。

---

## 一、文档入口与前置要求

- **官方文档首页**：<https://blowfish.page/docs/>
- **前置**：已安装 Hugo，并在项目中通过子模块或复制方式引入了 Blowfish 主题（`themes/blowfish`）。
- **可选**：使用官方 CLI 快速初始化配置：
  ```bash
  npx blowfish-tools
  ```

---

## 二、配置方式：根目录 vs config 目录

Blowfish 推荐使用 **配置目录** 管理站点与主题配置，便于多环境与模块化。

### 2.1 推荐方式：使用 `config/_default/`

1. 在项目根目录创建 `config/_default/` 文件夹。
2. 将主题自带的配置复制到 `config/_default/` 中并按需修改（主题内路径一般为 `themes/blowfish/config/_default/`）。
3. **根目录的 `hugo.toml`**：可删除或仅保留最少内容（如 `theme = "blowfish"`）。  
   若保留根目录配置，Hugo 会与 `config/_default/` 中的配置合并，同名前者的值可能被后者覆盖，需注意 `defaultContentLanguage` 等与语言文件名一致。

### 2.2 配置文件与语言对应关系

- `hugo.toml`：站点级（baseURL、语言、分页、taxonomies、outputs 等）。
- `params.toml`：主题参数（颜色、首页布局、文章/列表行为等）。
- `languages.[语言码].toml`：该语言的标题、作者、日期格式、版权等。**文件名中的语言码需与 `defaultContentLanguage` 一致**（如中文：`zh-cn`）。
- `menus.[语言码].toml`：该语言的主菜单、页脚菜单。语言码需与语言配置一致。

例如仅使用简体中文时，可保留或创建：

- `config/_default/hugo.toml`
- `config/_default/params.toml`
- `config/_default/languages.zh-cn.toml`
- `config/_default/menus.zh-cn.toml`

---

## 三、基础配置详解

### 3.1 站点配置（`hugo.toml`）

在 `config/_default/hugo.toml` 中建议至少设置：

```toml
theme = "blowfish"
baseURL = "https://你的域名.com/"
defaultContentLanguage = "zh-cn"

enableRobotsTXT = true
summaryLength = 0

[pagination]
  pagerSize = 10

[taxonomies]
  tag = "tags"
  category = "categories"
  author = "authors"
  series = "series"

[outputs]
  home = ["HTML", "RSS", "JSON"]
```

- `outputs.home` 必须包含 `HTML`、`RSS`、`JSON`，否则搜索等主题功能可能异常。
- 多语言时，为每种语言准备对应的 `languages.[code].toml` 和 `menus.[code].toml`。

### 3.2 语言与作者（`languages.zh-cn.toml` 示例）

控制站点标题、作者信息（用于首页 Profile 等布局）、日期格式、版权：

```toml
languageCode = "zh-cn"
languageName = "简体中文"
weight = 1
title = "彭知书的小窝"

[params]
  displayName = "简体中文"
  isoCode = "zh-cn"
  rtl = false
  dateFormat = "2006年1月2日"
  description = "站点简介，用于 SEO 等。"
  copyright = "© { year } 彭知书。保留所有权利。"

[params.author]
  name = "彭知书"
  image = "img/avatar.png"    # 放在 assets/img/ 下，建议 1:1 比例
  imageQuality = 96
  headline = "欢迎来到我的小窝"
  bio = "一句话介绍自己。"
  links = [
    { email = "mailto:your@email.com" },
    { github = "https://github.com/yourusername" },
    # 更多见官方文档支持的平台
  ]
```

- 作者头像路径相对于 `assets/`，例如 `img/avatar.png` 即 `assets/img/avatar.png`。
- `params.author.links` 决定首页展示的社交链接及顺序；支持平台见 [Configuration - Author](https://blowfish.page/docs/configuration/#author)。

### 3.3 主题参数（`params.toml`）

在 `config/_default/params.toml` 中可覆盖主题默认行为，常用项包括：

| 参数 | 说明 | 示例 |
|------|------|------|
| `colorScheme` | 配色方案 | `blowfish`、`avocado`、`ocean`、`noir`、`terminal` 等 |
| `defaultAppearance` | 默认外观 | `light` 或 `dark` |
| `autoSwitchAppearance` | 随系统深浅色切换 | `true` / `false` |
| `enableSearch` | 站内搜索 | `true` / `false`（依赖 outputs 含 JSON） |
| `enableCodeCopy` | 代码块复制按钮 | `true` / `false` |
| `[homepage]` | 首页布局与最近文章 | 见下一节 |
| `[article]` | 文章页：日期、作者、目录、阅读时间等 | 按需开启 |
| `[list]` | 列表页：摘要、按年分组等 | 按需开启 |

配色可选：`blowfish`、`avocado`、`fire`、`ocean`、`forest`、`princess`、`neon`、`bloody`、`terminal`、`marvel`、`noir`、`autumn`、`congo`、`slate`、`github`、`one-light` 等。

### 3.4 菜单（`menus.zh-cn.toml` 示例）

主导航（header）与页脚菜单分别用 `[[main]]` 和 `[[footer]]`：

```toml
[[main]]
  name = "首页"
  url = "/"
  weight = 10

[[main]]
  name = "文章"
  pageRef = "posts"
  weight = 20

[[main]]
  name = "标签"
  pageRef = "tags"
  weight = 30

[[footer]]
  name = "标签"
  pageRef = "tags"
  weight = 10
```

- `pageRef`：指向站点内页面或 section（如 `posts`、`tags`）。
- `url`：外链或首页等固定路径（如首页用 `url = "/"`）。
- `weight`：数字越小越靠前；可配合 `parent` 做嵌套菜单（仅 main 支持），详见 [Getting Started - Menus](https://blowfish.page/docs/getting-started/#menus)。
- 需要图标时使用 `pre = "图标名"`（图标名参考 [Samples - Icons](https://blowfish.page/samples/icons/)）。

---

## 四、首页布局（完善首页必读）

首页布局由 `params.toml` 中的 `[homepage]` 控制。

### 4.1 布局类型

在 `config/_default/params.toml` 中设置：

```toml
[homepage]
  layout = "profile"   # 可选见下表
```

| 值 | 说明 |
|----|------|
| `profile` | 作者信息（头像、名字、headline、链接）+ 首页正文 + 可选最近文章 |
| `page` | 仅展示首页 Markdown 内容，无作者区 |
| `hero` | 大图 + 作者信息 + 正文，需设置 `homepageImage` |
| `card` | 卡片式，需设置 `homepageImage` |
| `background` | 背景图风格，需设置 `homepageImage` |
| `custom` | 完全自定义，需自建 `layouts/partials/home/custom.html` |

`hero`、`card`、`background` 需要在 `[homepage]` 中配置 `homepageImage`（图片放 `assets/` 下并写相对路径）。

### 4.2 最近文章

在首页下方显示最近文章列表：

```toml
[homepage]
  showRecent = true
  showRecentItems = 6
  showMoreLink = true
  showMoreLinkDest = "/posts/"
```

文章来源于 `mainSections`（在 `params.toml` 中设置，如 `mainSections = ["posts"]`）。

### 4.3 首页正文内容

- 使用 **Profile / Page / Hero 等** 布局时，首页正文来自 **首页的 Markdown**。
- 若使用 **分支（branch）首页**：在 `content/_index.md` 写首页内容。
- 若使用 **叶子首页**：在 `content/index.md` 或项目约定的首页文件写内容。

首页 Markdown 中可使用 Blowfish 的 [Shortcodes](https://blowfish.page/docs/shortcodes/)（如 `lead`、`button`、`alert`）丰富版式。

---

## 五、内容组织与缩略图

### 5.1 推荐内容结构

```
content
├── _index.md          # 首页正文（若用 branch bundle 首页）
├── about.md           # 关于页等
└── posts
    ├── _index.md      # 文章列表页说明（可选）
    ├── 第一篇文章.md
    └── 某文章
        ├── index.md
        └── feature.png   # 该文章的特色图/缩略图
```

文章既可以是单文件 `posts/xxx.md`，也可以是目录 `posts/xxx/index.md`。需要**特色图**时，请使用**目录形式**并在同目录下放置以 `feature` 开头的图片。

### 5.2 缩略图 / 特色图（Thumbnails）

- 在**文章所在目录**内放置文件名以 `feature` 开头的图片（如 `feature.png`、`featured.jpg`）。
- Blowfish 会将其用作：
  - 列表/卡片中的缩略图；
  - 文章内 Hero 图（若开启）；
  - 社交分享（oEmbed）等。
- 若文章当前是单文件 `post-name.md`，需改为目录 `post-name/index.md` 再放入 `feature*.xxx`。
- 更多见 [Thumbnails](https://blowfish.page/docs/thumbnails/)。

### 5.3 背景图（Hero 背景等）

若希望文章或列表使用与特色图不同的背景图，可在同一文章目录下放置以 `background` 开头的图片。

---

## 六、文章 Front Matter（单篇定制）

除 [Hugo 默认 front matter](https://gohugo.io/content-management/front-matter/) 外，Blowfish 支持每篇文章覆盖主题行为，常用项包括：

| 参数 | 作用 |
|------|------|
| `title`、`description` | 标题与摘要/描述 |
| `summary` | 列表页摘要（不设则按 `summaryLength` 自动截取） |
| `showHero` | 是否在文章内显示特色图作为 Hero |
| `heroStyle` | `basic`、`big`、`background`、`thumbAndBackground` |
| `showDate`、`showDateUpdated` | 显示发布日期/更新日期 |
| `showAuthor` | 是否显示作者信息 |
| `showTableOfContents` | 是否显示目录 |
| `showReadingTime` | 是否显示阅读时间 |
| `showTaxonomies` | 是否显示标签/分类等 |
| `tags`、`categories` | 标签与分类 |
| `series`、`series_order` | 系列与顺序 |

完整列表见 [Front Matter](https://blowfish.page/docs/front-matter/)。

---

## 七、Shortcodes 速览（丰富页面内容）

在 Markdown 中可直接使用 Blowfish 提供的 Shortcode，用于首页或文章内。

| Shortcode | 用途 |
|-----------|------|
| `alert` | 提示框，可配图标与颜色 |
| `lead` | 突出显示一段导语 |
| `button` | 按钮链接 |
| `badge` | 标签/徽章 |
| `icon` | 图标 |
| `accordion` / `accordionItem` | 手风琴折叠 |
| `tabs` / `tab` | 标签页 |
| `figure` | 带标题的图片 |
| `gallery` | 图片画廊 |
| `mermaid` | Mermaid 图表 |
| `youtube` / `video` | 视频嵌入 |

示例：

```markdown
{{< alert >}}
**注意**：这是一条提示。
{{< /alert >}}

{{< lead >}}
这是一段醒目的导语，适合放在首页或章节开头。
{{< /lead >}}

{{< button href="/about/" >}}关于我{{< /button >}}
```

更多用法与参数见 [Shortcodes](https://blowfish.page/docs/shortcodes/)。

---

## 八、如何一步步完善页面（清单）

按下面顺序做，可系统地把 Blowfish 个人站从“能跑”做到“好用、好看”。

1. **统一配置方式**
   - 决定使用根目录 `hugo.toml` 还是 `config/_default/`；若用后者，从主题复制 `hugo.toml`、`params.toml`、`languages.*.toml`、`menus.*.toml` 到 `config/_default/` 并修改。

2. **站点与语言**
   - 在 `hugo.toml` 中设置 `baseURL`、`defaultContentLanguage`（如 `zh-cn`）。
   - 准备对应 `languages.zh-cn.toml`：`title`、`params.description`、`params.dateFormat`、`params.copyright`。

3. **作者与首页形象**
   - 在 `languages.zh-cn.toml` 中配置 `[params.author]`：`name`、`image`、`headline`、`bio`、`links`。
   - 在 `assets/img/` 下放置头像，保证 `image` 路径正确。

4. **首页布局与内容**
   - 在 `params.toml` 的 `[homepage]` 中设置 `layout`（如 `profile`）。
   - 如需最近文章：`showRecent = true`、`showRecentItems`、`showMoreLink`、`showMoreLinkDest`。
   - 在 `params.toml` 中设置 `mainSections = ["posts"]`（或你的主 section）。
   - 编写首页 Markdown（如 `content/_index.md`），用 `lead`、`button`、`alert` 等 shortcode 丰富版式。

5. **菜单**
   - 编辑 `menus.zh-cn.toml`，添加首页、文章、标签、分类等 `[[main]]` 和需要的 `[[footer]]`。

6. **主题外观**
   - 在 `params.toml` 中调整 `colorScheme`、`defaultAppearance`、`autoSwitchAppearance`。

7. **功能开关**
   - 按需开启 `enableSearch`、`enableCodeCopy`；在 `[article]`、`[list]` 中开启目录、摘要、阅读时间等。

8. **文章与缩略图**
   - 重要文章改为目录形式（`posts/文章名/index.md`），并放入 `feature*.png/jpg` 作为缩略图。
   - 在文章 front matter 中按需设置 `showHero`、`heroStyle`、`summary`、`tags`、`categories`。

9. **高级（可选）**
   - 多作者：[Multiple Authors](https://blowfish.page/docs/multi-author/)
   - 系列：[Series](https://blowfish.page/docs/series/)
   - 评论、统计、自定义 CSS：[Partials](https://blowfish.page/docs/partials/)、[Advanced Customisation](https://blowfish.page/docs/advanced-customisation/)
   - 部署：[Hosting & Deployment](https://blowfish.page/docs/hosting-deployment/)

---

## 九、参考链接汇总

| 主题 | 链接 |
|------|------|
| 文档首页 | https://blowfish.page/docs/ |
| 安装 | https://blowfish.page/docs/installation/ |
| 快速开始 | https://blowfish.page/docs/getting-started/ |
| 配置详解 | https://blowfish.page/docs/configuration/ |
| 首页布局 | https://blowfish.page/docs/homepage-layout/ |
| 缩略图 | https://blowfish.page/docs/thumbnails/ |
| Front Matter | https://blowfish.page/docs/front-matter/ |
| Shortcodes | https://blowfish.page/docs/shortcodes/ |
| Partials | https://blowfish.page/docs/partials/ |
| 高级定制 | https://blowfish.page/docs/advanced-customisation/ |
| 部署 | https://blowfish.page/docs/hosting-deployment/ |

本地预览：

```bash
hugo server -D
```

访问终端中显示的地址（通常为 `http://localhost:1313`）即可查看效果。按本文档从“配置方式 → 语言与作者 → 首页 → 菜单 → 内容与 Shortcodes”依次完善，即可系统掌握 Blowfish 并打造个人主页。
