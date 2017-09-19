# [DOCS LIST](/)

# _Nodejs规范文档_

---

> ## **快速入门**

### 介绍

> 特征

- 基于express@4.13.4,nodejs@6.9.0

> 架构

![架构图](https://thumbnail0.baidupcs.com/thumbnail/4b8c61f9b004f6055b847c61ccdc508a?fid=2483048885-250528-339503342833614&time=1505811600&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-4kLUJlccDNjuR%2B1P5XZ94DyhjpI%3D&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=6069955521780655514&dp-callid=0&size=c710_u400&quality=100&vuk=-&ft=video)

> ## **基础功能**



### 运行流程

> 安装依赖

```
npm install

```
> 运行

```
node server.js local

```


### 配置

> 目录结构

- app：配置项目基本的参数，比如整站的API接口和专题的API接口以及GET/POST方法等。
	- base：基础代码
	- lib：独立扩展的功能
	- utils：公共静态方法、error处理
- controllers：存放业务逻辑层代码
- logic：进入业务逻辑层之前的过滤层
- config：存放不同环境配置文件，比如服务器配置，端口监听等。
- node_modules：存放 package.json 中安装的模块，当你在 package.json 添加依赖的模块并安装后，存放在这个文件夹下。
- routes：存放路由文件。
- models:	存放数据层
- views：存放模版文件。
- public： 存放html，css，js等静态文件。
- utils:	公共方法、函数
- middleware:	独立功能的中间件	
- package.json：存储着工程的信息及模块依赖，当在 dependencies 中添加依赖的模块时，运行 npm install，npm 会检查当前目录下的 package.json，并自动安装所有指定的模块。

> 环境配置文件

目前设置了四个环境，分别是local(本地),dev(开发),test(QA),prod(线上)

```

NODE_ENV=local
PORT=3000 
LOGDIR=logs
RDS_PORT=6379
RDS_HOST='127.0.0.1'
RDS_PWD=''

```

### Router

当用户访问一个 URL 时，最后是通过 controller 里具体的 action 来响应的。所以就需要解析出 URL 对应的 controller 和 action，

### Logic

当在 Action 里处理用户的请求时，经常要先获取用户提交过来的数据，然后对其校验，如果校验没问题后才能进行后续的操作；当参数校验完成后，有时候还要进行权限判断等，这些都判断无误后才能进行真正的逻辑处理。如果将这些代码都放在一个 Action 里，势必让 Action 的代码非常复杂且冗长。

为了解决这个问题， 在控制器前面增加了一层 Logic，Logic 里的 Action 和控制器里的 Action 一一对应，系统在调用控制器里的 Action 之前会自动调用 Logic 里的 Action。

### Controller

MVC 模型中，控制器是用户请求的逻辑处理部分。

### View

目前view层只是作为页面静态化的模板。

> ## **缓存、数据库**

目前使用redis作为缓存，通过set、get来存储和获取缓存数据。

### redis推送、订阅

用作静态化页面方案的通知

> ## **进阶应用**

### 多进程

使用nodejs自带的cluster模块实现，使用graceful模块设置自启动。


```js

    if (cluster.isMaster) {
      //console.log("宿主启动...");
      var workers = [];
      var worker;

      for (var i = 0; i < numCPUs; i++) {
        worker = cluster.fork();
        worker.on("message", function(data){
          //console.log("the worker " + worker.id + "|" + worker.process.pid);
  　　　　　　workers.forEach(function(wk){
  　　　　　　　　wk.send(wk);
  　　　　　　});
        });
        workers.push(worker);
        
      }
      /**
       * 监听worker向master状态事件
       * @param  {[type]} worker   [description]
       * @param  {[type]} address) {                                 } [description]
       * @return {[type]}          [description]
       */
      cluster.on('listening', function(worker, address) {
        //console.log('核心'+i+' pid:'+ worker.process.pid+', Address: '+address.address+":"+address.port);
        //console.log(cluster.workers+"===="+cluster.worker)
      });

      //异常退出重新启动一个worker 2秒后
      cluster.on('exit', function(worker, code, signal) {
        //console.log('核心'+i+' pid:'+ worker.process.pid+' 重启')
        setTimeout(function() {
          cluster.fork();
        }, 2000);
      });

      cluster.on('fork', function (worker) {
          //console.log('[master] ' + 'fork: worker' + worker.id);
      });

      cluster.on('online', function (worker) {
          //console.log('[master] ' + 'online: worker' + worker.id);
      });
      /**
       * 监听master和worker的message事件
       * @param  {[type]} id) {                   cluster.workers[id].on('message', function(pid) {          workerObj['process id:  ' + pid] + [description]
       * @return {[type]}     [description]
       */

    } else if (cluster.isWorker) {
      //console.log(cluster.worker.id);
      var server = this.app.listen(process.env.PORT, function() {
        console.log('Express server listening on port ' + process.env.PORT);
        //console.log('worker' + cluster.worker.id);
        logger.info('worker' + cluster.worker.id+" success");

      });
        process.on('message', function(msg){
          process.send(msg);

        });

      /**
       * 当uncaughtException触发后，延迟一段时间退出进程 uncaughtException emit, base on process.on('uncaughtException')
       * @param  {[type]} {                       server: [ server ],          error: (err, throwErrorCount [description]
       * @return {[type]}    [description]
       */
      graceful({
        server: [server],
        killTimeout: 10000, //延迟触发
        error: (err, throwErrorCount) => {
          if (err.message) {
            err.message += ' (uncaughtException throw ' + throwErrorCount + ' times on pid:' + process.pid + ')';
            logger.error(err.message);
          }
          
        },
      });

    }

```

### 日志

使用log4js设置info和error两级日志。

```js

        log4js.configure({
            "appenders": [{
                "category": "console",
                "type": "console"
            }, {
                "category": "info",
                "type": "dateFile",
                "layout":{
                    "type":"pattern",
                    "pattern":'{"date":"%d","level":"%p","category":"%c","host":"%h","pid":"%z","data":\'%m\'}'
                },                
                "filename": path.join(process.env.LOGDIR, "/info/info"),
                "alwaysIncludePattern": true,
                "pattern": "-yyyy-MM-dd.log"
            }, {
                "category": "error",
                "type": "dateFile",
                "layout":{
                    "type":"pattern",
                    "pattern":'{"date":"%d","level":"%p","category":"%c","host":"%h","pid":"%z","data":\'%m\'}'
                },                
                "filename": path.join(process.env.LOGDIR, "/error/error"),
                "alwaysIncludePattern": true,
                "pattern": "-yyyy-MM-dd.log"
            }, {
                "category": "error_http",
                "type": "dateFile",
                "layout":{
                    "type":"pattern",
                    "pattern":'{"date":"%d","level":"%p","category":"%c","host":"%h","pid":"%z","data":\'%m\'}'
                },                
                "filename": path.join(process.env.LOGDIR, "/error/error_http"),
                "alwaysIncludePattern": true,
                "pattern": "-yyyy-MM-dd.log"
            }, {
                "category": "http",
                "type": "dateFile",
                "layout":{
                    "type":"pattern",
                    "pattern":'{"date":"%d","level":"%p","category":"%c","host":"%h","pid":"%z","data":\'%m\'}'
                },
                "filename": path.join(process.env.LOGDIR, "/http/http"),
                "alwaysIncludePattern": true,
                "pattern": "-yyyy-MM-dd.log"
            }],
            "replaceConsole": true,
            "levels": {
                "all": "auto"
            }
        });


```        

### 安全性

XSS、CSRF

### 单元测试

使用mocha模块进行单元测试。

### 性能测试、压力测试

apache bin目录下的ab.exe


### 异常、错误处理