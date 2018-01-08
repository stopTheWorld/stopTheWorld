* 一个全功能的servlet容器会为 servlet的每个HTTP请求做下面一些工作：
  * 当第一次调用`servlet`时候，`加载`该`servlet类`并调用`init`方法（仅仅一次）
  * 对每次请求，构造一个`ServletRequest`实例和一个`ServletResponse`实例
  * 调用`servlet`的`service`方法，同时传递`ServletRequest`和`ServletResponse`对象
  * 当`servlet`被关闭的时候，调用`servlet`的`destroy`方法并`卸载servlet类`
* Catalina由两个主要模块组成：
  * `连接器`(connector)：用来“连接”容器里边的请求，他的工作是为接收到的每一个HTTP请求构造一个`request`对象和`response`对象。然后它把流程传递给容器
  * `容器`(container)：从连接器接收到`request`和`response`对象后调用`servlet`的`service`方法用于响应

---

* HTTP
  * HTTP请求
    * Method URI Version
    * Header
    * CRLF
    * Body
  * HTTP响应
    * Version Code Status
    * Header
    * CRLF
    * Body
* Socket
* ServerSocket

---

* Servlet
  * 生命周期：
    * init
    * service
    * destroy


---

`HttpConnector`：监听客户端连接，创建处理对象池，传递请求到处理线程

1. 【主线程】：initialize()
   1. 调用`open()`从***ServerSocketFactoty***获取一个***ServerSocket***
2. 【主线程】：start()
   1. 启动***conector线程***
   2. 循环***minProcessors***次
      1. 调用`newProcessor()`创建并启动***processor线程***
      2. 调用`recycle()`将创建的***HttpProcessor***放入处理器***对象池***
3. 【conector线程 】：run()
   1. ***while(!stopped)***：
      1. 调用`ServerSocket.accept()`等待客户端连接获取***Socket***
      2. 调用`createProcessor()`从处理器对象池获取或创建***HttpProcessor***
      3. 调用`HttpProcessor.assgin(Socket)`唤醒***processor线程***
4. 【主线程】：stop()
   1. 调用所有`HttpProcessor.stop()`停止***processor线程***
   2. 关闭***ServerSocket***
   3. 停止***conector线程***

---

`HttpProcessor`：创建Request/Response对象，从Socket输入流解析Http请求头、Header

1. 【主线程/connector线程】：HttpProcessor(HttpConnector, int)
   1. 存储***HttpConnector***
   2. 调用`HttpConnector.createRequest()`和`HttpConnector.createRequest()`创建***Request对象***和***Response对象***
2. 【主线程/connector线程】：start()
   1. 创建并启动***processor线程***
3. 【processor线程】：run()
   1. ***while(!stopped)***：
      1. 调用`await()`等待获取***Socket***
      2. 调用`process(Socket)`进行处理
      3. 调用`HttpConnector.recycle()`将***HttpProcessor***重新放入处理器对象池中
4. 【processor线程】：await()
   1. 阻塞直到***available == true***
   2. 获取***Socket***
   3. ***available = false***
   4. 唤醒所有阻塞
5. 【conector线程】 ：assign(Socket)
   1. 阻塞直到***available == false***
   2. 存储***Socket***
   3. ***available = true***
   4. 唤醒所有阻塞
6. 【processor线程】：process(Socket)
   1. 从***Socket***获取并包装输入流***SocketInputStream***
   2. ***while(!stopped  && ok && keepAlive)***
      1. 设置***Request对象***的***输入流***和***Response对象***的***输出流***
      2. 调用`parseConnection(Socket)`
      3. 调用`parseRequest(SocketInputStream,OutputStream)`
      4. 调用`parseHeaders(SocketInputStream)`
      5. 调用`Container.invoke(Request, Response)`
      6. 如果***finishResponse==true***
         1. `Response.finishResponse()`
         2. `Request.finishRequest()`
         3. 结束输出`output.flush()`
      7. 回收***Request对象***和***Response对象***
         1. `Request.recycle()`
         2. `Response.recycle()`
   3. `shutdownInput()`
   4. 关闭***Socket***
7. 【processor线程】parseRequest(SocketInputStream,OutputStream)
   1. 从***SocketInputStream***解析***请求行信息***并填充到***HttpRequestLine***
   2. 获取***method***、***protocol***
   3. 获取***请求参数***
   4. 从***绝对url获取uri***
   5. 获取***jsessionid***
   6. 调用`normalize()`规范uri
   7. 填充***Request对象***的method、protocol、uri、queryString、SessionId、SessionURL、secure、scheme
8. 【processor线程】parseHeaders(SocketInputStream)
   1. ***whie(true)***
      1. 从***Request***分配一个***HttpHeader***
      2. 从***SocketInputStream***解析***一行Header***并填充到***HttpHeader***
      3. 获取***value***
      4. 填充***Requets对象***的一些***DefaultHeaders***
         1. 调用`RequestUtil.parseCookieHeader()`解析并填充***Cookie***
9. 【主线程】：stop()
   1. 停止***processor线程***

---

Container

* Engine：表示整个Catalina的servlet引擎
* Host：表示一个拥有数个Context的虚拟主机
* Context：表示一个Web应用，一个Context包含一个或多个Wrapper
* Wrapper：表示一个独立的servlet

---

Pipeline

Vavle

VavleContext

Contained

