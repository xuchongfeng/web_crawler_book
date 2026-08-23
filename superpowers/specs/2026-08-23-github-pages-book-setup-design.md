# GitHub Pages 书籍站 Setup 设计

**日期：** 2026-08-23  
**状态：** 已通过对话确认，待实现  
**范围：** 只做托管与写作脚手架，不写爬虫正文

## 1. 目标

在空仓库 `web_crawler_book` 上搭好一本中文技术书的阅读站点：左侧目录、全文搜索、暗色模式。读者通过 GitHub Pages **项目站**访问：

```text
https://<GitHub用户名>.github.io/web_crawler_book/
```

作者在本地用 Markdown 写作、`mkdocs serve` 预览；push 到 `main` 后由 GitHub Actions 自动构建并发布。

第一版成功标准见第 7 节。书籍章节内容不在本规格内。

## 2. 已锁定的决策

| 项 | 选择 |
|----|------|
| 阅读形态 | 文档站（侧栏 + 搜索 + 暗色），不是 mdBook 式电子书 |
| 工具链 | Python + Material for MkDocs |
| 托管 | GitHub Pages 项目站（子路径 `/web_crawler_book/`） |
| 发布方式 | GitHub Actions 构建 `site/`，用官方 Pages artifact 部署 |
| 第一版范围 | 最小可用：本地预览、自动发布、中文 UI、搜索、暗色、2–3 个占位页 |
| 规格存放 | 仓库根目录 `superpowers/`，不进入 MkDocs 的 `docs/` |

明确不采用：`mkdocs gh-deploy` / `gh-pages` 分支；把构建产物 `site/` 提交进 git。

## 3. 仓库结构

```text
web_crawler_book/
├── mkdocs.yml
├── requirements.txt
├── .gitignore
├── README.md
├── .github/workflows/pages.yml
├── docs/
│   ├── index.md
│   ├── preface.md
│   └── outline.md
└── superpowers/
    ├── specs/
    └── plans/
```

职责：

| 路径 | 职责 |
|------|------|
| `docs/` | 只给读者看的书稿 Markdown |
| `mkdocs.yml` | 站点名、中文、导航、主题、项目站 URL |
| `requirements.txt` | 唯一运行时依赖声明 |
| `.github/workflows/pages.yml` | 构建并部署到 Pages |
| `superpowers/` | 内部设计与实现计划，MkDocs **不收录** |
| `site/` | 构建产物，由 `.gitignore` 排除 |

写作约定：新增章节 = 在 `docs/` 增加 `.md`，并在 `mkdocs.yml` 的 `nav` 挂上。第一版不预建 `ch01-*.md`。

## 4. 站点配置

### 4.1 元信息

- `site_name`：`Web 爬虫实战`
- `site_description`：`一本面向实践的 Web 爬虫教程`
- `site_url`：`https://YOUR_GITHUB_USERNAME.github.io/web_crawler_book/`
- `repo_url`：`https://github.com/YOUR_GITHUB_USERNAME/web_crawler_book`
- `repo_name`：`YOUR_GITHUB_USERNAME/web_crawler_book`

`YOUR_GITHUB_USERNAME` 是**必须替换的固定占位符**，不是未决项。作者创建远程仓库后，在 `mkdocs.yml` 与 `README.md` 中各替换一次。未替换时，本地预览仍可用；线上 CSS/JS/搜索索引路径会错，表现为样式丢失或搜索 404。

### 4.2 主题与读者体验

- 主题：`material`
- 语言：`zh`（搜索框、目录、页脚等 UI 为中文）
- 外观：`theme.palette` 提供「跟随系统 / 浅色 / 深色」三档切换
- 搜索：Material 自带搜索，不接 Algolia
- 代码高亮语言预置：`python`、`bash`、`json`、`yaml`
- 导航 `nav` 仅三项：`首页` → `index.md`，`前言` → `preface.md`，`目录骨架` → `outline.md`
- 第一版不加 “Edit on GitHub”

