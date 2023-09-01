# HTTP

## http常见状态码

- 1xx：提示信息，协议处理的中间状态
- 2xx：服务器成功处理了客户端请求

​	200 OK：最常见的成功状态码

​	204 No Content：常见的成功状态码，响应头没有body数据

​	206 Partial Content：应用于HTTP分块下载或断点续传，表示相应返回的Body数据不是资源的全部

- 3xx：客户端请求的资源发生了变动，需要客户端用新的URL

​	301 Moved Permanently：永久重定向，请求的资源已经不存在了，需要用新的URL访问

​	302 Found：临时重定向，请求的资源还在，暂时需要用新的URL访问

​	301和302都会在响应头里使用字段location，指明需要跳转的URL，浏览器会自动重定向

​	304 Not Modified：不具有跳转的含义，表示资源未修改，重定向已存在的缓冲文件，也称缓存重定向，也就是告诉客户端可以继续使用缓存资源，用于缓存控制

- 4xx：客户端发送报文有误

  400 bad request：客户端请求报文有错误

  403 fobidden：服务器禁止访问资源

  404 Not Found：请求的资源在服务器不存在或者未找到

- 5xx：客户端请求报文正确，服务器处理时发生了错误

  500 Internal Server Error：与400类型一样，是一个笼统的错误代码

  501 Not Implemented：表示客户端请求的功能还不支持

  502 Bad Gateway：服务器作为网关或者代理时返回的错误代码

  503 Service Unavailable：服务器当前正忙，暂时无法响应客户端

  504 Gateway timeout：服务器作为网关或者代理，没有及时从上游服务器收到请求

## http常见字段

1. Host:客户端发送请求时，用来指定服务器的域名
2. Content-Length:本次数据长度(http通过设置\r\n和Content-Length都是为了解决粘包问题)
3. Connection字段:最常用于客户端要求服务器使用长连接
4. Content-Type:用于服务器回应时，告诉客户端，本次数据是什么格式
5. Content-Encoding:说明数据的压缩方法,表示服务器返回的数据使用了什么压缩格式

## get和post

1.  get是从服务器获取指定资源,post是根据报文body对指定的资源做出处理
2. GET 方法就是安全且幂等的(只读),post是不安全的,不是幂等的.
3. 可以对get请求的数据做缓存,缓存可以放到浏览器上,也可以放到代理上,在浏览器中get请求可以保存为书签
4. 浏览器一般不会缓存 POST 请求，也不能把 POST 请求保存为书签

**面试题: 使用post请求删除服务器资源和使用delete删除服务器资源有什么区别?**

- 使用POST请求资源实际上并未立即从服务器中删除，而是将其标记为已删除状态.
- 使用DELETE请求直接从服务器中永久删除资源

## http缓存技术

强制缓存和协商缓存

1. 强制缓存

   强缓存指的是只要浏览器判断缓存没有过期，则直接使用浏览器的本地缓存

   cache-control(相对时间),  expires(绝对时间)两个字段实现,cache-control的优先级更高

   cache-control流程如下:

   - 浏览器第一次请求服务器资源时,服务器返回资源并且在response加上cache-control设置国企时间大小
   - 浏览器再次访问服务器资源时,先通过请求资源时间与cache-control设置的过期时间比较,判断是否过期.没有过期,使用缓存.过期则重新请求
   - 服务器再次收到请求后,会再次更新response头部的cache-control

2. 协商缓存

  协商缓存就是与服务端协商之后，通过协商结果来判断是否使用本地缓存。一般通过http响应码304

  两种方式：If-Modified-Since字段和Last-Modified字段、If-None-Match和ETag字段

  - If-Modified-Since字段和Last-Modified字段
    响应头的Last-Modified标示这个响应资源的最后修改时间

    请求头的If-Modified-Since：发现响应头有Last-Modified声明，再次请求的时候带上Last-Modified的时间，服务器根据 If-Modified-Since和最后修改时间进行对比，判断是否被修改，然后返回不同的状态码（200 or 304）
  - If-None-Match和ETag字段
    Etag：标识唯一响应资源
    If-None-Match：当资源过期时，浏览器发现响应头里有 Etag，则再次向服务器发起请求时，会将请求头 If-None-Match 值设置为 Etag 的值。服务器收到请求后进行比对，如果资源没有变化返回 304，如果资源变化了返回 200。

## http/1.1与https
1. http/1.1优点
   简单、灵活易拓展、应用广泛、跨平台
   长链接
   管道化
3. http/1.1缺点
   无状态、明文传输、不安全
   session-cookie
5. 


## socket与tcp握手/挥手

1. tcp的三次握手

   **socket操作：**选择ip和port，服务器bind绑定ip和port，listen监听客户端连接

   **第一次握手：**客户端connect连接服务器，SYN = 1,seq =x

   **第二次握手：**服务器accept这个连接，SYN = 1，seq= y, ack = x+1，第二次握手完成，connect返回。

   **第三次握手：**connect返回后，客户端发送第三次握手ACK = 1， seq = x + 1，第三次握手完成，accept返回。连接建立

2. 为什么需要三次握手？

   - 防止已经失效的连接请求被服务器误认为是新连接：一些滞留的连接请求，到了服务器端，导致服务端建立连接，造成资源浪费
   - 防止连接重复建立

3. tcp 的四次挥手

   **第一次挥手（客户端）：**FIN= 1 seq= m，客户端进入FIN-WAIT1状态

   **第二次挥手（服务器）：**ACK =1 ，seq = n，ack=m+1，服务器进入close-wait状态

   客户端收到第二次挥手时，进入FIN-WAIT2状态，等待服务器传输最后的数据

   **第三次挥手（服务器）：**FIN = 1 ACK = 1 seq = u， ack=m+1，服务器进入LAST-ACK状态

   **第四次挥手（客户端）：**ACK = 1 seq =m+1 ack= u + 1，服务器进入CLOSE，客户端进入TIME-WAIT

   经过2msl后，客户都安进入close

   TIME-WAIT：处理残留报文（最后一次挥手可能在网络中延迟或者丢弃）、保证完整的连接释放、避免旧连接的干扰

4. 什么是全连接？什么是半连接

  全连接是已经建立好tcp连接，双方完成了三次握手

  半连接是tcp建立过程的中间状态，其中一方已经完成了三次握手，另一方还未完成。

  半连接的产生通常由于：网络异常（延迟、超时、拥塞）、服务器忙碌、防火墙限制、syn flood攻击（攻击者发送大量的伪造的syn到服务器，不进行后续的ack，占用服务器资源）
  
5. 什么样的情况导致机器上存在大量的time_wait连接，time_wait如何优化？

  time_wait出现情况：高并发、短连接频繁建立和关闭、客户端连接重用

  如何优化：调整tcp连接参数、调整应用程序逻辑（复用连接）、负载均衡/反向代理、增加服务器资源

  
