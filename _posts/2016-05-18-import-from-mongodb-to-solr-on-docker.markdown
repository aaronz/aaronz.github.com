---
layout: post
title:  "docker实战：mongodb数据导入solr"
date:   2016-05-18 00:00:00
categories: Tools
---

最近遇到一个需求是要通过solr将mongodb中的数据index，提供搜索的支持。原始数据是mongodb的一个导出文件，要自己搭建solr和mongodb的环境将数据导入。为了避免环境配置的麻烦，于是采用了docker image来完成这项工作。

# docker环境准备
首先是准备docker环境。我使用的是ubuntu 14.04。然后通过下面脚本安装docker。
[https://github.com/aaronz/vm_setup/blob/master/tools/ubuntu/install_docker.sh](https://github.com/aaronz/vm_setup/blob/master/tools/ubuntu/install_docker.sh)

安装好之后执行version命令得到如下输出。

    administrator@ubuntu:~$ docker version
    Client:
    Version:      1.11.1
    API version:  1.23
    Go version:   go1.5.4
    Git commit:   5604cbe
    Built:        Tue Apr 26 23:30:23 2016
    OS/Arch:      linux/amd64

    Server:
    Version:      1.11.1
    API version:  1.23
    Go version:   go1.5.4
    Git commit:   5604cbe
    Built:        Tue Apr 26 23:30:23 2016
    OS/Arch:      linux/amd64


# 安装mongodb
docker安装成功之后就可以从dockerhub上拉取各种image为我所用。先来个mongodb的offical包运行起来。

    docker run -p 27017:27107 --name my_mongo -v /home/admin:/home/hostadmin -d mongo --replSet "rs0"

这里用到的几个参数
 - -p 指定container和host暴露的端口对应关系
 - -v 指定container和host文件目录的映射关系，这里将host的/home/admin目录映射给container中的/home/hostadmin,后续只需要将备份的数据放在host的/home/admin下面，在container里面就可以访问到。
 - -d 指定执行完命令后detach
 - mongo 指定了拉取的image名称，这里会根据mongo这个名字找到对应的官方镜像
 - --replSet "rs0"指定将mongo启动为一个replicateSet，这是后需要用到的mongo-connector所需要的。 
 - --name 指定了container的名称。后续可以通过名称直接引用container。

## 导入数据
接下来就是在mongodb中导入数据。首先将数据拷贝到host的/home/admin目录下，我们这里的数据文件名叫arch.dat。导入数据到mongodb需要用到mongo自带的mongoimport命令，这时我们就需要进入到mongo container去执行命令。

通过下面命令进入mongo container的bash。

    docker exec -it my_mongo /bin/bash

在mongo container的bash里面我们首先要初始化mongo的replicate set。通过mongo命令启动mongo shell。然后执行下面命令。

    rs.initiate() # 初始化
    rs.conf()     # 校验配置

退出mongo shell然后导入数据。

    mongoimport -d test -c arch /home/hostadmin/arch.dat
    
 - -d 指定了数据库为test
 - -c 指定了collection的名称
 

# 安装solr

接下来搭建solr环境，直接拉下来solr的官方镜像即可。就免去了自己装各种依赖的痛苦。不过这里因为我们要改写配置solr配置文件，所以还是要将host目录映射到solr container内部，这样方便修改和保存。

    # create a directory to store the server/solr directory
    $ mkdir /home/admin/mysolr1

    # make sure its host owner matches the container's solr user
    $ sudo chown 999:999 /home/admin/mysolr1

    # copy the solr directory from a temporary container to the volume
    $ docker run -it --rm -v /home/admin/mysolr1:/target makuk66/docker-solr cp -r server/solr /target/

    # pass the solr directory to a new container running solr
    $ SOLR_CONTAINER=$(docker run -d -P -v /home/admin/mysolr1/solr:/opt/solr/server/solr makuk66/docker-solr)

    

这是在外部访问host的8983端口即可得到以下管理界面。

![solr-admin](../assets/images/posts/solr-admin.png)
    
solr启动之后要创建一个core，solr中每一个core可以对应一个搜索应用。

    # create a new core
    $ docker exec -it --user=solr $SOLR_CONTAINER bin/solr create_core -c arch

    # check the volume on the host:
    $ ls /home/admin/mysolr1/solr/
    configsets  arch  README.txt  solr.xml  zoo.cfg

## 修改配置

这样在host的/home/admin/mysolr1/solr/arch/conf目录下就有了arch对应的配置文件。这里我们要对配置文件进行相应的更改以便mongo-connector接入数据。

首先修改solrconfig.xml文件添加以下handler

    <requestHandler name="/admin/luke" class="org.apache.solr.handler.admin.LukeRequestHandler" />

然后修改managedschema文件，添加对应的字段

    <field name="_ts" type="long" indexed="true" stored="true" />
    <field name="ns" type="string" indexed="true" stored="true"/>

同时还要修改id的对应关系，因为mongodb里面用的是_id，所以要将以下两处改为_id。
    
    <field name="_id" type="string" indexed="true" stored="true" />
    <uniqueKey>_id</uniqueKey>
    
最后是添加mongodb表中定义的字段。
    
    <field name="body" type="string" indexed="true" stored="true"/>
    <field name="title" type="string" indexed="true" stored="true"/>
    <field name="url" type="string" indexed="true" stored="true"/>

# 安装mongo-connector

首先要安装python和pip。

    sudo add-apt-repository ppa:fkrull/deadsnakes
    sudo apt-get update
    sudo apt-get install python2.7

接下来就是安装mongo-connector，
    
    pip install mongo-connector

# solr导入数据

接下来就是执行命令导入数据。

    mongo-connector -m localhost:27017 -t http://localhost:8983/solr/arch -d solr_doc_manager

# 测试

命令执行之后在solr的管理界面上是看不到index有变化的，这里可以通过管理界面另外提交一个文档。

![solr-doc-submit](../assets/images/posts/solr-doc-submit.png)

这时候再看index就可以看到对应的变化了。此时通过管理界面执行查询也可以得到相应的结果。

![solr-query](../assets/images/posts/solr-query.png)


# 参考链接

- [mongo-connector](https://github.com/mongodb-labs/mongo-connector/wiki/Usage%20with%20Solr)
- [docker-solr](https://github.com/makuk66/docker-solr/blob/master/Docker-FAQ.md)
