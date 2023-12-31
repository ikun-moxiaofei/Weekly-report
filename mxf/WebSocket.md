WebSocket 是基于 TCP 的一种新的**网络协议**。它实现了浏览器与服务器全双工通信——浏览器和服务器只需要完成一次握手，两者之间就可以创建**持久性**的连接， 并进行**双向**数据传输。

**HTTP协议和WebSocket协议对比：**

- HTTP是**短连接**
- WebSocket是**长连接**
- HTTP通信是**单向**的，基于请求响应模式
- WebSocket支持**双向**通信
- HTTP和WebSocket底层都是TCP连接

**WebSocket缺点：**
服务器长期维护长连接需要一定的成本各个浏览器支持程度不一WebSocket 是长连接，受网络限制比较大，需要处理好重连
**结论：**WebSocket并不能完全取代HTTP，它只适合在特定的场景下使用，例如（视频弹幕，网页在线聊天）

**实现步骤：**
1). 直接使用websocket.html页面作为WebSocket客户端
2). 导入WebSocket的maven坐标
3). 导入WebSocket服务端组件WebSocketServer，用于和客户端通信
4). 导入配置类WebSocketConfiguration，注册WebSocket的服务端组件
5). 导入定时任务类WebSocketTask，定时向客户端推送数据
#### 示例代码开发
#### 1). 定义websocket.html页面
```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="UTF-8">
    <title>WebSocket Demo</title>
  </head>
  <body>
    <input id="text" type="text" />
    <button onclick="send()">发送消息</button>
    <button onclick="closeWebSocket()">关闭连接</button>
    <div id="message">
    </div>
  </body>
  <script type="text/javascript">
    var websocket = null;
    var clientId = Math.random().toString(36).substr(2);

    //判断当前浏览器是否支持WebSocket
    if('WebSocket' in window){
      //连接WebSocket节点
      websocket = new WebSocket("ws://localhost:8080/ws/"+clientId);
    }
    else{
      alert('Not support websocket')
    }

    //连接发生错误的回调方法
    websocket.onerror = function(){
      setMessageInnerHTML("error");
    };

    //连接成功建立的回调方法
    websocket.onopen = function(){
      setMessageInnerHTML("连接成功");
    }

    //接收到消息的回调方法
    websocket.onmessage = function(event){
      setMessageInnerHTML(event.data);
    }

    //连接关闭的回调方法
    websocket.onclose = function(){
      setMessageInnerHTML("close");
    }

    //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
    window.onbeforeunload = function(){
      websocket.close();
    }

    //将消息显示在网页上
    function setMessageInnerHTML(innerHTML){
      document.getElementById('message').innerHTML += innerHTML + '<br/>';
    }

    //发送消息
    function send(){
      var message = document.getElementById('text').value;
      websocket.send(message);
    }

    //关闭连接
    function closeWebSocket() {
      websocket.close();
    }
  </script>
</html>
```
2). 导入maven坐标
在sky-server模块pom.xml中已定义
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```
3). 定义服务端组件
直接导入到sky-server模块即可
```java
package com.sky.websocket;

import org.springframework.stereotype.Component;
import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

/**
 * WebSocket服务
 */
@Component
@ServerEndpoint("/ws/{sid}")
public class WebSocketServer {

    //存放会话对象
    private static Map<String, Session> sessionMap = new HashMap();

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session, @PathParam("sid") String sid) {
        System.out.println("客户端：" + sid + "建立连接");
        sessionMap.put(sid, session);
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, @PathParam("sid") String sid) {
        System.out.println("收到来自客户端：" + sid + "的信息:" + message);
    }

    /**
     * 连接关闭调用的方法
     *
     * @param sid
     */
    @OnClose
    public void onClose(@PathParam("sid") String sid) {
        System.out.println("连接断开:" + sid);
        sessionMap.remove(sid);
    }

    /**
     * 群发
     *
     * @param message
     */
    public void sendToAllClient(String message) {
        Collection<Session> sessions = sessionMap.values();
        for (Session session : sessions) {
            try {
                //服务器向客户端发送消息
                session.getBasicRemote().sendText(message);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```
4). 定义配置类，注册WebSocket的服务端组件
```java
package com.sky.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

/**
 * WebSocket配置类，用于注册WebSocket的Bean
 */
@Configuration
public class WebSocketConfiguration {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
```
5). 定义定时任务类，定时向客户端推送数据
```java
package com.sky.task;

import com.sky.websocket.WebSocketServer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Component
public class WebSocketTask {
    @Autowired
    private WebSocketServer webSocketServer;

    /**
     * 通过WebSocket每隔5秒向客户端发送消息
     */
    @Scheduled(cron = "0/5 * * * * ?")
    public void sendMessageToClient() {
        webSocketServer.sendToAllClient("这是来自服务端的消息：" + DateTimeFormatter.ofPattern("HH:mm:ss").format(LocalDateTime.now()));
    }
}
```

 
