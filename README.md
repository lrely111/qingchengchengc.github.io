# 王文龙的 Linux 学习博客（Hugo + PaperMod）

课程《Linux基础及应用》结课作业：在 Ubuntu 虚拟机里用 Hugo 搭建个人博客，
并用 GitHub Pages 做外网托管。

## 功能

- 首页：个人简介（`[params.homeInfoParams]`）+ 最新文章列表
- 关于：姓名、专业、个人照片
- 标签 / 分类 / 归档：自动由文章的 `tags`、`categories`、`date` 生成
- 搜索：`content/search.md` + `[outputs] home = ["HTML","RSS","JSON"]`

## 本地运行

```bash
# 1. 克隆（主题是 submodule，必须带 --recursive）
git clone --recursive https://github.com/<你的用户名>/<你的用户名>.github.io.git
cd <你的用户名>.github.io

# 2. 启动（允许局域网访问）
hugo server --bind 0.0.0.0 -D --baseURL http://<虚拟机IP>:1313/
```

浏览器打开 `http://<虚拟机IP>:1313/` 即可。

## 部署到 GitHub Pages

1. 新建仓库 `<你的用户名>.github.io`（Public）；
2. 推送源码到 `main` 分支：
   ```bash
   git remote add origin https://github.com/<你的用户名>/<你的用户名>.github.io.git
   git push -u origin main
   ```
3. 仓库 Settings → Pages → Source 选择 **Deploy from a branch** → 分支 `gh-pages` / 目录 `/(root)`；
4. 等 Actions 变成绿色后访问 `https://<你的用户名>.github.io/`。

> 工作流文件：`.github/workflows/hugo.yaml`，其中 `actions/checkout` 必须带
> `submodules: recursive`，否则拉不到 PaperMod 主题，构建会报
> `module "themes/PaperMod" not found`。

## 目录结构

```
.
├── hugo.toml                  # 站点配置（菜单、输出格式、首页信息）
├── content/
│   ├── about.md               # 关于页
│   ├── search.md              # 搜索页（layout: search）
│   ├── archives.md            # 归档页（layout: archives）
│   └── posts/                 # 文章
├── static/images/avatar.png   # 关于页照片
├── themes/PaperMod            # 主题（git submodule）
└── .github/workflows/hugo.yaml
```

## 遇到的问题与解决

| 问题 | 原因 | 解决办法 |
| --- | --- | --- |
| `hugo: command not found` | 只装了系统源的旧版或没装 | 装 Hugo extended（本作业 0.170.1），`hugo version` 能看到版本号 |
| `module "themes/PaperMod" not found` | 克隆仓库时没拉 submodule | `git clone --recursive`，或 `git submodule update --init --recursive` |
| 页面没有样式（纯 HTML） | `baseURL` 与访问地址不一致，CSS 404 | 启动时用 `--baseURL http://<IP>:1313/`，与浏览器地址保持一致 |
| 搜索框输入没反应 | 没生成 `index.json` | `hugo.toml` 加 `[outputs] home = ["HTML","RSS","JSON"]`，`content/search.md` 的 `layout: "search"` |
| 标签/分类页 404 | 菜单里 URL 写错或文章没有 `tags` | 菜单用 `/tags/`、`/categories/`、`/archives/`，文章 front matter 补上 `tags`、`categories` |
| GitHub Actions 构建失败 | workflow 里 checkout 没带子模块 | `actions/checkout@v4` 加 `submodules: recursive` |
| Pages 打开 404 | Pages 源分支选错 | Settings → Pages 选 `gh-pages` 分支，等 Actions 跑完 |
| 主题目录被当成普通文件提交 | 直接 `git add themes/PaperMod` | 用 `git submodule add`，检查 `.gitmodules` 是否存在 |
