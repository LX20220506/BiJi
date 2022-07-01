# SignalR预备知识

## 实时Web简述

大家都见过和用过实时Web, 例如网页版的即时通讯工具, 网页直播, 网页游戏, 还有股票仪表板等等.

 

传统的Web应用是这样工作的:

![](img\SignalR\SignalR_传统HTTP请求.png)

浏览器发送HTTP请求到ASP.NET Core Web服务器, 如果一切顺利的话, Web服务器会处理请求并返回响应, 在Payload里面会包含所请求的数据.

 

但是这种工作方式对实时Web是不灵的. 实时Web需要服务器可以主动发送消息给客户端(可以是浏览器):

![](E:\筆記\img\SignalR\SignalR_Server主动发送到Client.png)

Web服务器可以主动通知客户端数据的变化, 例如收到了新的对话消息.

## "底层"技术

而SignalR使用了三种"底层"技术来实现实时Web, 它们分别是Long Polling, Server Sent Events 和 Websocket.

首先, 得知道什么是Ajax. 这个就不介绍了.

## Long Polling

### Polling

介绍Long Polling之前, 首先介绍一下**Polling**.

Polling是实现实时Web的一种笨方法, 它就是通过**定期的向服务器发送请求**, 来查看服务器的数据是否有变化.

如果服务器数据没有变化, 那么就返回204 No Content; 如果有变化就把最新的数据发送给客户端:

![](img\SignalR\SignalR_Polling.png)

下面是Polling的一个实现, 非常简单:

![](img\SignalR\SignalR_1.png)

就看这个Controller的Get方法即可. 用到了MyService, 它在项目里是单例的. 它的方法非常简单:

![](E:\筆記\img\SignalR\SignalR_2.png)

MyService就是做了一个全局的Count, 它的GetLatestCount会返回最新的Count.

Controller里面的代码意思是: 如果Count > 6 就返回一个对象, 里面包含count的值和传进来的id; 如果 count > 10, 还要返回一个finished标志.

看一下前端代码:

![](E:\筆記\img\SignalR\SignalR_3.png)

也是非常的简单, 点击按钮后定时发送请求, 如果有结果就显示最新count值; 如果有finished标志, 就显示最新值和已结束.

注意这里使用的是fetch API.



运行项目, count > 6的时候:

![](img\SignalR\SignalR_4.png)

count > 10的时候结束:

![](img\SignalR\SignalR_5.png)

这就是Polling, 很简单, 但是比较浪费资源.

**SignalR没有采用Polling这种技术**

### Long Polling

**Long Polling**和Polling有类似的地方, 客户端都是发送请求到服务器. 但是不同之处是: **如果服务器没有新数据要发给客户端的话, 那么服务器会继续保持连接, 直到有新的数据产生, 服务器才把新的数据返回给客户端**.

**如果请求发出后一段时间内没有响应, 那么请求就会超时. 这时, 客户端会再次发出请求**.

![](E:\筆記\img\SignalR\SignalR_LongPolling.png)

![](img\SignalR\SignalR_LongPolling_超时.png)

例子, Controller的代码稍有改动:

![](img\SignalR\SignalR_6.png)

改动的目的就是在符合要求的数据出现之前, 保持连接开放.

 

前端也有一些改动:

![](E:\筆記\img\SignalR\SignalR_7.png)

pollWithTimeout方法使用了race, 如果请求后超过9秒没有响应, 那么就返回超时错误.

poll里面, 如果请求返回的结果是200, 那么就更新UI. 但是如果没有finished标志, 就继续发出请求.

 

运行:

![](E:\筆記\img\SignalR\SignalR_8.png)

可以看到只有一个请求, 请求的时间很长, 标识连接开放了很长时间.

这里需要注意的一点是, 服务器的超时时长和浏览器的超时时长可能不一样.

前边介绍的Polling和Long Polling都是HTTP请求, 这其实并不是很适合.

## Server Sent Events (SSE)

使用**SSE**的话, **Web服务器可以在任何时间把数据发送到浏览器, 可以称之为推送. 而浏览器则会监听进来的信息, 这些信息就像流数据一样, 这个连接也会一直保持开放, 直到服务器主动关闭它**.

**浏览器会使用一个叫做EventSource的对象用来处理传过来的信息**.

![](E:\筆記\img\SignalR\SignalR_ServerSentEvents.png)

例子, 这和之前的代码有很多地方不同, 用到了Reponse:

![](E:\筆記\img\SignalR\SignalR_9.png)

注意SSE返回数据的只能是字符串, 而且以data:开头, 后边要跟着换行符号, 否则EventSource会失败.

 

客户端:

![](E:\筆記\img\SignalR\SignalR_10.png)

这个就很简单了, 使用EventSource的onmessage事件. 前一个请求等到响应回来后, 会再发出一个请求.

 

运行:

![](E:\筆記\img\SignalR\SignalR_11.png)

这个EventSource要比Polling和Long Polling好很多.

