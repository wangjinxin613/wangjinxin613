---
title: child_process子进程
date: 2024-05-05 14:52:32
tags:
	- nodejs
---
# child_process 子进程

在node中，child_process这个模块非常重要。掌握了它，等于在node的世界开启了一扇新的大门。熟悉shell脚本的同学，可以用它来完成很多有意思的事情，比如文件压缩、增量部署等，nodejs创建子进程有四种方法,分别是spawn、fork、exec、execFile。

### 区别 ：

1. **格式** : spawn和execFile的格式都是(command,[args])；fork的参数直接(文件名);exec的command相当于spawn的command+args；
2. **回调** : spawn和fork没有直接的回调。spawn通过事件监听处理; fork相当于直接执行一个node程序;其余两个有回调,回调的参数为error,stdout,stderr;
3. **作用** : [这里我也不是很明白,引用网上的],fork用于启动一个node进程,可以进程进程之间通信;execFile用于执行一个外部应用;spawn方法会在新的进程执行外部应用;exec这个方法将会生成一个子shell，能够在shell中执行命令。

### child_process.exec(command[, options][, callback])

创建一个shell，然后在shell里执行命令。执行完成后，将stdout、stderr作为参数传入回调方法。                                     

例子如下：

1. 执行成功，`error`为`null`；执行失败，`error`为`Error`实例。`error.code`为错误码，
2. `stdout`、`stderr`为标准输出、标准错误。默认是字符串，除非`options.encoding`为`buffer`

```javascript
var exec = require('child_process').exec;

// 成功的例子  执行了命令ls -al
exec('ls -al', function(error, stdout, stderr){
    if(error) {
        console.error('error: ' + error);
        return;
    }
    console.log('stdout: ' + stdout);
    console.log('stderr: ' + typeof stderr);
});

// 失败的例子
exec('ls hello.txt', function(error, stdout, stderr){
    if(error) {
        console.error('error: ' + error);
        return;
    }
    console.log('stdout: ' + stdout);
    console.log('stderr: ' + stderr);
});
```

#### 参数说明：

- `cwd`：当前工作路径。
- `env`：环境变量。
- `encoding`：编码，默认是`utf8`。
- `shell`：用来执行命令的shell，unix上默认是`/bin/sh`，windows上默认是`cmd.exe`。
- `timeout`：默认是0。
- `killSignal`：默认是`SIGTERM`。
- `uid`：执行进程的uid。
- `gid`：执行进程的gid。
- `maxBuffer`： 标准输出、错误输出最大允许的数据量（单位为字节），如果超出的话，子进程就会被杀死。默认是200*1024（就是200k啦）

备注：

1. 如果`timeout`大于0，那么，当子进程运行超过`timeout`毫秒，那么，就会给进程发送`killSignal`指定的信号（比如`SIGTERM`）。
2. 如果运行没有出错，那么`error`为`null`。如果运行出错，那么，`error.code`就是退出代码（exist code），`error.signal`会被设置成终止进程的信号。（比如`CTRL+C`时发送的`SIGINT`）

### child_process.execFile(file[, args][, options][, callback])

跟`.exec()`类似，不同点在于，没有创建一个新的shell。至少有两点影响

1. 比`child_process.exec()`效率高一些。（实际待测试）
2. 一些操作，比如I/O重定向，文件glob等不支持。

### child_process.fork(modulePath[, args][, options])

`modulePath`：子进程运行的模块。

参数说明：（重复的参数说明就不在这里列举）

- `execPath`： 用来创建子进程的可执行文件，默认是`/usr/local/bin/node`。也就是说，你可通过`execPath`来指定具体的node可执行文件路径。（比如多个node版本）
- `execArgv`： 传给可执行文件的字符串参数列表。默认是`process.execArgv`，跟父进程保持一致。
- `silent`： 默认是`false`，即子进程的`stdio`从父进程继承。如果是`true`，则直接`pipe`向子进程的`child.stdin`、`child.stdout`等。
- `stdio`： 如果声明了`stdio`，则会覆盖`silent`选项的设置。

```javascript
var child_process = require('child_process');

var child = child_process.fork('./child.js', {
  silent: true,
  execArgv: process.execArgv
});

child.stdout.setEncoding('utf8');

child.stdout.on('data', function(data){
    console.log(data);
});
```

child.js

```javascript
console.log('output from another silent child');
```

## exec()与execFile()之间的区别

首先，exec() 内部调用 execFile() 来实现，而 execFile() 内部调用 spawn() 来实现。

> exec() -> execFile() -> spawn()

其次，execFile() 内部默认将 options.shell 设置为false，exec() 默认不是false。

### 各种事件

### close

当stdio流关闭时触发。这个事件跟`exit`不同，因为多个进程可以共享同个stdio流。
参数：code（退出码，如果子进程是自己退出的话），signal（结束子进程的信号）
问题：code一定是有的吗？（从对code的注解来看好像不是）比如用`kill`杀死子进程，那么，code是？

### exit

参数：code、signal，如果子进程是自己退出的，那么`code`就是退出码，否则为null；如果子进程是通过信号结束的，那么，`signal`就是结束进程的信号，否则为null。这两者中，一者肯定不为null。
注意事项：`exit`事件触发时，子进程的stdio stream可能还打开着。（场景？）此外，nodejs监听了SIGINT和SIGTERM信号，也就是说，nodejs收到这两个信号时，不会立刻退出，而是先做一些清理的工作，然后重新抛出这两个信号。（目测此时js可以做清理工作了，比如关闭数据库等。）

SIGINT：interrupt，程序终止信号，通常在用户按下CTRL+C时发出，用来通知前台进程终止进程。
SIGTERM：terminate，程序结束信号，该信号可以被阻塞和处理，通常用来要求程序自己正常退出。shell命令kill缺省产生这个信号。如果信号终止不了，我们才会尝试SIGKILL（强制终止）。

> Also, note that Node.js establishes signal handlers for SIGINT and SIGTERM and Node.js processes will not terminate immediately due to receipt of those signals. Rather, Node.js will perform a sequence of cleanup actions and then will re-raise the handled signal.

### error

当发生下列事情时，error就会被触发。当error触发时，exit可能触发，也可能不触发。（内心是崩溃的）

- 无法创建子进程。
- 进程无法kill。（TODO 举例子）
- 向子进程发送消息失败。（TODO 举例子）

### message

当采用`process.send()`来发送消息时触发。
参数：`message`，为json对象，或者primitive value；`sendHandle`，net.Socket对象，或者net.Server对象（熟悉cluster的同学应该对这个不陌生）

**.connected**：当调用`.disconnected()`时，设为false。代表是否能够从子进程接收消息，或者对子进程发送消息。

**.disconnect()**：关闭父进程、子进程之间的IPC通道。当这个方法被调用时，`disconnect`事件就会触发。如果子进程是node实例（通过child_process.fork()创建），那么在子进程内部也可以主动调用`process.disconnect()`来终止IPC通道。参考[process.disconnect](https://nodejs.org/api/process.html#process_process_disconnect)。



请注意，`exec`方法默认的最大允许输出到stdout和stderr的数据量不超过200K，如果超过了，子进程就会被杀死