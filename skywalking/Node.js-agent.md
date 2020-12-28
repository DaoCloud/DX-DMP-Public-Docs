# Node.js 探针接入

如果你的服务涉及到 Node.js，可以参考该文档。本文以 Node.js express应用为例讲解如何将 express 的endpoints 接入到分布式链路追踪。

## 前置条件

### 依赖包与环境安装

* 环境 Node.js 14.15.3

* Node.js agent module可以通过一下方式安装：

```bash
npm install skyapm-nodejs@latest --save
```

本文档示例中其他库安装：

```bash
//go2sky的gin插件
npm install mysql --save
npm install express --save
```

## 探针接入

```javascript
/*
参数说明：
@serviceName 服务名字, 以@结尾代表该服务所在 DMP 租户。
@instanceName 实例名字
@directServers SkyWalking后端收集器地址(使用gRPC 协议传输数据)
注意：
	在你使用其他module之前，必须先启动 skyapm-nodejs，否则无法收集数据
*/
require('skyapm-nodejs').start({
    serviceName: 'express-demo@devTenant',
    instanceName: 'test',
    directServers: '172.16.200.201:11800'
});
var mysql = require('mysql');
var express = require('express');
var app = express();

app.get('/user', function (req, res) {
  var connection = mysql.createConnection({
  host     : '172.0.0.0',
  user     : 'root',
  password : '123456',
  database : 'node.js'
});
  connection.connect();
  var myerror = "";
  /*
  用到的表：
  CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `message` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=latin1;
  */
  connection.query('SELECT * FROM user;', function (error, results, fields) {
		if(error){
          console.log('[SELECT ERROR] - ',error.message);
		  myerror = error;
          return;
        }
	   console.log('--------------------------SELECT----------------------------');
       console.log(results);
	   res.send(results);
       console.log('------------------------------------------------------------\n\n');
	   result = results;
	});
	if(myerror != ""){
		res.send('error!');
	}
    connection.end();
})
var server = app.listen(8081, function () {
  var host = server.address().address
  var port = server.address().port
  console.log("应用实例，访问地址为 http://%s:%s", host, port)
})

```

## 现在已经支持的module：

1. [Http](https://nodejs.org/api/http.html)
1. [Mysql](https://github.com/mysqljs/mysql)
1. [Egg](https://github.com/eggjs/egg)

在Node.js中，使用以上module，agent可以自动将其涉及到endpoints的接入到分布式链路追踪。