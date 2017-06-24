### tornado简单的WEB服务

* 在本节开始前，确认你有一定的python基础，并且pycharm和服务器环境搭建完毕。

* 本节目标，搭建一个微型的web服务，监听8000端口，如有接收到客户端的网页请求，根据请求的路径 返回 对应的信息。

  > 分布来做：
  >
  > 1. 监控端口设置
  > 2. 创建web应用 ， 将请求路径 和 响应 程序 绑定。
  > 3. 响应程序 编写。
  > 4. 创建web服务器，添加 web 应用 ， 监听 端口， 开启事件处理。

#### 端口设置

* 监听端口设置：命令行或是python脚本内？如何设置

  > 我们这里打算使用python程序做web服务器，那么在linux下就可以用`python web.py &` 来让进程在后台执行。也或是直接 使用`python web.py` 让服务在前端执行，并返回信息，便于 停止程序，并对web.py脚本 进行调整。

  * web.py程序运行时，必须监听服务器端口，以便向客户提供服务。如果我们将端口，定义在脚本 内部，那么，如果想要改动监听的端口，我们必须要修改脚本。如果改动次数过多，非常麻烦。所以在运行脚本 时，如果给定监听端口号，就不必再修改内部代码。 **命令给定端口号**，便于修改监听的端口。
  * 但是，有时命令 不写端口，那么内部，就不知道该 监听 那个端口。所以，内部，还是要写一个端口号。这个端口号，要求，如果外部命令，没有指定监听的端口，就做**默认监听端口**。 

* 脚本实现。

  * tornado给我们提供了一个库**options**，这个库可以从外部命令中获取指定的参数。这样，就可以通过对象option来传递命令参数。 

  * options有一个方法：define，来设定字符串默认的值。

  * 下面是实现方法：web.py脚本。

    ```python
    #coding=utf-8
    import tornado.options
    from tornado.options import define,options	
    #端口这里我们用port来表示。比如外部命令：python web.py --port=8000
    #那么在脚本内部，options对象会添加一属性：options.port=8000

    define("port",default=8000,help="run on given port,default 8000",type=int) 
    #这里define 是options库里的一个函数方法。
    #第一个参数传递设定的对象名称。
    #default 表示如果该对象没有值 就以后面的8000作为默认值。
    #help 表示外部命令调用帮助时显示的信息。如外部命令：python web.py --help
    #type 表示对象 值 的属性，int 为数字整形。

    tornado.options.parse_command_line()
    #parse_command_line函数方法，表示根据命令行的输入，处理对应的参数。
    ```

#### 创建web应用和请求响应类

* web应用和web服务器关系

  > 一个web服务器中可以包含多个应用，每一个应用对应一个虚拟主机，做路由映射。

* 路由映射

  > 这里做一个简单的路由映射，将对“/”的访问，交由IndexHandler 类来处理。

* IndexHandler 类定义。

  > 客户请求方法：分为八类。常用的是get和post。这里定义get函数，表示处理get请求。

* 代码实现如下：

  * tornado创建web应用，需要利用web库的Application 类来生成对象。

  ```python
  #coding=utf-8
  import tornado.web

  handlers=[(r"/",IndexHandler)]
  #定义了路由映射，以元组的形式。IndexHandler 是一个RequestHandler类。

  app = tornado.web.Application(
  	handlers
  )
  #创建web应用app，在创建时，需要用参数handlers指定路由映射。
  # 用 default_host 来指定响应的域名，如果没有指定 那么用服务器IP地址做为服务地址。
  # 在客户机上，就使用http://主机ip，来请求服务信息。

  class IndexHandler(tornado.web.RequestHandler):
      def get(self):
          self.write("hello,world!")
   #tornado.web.RequestHandler 类封装了客户请求的头部信息，这样继承后，可以让IndexHandler 类，得到请求信息，来响应客户请求。
  #重写get函数。
  ```

#### 创建web服务器

> 在上面，我们定义了监听端口，web应用。下面，就创建web服务器，添加web应用。绑定端口，开启循环监听。

* 创建web服务，需要用到tornado.httpserver库下的HTTPServer类。

* 循环事务，需要用 tornado.ioloop库下的IOLoop类，做实例化。

* 代码如下：

  ```python
  #coding=utf-8
  import tornado.httpserver
  import tornado.ioloop

  http_server = tornado.httpserver.HTTPSserver(app)
  http_server.listen(options.port)			#绑定端口
  tornado.ioloop.instace().start()
  ```



----------------------------------

完整的代码如下：此处做一些修改`__name__ == __main__`。防止 其它程序 调用 web.py时出错。

```python
#coding=utf-8
import tornado.httpserver
import tornado.options
import tornado.web
import tornado.ioloop

from tornado.options import define,options
#定义port,默认值
define("port",default=8000,help="run on the given port,default 8000",type=int)
#处理请求类
class IndexHandler(tornado.web.RequestHandler):
    def get(self):
       self.write("hello.")
#路由映射
handlers =[(r"/",IndexHandler)]

#当该脚本为自动调用时
if __name__ == "__main__":
    tornado.options.parse_command_line()	#解析命令行。
    #创建web应用
    app = tornado.web.Application(
            handlers
    )
    
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