不额外安装 MkDocs 插件。

### 4.3 依赖与运行时

- `requirements.txt` 只声明：`mkdocs-material>=9,<10`
- 包管理：pip + venv，不引入 poetry / uv
- Python：本地与 CI 均为 **3.12**

### 4.4 占位页职责

- `docs/index.md`：书名、一句话介绍、指向前言与目录骨架的链接
- `docs/preface.md`：前言占位（说明本书稍后填充，不含爬虫教程）
- `docs/outline.md`：未来章节目录骨架（标题列表即可，不写正文）

## 5. 本地写作流

```text
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve
```

浏览器打开 `http://127.0.0.1:8000`。本地预览使用普通 `mkdocs serve`，**不**加 `--strict`，方便起草时暂时缺页。

发布前或 CI 使用：

```text
mkdocs build --strict
```

坏链、`nav` 指向不存在的文件时必须失败。

## 6. 发布链路

默认分支：`main`。

```text
push 到 main（或手动 workflow_dispatch）
  → checkout
  → setup-python 3.12
  → pip install -r requirements.txt
  → mkdocs build --strict
  → 将 site/ 上传为 Pages artifact
  → deploy-pages
  → https://<用户名>.github.io/web_crawler_book/
```

规则：

- 触发：`main` 的 `push`，以及 `workflow_dispatch`。Pull Request **不**部署。
- Pages 来源：仓库 Settings → Pages → Source = **GitHub Actions**（不是 branch / `gh-pages`）。
- 权限：`contents: read`，`pages: write`，`id-token: write`。不把 token 写入仓库。
- 使用官方 action：`actions/upload-pages-artifact` 与 `actions/deploy-pages`。
- 并发：同一 workflow 同时只运行一个部署作业，避免两次 push 交叉覆盖。
- 失败行为：本次部署失败时，上一次成功的 Pages 内容继续可访问，直到下一次成功覆盖。不实现自动回滚脚本。
- 失败排查顺序：依赖安装 → `mkdocs build --strict` → workflow 权限 → 仓库未开启 Pages。

第一次启用 Pages 必须在 GitHub 网页上手动选择 Source = GitHub Actions。workflow 不能代替这一步。

## 7. 验收标准

1. 按第 3 节创建全部文件，仓库已 `git init`，`site/` 与 `.venv/` 被 git 忽略。
2. 本地 `mkdocs serve` 可打开三个占位页；可切换浅色/深色；搜索框能搜到占位标题（如「前言」）。
3. 将 `YOUR_GITHUB_USERNAME` 替换为真实账号后，创建 GitHub 远程仓库并 push `main`。
4. 在仓库 Settings → Pages 将 Source 设为 GitHub Actions。
5. Actions 成功后，项目站三个页面均可访问；硬刷新后 CSS 与搜索资源不是 404（子路径配置正确）。
6. 故意把 `nav` 指到不存在的文件后，`mkdocs build --strict` 必须以非零退出码失败。

## 8. 明确不做（第一版）

- 书籍正文、Giscus 评论、访问统计、mike 多版本、PDF 导出
- 自定义域名
- `gh-pages` 分支或提交 `site/`
- 在 `docs/` 内放置 `superpowers/` 或其他内部文档
- 评论、统计、自定义域名可在后续独立规格中增加

## 9. 实现顺序（给后续计划用）

1. 初始化 git 与忽略规则
2. 加入 `requirements.txt`、`mkdocs.yml`、三份占位 Markdown、`README.md`
3. 加入 `pages.yml`
4. 按第 7 节做本地验收（含 `--strict` 失败用例）
5. 作者替换用户名、创建远程、打开 Pages、确认线上

第 5 步依赖作者的 GitHub 账号与网页设置，实现计划需把它标为人工步骤，不能由 CI 单独完成。
