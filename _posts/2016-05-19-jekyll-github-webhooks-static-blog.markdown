---
layout: post
title:  "Github Webhooks + jekyll-hook即时更新jekyll博客"
date:   2016-05-19 00:00:00
categories: jekyll
---

越来越多的技术博客开始使用jekyll来生成静态网站文件了。如果每次文章更新都需要手动去编译markdown到静态文件就太烦了，本文介绍一种常用的自动化编译方法。使用github的webhooks和jekyll-hook来协同完成。

![jekyll](../assets/images/posts/jekyll.png)

<!--more-->


# 安装docker

首先安装git，然后clone [vm_setup repro](https://github.com/aaronz/vm_setup.git)
设置安装脚本的执行权限

    chmod +x chmod +x vm_setup/tools/ubuntu/install_docker.sh

执行安装脚本

    ~/vm_setup/tools/ubuntu/install_docker.sh

安装好之后执行docker version命令可以得到类似以下输出。

    administrator@ubuntu:~$ docker version
    Client:
    Version:      1.11.1
    API version:  1.23
    Go version:   go1.5.4
    Git commit:   5604cbe
    Built:        Tue Apr 26 23:30:23 2016
    OS/Arch:      linux/amd64


# 启动jekyll

jekyll也在docker中提供了官方镜像。可以通过以下命令拉取，

    docker run -p 80:4000 -v /home/admin:/home/host jekyll/jekyll
    
这里我们用 -p 参数将jekyll 4000端口映射到host的80端口，然后将host的admin目录映射为container的/home/host目录。

# 创建博客

这里直接从git里面拉取一些测试文章。

    git clone https://github.com/ivyzhcl/ivyzhcl.github.com.git
    
在/home/admin/ivyzhcl.github.com目录下就会有了以下文件。

    _posts       copyrights   sample-page

接下来进入container的bash

    sudo docker exec -it 5410 /bin/bash #5410 is the container id

将host中的博客文件拷贝到jekyll的目录下

    cp -r /home/host/* .

# 测试访问

从host外部访问host的80端口，可以显示页面如下。

![jekyll-blog](../assets/images/posts/jekyll-blog-01.png)

在container jekyll/_sites目录下可以看到我们刚刚拷贝进去的markdown博文被编译成了html页面。

    root /srv/jekyll → ls _site/
    2013           2014           copyrights     sample-page    uncategorized  原创           拾贝           经典

# 配置jekyll-hook

jekyll-hook的作用是接收github的hook事件，然后pull最新的github博客更新，然后将jekyll的文章更新。

所以我们要将jekyll-hook安装到jekyll的container内部。具体步骤如下。

    npm install -g forever
    git clone https://github.com/developmentseed/jekyll-hook.git
    cd jekyll-hook
    npm install
    cp config.sample.json config.json
    vi config.json
    
修改config.json文件如下

    {
        "gh_server": "github.com",
        "temp": "/srv/jekyll-hook", #此处修改为本地存放拉取github博客repro的临时目录
        "public_repo": true,
        "scripts": {
        "#default": {
            "build": "./scripts/build.sh",
            "publish": "./scripts/publish.sh"
        }
        },
        "secret": "",
        "email": {
            "isActivated": false,
            "user": "",
            "password": "",
            "host": "",
            "ssl": true
        },
        "accounts": [
            "ivyzhcl"   #此处修改为git发送通知的账号
        ]
    }

修改jekyll-hook/scripts/publish.sh文件，这个文件配置了本地服务器文件目录。默认使用的是nginx的服务器目录，可以将其修改为container的/srv/jekyll/目录
    
    # Set the path of the hosted site
    site="/srv/jekyll"

# 启动 forever jekyll-hook

通过forever启动jekyll-hook服务，让其在后台一直接收github的webhook请求。

    forever start jekyll-hook.js

验证一下

    $: forever list
    info:    Forever processes running
    data:        uid  command         script         forever pid  logfile                        uptime
    data:    [0] ZQMF /usr/bin/nodejs jekyll-hook.js 4166    4168 /home/ubuntu/.forever/ZQMF.log 0:0:1:22.176
    $: forever stop 0

# 配置 github webhook

Github webhook是在repro有更新的时候通知第三方的一种机制。原理就是触发一个http的请求。配置webhook直接到repro的setting页面。实例如下。

![github webhook](../assets/images/posts/github-webhooks.png)

这样在github中修改任何文件服务器都会接受到消息重新拉取更新，然后你的前台就可以自动随着github push更新了。


# 参考文章

- [jekyll](http://jekyllrb.com/)
- [jekyll-hook](https://github.com/developmentseed/jekyll-hook)
- [jekyll-docker](https://hub.docker.com/r/jekyll/jekyll/~/dockerfile/)
