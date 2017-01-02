---
layout: post
title: Go 中的 MySql 和 ORM
date: 2015-11-18 21:46:17 +0800
categories: Go
---

本篇文章主要内容是 Mysql 基础以及 Go 如何操作 MySql 数据库。

### Mysql 基础

在你的终端输入如下代码：

```shell
/usr/local/mysql/bin/mysql -u root -p
```

执行命令之后，输入你的 Mysql 数据库密码，成功之后显示：

```shell
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 19
Server version: 5.7.17 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

创建 test 数据库：

```shell
mysql> create database test;
Query OK, 1 row affected (0.00 sec)
```

显示所有存在的数据库：

```shell
mysql> show databases;
```

```shell
+--------------------+
| Database           |
+--------------------+
| information_schema |
| DB                 |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
6 rows in set (0.00 sec)
```

可以看到 test 数据库已经成功创建。接下来我们在 test 数据库创建两张表：userinfo 和 userdetail。首先切换到 test 数据库。我们先看看当前正在使用的数据库：

```shell
mysql> select database()
    -> ;
+------------+
| database() |
+------------+
| test       |
+------------+
1 row in set (0.00 sec)
```

切换操作：

```shell
mysql> use test;
Database changed
```

如上，我们就成功切换到 test 数据库中了，接下来我们创建两张表啦：

```shell
mysql> CREATE TABLE `userinfo` (
    ->         `uid` INT(10) NOT NULL AUTO_INCREMENT,
    ->         `username` VARCHAR(64) NULL DEFAULT NULL,
    ->         `departname` VARCHAR(64) NULL DEFAULT NULL,
    ->         `created` DATE NULL DEFAULT NULL,
    ->         PRIMARY KEY (`uid`)
    ->     );
mysql> CREATE TABLE `userdetail` (
    ->         `uid` INT(10) NOT NULL DEFAULT '0',
    ->         `intro` TEXT NULL,
    ->         `profile` TEXT NULL,
    ->         PRIMARY KEY (`uid`)
    ->     );
Query OK, 0 rows affected (0.02 sec)
```

查看表是否显示成功：

```shell
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| userdetail     |
| userinfo       |
+----------------+
2 rows in set (0.00 sec)
```

我们想看看所建表的结构的话：

```shell
mysql> desc userinfo;
+------------+-------------+------+-----+---------+----------------+
| Field      | Type        | Null | Key | Default | Extra          |
+------------+-------------+------+-----+---------+----------------+
| uid        | int(10)     | NO   | PRI | NULL    | auto_increment |
| username   | varchar(64) | YES  |     | NULL    |                |
| departname | varchar(64) | YES  |     | NULL    |                |
| created    | date        | YES  |     | NULL    |                |
+------------+-------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

向表中插入数据，并显示出来：

```shell
mysql> insert into userdetail VALUES ('2','Hello world', "I am allenuw");
Query OK, 1 row affected (0.00 sec)

mysql> select * from userdetail;
+-----+-------------+--------------+
| uid | intro       | profile      |
+-----+-------------+--------------+
|   2 | Hello world | I am allenuw |
+-----+-------------+--------------+
1 row in set (0.00 sec)
```

上面就是 Mysql 数据表最基本的操作了，还有很多要学习，其他的话大家 Google 一下就好了。接下来我们讲述 Go 中如何操作 Mysql 数据库。

### Golang 操作 Mysql

Go没有内置的驱动支持任何的数据库，但是Go定义了database/sql接口，用户可以基于驱动接口开发相应数据库的驱动。这一节来尝试与喜爱 mysql 的驱动

```shell
go get github.com/go-sql-driver/mysql
```

安装好 mysql 驱动之后，我们按照上述的步骤在终端创建两张表分别是：userinfo 和 userdetail。接下来看看 Go 官方提供的示例代码：

```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	//"time"
)

func main() {
	db, err := sql.Open("mysql", "root:123456@/test?charset=utf8")
	checkErr(err)
	//插入数据
	stmt, err := db.Prepare("INSERT userinfo SET username=?,departname=?,created=?")
	checkErr(err)
	res, err := stmt.Exec("astaxie", "WeChat", "2012-12-09")
	checkErr(err)
	id, err := res.LastInsertId()
	checkErr(err)
	fmt.Println(id)
	//更新数据
	stmt, err = db.Prepare("update userinfo set username=? where uid=?")
	checkErr(err)
	res, err = stmt.Exec("astaxieupdate", id)
	checkErr(err)
	affect, err := res.RowsAffected()
	checkErr(err)
	fmt.Println(affect)
	//查询数据
	rows, err := db.Query("SELECT * FROM userinfo")
	checkErr(err)

	for rows.Next() {
		var uid int
		var username string
		var department string
		var created string
		err = rows.Scan(&uid, &username, &department, &created)
		checkErr(err)
		fmt.Println(uid)
		fmt.Println(username)
		fmt.Println(department)
		fmt.Println(created)
	}
	//删除数据
	stmt, err = db.Prepare("delete from userinfo where uid=?")
	checkErr(err)
	res, err = stmt.Exec(id)
	checkErr(err)
	affect, err = res.RowsAffected()
	checkErr(err)
	fmt.Println(affect)
	db.Close()
}
// 存在 error 进行 painc
func checkErr(err error) {
	if err != nil {
		panic(err)
	}
}
```

输出结果如下所示：

```shell
[ `go run mysql_demo.go` | done: 345.222169ms ]
  1
  1
  1
  astaxieupdate
  WeChat
  2012-12-09
  1
```

#### Beego 进行 ORM

ORM 是一种对象关系的映射。接下来体验一下 Beego 中内部支持的 ORM。我们还是先来安装 beego 框架。

```go
go get github.com/astaxie/beego
```

```go
package main

import (
    "fmt"
    "github.com/astaxie/beego/orm"
    _ "github.com/go-sql-driver/mysql" // 导入数据库驱动
)

// Model Struct
type User struct {
    Id   int
    Name string `orm:"size(100)"`
}

func init() {
    // 设置默认数据库
    orm.RegisterDataBase("default", "mysql", "root:root@/my_db?charset=utf8", 30)

    // 注册定义的 model
    orm.RegisterModel(new(User))
    //RegisterModel 也可以同时注册多个 model
    //orm.RegisterModel(new(User), new(Profile), new(Post))

    // 创建 table
    orm.RunSyncdb("default", false, true)
}
```

其中第二个 `root`是你的本地 Mysql 数据库密码，而 my_db 就是你的数据库名称。下面是一些常用的操作：

```go
func main() {
    o := orm.NewOrm()
	// 基本的赋值
    user := User{Name: "slene"}

    // 插入表
    id, err := o.Insert(&user)
    fmt.Printf("ID: %d, ERR: %v\n", id, err)

    // 更新表
    user.Name = "astaxie"
    num, err := o.Update(&user)
    fmt.Printf("NUM: %d, ERR: %v\n", num, err)

    // 读取 one
    u := User{Id: user.Id}
    err = o.Read(&u)
    fmt.Printf("ERR: %v\n", err)

    // 删除表
    num, err = o.Delete(&u)
    fmt.Printf("NUM: %d, ERR: %v\n", num, err)
}
```

如上可见，我们基本上没有直接对 Mysql 数据库进行操作，而是通过一个实体类型来对数据库某个表进行各种操作。