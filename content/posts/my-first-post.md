+++
title = '使用 Hugo + Github Page + Github Action 打造個人網站'
date = 2024-07-01T21:29:19+08:00
description = ""
tags = [
    "git",
    "github",
    "github page",
    "github action",
    "hugo"]
categories = ["Dev_流程記錄"]
draft = false
+++

## Preface

- 使用 Notion 一陣子後，漸漸不喜歡他複雜的排版，水平向的擴展也有限
- 越來越習慣使用 Markdown 寫筆記，大多數筆記已遷移至 Obsidian
- 想練習輸出，有什麼能夠直接使用 Markdown 語法上傳的呢？
- Hugo 使用 Go 語言開發，以 Markdown 語法編寫內容，透過模板系統生成最終 HTML 頁面
- 此篇紀錄自己使用 Hugo + Github Page + Github Action 打造個人網站的流程

## Local

1. 安裝 Hugo

- [macOS | Hugo (gohugo.io)](https://gohugo.io/installation/macos/)，使用 `homebrew` 進行安裝

```shell
    brew install hugo
```

2. 創一個資料夾放置相關資料 ex. `blog`

```shell
hugo new site <資料夾名稱>

hugo new site <github帳號.github.io>
#此處命名為github帳號，方便後續使用github page

cd <資料夾名稱>
cd <github帳號.github.io>

git init
```

3. Hugo 框架資料夾結構

   - [Directory structure | Hugo (gohugo.io)](https://gohugo.io/getting-started/directory-structure/)

   ```
    my-site/
    ├── archetypes/  (放置md文章的模板)
    │   └── default.md
    ├── assets/
    ├── config/   <-- site configuration
    │   └── _default/
    │       └── hugo.toml <-- site configuration
    ├── content/ (文章放置的位置)
    ├── data/
    ├── i18n/
    ├── layouts/
    ├── public/     <-- created when you build your site
    ├── resources/  <-- created when you build your site
    ├── config.toml(or hugo.toml)/ (一些參數的設定檔)
    ├── static/ (靜態檔案放置的位置(如圖片檔))
    └── themes/ (落格使用的主題放置的位置)
   ```

4. 安裝 blog 主題

- 使用`PaperMod` 主題(替換使用的第四個...)

  - [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/)

- git submodule 方式安裝

```shell
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

- 建立 config.yaml(recommend to use yaml over toml as it is easier to read.), 更改 theme to "PaperMod"，依據需求調整配置

```
theme = "PaperMod"
```

- 建立 Archives 頁面
- 建立 Search 頁面

5. 啟動 Hugo 伺服器查看網站

```shell
hugo server
#Press `Ctrl + C` to stop Hugo’s development server.
```

6. 嘗試新增 content

```shell
hugo new content posts/my-first-post.md
#Hugo created the file in the `content/posts` directory

```

```
+++
author = ""
title = ""
date =
description = ""
tags = [""]
series = [""]
categories = [""]
draft = true
+++

```

- `archetypes/default.md ` 可在此調整文章樣版格式

7. 部署時，刪去 draft = true，terminal 輸入`hugo` 產生部署使用的`public`資料夾

8. 新增留言功能

- [Hugo comments](https://gohugo.io/content-management/comments/)
- 使用 giscus
  1. 依照指示建立留言專用的 github 資料夾
  2. 將 giscus 產生的程式碼貼至 `layouts/partials/comments.html`

## Github

### Github pages

- [Host on GitHub Pages | Hugo (gohugo.io)](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

1.  設定 Github pages

- 在自己的 Github 新增名稱為`<Github的名稱>.github.io`的 repo 然後在本地端的 blog 目錄中加入這個 repo 的 remote 位址

```shell
git remote add origin <repo位址>
```

- 使用 Github pages 的網頁，網址會是.github.io，所以要修改 config.yaml，把 url 改成相對應的網址

2. 將資料夾 commit 完後 push 至 Github

```shell
  git add .
  git commit -m "Test"

  git push origin

```

### Github action

1.  至 GitHub repository 頁面，選取  **Settings** > **Pages**

2.  更改  **Source**  為  `GitHub Actions`

3.  在本機資料夾中創造一個空的資料夾`.github`

```
  .github/workflows/hugo.yaml
```

4. 在 hugo.yaml 中貼上下方文字，根據需要修改分支名稱和 Hugo 版本

```YAML
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.128.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

5. git commit `Add workflow`，將檔案 push 至 GitHub

6. 在 git hub 首頁點選  **Actions** ，會看到以下畫面，當部署完成燈號會由黃轉綠

7. 點 `commit message` 可以看到部署完成的網站，未來重新 push 變更時，GitHub 都會自動重新部署。

---

References:

- [Caching dependencies to speed up workflows - GitHub Docs](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [GitHub Actions documentation - GitHub Docs](https://docs.github.com/en/actions)
