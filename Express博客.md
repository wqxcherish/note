```
var express = require('express');
var app = express();

//装配中间件
app.use('/',function (req,res){
	res.send('Hello Express');
})

app.listen(3000);
console.log('Server running at http://localhost:3000/')

module.exports = app;
```
app.use()沿用了Connect的方式,注册middleware
res.send()方法集成了设置Content-Type.和res.end()方法
module.exports方法导出app

# Application
帮助开发者配置web服务,常用方法:
- app.set(name,value)   //设置环境变量
- app.get(name)    //获取环境变量
- app.engine(ext,callback)  //设置模板引擎来渲染文件
- app.('html',require('ejs').renderFile)  //用ejs模板引擎来模板化HTML文件
- app.VERB(path,[callback...],callback) //定义一个或多个middleware
- app.route(path).VERB([callback...],callback) //定义一个或多个middleware,以及他们对于的web和路径
- app.param([name],callback)  //给一个请求所对应的方法绑定路由参数

# Request
处理http请求,常用变量/方法:
- req.query  //包含了query-string变量
- req.params  //包含了路径变量
- req.body  //检索请求body
- req.param(name) //检索一个请求参数值
- req.path,req.host,req.ip //请求的路径/主机/IP
- req.cookies  //和中间件cookieParser()一起使用,来检索用户发出的cookie

#Response
处理http响应,常用方法
- res.status(code) //设置响应HTTP状态码
- res.set(field[value])  //设置响应头HTTP header
- res.cookie(name,value,[option])  //设置响应cookie,option参数用来配置cookie
- res.redirect([status],url)  //再次定向请求一个新的URL,同时可以传递一个HTTP状态码,如果不设置,默认302
- res.send([body|status],[body])  //用于non-streaming响应.这个方法同时做了诸如设置Content-Type,Content-Length头和缓存头的操作
- res.json([status|body],[body])  //和res.send()一样,发送object或者array.多数时候用作语法糖,也有强制发送一个空json响应
- res.render(view,[locals],callback)  //渲染一个view或发出一个HTMl响应


#外部middleware
- Morgan //HTTP请求logger
- body-parser  //处理请求body,支持多种请求类型
- method-override  //支持HTTP verb 例如PUT DELETE
- Compression  //gzip/deflate 压缩相应数据
- express.static //处理静态文件
- cookie-parser  //包含于res.cookie实例
- Session 持久化session

# 项目文件结构
1. app  //存放Express应用逻辑的文件夹
 - controllers //存放应用的控制器
 - models   //存放model
 - routes   //存放路由中间件middleware
 - views    //存放界面view
2. config    //存放Express应用的配置文件,当需要加入更多的module的时候,每一个model的配置文件也放在这里
  - env      //存放环境配置文件
  - config.js //存放整个应用的配置
  - express.js //存放Express的初始化配置
3.  public //存放静态客户端文件
 - config //存放AngularJS应用的配置文件
 - controllers  //存放控制器
 - css
 - directives   //存放AngularJS的指示
 - filters     //存放AngularJS的过滤器
 - img
 - services
 - views       //存放AngulaJS的view
 - appliction.js   //用来初始化AngularJS应用
4. - server.js     //Node.js的main文件,用来加载express.js文件,是Express应用启动的文件
5. - package.json  //用来管理项目的依赖包

