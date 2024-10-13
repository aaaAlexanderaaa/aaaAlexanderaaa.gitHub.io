---
title: Github Pages折腾了Hexo 作为 blog
date: 2024-10-13 19:02:10
updated: 2024-10-13 19:02:10
tags:
- Hexo
- Blog
- Tools
categories: 
- ['折腾','Hexo']
keywords:
---

# Github Pages折腾了Hexo 作为 blog



## 0x01 Background



最近刷Github trending的时候看到了AFFiNE，发现加入了AI功能，就搭了一套环境配了AI、改了quota，想要作为写作的平台，但是发现写好的文章不能公开发布。

既然已经折腾到了这个地步，不如再进一步干脆搭个博客吧。



## 0X02 Investigation



在过去的很多刷博客的经验中，我主要接触到的是两种：

1. 自己搭的服务器做维护，比如wordpress这样的支持交互的网站
2. 静态页面，依赖其他平台的现成能力，自己只需要写文章即可，比如利用github pages在github.io上托管页面



我估摸这我这样的需求一定是有很多人产生过，也被很多人解决过，于是直接Google，大致搜索了一下，在这个帖子里看到了比较多的各种各样的方案：[**想搭建自己的博客，求大佬们推荐方案**](https://hk.v2ex.com/t/1009591)

简单归个类：

* 静态博客生成器，比如Hugo, Hexo
  * 无需数据库, 性能好
  * 托管平台
    * GitHub Pages, Vercel, Cloudflare Pages
* 动态博客系统，比如WordPress
  * 功能丰富, 插件生态好, 需要服务器和数据库
* 现成平台，比如Notion、简书、Medium
  * 无需自己搭建维护, 功能和自由度有限
* 完全自建
  * 约等于啥也没说



总体看下来，发现提到最多的无非就是GitHub Pages + Hexo，也不想把选型搞复杂了，看了下UI挺不错直接就上。



## 0X03 Implementation

直接搜GitHub Pages + Hexo能搜到很多的博客和教程，这部分就不多讲了，大概介绍下做了什么，踩了什么坑。



### 01 Dependencies

* [ ] GitHub Pages
  * [ ] 需要Github账户
* [ ] Hexo
  * [ ] 需要Node (我使用的nvm安装的node)
  * [ ] 需要Git



### 02 Deployment

我的环境是macos 包管理用的brew，编辑器用的VScode



1. GitHub Pages配置
   1. 新建一个git repo作为github pages的repo
   2. Setting->Pages->Build and deployment-> Source 选择Github Actions （用Deploy from a brance也行，后边通过 `hexo d -g` 部署，但我不喜欢）
2. Node 配置
   1. `brew install nvm`
      1. 安装后按要求配置path 或 source文件，忘了的话`brew info nvm` 也可以
   2. `nvm install 18`
   3. `nvm use 18`
3. Hexo 初始化
   1. Hexo安装
      ```
      npm install -g hexo-cli
      mkdir my_github_pages && cd my_github_pages && hexo init ./ && git init
      git remote add origin git@github.com:your_username/your_repo.git
      git branch -M main
      ```
   2. github actions配置
      1. 记得修改一下你的branches和node version，我的是main和node 18
      ```
      vim .github/workflows/pages.yml


      ####here is the content of pages.yml
      name: Pages

      on:
        push:
          branches:
            - main # default branch

      jobs:
        build:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v4
              with:
                token: ${{ secrets.GITHUB_TOKEN }}
                # If your repository depends on submodule, please see: https://github.com/actions/checkout
                submodules: recursive
            - name: Use Node.js 18
              uses: actions/setup-node@v4
              with:
                # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
                # Ref: https://github.com/actions/setup-node#supported-version-syntax
                node-version: "18"
            - name: Cache NPM dependencies
              uses: actions/cache@v4
              with:
                path: node_modules
                key: ${{ runner.OS }}-npm-cache
                restore-keys: |
                  ${{ runner.OS }}-npm-cache
            - name: Install Dependencies
              run: npm install
            - name: Build
              run: npm run build
            - name: Upload Pages artifact
              uses: actions/upload-pages-artifact@v3
              with:
                path: ./public
        deploy:
          needs: build
          permissions:
            pages: write
            id-token: write
          environment:
            name: github-pages
            url: ${{ steps.deployment.outputs.page_url }}
          runs-on: ubuntu-latest
          steps:
            - name: Deploy to GitHub Pages
              id: deployment
              uses: actions/deploy-pages@v4
      ```
   3. github push 并且通过github actions构建你的网站
      ```

      #rm -rf .git
      #git remote add origin https://github.com/aaaAlexanderaaa/aaaAlexanderaaa.gitHub.io.git
      #git branch -m main

      #如果遇到了错误：源引用规格 main 没有匹配，应该是没有add/commit就直接push了
      #如果已经执行了很多自己也搞不清楚的操作，你可以remove .git文件夹重新初始化git重新push，但是不建议
      git add .
      git commit -m "init"
      git push -u origin main

      #如果你做了一些奇怪的操作导致github和本地的仓库数据不一致提示： 更新被拒绝，因为远程仓库包含您本地尚不存在的提交
      #你可以给push加上-f强制覆盖，但是这个操作需要额外谨慎
      ```
4. theme配置
   在[github topics hexo-theme](https://github.com/topics/hexo-theme)下有很多hexo的主题，其中我分别尝试了next（更简洁）和fluid（更美观），但是我还是选择了fluid
   以下操作二选一即可
   1. [hexo-theme-next](https://github.com/theme-next/hexo-theme-next)
      1. 我用的git submodule来嵌入next，否则在通过github actions的机制下，git push会遇到致命错误：在 .gitmodules 中未找到子模组路径 'themes/next' 的 url
      ```
      git submodule add https://github.com/iissnan/hexo-theme-next themes/next
      git rm --cached themes/next
      git submodule update --init --recursive
      #这个时候去编辑配置吧
      git submodule status
      git add .
      git commit -m "Change theme"
      git push -u origin main
      ```

   2. [hexo-theme-fluid](https://github.com/fluid-dev/hexo-theme-fluid)
      ```
      npm install --save hexo-theme-fluid
      hexo new page about
      vim ./_config.fluid.yml
      #这个时候去编辑配置吧
      git add .
      git commit -m "Change theme"
      git push -u origin main

      ```


## 0x04 Reflections



整个过程下来花了两个小时吧，从对github page不了解、对hexo不了解、对git 操作不熟悉，到一整套部署下来，觉得接触了挺多新鲜东西的，有点意思和成就感，就把过程记录下来，用作分享，也给大家一些参考。





## 0x99 References

1. https://hk.v2ex.com/t/1009591
2. https://hexo.io/zh-cn/docs/index.html
3. https://hexo.io/zh-cn/docs/github-pages
4. https://sanonz.github.io/2020/deploy-a-hexo-blog-from-github-actions/
5. https://medium.com/@bebebobohaha/%E4%BD%BF%E7%94%A8-hexo-gitpage-%E6%90%AD%E5%BB%BA%E5%80%8B%E4%BA%BA-blog-5c6ed52f23db
6. [https://hexo.fluid-dev.com/docs/guide/](https://hexo.fluid-dev.com/docs/guide/#%E5%85%B3%E4%BA%8E%E6%8C%87%E5%8D%97)

