### Understanding the NodeJS cluster module

Node.js is becoming more and more popular as a server-side run-time environment, especially for high traffic websites, as statistics show. Also, the availability of several frameworks make it a good environment for rapid prototyping. Node.js has an event-driven architecture, leveraging a non-blocking I/O API that allows requests being processed asynchronously.

Each Node.js process runs in a single thread and by default it has a memory limit of 512MB on 32-bit systems and 1GB on 64-bit systems. Although the memory limit can be bumped to ~1GB on 32-bit systems and ~1.7GB on 64-bit systems, both memory and processing power can still become bottlenecks for various processes

The elegant solution Node.js provides for scaling up the applications is to split a single process into multiple processes or workers, in Node.js terminology. This can be achieved through a cluster module. The cluster module allows you to create child processes (workers), which share all the server ports with the main Node process (master).

In this article you’ll see how to create a express application and  cluster for speeding up your applications.

#### Node.js Cluster Module: what it is and how it works

A cluster is a pool of similar workers running under a parent Node process. Workers are spawned using the fork() method of the child_processes module. This means workers can share server handles and use IPC (Inter-process communication) to communicate with the parent Node process.

Although using a cluster module sounds complex in theory, it is very straightforward to implement. To start using it, you have to include it in your Node.js application:
```
var cluster = require('cluster);
```
A cluster module executes the same Node.js process multiple times. Therefore, the first thing you need to do is to identify what portion of the code is for the master process and what portion is for the workers. The cluster module allows you to identify the master process as follows:
```
if(cluster.isMaster) { ... }
```
The master process is the process you initiate, which in turn initialize the workers. To start a worker process inside a master process, we’ll use the fork() method:
```
cluster.fork();
```
This method returns a worker object that contains some methods and properties about the forked worker. We’ll see some examples in the following section.

A cluster module contains several events. Two common events related to the moments of start and termination of workers are the online and the exit events. online is emitted when the worker is forked and sends the online message. exit is emitted when a worker process dies. Later, we’ll see how we can use these two events to control the lifetime of the workers.


Let’s build a simple Express application to start with.

Create a new directory for this tutorial, and add a file called package.json with the following code:
```
{
  "name": "nodecluster",
  "version": "1.0.0",
  "dependencies": {
    "cluster": "^0.7.7",
    "express": "^4.16.3"
  }
}
```


###### Run npm install from within your project directory, this will install Express. Now we can create a new file, app.js:
```
// Include Express
const express = require('express');

// Create a new Express application
var app = express();

// Add a basic route – index page
app.get('/', function (req, res) {
    res.send('Hello World!');
});

// Bind to a port
app.listen(5000, function() {
    console.log('Your node is running on port 5000');
  });

```
###### Let’s now put together everything we’ve seen so far and show a complete working example.

### Examples

```
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;
const express = require('express');
var app = express();

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  //Check if work id is died
  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });

} else {
  // This is Workers can share any TCP connection
  // It will be initialized using express
  console.log(`Worker ${process.pid} started`);

  app.get('/cluster', (req, res) => {
    let worker = cluster.worker.id;
    res.send(`Running on worker with id ==> ${worker}`);
  });

  app.listen(5000, function() {
    console.log('Your node is running on port 5000');
  });
```

### Conclusion
The cluster module offers to NodeJS the needed capabilities to use the whole power of a CPU. Although not seen in this post, the cluster module is complemented with the child process module that offers plenty of tools to work with processes: start, stop and pipe input/out, etc.
