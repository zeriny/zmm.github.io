# 个人学术主页

纯静态站点，无构建步骤。结构与交互克隆自一个极简学术主页模板（顶栏导航、侧栏个人信息、论文年份/主题切换、BibTeX 折叠、章节折叠、滚动导航高亮），配色改为藏青主题。

## 站点结构

```
index.html    主页（Home / Publications / Honors & Awards / Teaching / Service）
blogs.html    博客页（顶部大标题 + 文章列表，文字为静态占位）
posts/        博文正文（每篇一个独立 HTML，复制现有文件即可）
styles.css    全部样式，配色参数集中在文件顶部 :root
app.js        渲染逻辑：读取 data/*.json 并填充各页面
data/         全部内容数据（见下表）
assets/profile.jpg          头像（直接替换同名文件即可）
files/publications/         论文 PDF（文件名对应 publications.json 的 file 字段）
files/slides/               演讲 slides PDF（文件名对应 publications.json 的 slides 字段）
```

## 本地预览

必须通过 HTTP 访问（页面用 fetch 加载 JSON，直接双击 index.html 会显示 "Preview error"，属预期行为）：

```
cd 本目录
python -m http.server 8000
```

浏览器打开 http://localhost:8000 （博客页：http://localhost:8000/blogs.html ）。

## 内容编辑（改哪个文件对应哪个版块）

| 要改的内容 | 改这个文件 | 说明 |
|---|---|---|
| 姓名、职称、单位、邮箱 | `data/profile.json` | `name` 字段同时驱动两个页面的左上角名字和浏览器标签标题；HTML 里的静态名字只是加载前的回退值 |
| 个人简介 | `data/profile.json` 的 `bio` | 支持 HTML（如 `<br>`、链接、`<code>`） |
| 招生启事（简介末尾彩色加粗一段） | `data/profile.json` 的 `recruiting` | 纯文字即可，颜色/加粗自动；可含 HTML 链接；不填则不显示。颜色由 `--recruiting` 变量控制 |
| 侧栏按钮（Scholar / CV） | `data/profile.json` 的 `links` | 只显示 label 为 Google Scholar / CV / Current Site 的条目 |
| 头像 | `assets/profile.jpg` | 直接替换同名文件 |
| 侧栏 News | `data/updates.json` | 每条 `{date: "MM/DD/YYYY", text: "可含HTML"}`；加 `"top": true` 可置顶 |
| Publications | `data/publications.json` | 字段见下方 |
| Honors & Awards | `data/honors.json` | 每条 `{year, title, url?, note?}`，按年份自动分组 |
| Patents | `data/patents.json` | 每条 `{year, title, url?, note?}`，按年份自动分组（与 Honors 相同渲染） |
| Teaching | `data/courses.json` | 分组结构：`[{group: "组名", items: [{title, role?, note?, terms: [..]}]}]`；`role`（如 "Teaching assistant"）显示在课程名前，`note` 为补充说明，`terms` 显示为括号内容 |
| Service | `data/service.json` | 每条 `{year, role, venue}`，按年份自动分组，同年多条各占一行 |
| 博客文章列表 | `data/blogs.json` | 每条 `{title, date, tag, excerpt, url}`；`url` 指向 `posts/` 下的本地文件，见下方"写一篇新博文" |
| 博客页顶部大标题和引言 | `blogs.html` | 这两段是静态文字，直接改 HTML |

### publications.json 字段说明

| 字段 | 作用 |
|---|---|
| `title`, `author`, `year`, `month` | 基本信息作者列表中与你 `profile.json` 的 `name` 相同的名字会自动加粗 |
| `category` | 数组，主题（Topic）视图的分组依据 |
| `type` | `inproceedings`（会议）/ `article`（期刊）/ `misc` |
| `booktitle_long/short`, `city`, `country`, `journal_long/short`, `howpublished` | 发表地信息；`short` 显示为括号缩写（如 IMC'26） |
| `file` | 论文 PDF：自动链接到 `files/publications/<file>.pdf`。目前三条占位都指向共用的 `placeholder.pdf`，放入你自己的 `<file>.pdf` 即可 |
| `note`, `dataset`, `software`, `talk` | 各链接按钮（Link / Data / Software / Talk），填 URL 即显示对应颜色标签 |
| `slides` | Slides 按钮，两种填法：填文件名则类似 `file`，自动链接到 `files/slides/<slides>.pdf`；填完整 `https://` URL 则链接外部 |
| `extra` | 附加信息行，显示在发表地下一行：字符串或字符串数组（多条时每条一行），支持 HTML。例如 `"* Presented in OARC 43"` |
| `award` | 金色标签，填文字（如 "Best Paper Award"） |
| `corresponding` | 通信作者标注：填某位作者的全名（须与 `author` 数组里的写法完全一致），该作者名后显示橙色 `*` 角标；不填则不显示 |

