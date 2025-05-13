+++
date = '2025-05-13T19:01:23+08:00'
draft = false
title = '如何使用Hugo和Github部署自定义博客'
+++
作为一个网上冲浪多年的浪子，白嫖客，CV战神，博主近期有一种特别想回馈社会的冲动，于是想把自己"求学经历"和实践结果记录一下，供后来者学习。

“工欲善其事必先利其器”，一个良好的分享地址是非常重要的。由于博主是shy人+功力不够，不想再B乎和某DN献丑，所以就有了下面部署的过程。

# 工具介绍
Hugo简介：基于Go编写的静态网站生成器，参考 [https://gohugo.com.cn/](https://gohugo.com.cn/)

```bash
# install
wget https://github.com/gohugoio/hugo/releases/download/v0.147.2/hugo_extended_0.147.2_Linux-64bit.tar.gz
tar -xvzf hugo_extended_0.147.2_Linux-64bit.tar.gz
sudo mv hugo /usr/local/bin/
rm -rf hugo_extended_0.147.2_Linux-64bit.tar.gz
hugo version

# how to use
hugo new site my-blog && cd my-blog
hugo new posts/hello-world.md # add something in this md file
hugo server --buildDrafts # visualization
hugo # generate static web file into public
```

**注意事项**
1. 记得把文章最上面的`draft = true`改为`false`
2. 想博主这个blog是加了主题的，主题网址 [https://themes.gohugo.io/](https://themes.gohugo.io/)
```bash
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo 'theme = "ananke"' >> hugo.toml
```

# 使用github部署
使用github部署这里需要先将仓库建立起来，并且得导入我们之前的在public生成的静态网址，博主参考的是github官方的博客 [https://pages.github.com/](https://pages.github.com/), 值得注意的是这里必须是 **usrname.github.io** 这种格式的。
1. 创建仓库，把之前创建好的内容直接导入到新仓库
```bash
(base) username@HOST-10-211-41-34:~/ws/wwliu.github.io$ tree -L 2
.
├── archetypes
│   └── default.md
├── assets
├── content
│   └── posts
├── data
├── hugo.toml
├── i18n
├── layouts
├── public
│   ├── 404.html
│   ├── ananke
│   ├── categories
│   ├── css
│   ├── images
│   ├── index.html
│   ├── index.xml
│   ├── posts
│   ├── sitemap.xml
│   └── tags
├── resources
│   └── _gen
├── static
│   └── images
└── themes
    └── no-style-please
```
2. 在本地创建两个分支 **main** 和 **gh-pages**，分别是主分支（所有代码）和发布分支（只包括生成后的内容）
3. 在主分支添加 Github Actions 支持的配置文件，因为需要自动化同步内容到发布分支
```yml
# .github/workflows/deploy.yml
name: Deploy Hugo site to GitHub Pages
on:
  push:
    branches: [main]
permissions:
  contents: write
  pages: write
  id-token: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: true
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.147.2'
          extended: true
      - name: Build
        run: |
          hugo --minify
          ls -lah ./public
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
```
4. 每次把生产改好的代码推到主分支上就可以看到自动化部署的流程，这里可以参考我的 [https://github.com/wwliu-coder/wwliu.github.io/actions](https://github.com/wwliu-coder/wwliu.github.io/actions)
5. 部署流程结果后就可以在你的仓库连接看到了 [https://wwliu-coder.github.io/wwliu.github.io/](https://wwliu-coder.github.io/wwliu.github.io/)

# 踩坑记录
1. 推代码前记得hugo生产一下静态网页
2. 需要替换对应的地址，否则有些图片不能正常加载
```
# hugo.toml 中baseURL
baseURL = 'https://wwliu-coder.github.io/wwliu.github.io/'
languageCode = 'zh-CN'
title = 'WWLiu Blog'
theme = "no-style-please"
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
```
```markdown
<p style="display: flex;">
    <img src="/wwliu.github.io/images/yeah.gif" width="200" />
    <img src="/wwliu.github.io/images/power.png" width="200" />
    <img src="/wwliu.github.io/images/yeah.gif" width="200" />
</p>
```

***
还有一件重要的事情，最近博主发现网上分享的东西越来越简单，都没有那种纯小白的分享过程，导致自己在创建过程也是踩过很多坑，所以后面的过程也是手把手教学，如果你觉得本人分享的东西还算有用，V5yuan，看看实力。

<p style="display: flex;">
    <img src="/wwliu.github.io/images/yeah.gif" width="200" />
    <img src="/wwliu.github.io/images/power.png" width="200" />
    <img src="/wwliu.github.io/images/yeah.gif" width="200" />
</p>