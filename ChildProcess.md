# exec()
exec()方法用于执行bash命令,它的参数是一个命令字符串
```JavaScript exec()1.0
var exec = require ('child_process').exec;
var ls = exec ('ls -l'),function(error,stdout,stderr) {
	if(error){
		console.log(error.stack);
		console.log('Error code:' + error.code);
	}
	console.log('Child Process STDOUT:' + stdout);
}
```
exec方法用于新建一个子进程,然后缓存它的运行结果,运行结束后调用回调函数
exec方法最大可以接受两个参数,第一个参数是所要执行的shell命令,第二个参数是回调函数,该函数接受三个参数,分别是发生的错误.标准输出的显示结果,标准错误的显示结果

```JavaScript 
exec()2.0
var exec = require('child_process').exec;
var child = exec('ls-l');
child.stdout.on('data',function(data){
	console.log('stdout:' + data);
});
child.stderr.on('data',function(data){
	console.log('stdout' + data);
})
child.on('close',function(code){
	console.log('closing code:' + code);
})
```
2.0代码监听data事件以后,可以实时输出结果,否则只要等到子进程结束,才会输出结果.若子进程运行时间较长,或者是持续运行,2.0写法更好

```
var exec = require ('child_process').exec;
exec('node -v',function(error,stdout,stderr){
	console.log('stdout:' + stdout);
	console.log('stderr:' + stderr);
	if(error !== null ) console.log('exec error:' + error);
})
```
# execSync()
同步执行版本,第一个参数是所要执行的命令,第二个参数用来配置执行环境
```
var SEPARATOR = process.platform === 'win32' ? ';':':';
var env = Object.assign({},process.env);

env.PATH = path.resolve('./node_modules/bin') + SEPARATOR + env.PATH;

function myExecSync(cmd) {
	var output = execSync(cmd,{
		cwd:process.cwd();
		env:env
	});
	console.log(output);
}
myExecSync('eslint .');
```