说明：期刊条目的 `vol`/`pp` 字段只作存档，BibTeX 生成器读取的是 `volume`/`number`，因此展开的 BibTeX 不含卷号——这是与原模板一致的行为，介意的话把字段改成 `volume`/`number` 即可。

## 写一篇新博文

博文正文是 `posts/` 目录下的独立 HTML 文件，无构建步骤：

1. 复制 `posts/` 里任意一篇（如 `posts/post-1.html`），改名为 `posts/my-post.html`；
2. 编辑文件内四处内容，其余（顶栏导航、样式引用）不用动：
   - `<title>`（浏览器标签标题）
   - `.post-meta` 一行（日期 · 分类，如 `Oct 2026 · Research`）
   - `#post-title` 大标题
   - `.post-body` 里的正文（标题/列表/代码/引用等样式均有示例，可含图片）
3. 在 `data/blogs.json` 里加一条，`url` 填 `posts/my-post.html`。

列表页（blogs.html）的排序 = `blogs.json` 里的数组顺序，新文章放数组最前面即可。本地链接在当前标签打开；若某篇放在外部站点（Medium 等），`url` 直接填完整 `https://` 地址，会自动在新标签打开。

## 换配色（只改 styles.css 顶部的 :root 变量块）

| 变量 | 当前值 | 影响范围 |
|---|---|---|
| `--primary` | `#1e3a5f` | 主色：所有版块标题、按钮和标签底色、年份/主题分组标题、顶栏中央、博客页大标题、金色 award 标签的文字 |
| `--primary-dark` | `#162c49` | 深一档：所有按钮/标签的 hover 色、顶栏渐变两端 |
| `--primary-tint` | `#eef2f8` | 博客页日期徽章的浅色底 |
| `--primary-tint-border` | `#c9d6e8` | 博客页日期徽章的描边 |
| `--accent` / `--accent-fill` | `#c64600` / `#e87722` | 橙色点缀：正文链接、标题下划线渐变、Link/Data 标签、博客页 eyebrow |
| `--stone` | `#75787b` | 灰色标签（Slides / Software / BibTeX）、博客页分类小字 |
| `--tag-talk` | `#5f7384` | Talk 标签的蓝灰色 |
| `--recruiting` | `#8b0000` | 简介末尾招生启事的颜色（加粗） |
| `--text` / `--white` | `#333333` / `#ffffff` | 正文文字 / 顶栏和按钮上的白字 |

不在变量里的颜色（改动较少的地方）：顶栏底部橙色装饰线是 `.topbar::after` 里的四段渐变；米色纸感页面底纹在 `body` 的 `background`；award 标签的金底 `#ffd700` 在 `.pub-tag--award`；头像相框的暖色边框在 `.profile-photo`。

## 增加或删除版块

这是唯一需要动多个文件的操作（顶栏导航在每个页面各存一份：`index.html`、`blogs.html`、以及 `posts/` 下所有博文）。以增加一个 Projects 版块为例：

1. **index.html**：导航 `<nav class="topnav">` 里加 `<a href="#projects">Projects</a>`；`<main class="content">` 里按现有版块的格式加一段：
   ```html
   <section id="projects" class="content-section">
     <h2 class="section-title">Projects</h2>
     <div id="projects-content"></div>
   </section>
   ```
2. **blogs.html 和 posts/ 下所有博文**：导航里加对应链接（`index.html#projects`，posts/ 下的文件要用 `../index.html#projects`），各页导航保持一致。
3. **app.js**：
   - `DATA_FILES.home` 里加一行 `projects: "data/projects.json",`
   - `init()` 里（`renderService(service);` 附近）加 `renderProjects(data.projects);`
   - 仿照 `renderHonors` 写一个 `renderProjects` 函数，把内容填进 `#projects-content`
4. 新建 `data/projects.json`。

删除版块即反向操作（两处 HTML 的导航链接 + section、app.js 里的渲染调用和函数、对应的 JSON 文件）。版块在页面上的顺序 = HTML 里 `<section>` 的排列顺序；折叠箭头和滚动导航高亮对新版块自动生效，无需额外配置。

## 部署

任意静态托管均可。GitHub Pages：把本目录推到一个仓库，在仓库 Settings → Pages 里选 main 分支即可，无需任何构建配置。
