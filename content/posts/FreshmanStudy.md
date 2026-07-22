+++
date = '2026-03-15T14:28:14+08:00'
draft = false
title = 'Hugo-Blowfish新手开发教学'
tags=["开发"]
categories=["学习"]
summary = "如何用hugo开发自己的网站，并采用Blowfish的模板"
+++
## hugo的下载安装和开发
- **windows用户**：先去`https://github.com/gohugoio/hugo/releases`下载对应的windows版本，解压后将exe文件放在自己建bin文件夹下，随后在环境变量中进行配置，在powershell中输入`hugo version`可以看到版本号即可。

- **创建第一个blog**：在你的文件夹中执行`hugo new site myblog`，随后`cd C:\hugo\myblog`，然后执行命令`git init`
`git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/你喜欢的主题`,主题可以在`https://themes.gohugo.io/`进行查找。

---

## Blowfish 调整主页面

官方文档：[Blowfish Configuration](https://blowfish.page/docs/configuration/)。运行 `hugo server`，浏览器打开本地地址热更新实时预览。

Blowfish 的配置主要在 `config/_default/` 目录：

| 文件 | 管什么 |
|------|--------|
| `hugo.toml` | 站点级总开关：网址、语言、主题名 |
| `languages.en.toml` | 站点标题、作者简介、社交链接 |
| `params.toml` | 首页布局、配色、页头页脚样式 |
| `menus.en.toml` | 顶部/底部导航菜单文字与链接 |

> 文件名里的 `en` 对应 `hugo.toml` 里的 `defaultContentLanguage`。若改成中文站点，通常要用 `languages.zh-cn.toml` / `menus.zh-cn.toml`，并同步改语言代码。

### 1. 站点基础信息：`hugo.toml`

这个文件决定「网站跑在哪里、用什么主题、默认什么语言」。改错 `baseURL` 时，图片、链接、部署后的路径很容易全坏。

重点字段（对照本站）：

```toml
theme = "blowfish"
baseURL = 'https://lucas2029460802.github.io/myblog/'
defaultContentLanguage = "en"
```

- **`theme`**：告诉 Hugo 使用 `themes/blowfish`。
- **`baseURL`**：线上完整网址，末尾一般带 `/`。GitHub Pages 项目站通常是 `https://用户名.github.io/仓库名/`。本地用 `hugo server` 时 Hugo 会临时处理，但部署前务必填对。
- **`defaultContentLanguage`**：默认语言代码，必须和 `languages.*.toml`、`menus.*.toml` 的后缀一致。

新手建议：先只改 `baseURL` 和语言，其他保持默认即可。

### 2. 站点标题与描述：`languages.en.toml`

浏览器标签页标题、SEO 描述、部分页面抬头都从这里读。

```toml
title = "welcome!"

[params]
  description = "welcome!"
  copyright = "© { year } Nicki"
```

- **`title`**：站点名，会出现在浏览器标题、部分布局的品牌位。
- **`description`**：站点简介，用于元数据和部分首页展示。
- **`copyright`**：页脚版权；`{ year }` 会自动替换成当前年份。

想改成中文站名，直接改这些字符串即可，不必先改文件名。

### 3. 作者信息与社交图标：`[params.author]`

仍在 `languages.en.toml` 里，控制首页/文章页的作者卡片：

```toml
[params.author]
  name = "Nicki"
  image = "img/chiikawa.png"   # 图片放在 assets/ 下
  headline = "welcome to my blog."
  bio = "Record and share here."
  links = [
    { github = "https://github.com/Lucas2029460802" },
    { steam = "https://steamcommunity.com/id/nickiDoe" },
    { email = "2029460802@qq.com" },
    { telegram = "https://t.me/nickiDoe" },
  ]
```

- **`name` / `headline` / `bio`**：名字、一句话介绍、更长的作者简介。
- **`image`**：头像路径相对 `assets/`，例如文件在 `assets/img/chiikawa.png`，这里就写 `img/chiikawa.png`。
- **`links`**：社交图标列表。**键名决定图标**（如 `github`、`email`、`telegram`），**值是跳转地址**。键名写错图标可能不显示；不需要的项直接删掉即可。

支持哪些图标键名，以官方文档 / 主题 icon 集合为准，常见有 `github`、`twitter`、`email`、`telegram`、`linkedin` 等。

### 4. 首页布局和样式：`params.toml`

这是「首页长什么样」的核心文件。本站当前是：

```toml
colorScheme = "blowfish"
defaultAppearance = "light"

[header]
  layout = "fixed"

[footer]
  showMenu = true
  showCopyright = true
  showAppearanceSwitcher = true

[homepage]
  layout = "background"
  homepageImage = "img/background.svg"
  showRecent = true
  showRecentItems = 6
  showMoreLink = true
  showMoreLinkDest = "posts/"
  layoutBackgroundBlur = true
```

分块理解：

- **`colorScheme`**：配色方案，可选 `blowfish`、`neon`、`forest`、`slate` 等，改一个词就能换皮肤。
- **`[header]`**：顶部导航栏样式，如 `fixed`（滚动时固定在顶部）。
- **`[footer]`**：是否显示页脚菜单、版权、明暗切换按钮等。
- **`[homepage]`**（最重要）：
  - `layout`：首页布局。常用 `profile`（个人简介）、`hero`、`card`、`background`（大背景图）。本站用的是 `background`。
  - `homepageImage`：首页背景/主图，路径相对 `assets/`。
  - `showRecent` / `showRecentItems`：是否在首页显示最近文章，以及显示几篇。
  - `showMoreLink` / `showMoreLinkDest`：是否显示「查看更多」以及跳转到哪里（通常是 `posts/`）。


### 5. 导航菜单：`menus.en.toml`

顶部栏（`[[main]]`）和页脚（`[[footer]]`）的文字与链接都在这里。每一项是一个菜单条目：

```toml
[[main]]
  name = "Home"
  url = "/"
  weight = 10

[[main]]
  name = "Posts"
  pageRef = "posts"
  weight = 20

[[main]]
  name = "友链"
  pageRef = "friends"
  weight = 45
```

- **`name`**：菜单上显示的文字，可改成中文，如「文章」「关于」。
- **`pageRef`**：指向站内内容，写内容路径即可（如 `about`、`posts`、`friends`），Hugo 会自动生成正确链接。
- **`url`**：外链或特殊路径时用（如 `https://...` 或 `/`）。`pageRef` 和 `url` 一般二选一。
- **`weight`**：排序权重，数字越小越靠前。

增删菜单：复制一整块 `[[main]] ...`，改 `name` / `pageRef` / `weight`；不需要的整块删掉。页脚同理，把 `[[main]]` 换成 `[[footer]]`。

### 改完怎么验证？

1. 在项目根目录运行 `hugo server`
2. 打开终端提示的地址（通常是 `http://localhost:1313/`）
3. 检查：站点标题、作者卡片、首页背景、顶部菜单是否符合预期
4. 确认无误后再部署；部署用的仍是 `hugo` 生成的 `public/` 目录

## 如何创建新帖子

- **创建新文章文件**：在项目根目录运行 `hugo new posts/你的文件名.md`，Hugo 会在 `content/posts/` 下创建一篇带 Front Matter 的文章。
- **编辑文章信息**：打开新建的 `.md` 文件，在顶部 Front Matter 中设置 `title`、`date`、`draft`、`tags`、`categories` 等；将 `draft` 设为 `false`，文章才会在正式环境中显示。
- **撰写正文内容**：在 Front Matter 下面用 Markdown 写正文，可以使用标题、列表、代码块、图片等常规 Markdown 语法。