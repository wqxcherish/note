#readFile(),readFileSync()
readFile()用于异步读取数据,readFileSync()用于同步
```
fs.readFile('./image.png',function (err,buffer){
	if (err) throw err;
	process(buffer);
})
```
readFile方法的一个参数是文件的路径,可以是绝对路径,也可以是相对路径.若是相对路径,是相对于当前进程所在的路径(process.cwd()),而不是相对于当前脚本所在的路径
readFile方法的第二个参数是读取完成后的回调函数.该函数的第一个参数是发生错误是时错误对象,第二个参数是代表文件内容的buffer实例
```
var text = fs.readFileSync(fileName,'utf-8');
text.split(/\r?\n/).forEach(function (line){
	
})

```  
readFileSync 方法的第一个参数是文件路径,第二个参数可以是一个表示配置的对象,也可以是一个表示文本文件编码的字符串.
默认的配置对象是 { encoding:null, flag:'r'}, 即文件编码默认为null, 读取模式默认为 r (只读)
如果第二个参数不指定编码(encoding),readFileSync 方法返回一个Buffer实例,否则返回的是一个字符串

不同系统的行结尾字符不同,可以用下面的方法判断
```
var EOL = fileContents.indexOf('\r\n') >= 0 ? '\r\n' : '\n';

var EOL = (process.platform === 'win32' ? '\r\n' : '\n');
 ```

#writeFile(),writeFileSync()

```
fs.writeFile('message.txt','Hello Node.js',(err) => {
	if (err) throw err;
	console.log('It\'s saved');
})
```

fs.writeFileSync(fileName,str,'utf8');

writeFile方法的第一个参数是写入的文件名,第二个参数是写入的字符串,第三个参数是回调函数
回调函数前面,还可以再加一个参数,表示写入字符串的编码(默认是utf8)


#exists(path,callback)

#mkdir(),writeFile(),readFile()

#mkdirSync(),writeFileSync(),readFileSync()

#readdir(),readdirSync()

#stat()

#watchFile(),unwatchFile()

#createReadStream()

#createWriteStream()