# child_process 子进程

在node中，child_process这个模块非常重要。掌握了它，等于在node的世界开启了一扇新的大门。熟悉shell脚本的同学，可以用它来完成很多有意思的事情，比如文件压缩、增量部署等，nodejs创建子进程有四种方法,分别是spawn、fork、exec、execFile。

### 区别 ：

1. **格式** : spawn和execFile的格式都是(command,[args])；fork的参数直接(文件名);exec的command相当于spawn的command+args；

2. **回调** : spawn和fork没有直接的回调。spawn通过事件监听处理; fork相当于直接执行一个node程序;其余两个有回调,回调的参数为error,stdout,stderr;

3. **作用** : [这里我也不是很明白,引用网上的],fork用于启动一个node进程,可以进程进程之间通信;execFile用于执行一个外部应用;spawn方法会在新的进程执行外部应用;exec这个方法将会生成一个子shell，能够在shell中执行命令。

   

### spawn `child_process.spawn(command, [args], [options])`

　　* command 命令指的是windows或者linux系统命令,如果报错`spawn xx ENOENT`则指在windows系统运行linux命令，或者相反。
　　* 这里的args是选填，有些命令需要加参数，比如`cat a.txt`,则格式为spawn('cat',['a.txt'])；有些命令不需要参数,比如`ls`，则直接spawn('ls'),一些辅助命令如'-a','-m'等也放到数组中;

```javascript
//windows系统下的命令;
var spawn = require('child_process').spawn,
    free  = spawn('cat', ['a.txt'],{cwd:'./a'});

// 捕获标准输出并将其打印到控制台
free.stdout.on('data', function (data) {
    console.log('标准输出：' + data);
});

// 捕获标准错误输出并将其打印到控制台
free.stderr.on('data', function (data) {
    console.log('标准错误输出：' + data);
});

// 注册子进程关闭事件
free.on('exit', function (code, signal) {
    console.log('子进程已退出，代码：' + code);
});
----------
//同步的spawn;
var spawn = require('child_process').spawnSync('cat',['a.txt']);
console.log(spawn.stdout.toString());
```


