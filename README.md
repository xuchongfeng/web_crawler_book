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
