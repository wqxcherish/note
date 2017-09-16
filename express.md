#express

##express()
```JavaScript
var express = require('express');
var app = express();
```
##Methods
express.static(root,[options])
express.static是Express中唯一的内建中部件.它以server-static模块为基础开发,负责托管Express应用内的静态资源
参数root为静态资源的所在的根目录
参数options是可选的,支持以下的属性:dotfiles,etag,extensions,index,lastModified,maxAge,redirect,setHeaders


##Application()
```JavaScript
var express = require('express');
var app = express();

app.get('/',function(req,res){
	res.send('hello world');
})

app.listen(3000);

app对象具有以下方法:路由HTTP请求,配置中间件,渲染HTML视图,注册模板引擎
```
##Properties

###app.locals 对象是一个JavaScript对象,它的属性就是程序本地的变量,一旦设定,app.locals的各属性值将贯穿程序的整个生命周期,与其相反的是
res.locals,它只在这次请求的生命周期中有效

###app.mountpath
属性是子程序过载的路径模式
```
var express = require('express');
var app = express();
var admin = express();

admin.get('/',function(req,res){
	console.log(admin.mountpath); //admin
	res.send("Admin Homepage");
})
```
app.use('/admin',admin); //app mount the admin
它和req对象的baseUrl属性比较相似,除了req.baseUrl是匹配的URL路径,而不是匹配的模式.如果一个子程序被挂载在多条路径模式,
app.mountpath就是一个关于挂载路径模式项的列表

```JavaScript
var admin = express();
admin.get('/',function(req,res){
	console.log(admin.mountpath);
	res.send('Admin Homepage');
})

var secret = express();
secret.get('/',function(req,res){
	console.log(secret.mountpath);
	res.send('Admin secret');
})
 
admin.use('/secr*t',secret); //load the 'secret' router on '/secr*t',on the 'admin' sub app
app.use(['/adm*n','/manager'],admin); //load the 'admin' router on '/adm*n' and '/manager' on the parent app
```
##Events
app.on('mount',callback(parent))
当子程序被挂载到父程序时,mount事件被发射.父程序对象作为参数,传递给回调方法
```
var admin = express();
admin.on('mount',function(parent){
	 console.log('Admin Mounted');
	 console.log(parent); // refers to the parent app
});
admin.get('/',function (req,res){
	res.send('Admin Homepage');
});
app.use('/admin',admin);

Methods
app.all(path,callback[,callback...])
```