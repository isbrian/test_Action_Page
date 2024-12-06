# 測試GitHub Actions部署page


使用 GitHub Actions 自動做佈署，將程式push到遠端的 main 分支，就會自動執行 YML的腳本
```
npm run build > 產生 dist 資料夾 > 放到 gh-pages 分支 > 瀏覽page
```

## 環境
- node 20
- vite
- vue 3

## 建立一個新專案
npm create vue@latest
記得要加入route

## 加入gitHub Page路由設定
在vite.config.js中加入
```javascript
export default defineConfig({
 base: '/test_Actions/', // Pages的名稱
 …
})
```

## 建立 GitHub 上的 Repo
將 Vite 專案資料夾，推到遠端的 GitHub Repo，使用的 branch 為 main。

## 設定 Workflows 權限
上方選單：點擊settings
左側選取：Action > General
右側內容：Workflow permissions，選擇Read and write premissions

## 製作YML
建立 .github/workflows/deploy.yml檔案：
```yml
name: Deploy
on:
  push:
    branches:
      - main

jobs:
  # 執行打包
  build:
    name: Build
    runs-on: ubuntu-latest

    steps:
      # 複製一份現在的 repo
      - name: Checkout repo
        uses: actions/checkout@v4

      # 建立 Node.js 執行環境
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      # 安裝 npm 相關的相依性
      - name: Install dependencies
        uses: bahmutov/npm-install@v1

      # 執行 Vite 的打包指令
      - name: Build project
        run: npm run build

      # 上傳 dist 資料夾
      - name: Upload production-ready build files
        uses: actions/upload-artifact@v4
        with:
          name: production-files
          path: ./dist

  # 執行佈署
  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      # 下載 dist 資料夾
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: production-files
          path: ./dist

      # 將 dist 資料夾佈署至 GitHub Pages
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

## 設定 GitHub Pages
上方選單：點擊settings
左側選取：pages
右側內容：Build and deployment > source，選取Deploy from sa branch
Branch：選取gh-pages (透過自動佈署後產生的分支)
```
自動生成的 dist 資料夾內容，會被部署到 gh-pages 這個分支內
```