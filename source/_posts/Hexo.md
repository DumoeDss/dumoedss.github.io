---
title: Hello Hexo
date: 2017-12-31 16:24:08
tags: Hexo
categories: Blog
---
![](http://wx2.sinaimg.cn/mw690/bcd85caely1fmzsv40h65j21hc0u01kx.jpg)

Theme:[Sagiri](https://github.com/DIYgod/hexo-theme-sagiri)

<!-- More-->

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

### Backup to github

```bash
$ hexo backup
```

More info: [Deployment](https://github.com/coneycode/hexo-git-backup)

## 插件

### git推送

```
cnpm install hexo-deployer-git --save
```
### [git备份](https://github.com/coneycode/hexo-git-backup)
```
cnpm install hexo-git-backup --save
```

### [Live2d](https://github.com/EYHN/hexo-helper-live2d)

```
cnpm install --save hexo-helper-live2d --save
```
### [APlayer](https://github.com/MoePlayer/APlayer)

```
cnpm install aplayer --save

{% aplayer "歌名" "歌手" "xx.mp3"  "xx.jpg" "autoplay=false" %}

```
### [DPlayer](https://github.com/MoePlayer/DPlayer)
```
cnpm install dplayer --save

{% dplayer "url=xx.mp4"  "pic=xx.jpg" "loop=yes" "theme=#FADFA3" "autoplay=false" "token=tokendemo" %}
```

Theme Dependence
```
cnpm install nprogress --save 

cnpm install balloon-css --save 
```