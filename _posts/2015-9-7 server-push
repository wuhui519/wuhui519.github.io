---
layout: post
title: 关于“服务器推”技术的调研
---
#关于“服务器推”技术的调研

##一、基于TCP长连接的“服务器推”技术

当网络通信时采用TCP协议时，在真正的读写操作之前，server与client之间必须建立一个连接，当读写操作完成后，双方不再需要这个连接时它们可以释放这个连接，连接的建立是需要三次握手的，而释放则需要四次握手，所以说每个连接的建立都是需要资源消耗和时间消耗的.

经典的三次握手示意图：

![image](http://pic002.cnblogs.com/images/2011/305779/2011062613192683.jpg)

经典的四次握手关闭图：

![image](http://pic002.cnblogs.com/images/2011/305779/2011062613210341.jpg)

所谓长连接，client向server发起连接，server接受client连接，双方建立连接。client与server完成一次读写之后，它们之间的连接并不会主动关闭，后续的读写操作会继续使用这个连接。而且，这种连接是全双工通讯，即客户端可以向服务端发送数据，服务端也可以主动向客户端发送数据，以便服务器更实时地推送客户端需要的信息。

基于TCP长连接方式的优点是iOS提供原生的socket连接支持，缺点是Socekt连接代码比较底层，对程序开发要求较高。另外，服务端还需要额外支持TCP连接的子网穿透和防火墙穿透问题，也增加了开发成本。

[raywenderlich](http://www.raywenderlich.com)网站上有一篇从零开始构建基于Socket的长连接文章[Networking Tutorial for iOS: How To Create A Socket Based iPhone App and Server](http://www.raywenderlich.com/3932/networking-tutorial-for-ios-how-to-create-a-socket-based-iphone-app-and-server)

##二、基于HTTP长连接的“服务器推”技术

###1. Comet

浏览器作为 Web 应用的前台，自身的处理功能比较有限。浏览器的发展需要客户端升级软件，同时由于客户端浏览器软件的多样性，在某种意义上，也影响了浏览器新技术的推广。在 Web 应用中，浏览器的主要工作是发送请求、解析服务器返回的信息以不同的风格显示。AJAX 是浏览器技术发展的成果，通过在浏览器端发送异步请求，提高了单用户操作的响应性。但 Web 本质上是一个多用户的系统，对任何用户来说，可以认为服务器是另外一个用户。现有 AJAX 技术的发展并不能解决在一个多用户的 Web 应用中，将更新的信息实时传送给客户端，从而用户可能在“过时”的信息下进行操作。而 AJAX 的应用又使后台数据更新更加频繁成为可能。

![图 1. 传统的 Web 应用模型与基于 AJAX 的模型之比较](http://www.ibm.com/developerworks/cn/web/wa-lo-comet/fig001.jpg) 

**图 1. 传统的 Web 应用模型与基于 AJAX 的模型之比较**

“服务器推”是一种很早就存在的技术，以前在实现上主要是通过客户端的套接口，或是服务器端的远程调用。因为浏览器
技术的发展比较缓慢，没有为“服务器推”的实现提供很好的支持，在纯浏览器的应用中很难有一个完善的方案去实现“服务器推”并用于商业程序。最近几年，因为 AJAX 技术的普及，以及把 IFrame 嵌在“htmlfile“的 ActiveX 组件中可以解决 IE 的加载显示问题，一些受欢迎的应用如 meebo，gmail+gtalk 在实现中使用了这些新技术；同时“服务器推”在现实应用中确实存在很多需求。因为这些原因，基于纯浏览器的“服务器推”技术开始受到较多关注，Alex Russell（Dojo Toolkit 的项目 Lead）称这种基于 HTTP 长连接、无须在浏览器端安装插件的“服务器推”技术为“Comet”。目前已经出现了一些成熟的 Comet 应用以及各种开源框架；一些 Web 服务器如 Jetty 也在为支持大量并发的长连接进行了很多改进。关于 Comet 技术最新的发展状况请参考关于 Comet 的 wiki。
下面将介绍两种 Comet 应用的实现模型。
####基于 AJAX 的长轮询（long-polling）方式
如 图 1 所示，AJAX 的出现使得 JavaScript 可以调用 XMLHttpRequest 对象发出 HTTP 请求，JavaScript 响应处理函数根据服务器返回的信息对 HTML 页面的显示进行更新。使用 AJAX 实现“服务器推”与传统的 AJAX 应用不同之处在于：
服务器端会阻塞请求直到有数据传递或超时才返回。
客户端 JavaScript 响应处理函数会在处理完服务器返回的信息后，再次发出请求，重新建立连接。
当客户端处理接收的数据、重新建立连接时，服务器端可能有新的数据到达；这些信息会被服务器端保存直到客户端重新建立连接，客户端会一次把当前服务器端所有的信息取回。

![图 2. 基于长轮询的服务器推模型](http://www.ibm.com/developerworks/cn/web/wa-lo-comet/fig002.jpg)

**图 2. 基于长轮询的服务器推模型**

一些应用及示例如 “Meebo”, “Pushlet Chat” 都采用了这种长轮询的方式。相对于“轮询”（poll），这种长轮询方式也可以称为“拉”（pull）。因为这种方案基于 AJAX，具有以下一些优点：请求异步发出；无须安装插件；IE、Mozilla FireFox 都支持 AJAX。
在这种长轮询方式下，客户端是在 XMLHttpRequest 的 readystate 为 4（即数据传输结束）时调用回调函数，进行信息处理。当 readystate 为 4 时，数据传输结束，连接已经关闭。Mozilla Firefox 提供了对 Streaming AJAX 的支持， 即 readystate 为 3 时（数据仍在传输中），客户端可以读取数据，从而无须关闭连接，就能读取处理服务器端返回的信息。IE 在 readystate 为 3 时，不能读取服务器返回的数据，目前 IE 不支持基于 Streaming AJAX。
####基于 Iframe 及 htmlfile 的流（streaming）方式
iframe 是很早就存在的一种 HTML 标记， 通过在 HTML 页面里嵌入一个隐蔵帧，然后将这个隐蔵帧的 SRC 属性设为对一个长连接的请求，服务器端就能源源不断地往客户端输入数据。

![图 3. 基于流方式的服务器推模型](http://www.ibm.com/developerworks/cn/web/wa-lo-comet/fig003.jpg)

**图 3. 基于流方式的服务器推模型**

上节提到的 AJAX 方案是在 JavaScript 里处理 XMLHttpRequest 从服务器取回的数据，然后 Javascript 可以很方便的去控制 HTML 页面的显示。同样的思路用在 iframe 方案的客户端，iframe 服务器端并不返回直接显示在页面的数据，而是返回对客户端 Javascript 函数的调用，如“<script type="text/javascript">js_func(“data from server ”)</script>”。服务器端将返回的数据作为客户端 JavaScript 函数的参数传递；客户端浏览器的 Javascript 引擎在收到服务器返回的 JavaScript 调用时就会去执行代码。
从 图 3 可以看到，每次数据传送不会关闭连接，连接只会在通信出现错误时，或是连接重建时关闭（一些防火墙常被设置为丢弃过长的连接， 服务器端可以设置一个超时时间， 超时后通知客户端重新建立连接，并关闭原来的连接）。
使用 iframe 请求一个长连接有一个很明显的不足之处：IE、Morzilla Firefox 下端的进度栏都会显示加载没有完成，而且 IE 上方的图标会不停的转动，表示加载正在进行。Google 的天才们使用一个称为“htmlfile”的 ActiveX 解决了在 IE 中的加载显示问题，并将这种方法用到了 gmail+gtalk 产品中。Alex Russell 在 “What else is burried down in the depth's of Google's amazing JavaScript?”文章中介绍了这种方法。Zeitoun 网站提供的 comet-iframe.tar.gz，封装了一个基于 iframe 和 htmlfile 的 JavaScript comet 对象，支持 IE、Mozilla Firefox 浏览器，可以作为参考。

###2. WebSocket 

可以把 WebSocket 看成是 HTTP 协议为了支持长连接所打的一个大补丁，它和 HTTP 有一些共性，是为了解决 HTTP 本身无法解决的某些问题而做出的一个改良设计。在以前 HTTP 协议中所谓的 keep-alive connection 是指在一次 TCP 连接中完成多个 HTTP 请求，但是对每个请求仍然要单独发 header；所谓的 polling 是指从客户端（一般就是浏览器）不断主动的向服务器发 HTTP 请求查询是否有新数据。这两种模式有一个共同的缺点，就是除了真正的数据部分外，服务器和客户端还要大量交换 HTTP header，信息交换效率很低。它们建立的“长连接”都是伪.长连接，只不过好处是不需要对现有的 HTTP server 和浏览器架构做修改就能实现。

WebSocket 解决的第一个问题是，通过第一个 HTTP request 建立了 TCP 连接之后，之后的交换数据都不需要再发 HTTP request了，使得这个长连接变成了一个真.长连接。但是不需要发送 HTTP header就能交换数据显然和原有的 HTTP 协议是有区别的，所以它需要对服务器和客户端都进行升级才能实现。在此基础上 WebSocket 还是一个双通道的连接，在同一个 TCP 连接上既可以发也可以收信息。此外还有 multiplexing 功能，几个不同的 URI 可以复用同一个 WebSocket 连接。这些都是原来的 HTTP 不能做到的。

另外说一点技术细节，因为看到有人提问 WebSocket 可能进入某种半死不活的状态。这实际上也是原有网络世界的一些缺陷性设计。上面所说的 WebSocket 真.长连接虽然解决了服务器和客户端两边的问题，但坑爹的是网络应用除了服务器和客户端之外，另一个巨大的存在是中间的网络链路。一个 HTTP/WebSocket 连接往往要经过无数的路由，防火墙。你以为你的数据是在一个“连接”中发送的，实际上它要跨越千山万水，经过无数次转发，过滤，才能最终抵达终点。在这过程中，中间节点的处理方法很可能会让你意想不到。

比如说，这些坑爹的中间节点可能会认为一份连接在一段时间内没有数据发送就等于失效，它们会自作主张的切断这些连接。在这种情况下，不论服务器还是客户端都不会收到任何提示，它们只会一厢情愿的以为彼此间的红线还在，徒劳地一边又一边地发送抵达不了彼岸的信息。而计算机网络协议栈的实现中又会有一层套一层的缓存，除非填满这些缓存，你的程序根本不会发现任何错误。这样，本来一个美好的 WebSocket 长连接，就可能在毫不知情的情况下进入了半死不活状态。

而解决方案，WebSocket 的设计者们也早已想过。就是让服务器和客户端能够发送 Ping/Pong Frame（RFC 6455 - The WebSocket Protocol）。这种 Frame 是一种特殊的数据包，它只包含一些元数据而不需要真正的 Data Payload，可以在不影响 Application 的情况下维持住中间网络的连接状态。

twitter、[pusher](https://pusher.com/)等产品采用的就是WebSocket的方式进行服务端-客户端全双工通信，见[Using websockets in native iOS and Android apps](http://www.elabs.se/blog/66-using-websockets-in-native-ios-and-android-apps)。这种方式的优点是构建在HTTP之上，接口比较高级，开发成本较低，客户端和服务端都有第三方组件支持。iOS的WebSocket第三方库见[SocketRocket](https://github.com/square/SocketRocket)

###3. SPDY, HTTP/2

五年以前，谷歌推出了 SPDY，一种新的协议，它定位为替代传统的超文本传输协议 (HTTP)，据称拥有更高的安全性和性能表现。不久前该公司宣布，将很快从 Chrome 去除 SPDY 支持 。这是因为 IETF 一直在努力更打造一种全新的协议：HTTP/2。

SPDY 的主要目标是降低延迟并提高安全性。为了减少延迟，它包括多路复用的支持 — — 支持单个链接的多个请求，不同的请求优先级 — — 为了安全，它强制性使用 TLS 。HTTP/2 采用了 SPDY 的一些特点，如多路复用技术，并使用一套略有不同的 TLS 扩展。

SPDY侧重于提升网页的加载速度，但是也提供server push的高级特性。SPDY通过X-Associated-Content头试验了服务器推送数据给客户端的选项。这个头告诉客户端服务器将在客户端请求资源之前，推送资源给它。Twitter实验室为SPDY开发了iOS版本[CocoaSPDY](https://blog.twitter.com/2013/cocoaspdy-spdy-for-ios-os-x)，但是Server Push功能暂时未能支持，源码见[CocoaSPDY](https://github.com/twitter/CocoaSPDY)


`参考：`

1. [Comet：基于 HTTP 长连接的“服务器推”技术](https://www.ibm.com/developerworks/cn/web/wa-lo-comet/)
2. [Networking Tutorial for iOS: How To Create A Socket Based iPhone App and Server](http://www.raywenderlich.com/3932/networking-tutorial-for-ios-how-to-create-a-socket-based-iphone-app-and-server) 
