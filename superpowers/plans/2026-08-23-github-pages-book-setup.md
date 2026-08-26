# GitHub Pages 书籍站 Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在空仓库上搭好 Material for MkDocs 书籍站，本地可预览，push `main` 后由 GitHub Actions 发布到项目站 `/web_crawler_book/`。

**Architecture:** 书稿只放 `docs/`，配置集中在根目录 `mkdocs.yml`。CI 在 Python 3.12 上 `mkdocs build --strict`，把 `site/` 作为 Pages artifact 部署。内部规格在 `superpowers/`，不进入 MkDocs 源码树。

**Tech Stack:** Python 3.12、pip、venv、`mkdocs-material>=9,<10`、GitHub Actions（`actions/checkout`、`actions/setup-python`、`actions/upload-pages-artifact`、`actions/deploy-pages`）。

**Spec:** `superpowers/specs/2026-08-23-github-pages-book-setup-design.md`

## Global Constraints

- `site_name`：`Web 爬虫实战`
- `site_description`：`一本面向实践的 Web 爬虫教程`
- `site_url`：`https://YOUR_GITHUB_USERNAME.github.io/web_crawler_book/`
- `repo_url`：`https://github.com/YOUR_GITHUB_USERNAME/web_crawler_book`
- `repo_name`：`YOUR_GITHUB_USERNAME/web_crawler_book`
- `YOUR_GITHUB_USERNAME` 是必须替换的固定占位符，不是未决项；实现阶段不要擅自换成某个真实账号
- 主题：`material`；语言：`zh`
- `theme.palette` 提供「跟随系统 / 浅色 / 深色」三档切换
- 搜索用 Material 自带搜索，不接 Algolia
- 代码高亮预置语言：`python`、`bash`、`json`、`yaml`（Pygments，不另装插件）
- `nav` 仅三项：`首页` → `index.md`，`前言` → `preface.md`，`目录骨架` → `outline.md`
- 第一版不加 “Edit on GitHub”（`edit_uri` 设为空字符串）
- `requirements.txt` 只声明：`mkdocs-material>=9,<10`
- 包管理：pip + venv，不引入 poetry / uv
- Python：本地与 CI 均为 3.12
- 不额外安装 MkDocs 插件 pip 包
- 第一版不预建 `ch01-*.md`
- `superpowers/` 不进入 MkDocs 的 `docs/`
- 不提交 `site/`，不用 `gh-pages` 分支，不用 `mkdocs gh-deploy`
- 触发：`main` 的 `push` 与 `workflow_dispatch`；Pull Request 不部署
- 不做：书籍正文、Giscus、访问统计、mike、PDF、自定义域名

## File Structure

| 路径 | 职责 |
|------|------|
| `.gitignore` | 忽略 `site/`、`.venv/`、`__pycache__/`、`.DS_Store` |
| `requirements.txt` | 唯一依赖：`mkdocs-material>=9,<10` |
| `mkdocs.yml` | 站点元信息、主题、导航、高亮 |
| `docs/index.md` | 读者首页 |
| `docs/preface.md` | 前言占位 |
| `docs/outline.md` | 目录骨架占位 |
| `README.md` | 本地预览、替换用户名、第一次打开 Pages |
| `.github/workflows/pages.yml` | 构建并部署到 GitHub Pages |

仓库已 `git init`，默认分支 `main`，已有规格文件。不要重新 init。

---

### Task 1: Git 忽略规则

**Files:**
- Create: `.gitignore`
- Test: 用 `git check-ignore`（不新增测试框架）

**Interfaces:**
- Consumes: 已存在的 git 仓库（分支 `main`）
- Produces: `.gitignore` 忽略 `site/`、`.venv/`、`__pycache__/`、`*.py[cod]`、`.DS_Store`

- [ ] **Step 1: 确认忽略规则尚不存在（失败用例）**

Run:

```bash
git check-ignore -v site/ .venv/ __pycache__/
echo "exit=$?"
```

