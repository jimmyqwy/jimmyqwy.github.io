---
layout: post
title: "[Memo] WebSocket"
date:   2015-08-05 12:00:00
tags:
- Web
- WebSocket
---

### Concepts
WebSocket is a framing protocol over TCP/IP protocol. It is initiated by a client using a special HTTP request to the server and, after the initial handshake, the client and server can freely and asynchronously send frames to each other. There are two types of frames: control and data. The minimal control frame size is 2 bytes while the data frame is starts at 6 bytes for a client and 2 bytes for a server. Data frames can be either text or binary. Text frames are UTF-8 encoded. Frames can be chunked; meaning that a big data set can be split over several frames. WebSocket doesn’t introduce any identification associated with frames. Therefore it isn’t allowed to mix frames of different messages; only control frames can appear in a sequence of intermediate frames of a big message.

The majority of web applications are designed on a request - response paradigm. Although AJAX allowed for asynchronicity , there is still artificial waiting for a response before proceeding to the next step of execution. Since a WebSocket connection is established once, it eliminates needs of reestablishing of a connection for every data exchange, and eliminates sending redundant HTTP headers in a sequential communication. These benefits are especially critical for SSL type of connections where initial connection handshake is a costly operation. Browser WebSocket sends are truly asynchronous and Java server side code can send messages without waiting for when they are requested. Such freedom of sending messages can require some bookkeeping organization to keep an application state consistent. 

A traditional AJAX based implementation can use a lazy mechanism to render the areas issuing a content request calls. However in the case of using a WebSocket, a server can deliver the content in a browser as it gets ready without responding to particular AJAX requests. The drawback of AJAX requests is that their server side processing can be not in an optimal order due to browser’s requests serialization. Giving a possibility to a server deciding the best way of a content calculation can improve an overall responsiveness of a web application.

### Notes
- Using WebSocket brings particular challenges in web application development. WebSocket session has no relation with HTTP session, though it can be used for a similar purpose.  Certain common data can be associated with the session so all messages processing can rely on certain state and data maintained in the session.

-  If a browser also applies a pool for WebSocket connections it can be serious limitation, because without any mechanism of closing WebSocket connection, it can stay active forever, and other attempts to create new connections will be deadlocked.  <=== a best practice recommendation is to use only one WebSocket connection.

- A browser can’t cache data transferred over WebSocket. Therefore using WebSocket for transferring a browser’s cacheable resources isn’t an efficient approach.

- Websocket .vs. restful
REST communication data format is usually limited to JSON and request parameters whereas a WebSocket message payload can be anything including pure binary data.

- WebSockets are natural for development in types of applications such as:
Games with real-time player collaboration
Real-time monitoring systems
System requiring user collaboration like chat, shareable document editing, etc.
However, WebSocket can be applied to traditional web applications with certain benefits.

### Reference
[RESTFUL vs WebSocket][RvsW]

=====

Please refer to [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
[RvsW]: http://www.pubnub.com/blog/websockets-vs-rest-api-understanding-the-difference/