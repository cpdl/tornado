### WEB请求的那些事

> 在上节中，主要搭建了web服务，来响应客户端的请求。
>
> 这节中，我们来根据客户请求报文中的请求行(Request line) ,请求头部(Request Header),请求主体(Request body)的信息，来描述服务端如何处理。
>
> 请求行的格式：`<method> <request-URL> <version>`

#### tornado.web库

再次重申：tornado 用 web.RequestHandler 类，来处理客户请求。每一次客户发来请求，先由服务器应用APP根据路由映射，调用处理类，生成一个RequestHandler类对象。这个类对象负责处理 客户请求的请求信息，并根据类的处理方法，生成返回的响应报文。

我们这里，根据客户的请求报文三个部分，来看看 客户的请求报文 提供的信息。

#### 处理请求方法Method--Request Line

* 客户的请求方式在HTTP/1.1中，有八种之多。在生产环境中常用到的是get和post两种方法。

  > get方法，主要用来请求数据。client端将请求的信息以URL编码的信息明文传递给服务器。
  >
  > post方法，主要用来传递数据给服务端。client端请求的信息以数据包形式传送给服务端。
  >
  > post请求，常用来提交表单的私密数据。get请求，网页中文件加载等。

* 同一个Request-URL，**可使用多个请求方法**。表示 将概念相关的处理方法绑定在同一个URL中。

  > 比如用户信息，用get表示请求用户信息，用post表示修改用户的信息等。
  >
  > 再如：Request-URL 为 `http://ip:/book/book_id` ，即可利用get 方法通过 book_id 得到书籍的详细信息。也可利用post方法，提交书籍的信息，从而修改书籍有详细信息。

* 实现过程：（这里测试同一个URL绑定多个请求方法）

  ```python
  #以一个请求地址的两种 请求方法为例。
  #在路由映射中添加：
  # (r"/book/(\d+)", BookInfoHandler)

  #添加处理类，注意这里的id来源。
  class BookInfoHandler(tornado.web.RequestHandler):
      def get(self,id):
          self.write(id);
      
      def post(self,id):
          self.write(id + "post")
          
  # 测试方法：在ubuntu中命令：
  # 测试get: 
  #curl http://192.168.128.140:8000/book/12345612
  # 测试post:
  #curl -d aaa=bbb http://192.168.128.140:8000/book/12345612
  ```

#### URL的秘密

* URL书写风格

  * URL有两种表示风格。早期，查询字符串风格，在URL中以？表示传递的参数 ，比如在百度中搜索`hello`,URL的请求地址：`https://www.baidu.com/s?wd=hello&ie=UTF-8` 。
    * 后期，出现RESTFUL风格的表示方式(**Representational State Transfe**). 这种概念中，每一个URL 都表示一种资源，而将对资源的操作放在了请求体中。简单说来，也是一种HTTP的分层思想，将资源和操作分离开。																			

  > 查询字符串风格 ：` http://192.168.128.140:8000/book?id=123456`
  >
  > RESTFUL风格 ： `http://192.168.128.140:8000/book/id/123456`
  >
  > 通过参数使用方法，通过路径调用方法

#### 自定义参数 --URL和Request body

  * 在客户端的请求报文中，网站开发者的自定义参数，出现在两个地方：URL的查询字符串中，和请求报文的请求体(Request body)中。

  * get_argument 获取URL查询字符串或请求体中的参数信息。而get_arguments 获取查询字符串或请求体中的信息对应元素**列表**。命令使用方法：`get_argument(target_name,default,strip=true)` ，注意返回的信息是unicode字符串。

  * get_query_argument ，get_query_arguments。只查询URL字符串信息。

  * get_body_argument, get_body_arguments 查询请求主体中信息。

    ```python
    # 常用get_argument 来获取参数。get_query_argument我在firefox测试才正常，而get_body_argument在curl下测试正常。所以建议使用get_argument.
    # get_argument和get_arguments,都接受一个参数的多个值，但get_argument只返回这个参数的最后一个值，get_arguments 返回这个参数值的列表。参考tornado源码的说明 。
    # 测试例子。添加路由：(r'/argument',ArgHandler) ,这里注意，如用上节的代码添加，需要先把根路由'/'删除或放在最下面，否则先匹配到根路由，就不再往下面匹配路由。
    class ArgHandler(tornado.web.RequestHandler):
        def get(self):
            arg1 = self.get_argument('arg1', '')
            print arg1
            self.write(arg1)
            
    # 这里再补充一个知识，便于服务器重启。
    # 正常情况下，我们每次修改这个服务器程序，都要在服务器端重新运行启动命令：python server.py
    # tornado为我们提供了一个服务器应用的参数，debug调试。开启会，如果代码有修改，它会重新加载，并且服务器出错，会在浏览器中返回错误信息。可在开发环境中使用，但不要在正式环境中使用。代码如此下：
    app = tornado.web.Application(
    	handlers,
        debug = True
    )
    #注意，handlers后，添加逗号。
    ```

* URL路由匹配问题

  第三节中，我们简单的说明了路由正则匹配的事情。这里我们需要注意几点：

  * 链接路由请求，在路由匹配时，只要匹配到。就不再向后匹配。


  * 如果使用正则分组，请求方法的参数要分组名保持个数相同。如果分组没命名，则按顺序传入参数。

  * 服务中处理 请求时，分组 做为参数 传递。对参数进行操作。(倒序)

    ```python
    # 以第一例来说：(r"/book/(\d+)", BookInfoHandler)，此处正则匹配时有一分组，分组将会以一个整体看待做为参数，传递给处理方法：BookInfoHandler。所以后面定义类的处理方法时有两个参数：get(self,id)。否则，web.RequestHandler类会报错。
    # 分组命令写法：(r"/book/(?P<bid>\d+)",BookInfoHandler),此处将分组命名为：bid.那么后面处理方法：BookInfoHandler必须将bid 作为参数,而不能用名字做为参数。如：get(self,bid)

    # 示例如下：
    # match with，匹配路由：(r"/book/(?P<bid>\d+)",BookInfoHandler)
    # 处理方法：倒着输出bid。
    class BookInfoHandler(tornado.web.RequestHandler):
        def get(self,bid):
            self.write(bid[::-1])
            
    # 运行服务器应用，在客户浏览器测试：http://192.168.128.140:8000/book/123456.
    # 在浏览器会显示 ：654321
    ```

####请求头部的元素--Request header

每一次客户请求报文中请求头部包含的客户端信息，我们都封装在这次请求生成的RequestHandler对象的request属性中。比如说请求头部包含的客户地址表示：`self.request.remote_ip`。

下面列出一些比较请求信息：

    * self.request.method HTTP请求方法。如：get,post,put等。
    * self.request.url 请求的完整URL。 path:路径部分。query: 查询部分。
    * self.request.body 请求主体。
    * self.request.remote_ip , 客户端的IP地址作为字符串。如果HTTPServer.xheaders设置，将传递由负载均衡器 
    在X-Real-Ip或X-Forwarded-For头中提供的真实IP地址。
    * self.request.cookie  客户存储的cookie值。
    * self.request.files 文件属性中使用文件上传。
    * self.request.connection 长连接。
    * self.request.request_time 返回此请求执行所花费的时间。

