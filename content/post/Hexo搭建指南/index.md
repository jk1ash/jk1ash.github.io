---
title: Hexo搭建指南
description: 
date: 2023-01-01 00:00:00+0000
image: 
categories:
    - 2023
tags:
    - Blog
weight: 1
---

## 1.环境准备

- node.js ([node.js安装教程](https://jklash.com/index.php/archives/12.html))
- nginx (使用宝塔安装)

## 2.安装Hexo

```bash
npm install hexo-cli -g
```

## 3.配置Hexo

### 创建第一个Blog

```bash
hexo init blog
cd blog
npm install
```

### 启动

```bash
npm install hexo --save
#清理
hexo clean
#生成静态文件
hexo g
hexo generate
#安装部署插件
npm install hexo-deployer-git --save
#推送远程
hexo d
hexo deploy
#启动服务
hexo s
hexo server
```

### 主题配置

hexo官方主题仓库地址：https://hexo.io/themes/

主题推荐：[butterfly](https://github.com/jerryc127/hexo-theme-butterfly)

```bash
#将主题解压至博客的themes目录下
tar -zxvf hexo-theme-butterfly-XXX.tar.gz -C themes

#修改config.yml
#添加或者修改, 和themes目录下的主题目录名一致
theme: butterfly
```

## 4.发布

### 方法1: 使用git自动发布到本地和GitHub Page

```bash
mkdir git
cd git
git init --bare blog.git
```

配置 git hook, 在 blog.git/hooks 目录下新建一个post-receive文件, 然后输入以下内容:

```bash
#!/bin/bash
git --work-tree=blog --git-dir=/git/blog.git checkout -f
```

然后添加权限

```bash
chmod +x blog.git/hooks/post-receive
```

**示例**

```bash
#!/bin/bash
git --work-tree=/www/wwwroot/hexo/blog --git-dir=/www/wwwroot/hexo/git/blog.git checkout -f
```

修改配置

修改config.yml, 绑定git仓库

```yaml
deploy:
  type: 'git'
  repo:
        github: github仓库地址 #可选
        local: 本地仓库地址
  branch: master
```

**示例**

```yaml
deploy:
  type: 'git'
  repo:
        github: https://xxxxx@github.com/jklash1996/jklash1996.github.io.git
        coding: https://xxxxx@e.coding.net/jklash/jklash-hexo/blog.git
        local: root@ipaddr:/www/wwwroot/hexo/git/blog
  branch: master
```

### 方法2: 使用GitHub Action自动部署

```yaml
name: Deploy Hexo

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with: 
          submodules: "true"
     
      - name: Install Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18.x"
     
      - name: Install Hexo
        run: |
          npm install hexo-cli -g
          npm install hexo --save
          npm install hexo-deployer-git --save
     
      - name: Install dependencies
        run: |
          npm install
          npm ci
     
      - name: Generate static files
        run: |
          hexo clean
          hexo generate
     
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.GH_TOKEN }}
          publish_dir: ./public
          external_repository: username/username.github.io
          publish_branch: main

```

## 5.常见错误

1. 如果在window下使用hexo, window下执行hexo出现错误

> hexo : 无法加载文件C:\Users\imwan\AppData\Roaming\npm\hexo.ps1，因为在此系统上禁止运行脚本。

**解决方法：**

cmd输入

```bash
set-ExecutionPolicy RemoteSigned
```

选择yes

## 6.ngnix缓存设置

```bash
location ~* \.(html|xml)$ {
        error_log /dev/null;
        access_log off;
        add_header Cache-Control no-cache;
    }
 
    location ~* \.(css|js|png|jpg|jpeg|gif|gz|svg|mp4|mp3|ogg|ogv|webm|htc|woff2|ico|woff|ttf)$ {
        error_log /dev/null;
        access_log off;
        add_header Cache-Control "public,max-age=864000";
    }
 
    location ~ .*\.(js|css)?$
    {
        error_log /dev/null;
        access_log off;
        add_header Cache-Control no-cache;
    }
```