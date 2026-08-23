# Web 爬虫实战

一本面向实践的 Web 爬虫教程。站点用 Material for MkDocs 构建，托管在 GitHub Pages 项目站。

线上地址：

`https://xuchongfeng.github.io/web_crawler_book/`

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

## GitHub 用户名

站点 URL 已设为 `xuchongfeng`。若要换账号，同时改 `mkdocs.yml` 的 `site_url`、`repo_url`、`repo_name` 和本文件的线上地址。

## 第一次发布

1. 创建 GitHub 仓库 `web_crawler_book`，push `main`。
2. 打开仓库 **Settings → Pages**，Source 选 **GitHub Actions**（不要选 branch / `gh-pages`）。
3. 等待 Actions 工作流成功。
4. 打开 `https://xuchongfeng.github.io/web_crawler_book/`，确认首页、前言、目录骨架都能打开，硬刷新后样式和搜索不是 404。

之后每次 push `main`（或手动 Run workflow）都会重新发布。Pull Request 不会部署。
