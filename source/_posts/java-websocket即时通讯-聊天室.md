---
title: java-websocket即时通讯-聊天室
date: 2017-11-29 16:13:06
tags:
---
###  效果预览
---
有小伙伴需要源码，该文章中的代码年代久远，重新弄了一个seven-im，请移步 [<seven-im，一个基于java springboot websocket的及时通讯(IM)，在线聊天室、群聊、私聊>](https://blog.csdn.net/qq_29485177/article/details/105844778)

![聊天室截图](https://i-blog.csdnimg.cn/blog_migrate/80d2700bf7ba0d2c71c0d166b05debcf.png)


### web即时通讯的解决方式

---

- 轮询：简单粗暴，过多的请求，服务器表示压力很大
- 长连接：发送请求到服务器，有消息时才返回，长时间占用资源
- websocket：基于tcp新的协议，双向通信

  废话少说，上代码（部分关键代码）！

### html

    <style>
		#chatroom_container{width:600px;height:500px;margin:70px auto 0px auto;background:#fff;border-radius:4px;overflow:hidden;box-shadow:0px 0px 20px #666;}
		#east{float:right;width:130px;height:350px;}
		#center{float:left;width:469px;height:350px;box-sizing:border-box;padding:8px;border-right:1px solid #ddd;overflow-y:auto;}
		#south{width:100%;height:40px;border-top:1px solid #ddd;position:absolute;left:0;bottom:0;box-sizing:border-box;padding:10px;background:#fff;}
		#chat{width:100%;height:100%;box-sizing:border-box;}
		#sendButton{position:absolute;bottom:5px;right:6px;}
		.record{margin:6px 5px;}
		.onlineStaff{margin:3px 4px;padding:3px 4px;cursor:pointer;}
		.onlineStaff:hover{background:#f0f0f0;}
    </style>
    
    <div id="chatroom_container">
    	<div id="north">交流平台</div>
    	<div id="online"></div>
    	<div id="center">
    		<div id="console"></div>
    	</div>
    	<div id="east"></div>
    	<div style="clear:both;"></div>
    	<div id="south">
    		<textarea id="chat" ></textarea>
    		<div id="sendButton" onclick="Chat.sendMessage()">发送</div>
    	</div>
    </div>

### js

    var Chat = {};
    Chat.socket = null;
    Chat.connect = (function(host){
        if ('WebSocket' in window) {
            Chat.socket = new WebSocket(host);
        } else if ('MozWebSocket' in window) {
            Chat.socket = new MozWebSocket(host);
        } else {
            Console.log('Error: WebSocket is not supported by this browser.');
            return;
        }
        //websocket连接打开时
        Chat.socket.onopen = function () {
            document.getElementById('chat').onkeydown = function(event) {
                if (event.keyCode == 13) {
                    Chat.sendMessage();
                    event.returnValue = false;
                    event.preventDefault();
                }
            };
        };
        //websocket连接关闭时
        Chat.socket.onclose = function () {
            document.getElementById('chat').onkeydown = null;
        };
        //websocket连接有消息时
        Chat.socket.onmessage = function (message) {
            var data = eval('(' + message.data + ')');
            //data即为后台发送的数据，在此根据数据内容进行判断属于Chat.info | Chat.self | Chat.other
        }
        
    });
    //初始化
    Chat.initialize = function() {
        if (window.location.protocol == 'http:') {
            Chat.connect('ws://' + window.location.host + '/websocket/chat');
        } else {
            Chat.connect('wss://' + window.location.host + '/websocket/chat');
        }
    };
    //发送的方法
    Chat.sendMessage = (function() {
        var message = document.getElementById('chat').value;
        if (message != '') {
            var data = {
        		'from':'',
        		'to':'',
        		'groupId':'',
        		'type':'',
        		'message':message
        	}//from,to,type为后期私聊，群里
        	var msg = JSON.stringify(data);
            Chat.socket.send(msg);
            document.getElementById('chat').value = '';
        }
    });
    
    var Console = {};//聊天主窗体
    //负责展现系统提示，例如 小明进入
    Console.info = (function(message) {
        var h = '<div style="text-align:center;color:#aca;margin:5px auto;">'+ message +'</div>';
        $('#center').append(h);
    });
    //负责展现自己发送的消息，显示在右边
    Console.self=(function(data){
    	var h = '';
    	//将data解析为html片段
    	$('#center').append(h);
    });
    //负责展现其他人发送的消息，显示在左边
    Console.other=(function(data){
    	var h = '';
    	//将data解析为html片段
    	$('#center').append(h);
    });
    
    Chat.initialize();
    
### java后台
- 核心处理类ChatWebSocket.java，负责websocket连接/关闭/收发消息的整个过程
- 消息类WebSocketMessage.java，发送消息的载体
- 消息处理类ServerEncoder.java，将消息进行编码转json设置
- 配置类GetHttpSessionConfigurator.java，获取httpSession

#### ChatWebSocket.java
websocket连接/关闭/收发消息的整个过程

    @ServerEndpoint(value = "/websocket/chat", configurator = GetHttpSessionConfigurator.class, encoders = ServerEncoder.class)
    public class ChatWebSocket {
        //存储所有的websocket连接
        private static final Map<String, ChatWebSocket> CONNECTIONS = new LinkedHashMap<String, ChatWebSocket>();
    	private User user;
    	private Session session;
        
        /**
        * 当websocket握手成功时触发
        * 该方法参数是可选的，session指的是websocket的session，不是httpSession.
        * 获取httpSession需要通过EndpointConfig类，而EndpointConfig类是由@ServerEndpoin()中的configurator进行设置
        */
        @OnOpen
        public void start(Session session, EndpointConfig config){
            this.session = session;            
            HttpSession httpSession = (HttpSession) config.getUserProperties().get(HttpSession.class.getName());
            WebSocketMessage webSocketMessage = new WebSocketMessage();
            if(httpSession == null){
    			webSocketMessage.setMessage("您未登录系统");
    			ChatWebSocket.sendToOne(this,webSocketMessage);
    			try {
    				this.session.close();
    			} catch (IOException e) {
    				e.printStackTrace();
    			}
    		}else{
    		    //获取httpSession中登录的用户
    		    User user = (User) httpSession.getAttribute("currentUser");
    		    if(user == null){
    				webSocketMessage.setMessage("登录错误，未知用户");
    				ChatWebSocket.sendToOne(this,webSocketMessage);
    				try {
    					this.session.close();
    				} catch (IOException e) {
    					e.printStackTrace();
    				}
    			}else{
    				this.user = user;
    				CONNECTIONS.put(this.user.getId(), this);
    				webSocketMessage.setUserId(this.user.getId());
    				ChatWebSocket.sendToOne(this,webSocketMessage);
    				String msg = this.user.getName() + " 进入";
    				ChatWebSocket.broadcastOnline(msg);
    			}
    		}
        }
        
        //当websocket握手关闭时触发
        @OnClose
        public void end(){
            CONNECTIONS.remove(this.user.getId());
    		String msg = this.user.getName() + " 离开";
    		ChatWebSocket.broadcastOnline(msg);
        }
        
        //当收到消息时
        @OnMessage
        public void incoming(String message){
            WebSocketMessage webSocketMessage = new WebSocketMessage();
    		webSocketMessage.setUserId(this.user.getId());
    		webSocketMessage.setUserName(this.user.getName());
    		webSocketMessage.setMessage(message);
    		webSocketMessage.setSendDate(new Date());
    		ChatWebSocket.broadcast(webSocketMessage);
        }
        
        //发送至单独用户
        private static void sendToOne(ChatWebSocket chatWebSocket,
    			WebSocketMessage webSocketMessage) {
    		chatWebSocket.session.getAsyncRemote().sendObject(webSocketMessage);
    	}
        
        //广播
        private static void broadcast(WebSocketMessage webSocketMessage) {
            Set<String> keys = CONNECTIONS.keySet();
            for (String key : keys) {
    			ChatWebSocket chatWebSocket = CONNECTIONS.get(key);
    			synchronized (chatWebSocket) {
    				ChatWebSocket.sendToOne(chatWebSocket, webSocketMessage);
    			}
    		}
        }
        
        //广播在线用户
        private static void broadcastOnline(String msg) {
    		Set<String> keys = CONNECTIONS.keySet();
    		WebSocketMessage webSocketMessage = new WebSocketMessage();
    		webSocketMessage.setType(WebSocketMessage.WEBSOCKETMESSAGE_TYPE_ALL);
    		webSocketMessage.setMessage(msg);
    		List<UserAO> userList = new ArrayList<UserAO>();
    		for (String key : keys) {
    			ChatWebSocket chatWebSocket = CONNECTIONS.get(key);
    			userList.add(chatWebSocket.user);
    		}
    		webSocketMessage.getAttributes().put("user", userList);
    		for (String key : keys) {
    			ChatWebSocket chatWebSocket = CONNECTIONS.get(key);
    			synchronized (chatWebSocket) {
    				ChatWebSocket.sendToOne(chatWebSocket, webSocketMessage);
    			}
    		}
    	}
        
    }
    

#### WebSocketMessage.java
发送消息的载体

    public class WebSocketMessage {
        private String userId;
    	private String userName;
    	private string from;
    	private string to;
    	private string groupId;
    	...
    	private String message;
    	private Date sendDate;
    	private String type;
    	private Map<String,Object> attributes = new HashMap<String, Object>();
    	
    	public WebSocketMessage(){
    		super();
    	}
    	
    	public WebSocketMessage(String userName,String message,Date sendDate) {
    		this.userName = userName;
    		this.sendDate = sendDate;
    		this.message = message;
    	}
    	
    	//getter and setter ...
    }
    
#### ServerEncoder.java
将消息进行编码转json设置

    public class ServerEncoder implements Encoder.Text<WebSocketMessage> {
    	@Override
    	public void init(EndpointConfig paramEndpointConfig) {
    		// TODO Auto-generated method stub
    	}
    	
    	@Override
    	public void destroy() {
    		// TODO Auto-generated method stub
    	}
    	
    	@Override
    	public String encode(WebSocketMessage paramT) throws EncodeException {
    	    //将WebSocketMessage转为json
    		return JsonMapper.buildNormalMapper().toJson(paramT);
    	}
    }
    
#### GetHttpSessionConfigurator
获取httpSession

    public class GetHttpSessionConfigurator extends Configurator {
    	@Override
    	public void modifyHandshake(ServerEndpointConfig sec,
    			HandshakeRequest request, HandshakeResponse response) {
    		HttpSession httpSession=(HttpSession) request.getHttpSession();
    		if(httpSession!=null){
    			sec.getUserProperties().put(HttpSession.class.getName(),httpSession);
    		}        
    	}
    	
    }
    
    
以上只是部分关键代码，简单的实现了即时通讯，若需要私聊，群聊，分组等功能，则需改造WebSocketMessage.java和前端js中Chat.socket.onmessage()

有小伙伴需要源码，该文章中的代码年代久远，重新弄了一个seven-im，请移步 [<seven-im，一个基于java springboot websocket的及时通讯(IM)，在线聊天室、群聊、私聊>](https://blog.csdn.net/qq_29485177/article/details/105844778)