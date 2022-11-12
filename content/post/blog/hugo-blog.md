---
title: "Quick Start Hugo Blog with Netlify"
date: 2022-11-11T21:27:07+09:00
draft: false
summary: "Easy way to start tect blog with Hugo + Netlify."
tags: ['blog']
---

# Create New Site using [Hugo](https://gohugo.io)

> For `macOS`, `homebrew`

1. Install hugo
    ```shell
    brew install hugo
    ```

2. Create new site
    ```shell
    hugo new site blog
    ```

3. Add theme (Customize theme as you want)
    > You should use `https` url when you add the theme to submodule.
    > If you use `SSH` url, it might occur an error when netlify builds the site.

    ```shell
    git init
    git submodule add https://github.com/MeiK2333/github-style.git themes/github-style
    ```
    
4. Start hugo server
    ```shell
    hugo server
    hugo server -D # (with drafts)
    ```

5. Build static pages
    ```shell
    hugo
    hugo -D # (with drafts)
    ```

6. Push to Github
    ```shell
    git remote add origin https://github.com/username/blog-repository.git
    git add .
    git commit -m "First blog"
    git push -u origin main
    ```

    > When you push to Github, you don't need to build by yourself.
    > Once you connect the blog repository to netlify, it will build the site automatically.

# Deploy using [Netlify](https://www.netlify.com)

> 💡 Create netlify account first.

1. Add new site
2. Import an existing project from Github
3. Authorize github account
4. Select blog repository
5. Check site and build settings
    - Check owner, branch name
    - Check Build command is `Hugo`, and Publish directory is `Publish`. `Publish` is default directory when hugo builds a site.

# References

- [Quick start Hugo](https://gohugo.io/getting-started/quick-start/)
- [Host on Netlify](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/)
- [Github Style Theme](https://github.com/MeiK2333/github-style)