Expected: 退出码 `1`，无匹配输出（还没有 `.gitignore`）。

- [ ] **Step 2: 写入 `.gitignore`**

Create `.gitignore`:

```gitignore
site/
.venv/
__pycache__/
*.py[cod]
.DS_Store
```

- [ ] **Step 3: 确认路径会被忽略**

Run:

```bash
git check-ignore -v site/ .venv/ __pycache__/
```

Expected: 三行均命中 `.gitignore`，退出码 `0`。例如：

```text
.gitignore:1:site/	site/
.gitignore:2:.venv/	.venv/
.gitignore:3:__pycache__/	__pycache__/
```

- [ ] **Step 4: Commit**

```bash
git add .gitignore
git commit -m "chore: ignore MkDocs site, venv, and bytecode"
```

---

### Task 2: 安装 mkdocs-material

**Files:**
- Create: `requirements.txt`
- Test: `.venv` 内 `mkdocs --version`（`.venv` 不被 git 跟踪）

**Interfaces:**
- Consumes: Task 1 的 `.gitignore`（确保 `.venv/` 不会被 add）
- Produces: 可执行的 `mkdocs` 命令，来自 `mkdocs-material>=9,<10`

- [ ] **Step 1: 确认 mkdocs 尚未安装到项目 venv（失败用例）**

Run:

```bash
test ! -x .venv/bin/mkdocs
echo "mkdocs_in_venv=$?"
```

Expected: `mkdocs_in_venv=0`（`test ! -x` 成功，说明还没有 `.venv/bin/mkdocs`）。

- [ ] **Step 2: 写入 `requirements.txt`**

Create `requirements.txt`:

```text
mkdocs-material>=9,<10
```

整份文件只有这一行，不要追加 pytest、poetry、uv 或其他插件包。

- [ ] **Step 3: 创建 venv 并安装**

Run:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -U pip
.venv/bin/pip install -r requirements.txt
```

Expected: pip 成功安装 `mkdocs-material` 9.x。若本机 `python3` 不是 3.12，改用已安装的 3.12 解释器（例如 `python3.12 -m venv .venv`）。CI 将固定 `3.12`。

- [ ] **Step 4: 确认 mkdocs 可用，且 venv 被忽略**

Run:

```bash
.venv/bin/mkdocs --version
git check-ignore -v .venv/
git status --short
```

Expected:

- `mkdocs` 版本行包含 `version`，且能导入 Material（安装 `mkdocs-material` 即提供 `mkdocs`）
- `.venv/` 被 ignore
- `git status --short` 只出现 `requirements.txt`（以及未提交的其他工作文件），**没有** `.venv/`

- [ ] **Step 5: Commit**

```bash
git add requirements.txt
git commit -m "chore: add mkdocs-material 9.x dependency"
```

---

### Task 3: MkDocs 站点配置与占位页

**Files:**
- Create: `mkdocs.yml`
- Create: `docs/index.md`
- Create: `docs/preface.md`
- Create: `docs/outline.md`
- Test: `.venv/bin/mkdocs build --strict` 与生成物断言

**Interfaces:**
- Consumes: Task 2 的 `.venv/bin/mkdocs` 与 `requirements.txt`
- Produces: 可 strict 构建的站点；`nav` 三项；`site/` 仅构建产物，不入库；构建结果不含 `superpowers/` 页面

- [ ] **Step 1: 无配置时构建必须失败**

Run:

```bash
.venv/bin/mkdocs build --strict
echo "exit=$?"
```

Expected: 非零退出码（缺少 `mkdocs.yml` / `docs/`）。不要在这一步“修好”它。

- [ ] **Step 2: 写入 `mkdocs.yml`**

Create `mkdocs.yml`:

```yaml
site_name: Web 爬虫实战
site_description: 一本面向实践的 Web 爬虫教程
site_url: https://YOUR_GITHUB_USERNAME.github.io/web_crawler_book/
repo_url: https://github.com/YOUR_GITHUB_USERNAME/web_crawler_book
repo_name: YOUR_GITHUB_USERNAME/web_crawler_book
edit_uri: ""

