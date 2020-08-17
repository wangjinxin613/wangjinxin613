# cluster模块

`Node.js`默认单进程运行，对于`32`位系统最高可以使用512MB内存，对于`64`位最高可以使用1GB内存。**`对于多核CPU的计算机来说，这样做效率很低，因为只有一个核在运行，其他核都在闲置`**。`cluster模块`就是为了解决这个问题而提出的。

- `cluster模块`允许设立一个主进程和若干个worker进程，由主进程监控和协调`worker`进程的运行。
- `worker`之间采用进程间通信交换消息，`cluster模块`内置一个负载均衡器，采用`Round-robin`算法协调各个worker进程之间的负载。
- 运行时，所有新建立的链接都由主进程完成，然后主进程再把TCP连接分配给指定的worker进程。

```javascript
var cluster = require('cluster');
var os = require('os');

if (cluster.isMaster){
  for (var i = 0, n = os.cpus().length; i < n; i += 1){
    cluster.fork();
  }
} else {
  http.createServer(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");
  }).listen(8000);
}
```

上面代码先判断当前进程是否为`主进程（cluster.isMaster）`，如果是的，就按照`CPU的核数`，新建若干个`worker进程`；如果不是，说明当前进程是`worker进程`，则在该进程启动一个服务器程序。
上面这段代码有一个缺点，就是一旦work进程挂了，主进程无法知道。
为了解决这个问题，可以在主进程部署`online事件`和`exit事件`的`监听函数`。

```javascript
var cluster = require('cluster');
if(cluster.isMaster) {
  var numWorkers = require('os').cpus().length;
  console.log('Master cluster setting up ' + numWorkers + ' workers...');
  for(var i = 0; i < numWorkers; i++) {
    cluster.fork();
  }
 cluster.on('online', function(worker) {
    console.log('Worker ' + worker.process.pid + ' is online');
  });
cluster.on('exit', function(worker, code, signal) {
    console.log('Worker ' + worker.process.pid + ' died with code: ' + code + ', and signal: ' + signal);
    console.log('Starting a new worker');
    cluster.fork();
  });
}
```

上面代码中，主进程一旦监听到`worker`进程的`exit`事件，就会重启一个`worker`进程。worker进程一旦启动成功，可以正常运行了，就会发出`online`事件。

## cluster模块的属性与方法

####  isMaster，isWorker

`isMaster`属性返回一个`布尔值`，表示当前进程`是否为主进程`。这个属性由`process.env.NODE_UNIQUE_ID`决定，如果process.env.NODE_UNIQUE_ID为未定义，就表示该进程是主进程。
`isWorker`属性返回一个`布尔值`，表示当前进程`是否为work进程`。它`与isMaster属性的值正好相反`。

#### fork()

`fork方法`用于`新建`一个`worker进程`，上下文都复制主进程。只有主进程才能调用这个方法。
该方法返回一个`worker`对象。

####  kill()

`kill方法`用于`终止worker进程`。它可以接受一个参数，表示系统信号。

- 如果当前是主进程，就会终止与`worker.process`的联络，然后将系统信号法发向worker进程。
- 如果当前是`worker进程`，就会终止与主进程的通信，然后退出，返回0。
- 在以前的版本中，该方法也叫做 `worker.destroy()` 。

### listening事件

`worker`进程调用`listen`方法以后，`“listening”`就传向该进程的服务器，然后传向主进程。
该事件的回调函数接受两个参数，一个是当前`worker对象`，另一个是`地址对象`，包含网址、端口、地址类型（IPv4、IPv6、Unix socket、UDP）等信息。这对于那些服务多个网址的Node应用程序非常有用。

```javascript
cluster.on('listening', function(worker, address) {
  console.log("A worker is now connected to " + address.address + ":" + address.port);
});
```

## worker对象

`worker对象`是`cluster.fork()`的返回值，代表一个`worker进程`。
它的属性和方法如下

### worker.id

`work.id`返回当前`worker`的独一无二的进程编号。这个编号也是`cluster.workers`中指向当前进程的索引值。

### worker.process

所有的`worker`进程都是用`child_process.fork()`生成的。`child_process.fork()`返回的对象，就被保存在`worker.process`之中。
通过这个属性，可以获取`worker`所在的进程对象。

### worker.send()

该方法用于在主进程中，向子进程发送信息。

```javascript
if (cluster.isMaster) {
  var worker = cluster.fork();
  worker.send('hi there');
} else if (cluster.isWorker) {
  process.on('message', function(msg) {
    process.send(msg);
  });
}
```

上面代码的作用是，worker进程对主进程发出的每个消息，都做回声。
在worker进程中，要向主进程发送消息，使用`process.send(message)`；要监听主进程发出的消息，使用下面的代码。

```javascript
process.on('message', function(message) {
  console.log(message);
});
```

发出的消息可以字符串，也可以是JSON对象。下面是一个发送JSON对象的例子。

```javascript
worker.send({
  type: 'task 1',
  from: 'master',
  data: {
    // the data that you want to transfer
  }
});
```

## cluster.workers对象

该对象`只有主进程`才有，包含了所有`worker进程`。每个成员的键值就是一个`worker进程对象`，键名就是该`worker进程的worker.id属性`。

```javascript
function eachWorker(callback) {
  for (var id in cluster.workers) {
    callback(cluster.workers[id]);
  }
}
eachWorker(function(worker) {
  worker.send('big announcement to all workers');
});
```

上面代码用来遍历所有`worker`进程。
当前`socket`的`data`事件，也可以用`id`属性识别`worker`进程。

```javascript
socket.on('data', function(id) {
  var worker = cluster.workers[id];
});
```

## PM2模块

`PM2`模块是`cluster`模块的一个包装层。它的作用是尽量将`cluster`模块抽象掉，让用户像使用单进程一样，部署多进程Node应用。

```javascript
// app.js
var http = require('http');
http.createServer(function(req, res) {
  res.writeHead(200);
  res.end("hello world");
}).listen(8080);
```

上面代码是标准的Node架设Web服务器的方式，然后用`PM2`从命令行启动这段代码。

```
$ pm2 start app.js -i 4
```

上面代码的`i`参数告诉`PM2`，这段代码应该在`cluster_mode`启动，且新建`worker`进程的数量是4个。
如果`i`参数的值是0，那么当前机器有几个CPU内核，PM2就会启动几个worker进程。
如果一个`worker`进程由于某种原因挂掉了，会立刻重启该worker进程。

```
# 重启所有worker进程
$ pm2 reload all
```

每个worker进程都有一个id，可以用下面的命令查看单个worker进程的详情。

```
$ pm2 show <worker id>
```

正确情况下，PM2采用`fork模式`新建worker进程，即主进程fork自身，产生一个worker进程。
`pm2 reload`命令则会用`spawn`方式启动，即一个接一个启动worker进程，一个新的worker启动成功，再杀死一个旧的`worker`进程。
采用这种方式，重新部署新版本时，服务器就不会中断服务。

```
$ pm2 reload <脚本文件名>
```

关闭worker进程的时候，可以部署下面的代码，让worker进程监听`shutdown`消息。一旦收到这个消息，进行完毕收尾清理工作再关闭。

```javascript
process.on('message', function(msg) {
  if (msg === 'shutdown') {
    close_all_connections();
    delete_logs();
    server.close();
    process.exit(0);
  }
});
```

**cluster的内部实现原理是封装了一层child_process--fork，而child_process--fork 则是封装了unix 系统的fork 方法**

[child_process](./child_process子进程.md)  是 nodejs的子进程

