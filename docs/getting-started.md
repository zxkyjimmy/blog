# VitePress部署在GitHub上

## Requirements

- 下載安裝[Node.js](https://nodejs.org/en/)
- 準備好[GitHub](http://github.com/)帳號
- [VS Code](https://code.visualstudio.com)

## Step. 1: Create a new repository

在GitHub上建立新的儲存庫，並設定為public，然後按下`Create repository`。

> `.gitignore`可以先不用選擇，等會在自己建立。
> License可以選擇跟VitePress官方一樣的MIT。

![](https://i.imgur.com/krNbLbc.png)


將該儲存庫clone下來，並用VS Code開啟：
> `<USERNAME>`改成自己的GitHub帳號，`<PROJECT>`則是剛才建立的儲存庫名稱

```shell
$ git clone https://github.com/<USERNAME>/<PROJECT> && cd <PROJECT>
$ code .
```

初始化npm：

```shell
$ npm init -y
```

## Step. 2: Install VitePress

利用npm安裝VitePress：

```shell
$ npm install --dev vitepress vue
```

建立第一個文件：

```shell
$ mkdir docs && echo '# Hello VitePress' > docs/index.md
```

修改`package.json`中的`scripts`，加入`dev`、`build`、`serve`：

```json
{
  ...
  "scripts" : {
    "dev": "vitepress dev docs",
    "build": "vitepress build docs",
    "serve": "vitepress serve docs"
  },
  ...
}
```

回到終端機執行下列指令來運行看看本地server：

```shell
$ npm run dev
```

如果出現底下的畫面表示成功，我們就不要動他，讓他持續幫我們運行這個本地server。

![](https://i.imgur.com/XChtFCQ.png)

接著用瀏覽器開啟`http://localhost:5173`，就能看到剛才建立的Hello VitePress

![](https://i.imgur.com/Cu5opfB.png)


## Step 3. Configuration

在`docs`資料夾中創建一個`.vitepress`資料夾，這是用來放置跟vitepress設置相關文件的地方。此時的架構大概是下面這個樣子：

```
.
├─ docs
│  ├─ .vitepress
│  │  └─ config.js
│  └─ index.md
└─ package.json
```

其中`config.js`的設置如下：
```javascript
/**
 * @type {import('vitepress').UserConfig}
 */
const config = {
  lang: 'zh-Hant-TW',
  title: 'VitePress',
  description: 'Just playing around.'
}

export default config
```
> 若想知道還有哪些設定可以調整，請參考[官網介紹](https://vitepress.vuejs.org/config/app-configs)

> 另外，由於`config.js`是新建立的，所以如果要測試網頁的話，可能需要重新執行`npm run dev`

## Step 4. Prettify home page

添加新的頁面：

```shell
$ echo '# Getting Started' > docs/getting-started.md
```

修改首頁`docs/index.md`：
```
---
layout: home

title: VitePress
titleTemplate: Vite & Vue Powered Static Site Generator

hero:
  name: VitePress
  text: Vite & Vue Powered Static Site Generator
  tagline: Simple, powerful, and performant. Meet the modern SSG framework you've always wanted.
  actions:
    - theme: brand
      text: Get Started
      link: /getting-started
    - theme: alt
      text: View on GitHub
      link: https://github.com/zxkyjimmy/blog
---
```

此時我們就有一個漂亮的首頁

![](https://i.imgur.com/4zCIgy2.png)

## Step 5. Deploy on GitHub

### 設定base

由於GitHub幫我們Host(託管)的網站，會包含儲存庫的名稱，因此我們需要修改`config.js`來讓生成的網站連結正確：

```javascript{5}
/**
 * @type {import('vitepress').UserConfig}
 */
const config = {
  base: '/blog',  // 儲存庫名稱
  lang: 'zh-Hant-TW',
  title: 'VitePress',
  description: 'Just playing around.'
}

export default config
```

此時本地路徑會變成`http://localhost:5173/blog/`

![](https://i.imgur.com/mKAKGjr.png)

### 建立GitHub Actions

在`.github/workflows`中建立`deploy.yml`：
> `-p`這個參數可以讓`mkdir`一次建立多層的資料夾

```shell
$ mkdir -p .github/workflows && touch .github/workflows/deploy.yml
```

此時架構大致如下：

```
.
├─ .github
│  └─ workflows
│     └─ deploy.yml
├─ docs
│  ├─ .vitepress
│  │  └─ config.js
│  ├─ getting-started.md
│  └─ index.md
└─ package.json
```

其中`deploy.yml`內容設定如下：

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v3
        with:
          node-version: 16
          cache: npm
      # 讓npm根據package.json和package-lock.json中的內容，安裝對應的套件
      - run: npm ci

      # 建置網站
      - name: Build
        run: npm run build

      # 部署到gh-pages分支
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: docs/.vitepress/dist

```

### 忽視非必要記錄的檔案

創建`.gitignore`檔案，並把我們不想讓git管理的內容寫在裡面：

```
.vscode
node_modules
dist
```

> `node_modules`裏面儲存的是`npm install --dev vitepress vue`時下載下來的套件，底下一共有一千多個檔案，管理起來會太過臃腫。

> `dist`是如果我們使用`npm run build`時會產生的網站資料夾。

### 上傳到GitHub

在上傳之前，先確認一下我們的資料夾有以下的東西：

```
.
├─ .github
│  └─ workflows
│     └─ deploy.yml
├─ docs
│  ├─ .vitepress
│  │  └─ config.js
│  ├─ getting-started.md
│  └─ index.md
├─ .gitignore
├─ package-lock.json
└─ package.json
```

然後我們使用VS Code協助我們上傳。

在commit之前，我們需要選出要讓git紀錄的檔案有哪些。首先點選VS Code左側第三個的版控圖標，然後藉由Changes旁邊的＋號，把這七個檔案都選起來。

![](https://i.imgur.com/reuXLFk.png)

然後在Message裏面打一些文字來紀錄我們做了什麼，然後按下`Commit`：

![](https://i.imgur.com/JSZNq5K.png)

最後將我們剛才的commit push到GitHub上，在VS Code中可以直接點選`Sync Changes`來同步。

![](https://i.imgur.com/bRYPHzE.png)


第一次上傳，VS Code可能會要驗證你的身份，按下`Authorize Visual-Studio-Code`就好

![](https://i.imgur.com/gGuwewI.png)

由於我們剛才建立了`.github/workflows/deploy.yml`，我們可以到儲存庫的頁面上點選Actions，就能看到部署的動作正在運行。

![](https://i.imgur.com/nPMGPkD.png)

稍微等一下後，就能看到該動作完成了。

![](https://i.imgur.com/Fq6vCaq.png)


### 設定GitHub Pages的目標

在儲存庫的`Settings`中找到`Pages`，並將`Branch`設定為`gh-pages`後按下`Save`：

![](https://i.imgur.com/rOBQTYb.png)

稍微耐心等一會兒，就能利用`zxkyjimmy.github.io/blog`來看到自己建立的網頁了。
> 格式為`https://<USERNAME>.github.io/<PROJECT>`

![](https://i.imgur.com/g97Jq7f.png)