theme:
  name: material
  language: zh
  palette:
    - media: "(prefers-color-scheme)"
      toggle:
        icon: material/brightness-auto
        name: 切换至浅色模式
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-7
        name: 切换至深色模式
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-4
        name: 切换至跟随系统

markdown_extensions:
  - pymdownx.highlight:
      guess_lang: false
  - pymdownx.superfences

nav:
  - 首页: index.md
  - 前言: preface.md
  - 目录骨架: outline.md
```

不要添加 `plugins` 下的第三方插件。不要设置非空 `edit_uri`。不要把 `YOUR_GITHUB_USERNAME` 换成真实账号。

`pymdownx.highlight` 使用 Pygments，已包含 `python`、`bash`、`json`、`yaml`，围栏写 ```` ```python ```` 即可，无需再装包。

- [ ] **Step 3: 写入三份占位 Markdown**

Create `docs/index.md`:

```markdown
# Web 爬虫实战

一本面向实践的 Web 爬虫教程。

- [前言](preface.md)
- [目录骨架](outline.md)
```

Create `docs/preface.md`:

```markdown
# 前言

本书正文稍后填写。当前页面只用于验证站点导航、搜索与发布流程。
```

Create `docs/outline.md`:

```markdown
# 目录骨架

- 第 1 章 环境与 HTTP
- 第 2 章 静态页面采集
- 第 3 章 结构化解析
- 第 4 章 动态页面
- 第 5 章 存储与任务调度
```

不要在这些文件里写爬虫教程正文。不要创建 `docs/superpowers/`。

- [ ] **Step 4: strict 构建必须成功**

Run:

```bash
.venv/bin/mkdocs build --strict
```

Expected: 退出码 `0`，生成 `site/`。

- [ ] **Step 5: 断言生成物与隔离规则**

Run:

```bash
test -f site/index.html
test -f site/preface/index.html
test -f site/outline/index.html
test ! -e site/superpowers
grep -q "Web 爬虫实战" site/index.html
grep -q "前言" site/index.html
grep -q "目录骨架" site/index.html
git check-ignore -v site/index.html
```

Expected: 三个 `test -f` 成功；`site/superpowers` 不存在；首页 HTML 含书名与两个导航名；`site/index.html` 被 git 忽略。

本地 `mkdocs serve` 挂在 `http://127.0.0.1:8000/`（站点根路径）。线上才是 `/web_crawler_book/`。这是预期行为，不要为了本地预览改 `site_url`。

- [ ] **Step 6: 坏链时 `--strict` 必须失败，然后恢复**

Run（先破坏，再恢复，中间不要 commit）：

```bash
cp mkdocs.yml mkdocs.yml.bak
printf '\n  - 坏链: missing.md\n' >> mkdocs.yml
set +e
.venv/bin/mkdocs build --strict
echo "strict_broken_exit=$?"
set -e
mv mkdocs.yml.bak mkdocs.yml
.venv/bin/mkdocs build --strict
echo "strict_restored_exit=$?"
```

Expected:

- `strict_broken_exit` 为非零
- `mkdocs.yml` 已恢复为 Step 2 的内容
- `strict_restored_exit=0`

- [ ] **Step 7: Commit**

```bash
git add mkdocs.yml docs/index.md docs/preface.md docs/outline.md
git commit -m "feat: scaffold MkDocs Material book site"
```

不要 `git add site/`。

---

### Task 4: README 写作与发布说明

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: Task 2 的 venv 命令、Task 3 的 `mkdocs.yml` 占位符与三个页面
- Produces: 作者按 README 能完成本地预览、替换 `YOUR_GITHUB_USERNAME`、第一次打开 Pages

- [ ] **Step 1: 确认 README 尚不存在（失败用例）**

Run:

```bash
test ! -f README.md
echo "exit=$?"
```

