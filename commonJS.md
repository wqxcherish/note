# CommonJS模块规范
- 每个文件就是一个模块,文件定义的变量,函数,类都是私有的,对其他文件不可见
- 如果想在多个文件分享变量,必须定义为global对象的属性
- 每个模块内部,module变量代表当前模块.这个变量是一个对象,其exports属性(module.exports)是对外的接口.(加载模块就是加载模块的module.exports属性)

## CommonJS模块的特点
 - 所有代码都运行在模块作用域,不会污染全局作用域
 - 模块可以多次加载,第一次加载之后运行结果就被缓存了,之后加载都读取缓存结果.若想模块再次运行,必须清除缓存
 - 模块加载的顺序,按照骑在代码中出现的顺序

## 模块中module对象的属性
 - module.id,module.filename
 - module.loaded 返回一个布尔值,表示模块是否已经完成加载
 - module.parent 返回一个对象,表示调用该模块的模块
 - module.children 返回一个数组,表示该模块要用到的其他模块
 - module.exports 表示模块对外输出的值

 判断一个函数是否为入口函数
 ```javascript
 if( !module.parent ) {
 	app.listen(8080,function(){
 		console.log('app listening on port 8080');
 	})
 } else {
 	module.exports = app;
 }
 ```

## exports 变量
Node 为每个模块提供一个exports 变量,指向module.exports.
注意:
  -不能直接将exports变量指向一个值,因为只有等于切换了exports与module.exports的联系
  -如果一个模块的对外接口就是一个单一的值,不能使用exports输出,只能使用module.exports输出

# AMD规范与CommonJS规范的兼容性
CommonJS规范加载模块是同步的,AMD规范则是非同步加载模块,允许制定回调函数.
由于Node.js主要用于服务器编程,模块文件一般已经存在于本地硬盘,所有加载起来比较快,不用考虑非同步加载的方式,所以CommonJS规范比较适用
如果是浏览器环境,要从服务器端加载模块,这是就必须采用非同步模式,因此浏览器端一般采用AMD规范

## AMD规范使用define方法定义模块
```JavaScript
define(['package/lib'],function(lib){
	function foo() lib.log("helloworld");
	return foo:foo;
});
```
## AMD规范允许输出的模块兼容CommonJS规范,这时define方法需要改写
```javascript
	define(function (require, exports,module){
		var someModule = require("someModule");
		var anotherModule = require("anotherModule");

		someModule.doTehAwesome();
		anotherModule.doMoarAwesome();

		exports.asplode = function (){
			someModule.doTehAwesome();
			anotherModule.doMoarAwesome();
		};
	})
```

# require命令
## 基本用法
- 读入并执行一个JavaScript文件,然后返回该模块的exports对象
- 若模块输出的是一个函数,那就不能定义在exports对象上面,而要定义在module.exports变量上

## 加载规则
1. 如果参数字符串以"/"开头,则表示加载的是一个位于绝对路径的模块文件.
2. 如果以"./"开通,则表示加载一个位于相对路径(跟当前执行脚本的位置相比)的模块文件
3. 如果参数字符不以"./"或"/"开头,则表示加载的是一个默认提供的核心模块,或者一个位于各级node_modules目录的已安装模块

举例来说,脚本/home/user/projects/foo.js执行了require('bar.js')命令,Node会依次搜索以下文件
*/usr/local/lib/node/bar.js
*/home/user/projects/node_modules/bar.js
*/home/user/node_modules/bar.js
*/home/node_modules/bar.js
*/node_modules/bar.js

4. 如果不以"./"或"/"开头,而且是一个路径,比如 require('example-module/path/to/file'),则将先找到example-module的位置,然后再以它为参数,找到后续路径
5. 如果指定的模块文件没有发现,Node会尝试为文件名添加.js,.json,.node后,再去搜索
6. 如果想得到require命令加载的确切文件名,使用require.resolve()方法

## 目录的加载规则
1. 为目录设置一个入口文件,让require方法可以通过这个入口文件,加载整个目录.
2. 在目录中放置一个package.json文件,并且将入口文件写入main字段.
3. require发现参数字符串指向一个目录以后,会自动查看该目录的package.json文件,然后加载面字段指定的入口文件.如果package.json文件没有字段,或者没有package.json文件,则会加载该目录下的index.js文件或index.node文件
 
## 模块的缓存
第一次加载某个模块时,Node会缓存该模块.以后再加载该模块,就直接从缓存去除该模块的module.exports属性
所有缓存的模块保存在require.cache之中,删除模块缓存的代码
```JavaScript
//删除指定模块的缓存
delete require.cache[moduleName];
//删除所有模块的缓存
Object.keys(require.cache).foreach(function(key){
	 delete require.cache[key];
})
```
注意,缓存是根据绝对路径识别模块的,如果同样的模块名,但是保存在不同的路径,require命令还是会重新加载该模块的

## 环境变量NODE_PATH
Node执行一个脚本时,会先查看环境变量NODE_PATH.
NODE_PATH是历史遗留下来的一个路径解决方案，通常不应该使用，而应该使用node_modules目录机制。

## 模块的循环加载
如果发生模块的循环加载,即A加载B,B又加载A,则B将加载A的不完整版本

## require.main
require方法有一个main属性,可以用来判断模块是直接执行,还是被调用执行
直接执行的时候,require.main属性指向模块本身
 
 require.main === module true为直接执行,false为调用执行

# 模块的加载机制
 CommonJS模块的加载机制是,输入的是被输出的值的拷贝

## require 的内部处理流程
 ```JavaScript
 Module._load = function(request, parent, isMain) {
  // 1. 检查 Module._cache，是否缓存之中有指定模块
  // 2. 如果缓存之中没有，就创建一个新的Module实例
  // 3. 将它保存到缓存
  // 4. 使用 module.load() 加载指定的模块文件，
  //    读取文件内容之后，使用 module.compile() 执行文件代码
  // 5. 如果加载/解析过程报错，就从缓存删除该模块
  // 6. 返回该模块的 module.exports
};
```
```JavaScript
Module.prototype._compile = function(content, filename) {
  // 1. 生成一个require函数，指向module.require
  // 2. 加载其他辅助方法到require
  // 3. 将文件内容放到一个函数之中，该函数可调用 require
  // 4. 执行该函数
};
```
require(): 加载外部模块
require.resolve()：将模块名解析到一个绝对路径
require.main：指向主模块
require.cache：指向所有缓存的模块
require.extensions：根据文件的后缀名，调用不同的执行函数

一旦require函数准备完毕，整个所要加载的脚本内容，就被放到一个新的函数之中，这样可以避免污染全局环境。该函数的参数包括require、module、exports，以及其他一些参数。
```JavaScript
(function (exports, require, module, __filename, __dirname) {
  // YOUR CODE INJECTED HERE!
});
```
Module._compile方法是同步执行的，所以Module._load要等它执行完成，才会向用户返回module.exports的值