---
layout: post
title: "Loopback - ACL (1)"
categories: jekyll update
date: 2015-03-13 16:49:30
tags: loopback, acl
---

#ACL

##General Process

控制管理的一般流程：
 1.  指定用户角色（User Role）
 2.  定义用户可执行的行为
 3.  执行验证（authentication）即代码实现


##Exponsing and hiding Models, methods and endpoints

--------------------------

这部份的主要内容是限制RESTful路由访问, Model 属性的控制。

--------------------------

如果想要把一个model的CRUD方法通过REST访问，只要把 `server/model-config.json` 中的*public*属性改为*true* 即可：

	"Photo":{
		"Role": ｛
			"dataSource": "db"，
			"public": true
		｝
	}

###Hide methods and REST endpoints
如果你想隐藏某些CRUD操作，不让用户通过RESTful api来进行操作，只需要在对应的model中调用 `disableRemoteMethod()`方法。

假设，我们想要隐藏`Photo`的删除endpoint, 那么，我们对应会去修改 `common/models/photo.js`：

	module.exports = function(Photo){
		var isStatic = true;
		Photo.disableRemoteMethod('deleteById', isStatic);
	}

	module.exports = function(Photo){
		var isStatic = false; //表明是此方法挂载在prototype原型上
		Photo.disableRemoteMethod('updateAttributes', isStatic);
	}

查看方法名称： [PersistedModel](http://apidocs.strongloop.com/loopback/#persistedmodel) 及 `loopback-datasource-juggler` 模块中的 `dao.js`

###Hide endpoints for related models

当不同的 models 之间存在关系时，loopback 会自动两个表之间的关系生成路由。例如， 一个 `User` 有多个 `Photo`,  那么，loopback 会为此关系自动生成如下路由：

	POST: /users/:id/photos --> create
	GET:  /users/:id/photos

如果想把相应的路由也隐藏掉，那么也可以使用 `disableRemoteMethod()`。如果想把以上的 `create` 路由隐藏，那么只需调用：`User.disableRemoteMethod('__create_photos__', false)` 即可。 其他相应的 `remote method` 名称可以查看[这里](http://docs.strongloop.com/display/public/LB/Restricting+access+to+related+models)。

###Hide Model properties
查看[这里](http://docs.strongloop.com/display/public/LB/Model+definition+JSON+file#ModeldefinitionJSONfile-Hiddenproperties)
