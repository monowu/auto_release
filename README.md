# auto_release

---
type: slide
---

# 懶人的DevOps：實現自動化 Release 的實用技巧 - 

## 鄭棋文 Steven

---

## 確認安裝項目

* Git V2.y.z
* Node ==V18== 以上
* IDE：VSCode、Jetbrain系列
* Github Account，可以進行登入以及clone repo。
* Windows 請使用 ==Git bash==

補充資料:
[Semantic Versioning](https://semver.org/lang/zh-TW/)
[Conventional Commits](https://www.conventionalcommits.org/zh-hant/v1.0.0/)
👋

---

## 新增 github repository

---



## 執行 git clone
```bash=
git clone https://github.com/your-account-name/your-repo-name.git
```

---

## 進入clone資料夾内,輸入
```bash=
cd your-repo-name
```

```bash=
echo "# your-repo-name" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/your-account-name/your-repo-name.git
git push -u origin main
```
### 確認可以commit,並且push
👋

---

```bash=
npm init
* package name: (automation-release) auto_release
* version: (1.0.0) 0.0.0
* description: demo
* entry point: (index.js)
* test command: jest
* git repository: https://github.com/monowu/auto_release.git
* keywords:
* author: monowu
* license: (ISC)
```

```bash=
git add package.json
```


```bash=
git commit -m "completed npm init"
```
## 隨便給一個Commit
👋

---

## 安裝：🐶 [Husky](https://typicode.github.io/husky/getting-started.html)

```bash=
npx husky-init && npm install
```
&nbsp;
```bash=
git add .husky/*
```
&nbsp;
```bash=
#再隨便給一個commit
git commit -m "add husky"
```
👋

---

.husky/pre-commit
將 npm test 刪除

---

## 新增：Commit-msg
```bash=
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
```
&nbsp;
```bash=
git add .husky/*
```
&nbsp;
```bash=
## 給一個commit
git commit -m "add commit-msg"
```
[husky](https://typicode.github.io/husky/getting-started.html)
👋

---

## 安裝：[Commitlint](https://github.com/conventional-changelog/commitlint#getting-started)
```bash=
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

---

[Commitlint](https://github.com/conventional-changelog/commitlint#getting-started)
## 新增檔案: commitlint.config.js
```bash=
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```
&nbsp;
```bash=
git add commitlint.config.js
```
&nbsp;
```bash=
#再給一個Commit
git commit -m "add commitlint.config.js"
```
👋

---

## Commit message 真的被擋住了

![](https://hackmd.io/_uploads/HkDix7kgT.png)

---

## 修改 commit message
```bash=
git commit -m "feat: add husky and commitlint"
```

---

## VSCode Extension
Conventional commit

## Jetbrain Plugins
1. Git commit Template
2. Conventional commit

---

## 產生changelog

安裝：semantic-release
```bash=
npm install --save-dev semantic-release
```

---

安裝: semantic-release [plugin](https://semantic-release.gitbook.io/semantic-release/extending/plugins-list)
&nbsp;
```bash=
npm install --save-dev @semantic-release/git @semantic-release/changelog
```


---

在根目錄新增 release.config.js
```javascript
//release.config.js
module.exports = {
    branches: ['main'],
    plugins: [
      '@semantic-release/commit-analyzer',
      '@semantic-release/release-notes-generator',
      ['@semantic-release/npm',{npmPublish: false}],
      [
        '@semantic-release/changelog',
        {
          mangle: false,
          headerIds: false,
          changelogFile: 'CHANGELOG.md',
        },
      ],
      [
        '@semantic-release/git',
        {
          assets: ['CHANGELOG.md', 'package.json'],
          message:
            'chore(release): ${nextRelease.version} [skip ci]',
        },
      ],
        
    ],
  };
```
&nbsp;
```bash=
git add release.config.js
git commit -m "chore: add release.config.js"
```
👋

---

## 產生CHANGELOG.md
```bash=
npx semantic-release
```

---

## 加上  --ci=false
```bash=
npx semantic-release –-ci=false
```
👋

---

## 建立Github Release
新增：.github/workflows/release.yaml

```YAML=
# This workflow will do a clean installation of node dependencies, cache/restore them, build the source code and run tests across different versions of node
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nodejs

name: Release workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
permissions:
  contents: read # for checkout

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write # to be able to publish a GitHub release
      issues: write # to be able to comment on released issues
      pull-requests: write # to be able to comment on released pull requests
      id-token: write # to enable use of OIDC for npm provenance
    strategy:
      matrix:
        node-version: [18.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm install
      env:
        HUSKY: 0
    - run: npx semantic-release
      env:
        GH_TOKEN: ${{ secrets.GH_TOKEN }}
      name: Release

```

---

給一個Commit
```bash=
git add .github/*
git commit -m "ci: add github workflow release.yaml"
```

---


## 修改release.config.js
```javascript
//release.config.js
module.exports = {
    branches: ['main'],
    plugins: [
      '@semantic-release/commit-analyzer',
      '@semantic-release/release-notes-generator',
      ['@semantic-release/npm',{npmPublish: false}],
      [
        '@semantic-release/changelog',
        {
          mangle: false,
          headerIds: false,
          changelogFile: 'CHANGELOG.md',
        },
      ],
      [
        '@semantic-release/git',
        {
          assets: ['CHANGELOG.md', 'package.json'],
          message:
            'chore(release): ${nextRelease.version} [skip ci]',
        },
      ],
        '@semantic-release/github',
    ],
  };
  
```

```bash=
npx semantic-release
```
👋

---

```bash=
git add package.json package-lock.json
git add release.config.js
git commit -m "chore: update release.config.js to add github plugin"
git pull
git push
```
👋

---

新增：
GH_TOKEN= Github Personal token


---

## 重跑一次workflow
看看裏面長怎樣?

---

## 給一個feature commit

```bash=
git commit -m "feat: add xxx"
git push
```
接著,看一下Workflow 的結果是啥?🤔

---

# 🎉 👏
# 掌聲鼓勵


---

# 🧐Q&A

---

# 👍🏼
## 一張寫下超讚的地方

# 👎🏼
## 一張寫下痛恨的地方

---

## 歡迎聯絡

FB: https://fb.ChiWenCheng.com 

LinkedIn: https://cv.ChiWenCheng.com 

✉️: steven@ChiWenCheng.com

如果本次的Workshop有幫助到你們
歡迎分享😍

