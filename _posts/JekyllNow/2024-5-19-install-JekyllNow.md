---
layout: post
title: 【Jekyll】Install JekyllNow
categories: [Jekyll]
---

## 1. How to Build a Personal Blog

Refer to this guide:

- [Learn How to Build Your Personal Blog with GitHub Pages](https://dev.to/alagrede/lean-how-to-build-your-personal-blog-with-github-pages-42ae)

## 2. Build Steps

- Since I cannot always code directly on the GitHub web page, I clone the repository.
- After cloning, I need to run the server locally. Refer to the [official readme](https://github.com/barryclark/jekyll-now/blob/master/README.md).
- Command: `gem install github-pages`

## 3. Comment System

- References:
  - Recommended: [Jekyll Comment System Using GitHub Issues](https://www.aleksandrhovhannisyan.com/blog/jekyll-comment-system-github-issues/)
  - [Adding Comments to a Static Site](https://medium.com/@raravi/adding-comments-to-a-static-site-31506e77fc41)

## 100. Problem Records

### 100.1 Problem 1: Package Issues

```
ERROR:  Error installing github-pages:
        The last version of drb (>= 0) to support your Ruby & RubyGems was 2.0.6. Try installing it with `gem install drb -v 2.0.6` and then running the current command again
        drb requires Ruby version >= 2.7.0. The current ruby version is 2.6.10.210.
```

Unfortunately, I don't have a definitive solution, so I tackled the issues one by one...

Some documentation:

- [Update ruby](https://stackoverflow.com/questions/38194032/how-can-i-update-ruby-version-2-0-0-to-the-latest-version-in-mac-os-x-v10-10-yo)

Some commands:

```
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/\
gem install faraday-net_http -v 3.0.2
sudo gem install faraday-net_http -v 3.0.2
sudo gem install faraday -v 2.8.1
sudo gem install drb -v 2.0.6
sudo gem install activesupport -v 6.1.7.7
sudo gem install -V github-pages
```