Expected: `exit=0`（文件还不存在）。若已有其他 README，停下来先和本任务内容合并，不要覆盖未知内容。

- [ ] **Step 2: 写入 `README.md`**

Create `README.md`（写入文件时不要包含最外层的四重围栏）：

````markdown
# Web 爬虫实战

一本面向实践的 Web 爬虫教程。站点用 Material for MkDocs 构建，托管在 GitHub Pages 项目站。

线上地址（把用户名换掉之后）：

`https://YOUR_GITHUB_USERNAME.github.io/web_crawler_book/`

## 本地预览

需要 Python 3.12。

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve
```

浏览器打开 `http://127.0.0.1:8000`。本地预览在站点根路径，没有 `/web_crawler_book/` 前缀。

发布前检查坏链：

```bash
mkdocs build --strict
```

`site/` 是构建产物，不要提交。

## 替换 GitHub 用户名

把下面两处的 `YOUR_GITHUB_USERNAME` 换成你的 GitHub 用户名：

1. `mkdocs.yml` 里的 `site_url`、`repo_url`、`repo_name`
2. 本文件 `README.md` 里的线上地址

不替换也能本地预览。不替换就发布的话，线上 CSS / JS / 搜索索引路径会错。

## 第一次发布

1. 创建 GitHub 仓库 `web_crawler_book`，push `main`。
2. 打开仓库 **Settings → Pages**，Source 选 **GitHub Actions**（不要选 branch / `gh-pages`）。
3. 等待 Actions 工作流成功。
4. 打开 `https://<你的用户名>.github.io/web_crawler_book/`，确认首页、前言、目录骨架都能打开，硬刷新后样式和搜索不是 404。

之后每次 push `main`（或手动 Run workflow）都会重新发布。Pull Request 不会部署。
````

- [ ] **Step 3: 确认 README 含三块必写说明**

Run:

```bash
grep -q "mkdocs serve" README.md
grep -q "YOUR_GITHUB_USERNAME" README.md
grep -q "GitHub Actions" README.md
```

Expected: 三个 `grep` 退出码均为 `0`。

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: add local preview and Pages setup README"
```

---

### Task 5: GitHub Actions 部署工作流

**Files:**
- Create: `.github/workflows/pages.yml`
- Test: Python 读取 YAML 关键字段（标准库 `pathlib` + 文本断言，不新增 PyYAML 依赖）

**Interfaces:**
- Consumes: Task 2 的 `requirements.txt`；Task 3 的 `mkdocs build --strict` 与 `site/` 输出目录
- Produces: `.github/workflows/pages.yml`——仅 `main` push 与 `workflow_dispatch` 触发；权限 `contents: read`、`pages: write`、`id-token: write`；Python 3.12；artifact 路径 `site`

- [ ] **Step 1: 确认工作流文件不存在（失败用例）**

Run:

```bash
test ! -f .github/workflows/pages.yml
echo "exit=$?"
```

Expected: `exit=0`。

- [ ] **Step 2: 写入 `.github/workflows/pages.yml`**

Create `.github/workflows/pages.yml`:

```yaml
name: Deploy GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Build site
        run: mkdocs build --strict
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

不要添加 `pull_request` 触发。不要调用 `mkdocs gh-deploy`。不要把 token 写进文件。

- [ ] **Step 3: 断言工作流关键字段**

Run:

```bash
python3 - <<'PY'
from pathlib import Path
text = Path(".github/workflows/pages.yml").read_text(encoding="utf-8")
required = [
    "branches:\n      - main",
    "workflow_dispatch:",
    "contents: read",
    "pages: write",
    "id-token: write",
    'python-version: "3.12"',
    "mkdocs build --strict",
    "actions/upload-pages-artifact@v3",
    "path: site",
    "actions/deploy-pages@v4",
]
for item in required:
    assert item in text, f"missing: {item!r}"
assert "pull_request" not in text
assert "gh-deploy" not in text
assert "gh-pages" not in text
print("workflow_ok")
PY
```

Expected: 打印 `workflow_ok`，退出码 `0`。

