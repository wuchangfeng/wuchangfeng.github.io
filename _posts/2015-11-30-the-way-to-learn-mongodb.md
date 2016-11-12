---
title: 非关系型数据库之 MongoDB 初探
date: 2016-05-07 18:42:54
tags: MongoDB
categories: About SQL
feature: 
---



使用 MongoDB 过程中的一些记录。

<!--more--> 



![mongodb_logo.jpg](http://7xrl8j.com1.z0.glb.clouddn.com/mongodb_logo.jpg)



在师兄的推荐下，开始使用 NoSQL 类型数据库。记录一下日常使用过程中一些基本的东西。主要的特点如下：

1. NoSQL，指的是非关系型的数据库。NoSQL 有时也称作Not Only SQL的缩写，是对不同于传统的关系型数据库的数据库管理系统的统称。
2. MongoDB 是一个基于**分布式**文件存储的数据库。
3. 主要用于超多规模型数据的存储，存储的数据量也非常大，因而其查询效率非常好，可以说为查询效率而生。

简单点说，NoSQL 简直为大数据而生。

大部分图标来自[菜鸟教程](http://www.runoob.com/mongodb/mongodb-create-database.html)

<!-- more --> 


### 一 . 安装 MongoDB

1. 在 windows 64 位的情况下，下载的版本为 mongodb-win32-x86_64-2008plus-ssl-3.2.6-signed.msi 

2. 自定义安装，安装在 D 盘新建的 Mongodb 文件夹中，并在该文件夹根目录下新建 /data/db 以及
   /log/log.txt 

3. 指定数据存储位置以及日志信息位置

   F:\MongoDB\bin>mongod.exe --logpath=F:\MongoDB\log\log.txt --dbpath=F:\MongoDB\data\db

4. 作为服务进行安装 

   F:\MongoDB\bin>mongod.exe --install --logpath=F:\MongoDB\log\log.txt --dbpath=F:\MongoDB\data\db

5. 启动命令以及终止命令，**必须以管理员身份启动**

   net start mongodb 
   net stop mongodb 

6. 进入 shellDB ，本质上为一个 javaScript shell


![](http://7xrl8j.com1.z0.glb.clouddn.com/mongodb.jpg)


​	 

### 二 . MongoDB 初窥

![](http://7xrl8j.com1.z0.glb.clouddn.com/mongodbtest1.jpg)

上面这个例子将 数字 10 插入到 runoob 集合的 x 字段中。
与关系型数据的对比

![](http://7xrl8j.com1.z0.glb.clouddn.com/mongodb1.jpg)

![](http://7xrl8j.com1.z0.glb.clouddn.com/mongodb2.jpg)

### 三 . MongoDB 用法

* **show dbs**, 可以显示所有数据库的列表
* **use db**,可以连接到一个指定的数据库,如果数据库不存在，则创建
* **db** 命令可以显示当前数据库对象或集合
* 删除数据库，先切换到指定数据库下，执行 db.dropDatabase()
* 插入文档 db.COLLECTION_NAME.insert(document) 而文档其实就是 json 数据
* 插入文档到集合 db.col.insert(document)
* 删除文档 db.col.remove({'title':'MongoDB 教程'}) 即移除 title 为 Mongodb 的一些文档数据
* **db.col.find().pretty()** ,格式化显示一系列 JSON 数据
* **show collections**,显示所有 documents，相当于 SQL 中的 table

### 四 . MongoDB 查询实例

查询了集合 col 中的数据：

``` python
> db.col.find().pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

MongoDB AND 条件

``` python
> db.col.find({"by":"菜鸟教程", "title":"MongoDB 教程"}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```
### 四 . 与 Java 相关

[Java Mongodb](http://www.runoob.com/mongodb/mongodb-java.html)

### 五 . 与 Python 相关

[Mongoengine 官方文档](http://docs.mongoengine.org/guide/querying.html)

### 六 . 与 API 设计相关

[GitHub Demo](https://github.com/mattbates/mycms_mongodb/blob/master/web.py)

[Flask 官网关于 API 的设计](http://docs.jinkan.org/docs/flask/index.html)