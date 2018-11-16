---
layout: post
title: "浏览器输入 URL 回车后"
---

前几天在网上看到这样一篇文章[《从输入 URL 到页面加载完成的过程中都发生了什么事情》](http://fex.baidu.com/blog/2014/05/what-happen/)，此文涉及面极广甚至还从硬件方面进行解析，所以打算按照自己的理解粗略写一写这个问题。操作是访问URL为 http://sakuyakun.com 网址。

1. 当在浏览器上按下回车按钮访问一个 URL 时，浏览器会开一个线程来处理这个请求。

2. 判断 URL 协议，如果是 HTTP/HTTPS 协议就按照 Web 方式来处理（常见的还有FTP）。

3. URL 是域名而不是IP地址的话，查找 DNS 过程（浏览器缓存 -> 系统缓存 -> 路由器缓存 -> ...），通过 DNS 将该域名解析成IP地址。

4. 确认了IP地址与端口号后，浏览器向服务器发起 TCP 连接，与浏览器建立 TCP 三次握手。

5. 握手成功后，浏览器向服务器发送 HTTP 请求报文（请求行，请求报头，请求正文）。

6. 服务器处理请求，检测是否已缓存。如果没有缓存，服务器返回 HTTP 报文。如果存在缓存，首先进行新鲜度检测（Cache-Control 与 Expires），如果未过期则直接返回缓存内容给客户端（200 from cache）。如果新鲜度检测没通过则进行资源二次校验（Last-Modified 与 ETag）。验证通过返回缓存给浏览器（304），失败则获取最新资源。

7. 假设无缓存文件，且服务器没问题，成功返回资源，或者可以说 HTTP 报文（状态码200，响应报头，响应报文）。

8. 浏览器接收到资源文件（HTML，CSS，JS），浏览器渲染引擎与JS引擎开始工作，JS引擎负责解析和执行JS。渲染引擎首先将HTML代码转为DOM，CSS代码转为CSSOM，然后结合DOM和CSSOM生成一颗渲染树（Render Tree）。生成布局（Layout），将渲染树的所有节点进行平面合成。然后进行绘制显示（Painting）。

响应头的 Cache-Control 与 Expires 的作用一致，用于有效期效验，如果同时设置 Cache-Control 优先级则更高且覆盖 Expires 因为是 HTTP/1.1 定义的规则，相比 Expires 配置项更多。而 Last-Modified 与 ETag ，用于资源二次校验。服务器会优先验证 ETag 再而比对 Last-Modified 文件最后修改时间，ETag 可以理解为与资源关联的一个标记， 通过对比这个标记差异从而判断是否应该使用缓存文件。

Reflow（回流）和 Repain（重绘）是页面性能优化比较关键的地方。DOM 节点中的各个元素都是以盒模型的形式存在，当浏览器去重新计算其位置和大小以及其他属性时的过程称为 Reflows。当浏览器重新绘制内容颜色，字体等不影响布局位置的属性的过程称为 Repaint。