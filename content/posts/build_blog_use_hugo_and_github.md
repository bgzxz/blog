---
title: "使用hugo和github搭建个人blog"
date: 2018-01-01T14:11:54+08:00
draft: false
---
# hugo简介
golang实现的高性能静态站点生辰工具

更多细节见[官方网站](https://gohugo.io/)
# 安装hugo
 Homebrew (macOS)

{{< highlight bash >}}
brew install hugo
{{< /highlight >}}
 源码安装
{{< highlight bash >}}
git clone https://github.com/gohugoio/hugo.git
cd hugo
go install --tags extended
{{< /highlight >}}

更多安装方式见[官方文档](https://gohugo.io/getting-started/installing/)

# 验证安装

  执行hugo version,显示版本号表示安装成功

# 创建一个site
{{< highlight bash >}}
hugo new site quickstart
{{< /highlight >}}

# 添加主题
{{< highlight bash >}}
cd quickstart;
git init;
git submodule add https://github.com/aos/temple.git themes/temple;

# Edit your config.toml configuration file
# and add the temple theme.
echo 'theme = "temple"' >> config.toml
{{< /highlight >}}
# 添加文章
{{< highlight bash >}}
hugo new posts/my-first-post.md
{{< /highlight >}}
# 启动本地服务并验证页面
{{< highlight bash >}}
hugo server -D

Start Hugo dev server...

                   | EN
+------------------+----+
  Pages            | 11
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  1
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Total in 26 ms
Watching for changes in /Users/bgxzx/my-site/blog/{content,data,layouts,static,themes}
Watching for config changes in /Users/bgxzx/my-site/blog/config.toml
Environment: "development"
Serving pages from /Users/bgxzx/my-site/blog/dev-public
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
{{< /highlight >}}

# 使用github部署站点
1. 在GitHub创建一个仓库用来保存站点的源码  (e.g. `blog`) 。

2. 创建 `<USERNAME>.github.io` 这样的仓库用来发布Hugo生成的静态站点。

3. `git clone <YOUR-PROJECT-URL> && cd <YOUR-PROJECT>`。

4. 确保站点本地可以工作 (`hugo server` or `hugo server -t <YOURTHEME>`)。

5. 结束hugo server同时删除hugo生成的public目录。

6. 添加github站点仓库为子public模块 `git submodule add -b master git@github.com:<USERNAME>/<USERNAME>.github.io.git public`。

# 创建自动化生成和部署脚本
{{< highlight bash >}}
    #!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..
{{< /highlight >}}

访问`<USERNAME>.github.io`验证效果

更多部署方式见[官网](https://gohugo.io/hosting-and-deployment/)