# 处理请求路由
```
app.get('/',function(res,req){
	res.send('This is a GET request')
});

app.post('/',function (res,req){
	res.send('This is POST request')
});
//支持通过定义一个路由链接一个或多个middleware的方式来实现处理多种方式请求
app.route('/').get(function(req,res){
	res.send('This is a GET request');
}).post(function(res,req){
	res.send('This is a POST request');
})
//middleware可以按照顺序被依次调用,如在执行请求前做验证
var express = require('express');
var hasName = function(req,res,next) {
	if(req.param('name')){  //检查是否有name参数
		next();
	}else{
 		res.send('What is your name?');
	}
}

var sayHello = function(req,res,next){
	res.send('Hello'+req.param('name'));
}

var app = express();
app.get('/',hasName,sayHello);
app.listen(3000);
console.log('Server is running at http://localhost:3000/');
```
# 在项目中加入路由文件
```
/* routes.js */
module.exports = function(app){
	var index =  require('../controllers/index.server.controller');
	app.get('/',index.render);  //将redner()方法当做一个middleware导入了根路径的GET请求
}

/* express.js */
var express = require('express');
module.exports = function (){
	var app = express();  //创建应用实例app
	//require路由文件并把实例app传入,路由文件使用app实例创建了路由配置并调用了controller的render方法
	require('../app/routes/index.server.routes')(app);  
	return (app);  通过返回app实例结束
}
/* server.js */
var express = require('./config/express');

var app = express();
app.listen(3000);
module.exports = app;
console.log('Server is running at http://localhost:3000/');

/* package.json */
{
	"name":"wqx",
	"version": "0.0.3",
     "dependencies": {
       "express": "~4.8.8",
       "morgan": "~1.3.0",
       "compression": "~1.0.11",
       "body-parser": "~1.8.0",
       "method-override": "~2.2.0"
    } 
} 
/* 引入module express.js*/
var express = require('express'),
	morgan = require('morgan'),    //简单的logger
	bodyParser = require('bodyParser'),  //一个response压缩
	methodOverride = require('method-override'),  //处理request数据
	compress = require('compression');  //提供DELETE,PUT等HTTP请求方式的处理
module.exports = function(){
	var app = express();
	if(process.env.NODE_ENV === 'development'){
		app.use(morgan('dev'));
	}else if(process.env.NODE_ENV === 'production'){
		app.use(compress());
	}
	app.use(bodyParser.urlencoded({
		extended:true
	}))
	app.use(bodyParser.json());
	app.use(methodOverride());
	require('../app/routes/index.server.routes')(app);
	return (app);
}
/* 引入module server.js*/
process.env.NODE_ENV = process.env.NODE_ENV || 'development'
var express = require ('./config/express');
var app = express();
app.listen(3000);
module.exports = app;
console.log('Server is running at http://localhost:3000/');
```
# 环境配置文件
```
//在config/env目录建立文件development.js
module.exports = {
	//Development configuration options
}
//修改配置加载,在config目录新建config.js文件
module.exports = require('./env/'+process.env.NODE_ENV+ '.js');
```
# 渲染界面
web框架的最基本功能是渲染界面,最基本的认知就是模板引擎提供数据,然后渲染成HTML.在mvc模式下,控制器使用模型把数据分配,通过界面模板渲染HTML
Express渲染界面的方法:
app.render()  //渲染界面将HTML发送给回调函数
res.render()  //本地渲染界面.发送HTML作为请求的响应

在使用渲染方法之前,首先需要配置界面系统,在package.json中加入依赖包,"ejs":"~1.0.0" 然后使用$npm update加入ejs模块.安装ejs后,在
config/express.js中增加ejs的配置

安装ejs后,在config/express.js文件中增加ejs的配置
```
var express = require('express'),
    morgan = require('morgan'),
    bodyParser = require('body-parser'),
    methodOverride = require('method-override'),
    compress = require('compression');
module.exports = function () {
    var app = express();

    if (process.env.NODE_ENV === 'development'){
        app.use(morgan('dev'));
    }else if(process.env.NODE_ENV == 'production'){
        app.use(compress());
    }

    app.use(bodyParser.urlencoded({
        extended: true
    }));

    app.use(bodyParser.json());
    app.use(methodOverride());

    //ejs
    app.set('views','./app/views');
    app.set('view engine','ejs');

    require('../app/routes/index.server.routes')(app);
    return(app);
};
```
# 渲染EJS view
EJS间基本是由HTML和EJS标签组成的.EJS模板将会跳转app/views文件夹,会有.ejs扩展.当你使用res.render()方法时,EJS引擎将会在views文件夹寻找模板,如果找到了就渲染HTML输出

创建第一个EJS view 
```
/* index.ejs*/
<!DOCTYPE html>
<html>
<head>
	<title><%= title %></title>
</head>
<body>
	<!--告诉ejs模板引擎来渲染模板变量,上例子中是title变量.接下来需要配置控制器自动渲染模板输出HTML响应 -->
	<h1><%= title %></h1> 
</body>
</html>
/* controller.js */
exports.render = function (req,res) {
	res.render('index',{
		title:'Hello EJS'
	})
}
```
# 保存静态文件
Express提供了express.static()  middleware来处理静态文件.
app.use(express.static('./public')); //告诉了expres静态文件存放的路径,注意要把这段代码放到路由之后,因为放在路由之前的express会先从HTTP请求path中寻找静态文件,这样会让性能变慢

# 处理Session
导入express-session包
express-session模块:
-cookie-stored  
-signed identifier
   要标记session identifier 需要用到secret string 来预防而已session干预.
   为了安全,cookie secret 在不同的环境需要不同得配置文件
```
module.exports = {
	//Development configuration options
	sessionSecret:'developmentSessionSecret'
}
```






