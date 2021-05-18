# Node.js Probe access

If your service involves Node.js, you can refer to this document. This article uses a Node.js express application as an example to explain how to connect express's endpoints to distributed link tracing.

## Prerequisites

### Dependency packages and environment installation

* Environment Node.js 14.15.3

* Node.js agent module can be installed in the following way：

```bash
npm install skyapm-nodejs@latest --save
```

Other library installations in this document example：

```bash
//go2sky gin plugin
npm install mysql --save
npm install express --save
```

## Probe access

```javascript
/*
Parameter description.
@serviceName Service name, Rule: tenantCode::namespace(K8S)::serviceName, via :: link.
@instanceName Instance name
@directServers SkyWalking backend collector address (uses gRPC protocol to transfer data)
Caution.
	You must start skyapm-nodejs before you can use other mods, otherwise you cannot collect data
*/
require('skyapm-nodejs').start({
    serviceName: 'devTenant::dmp-t::express-demo',
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
  Table used：
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
  console.log("Application instance, accessed at http://%s:%s", host, port)
})

```

## Mods that are now supported：

1. [Http](https://nodejs.org/api/http.html)
1. [Mysql](https://github.com/mysqljs/mysql)
1. [Egg](https://github.com/eggjs/egg)

In Node.js, using the above mods, the agent can automatically connect its access to the distributed link trace involving endpoints.