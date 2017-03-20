
> Stability: 2 - Stable
一个NodeJs实例运行在一个单独的线程中。为了更高效的利用多核操作系统，我们希望启动一个NodeJs的集群去执行我们的任务。
A single instance of Node.js runs in a single thread. To take advantage of
multi-core systems the user will sometimes want to launch a cluster of Node.js
processes to handle the load.
集群模块使我们可以简单地创建子任务并通过共享服务器端口处理他们。
The cluster module allows you to easily create child processes that
all share server ports.

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```
现在运行Node.js 将共享8000端口执行多个任务：
Running Node.js will now share port 8000 between the workers:

```txt
$ node server.js
Master 3596 is running
Worker 4324 started
Worker 4520 started
Worker 6056 started
Worker 5644 started
```

Please note that on Windows, it is not yet possible to set up a named pipe
server in a worker.

