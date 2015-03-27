---
layout: post
title: "Mongodb 用户身份管理"
categories: jekyll update
date: 2015-03-27 9:49:30
---

#MongoDB Security


##Generate a key file

1.  生成密钥文件

        openssl rand -base64 741 > mongodb-keyfile

        chmod 600 mongodb-keyfile

2.  启动Mongodb进程时，指定密钥路径。此时，常规登入将不在有效 (*`localhost exception`*)，需提供身份认证信息。

        mongod --dbpath ./data/db --keyFile /privat/var/key.pem

##create a user Admin
在 `admin` 数据库中创建管理员，`usrAdminAnyDatabase` 表明该管理员可以对任何数据库进行管理。

    use admin;
    db.createUser(
      {
        user: "siteUserAdmin",
        pwd: "password",
        roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
      }
    )


##Add a user to a database
添加新用户，需要以合适的身份登入才行，比如用户管理员。添加的用户是在哪个DB中添加的就属于哪个DB, 不同 DB 可以有一样的用户。下例中生成的新用户只属于  `reporting` 数据库。



    use reporting
    db.createUser(
        {
          user: "reportsUser",
          pwd: "12345678",
          roles: [
             { role: "read", db: "reporting" },
             { role: "read", db: "products" },
             { role: "read", db: "sales" },
             { role: "readWrite", db: "accounts" }
          ]
        }
    )

##Assign a user a role

当创建完用户后，如若想对用户援于读写的权利，通过如下方法创建。

    db.grantRolesToUser('neil', [
    	{role: "readWrite", db: "test"},
    	{role: "read", db: "admin"}
    ])