- [ ] **Step 4: Commit**

```bash
git add .github/workflows/pages.yml
git commit -m "ci: deploy MkDocs site to GitHub Pages"
```

---

### Task 6: 作者发布清单（人工步骤）

**Files:**
- Modify: 无（实现代理不改代码；作者按本任务操作远程）
- Test: 线上三个页面可访问，硬刷新后静态资源不是 404

**Interfaces:**
- Consumes: Task 3 的 `YOUR_GITHUB_USERNAME` 占位符；Task 4 的 README；Task 5 的 workflow
- Produces: 远程仓库 + Pages Source = GitHub Actions + 可访问的项目站

实现代理**不要**猜测或写入某个真实 GitHub 用户名。下列命令由作者在已登录 `gh` 的机器上执行，或在 GitHub 网页上等价完成。

- [ ] **Step 1: 替换占位符**

作者在 `mkdocs.yml` 与 `README.md` 中，把所有 `YOUR_GITHUB_USERNAME` 替换为自己的 GitHub 用户名。替换后：

```bash
grep -n 'YOUR_GITHUB_USERNAME' mkdocs.yml README.md
```

Expected: 无输出（退出码 `1` 表示 grep 没找到，这是成功）。

然后提交（作者本地执行，信息用真实意图即可）：

```bash
git add mkdocs.yml README.md
git commit -m "chore: set GitHub username for project Pages URL"
```

- [ ] **Step 2: 创建远程并 push `main`**

若使用 GitHub CLI：

```bash
gh repo create web_crawler_book --public --source=. --remote=origin --push
```

Expected: 远程出现 `web_crawler_book`，`main` 已推送。不要把仓库命名为 `<用户名>.github.io`（那是用户站，本规格是项目站）。

- [ ] **Step 3: 打开 Pages**

在 GitHub 网页：仓库 **Settings → Pages → Source** 选 **GitHub Actions**。不要选 Deploy from a branch。

- [ ] **Step 4: 确认 Actions 成功**

打开仓库 Actions，工作流 `Deploy GitHub Pages` 为绿色。若失败，按顺序查：依赖安装 → `mkdocs build --strict` → workflow 权限 → Pages 是否已开启。

- [ ] **Step 5: 线上验收**

打开：

- `https://<用户名>.github.io/web_crawler_book/`
- `https://<用户名>.github.io/web_crawler_book/preface/`
- `https://<用户名>.github.io/web_crawler_book/outline/`

Expected: 三个页面 200；硬刷新后 CSS / JS / 搜索请求不是 404；可切换浅色/深色；搜索「前言」能出结果。

---

## Self-Review

**Spec coverage**

| 规格项 | 任务 |
|--------|------|
| 仓库结构第 3 节全部文件 | Task 1–5 |
| `site/` / `.venv/` 忽略 | Task 1、Task 3 Step 5 |
| `mkdocs-material>=9,<10`、pip+venv、Python 3.12 | Task 2、Task 5 |
| 元信息、中文、三色、nav、空 `edit_uri` | Task 3 |
| 三份占位页职责 | Task 3 |
| `superpowers/` 不进站点 | Task 3 Step 5 |
| 本地 serve 不加 `--strict`；CI/发布前加 | Task 4、Task 5 |
| Actions artifact 部署、权限、并发、无 PR 部署 | Task 5 |
| 坏链 `--strict` 失败 | Task 3 Step 6 |
| 替换用户名、创建远程、打开 Pages、线上验收 | Task 6（人工） |
| 不做评论/统计/mike/PDF/自定义域名/gh-pages | 全任务未引入 |

**Placeholder scan:** 计划中的 `YOUR_GITHUB_USERNAME` 与规格锁定的占位符相同。没有 TBD / 另写测试但不给命令 / “类似 Task N”。

**Consistency:** `site_name`、三份文档文件名、workflow 名 `Deploy GitHub Pages`、artifact 路径 `site`、Python `3.12` 前后一致。