它有以下优点: 使用简单(HTTP), 自动重连, 虽然不支持老浏览器但是很容易polyfill.

而缺点是: 很多浏览器都有最大并发连接数的限制, 只能发送文本信息, 单向通信.

## Web Socket

Web Socket是不同于HTTP的另一个TCP协议. 它使得浏览器和服务器之间的交互式通信变得可能. 使用WebSocket, 消息可以从服务器发往客户端, 也可以从客户端发往服务器, 并且没有HTTP那样的延迟. 信息流没有完成的时候, TCP Socket通常是保持打开的状态.

使用线代浏览器时, SignalR大部分情况下都会使用Web Socket, 这也是最有效的传输方式. 

**全双工通信**: 客户端和服务器可以同时往对方发送消息.

并且不受SSE的那个浏览器连接数限制(6个), 大部分浏览器对Web Socket连接数的限制是50个.

消息类型: 可以是文本和二进制, Web Socket也支持流媒体(音频和视频).

其实正常的HTTP请求也使用了TCP Socket. Web Socket标准使用了握手机制把用于HTTP的Socket升级为使用**WS协议**的 WebSocket socket.

### 生命周期

Web Socket的生命周期是这样的:

![](E:\筆記\img\SignalR\SignalR_12.png)

所有的一切都发生在TCP Socket里面, 首先一个常规的HTTP请求会要求服务器更新Socket并协商, 这个叫做HTTP握手. 然后消息就可以在Socket里来回传送, 直到这个Socket被主动关闭. 在主动关闭的时候, 关闭的原因也会被通信.

### HTTP 握手

每一个Web Socket开始的时候都是一个简单的HTTP Socket.

客户端首先发送一个GET请求到服务器, 来请求升级Socket. 

如果服务器同意的话, 这个Socket从这时开始就变成了Web Socket.

![](E:\筆記\img\SignalR\SignalR_WebSocket握手.png)

这个请求的示例如下:

![](E:\筆記\img\SignalR\SignalR_13.png)

第一行表明这就是一个HTTP GET请求.

Upgrade 这个Header表示请求升级socket到Web Socket.

Sec-WebSocket-Key, 也很重要, 它用于防止缓存问题, 具体请查看[官方文档](https://tools.ietf.org/html/rfc6455).

 

服务器理解并同意请求以后, 它的响应如下:

![](E:\筆記\img\SignalR\SignalR_14.png)

返回101状态码, 表示切换协议.

如果返回的不是101, 那么浏览器就会知道服务器没有处理WebSocket的能力.

此外Header里面还有Upgrade: websocket.

Sec-WebSocket-Accept是配合着Sec-WebSocket-Key来运作的, 具体请查阅[官方文档](https://tools.ietf.org/html/rfc6455).

### 消息类型

Web Socket的消息类型可以是文本, 二进制. 也包括控制类的消息: Ping/Pong, 和关闭.

每个消息由一个或多个Frame组成:

![](E:\筆記\img\SignalR\SignalR_15.png)

所有的Frame都是二进制的. 所以文本的话, 就会首先转化成二进制.

 

Frame 有若干个Header bits.

有的可以表示这个Frame是否是消息的最后一个Frame;

有的可以表示消息的类型.

有的可以表示消息是否被掩蔽了. 客户端到服务器的消息被掩蔽了, 为了防止缓存投毒(使用恶意数据替换缓存).

有的可以设置payload的长度, payload会占据frame剩下的地方.

 

实际上用的时候, 你基本不会观察到frame, 它是在后台处理的, 你能看到的就是完整的消息.

但是在浏览器调试的时候, 你看到的是frame挨个传递进来而不是整个消息.

 

看下例子:

首先ASP.NET Core项目里已经内置了WebSocket, 但是需要配置和使用这个中间件, 在Startup:

![](E:\筆記\img\SignalR\SignalR_16.png)

这里我们设置了每隔120秒就ping一下. 还设置用于接收和解析frame的缓存大小. 其实这两个值都是默认的值.

 

修改后的Controller:

![](E:\筆記\img\SignalR\SignalR_17.png)

这里需要注入HttpContextAccessor. 然后判断请求是否是WebSocket请求, 如果是的话, 客户端会收到回复, 这时Socket就升级完成了. 升级完返回一个webSocket对象, 然后我把events通过它发送出去. 随后我关闭了webSocket, 并指明了原因NormalClosure.

 

然后看看SendEvents方法:

![](E:\筆記\img\SignalR\SignalR_18.png)

这里的重点就是webSocket对象的SendAsync方法. 我需要把数据转化成buffer进行传送. 数据类型是Text. 具体参数请查看文档.

 

看一下客户端:

![](E:\筆記\img\SignalR\SignalR_19.png)

也很简单, 这里有一个WebSocket对象, 注意这里的url开头是**ws**而不是http, 还有一个**wss**, 就先当与http里的https.

然后eventhandler和SSE的差不多. 返回的json数据需要先parse, 然后再使用.
