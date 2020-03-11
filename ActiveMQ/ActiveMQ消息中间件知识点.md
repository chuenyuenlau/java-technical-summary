# ActiveMQ消息中间件

# 一、MQ简介

​	MQ全称为Message Queue, 消息队列（MQ）是一种应用程序对应用程序的通信方法。应用程序通过写和检索出入列队的针对应用程序的数据（消息）来通信，而无需专用连接来链接它们。消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信，直接调用通常是用于诸如远程过程调用的技术。排队指的是应用程序通过队列来通信。队列的使用除去了接收和发送应用程序同时执行的要求。

# 二、MQ特点

MQ的消费-生产者模型的一个典型的代表，一端往消息队列中不断的写入消息，而另一端则可以读取或者订阅队列中的消息。MQ和JMS类似，但不同的是JMS是SUN JAVA消息中间件服务的一个标准和API定义，而MQ则是遵循了AMQP协议的具体实现和产品。 

> JMS：即Java消息服务（Java Message Service）应用程序接口是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。
>
> AMQP：Advanced Message Queuing Protocol ,一个提供统一消息服务的应用层标准高级消息队列协议,是应用层协议的一个开放标准,为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。
> 简单点说：就是对于消息中间件所接受的消息传输层的协议，只有这样才能保证客户端和消息中间件能够进行交互（换位思考：HTTP和HTTPS甚至说是TCP/IP与UDP协议都要的道理）。



# 三、优势

1.流量肖锋

2.任务异步处理

特点：可以解耦合



# 四、通信模式

## 4.1  点对点（queue）

- 一个消息只能被一个服务接收
- 消息一旦被消费，就会消失
- 如果没有被消费，就会一直等待，直到被消费
- 多个服务监听同一个消费空间，先到先得

```java
/**
 * queue模式：生产者
 */
public class MyQueueProducer {
    //定义ActivMQ的连接地址
    private static final String ACTIVEMQ_URL = "tcp://127.0.0.1:61616";
    //定义发送消息的队列名称
    private static final String QUEUE_NAME = "MyMessage";

    public static void main(String[] args) throws JMSException {
        //创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        //打开连接
        connection.start();
        //创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建队列目标
        Destination destination = session.createQueue(QUEUE_NAME);
        //创建一个生产者
        javax.jms.MessageProducer producer = session.createProducer(destination);
        //创建模拟100个消息
        for (int i = 1 ; i <= 100 ; i++){
            TextMessage message = session.createTextMessage("我发送message:" + i);
            //发送消息
            producer.send(message);
            //在本地打印消息
            System.out.println("我现在发的消息是：" + message.getText());
        }
        //关闭连接
        connection.close();
    }
}}
```



```java
/**
 * queue模式：消费者
 */
public class QueueConsumer {
    //定义ActivMQ的连接地址
    private static final String ACTIVEMQ_URL = "tcp://127.0.0.1:61616";
    //定义发送消息的队列名称
    private static final String QUEUE_NAME = "MyMessage";

    public static void main(String[] args) throws JMSException {
        //创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        //打开连接
        connection.start();
        //创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建队列目标
        Destination destination = session.createQueue(QUEUE_NAME);
        //创建消费者
        javax.jms.MessageConsumer consumer = session.createConsumer(destination);
        //创建消费的监听
        consumer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    System.out.println("获取消息：" + textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```







## 4.2  发布/订阅模式（topic）

- 一个消息可以被多个服务接收
- 订阅一个主题的消费者，只能消费自它订阅之后发布的消息
- 消费端如果在生产端发送消息之后启动，是接收不到消息的，除非生产端对消息进行了持久化（例如广播，只有当时听到的人能听到消息）

```java
/**
 * topic模式：生产者
 */
public class MyTopicProducer {
    //定义ActivMQ的连接地址
    private static final String ACTIVEMQ_URL = "tcp://127.0.0.1:61616";
    //定义发送消息的主题名称
    private static final String TOPIC_NAME = "MyTopicMessage";

    public static void main(String[] args) throws JMSException {
        //创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        //打开连接
        connection.start();
        //创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建队列目标
        Destination destination = session.createTopic(TOPIC_NAME);
        //创建一个生产者
        javax.jms.MessageProducer producer = session.createProducer(destination);
        //创建模拟100个消息
        for (int i = 1; i <= 100; i++) {
            TextMessage message = session.createTextMessage("当前message是(主题模型):" + i);
            //发送消息
            producer.send(message);
            //在本地打印消息
            System.out.println("我现在发的消息是：" + message.getText());
        }
        //关闭连接
        connection.close();
    }
}
```



```java
/**
 * topic模式：消费者
 */
public class TopicConsumer1 {
    //定义ActivMQ的连接地址
    private static final String ACTIVEMQ_URL = "tcp://127.0.0.1:61616";
    //定义发送消息的队列名称
    private static final String TOPIC_NAME = "MyTopicMessage";
    public static void main(String[] args) throws JMSException {
        //创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        //打开连接
        connection.start();
        //创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建队列目标
        Destination destination = session.createTopic(TOPIC_NAME);
        //创建消费者
        javax.jms.MessageConsumer consumer = session.createConsumer(destination);
        //创建消费的监听
        consumer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    System.out.println("获取消息：" + textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```



# 五、配置文件实现

## 5.1 服务端

### - xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jms="http://www.springframework.org/schema/jms"
	xsi:schemaLocation="http://www.springframework.org/schema/beans  
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
     http://www.springframework.org/schema/context  
     http://www.springframework.org/schema/context/spring-context-3.0.xsd  
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
    http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms-3.0.xsd">

	<!-- 这里暴露内部统一使用的MQ地址 -->
    <!-- 连接工厂 -->
	<bean id="internalTargetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://localhost:61616" />
	</bean>
	<bean id="internalConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory"
		destroy-method="stop">
		<property name="connectionFactory" ref="internalTargetConnectionFactory" />
		<property name="maxConnections" value="20" />
	</bean>
	<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
	<bean id="internalJmsTemplate" class="org.springframework.jms.core.JmsTemplate">
		<property name="connectionFactory" ref="internalConnectionFactory" />
	</bean>

	<!-- 推送给用户信息  创建一个Queue-->
	<bean id="userServiceQueue" class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg>
			<value>user.service.queue</value>
		</constructor-arg>
	</bean>
	<!-- 推送给新闻信息   创建一个Queue-->
	<bean id="newsServiceQueue" class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg>
			<value>news.service.queue</value>
		</constructor-arg>
	</bean>
	<!-- 推送给客户信息   创建一个Queue-->
	<bean id="clientServiceQueue" class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg>
			<value>client.service.queue</value>
		</constructor-arg>
	</bean>

	 
</beans>
```



### - Service层

```java
/**
 * 推送的接口
 * @author Administrator
 * @create 2016-8-10 下午3:41:03
 * @version 1.0
 */
public interface PushService {

	public final ExecutorService pushExecutor = Executors.newFixedThreadPool(10);
	
	public void push(Object info);
	
}
```



```java
@Service("userPushService")
public class UserPushServiceImpl implements PushService {

	
	@Autowired
	private JmsTemplate jmsTemplate;
	
	/**
	 * 这里是根据MQ配置文件定义的queue来注入的，也就是这里将会把不同的内容推送到不同的queue中
	 */
	@Autowired
	@Qualifier("userServiceQueue")
	private Destination destination;
	
	@Override
	public void push(final Object info) {
		pushExecutor.execute(new Runnable() {
			@Override
			public void run() {
				jmsTemplate.send(destination, new MessageCreator() {
					public Message createMessage(Session session) throws JMSException {
						 User p = (User) info;
						return session.createTextMessage(JSON.toJSONString(p));
					}
				});
			}			
		});
	}

}
```



### - Controller层

```java
@Controller
@RequestMapping("/push")
public class PushController {

	@Resource(name="userPushService")
	private PushService userPushService;
	
	/**
	 * 用户推送
	 * @param info
	 * @return
	 * @author Administrator
	 * @create 2016-8-10 下午4:22:28
	 */
	@RequestMapping(value="/user",method=RequestMethod.POST)
	@ResponseBody
	public ResultRespone userPush(User info){
		ResultRespone respone = new ResultRespone();
		try {
			userPushService.push(info);
			respone.setData(info);
		} catch (Exception e) {
			e.printStackTrace();
			respone = new ResultRespone(false, e.getMessage());
		}
		return respone;
	}
}

```



### - 前端页面

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
          <script type="text/javascript" src="resources/jquery-1.9.1.js"></script>
     </head>   
<body>
<br/><br/><br/>
用户姓名：<input type="text" id="username" />
用户密码：<input type="text" id="password" />
用户性别：<input type="text" id="sex" />
 <input type="button" value="推送用户信息" id="pushUser" /> 




<script type="text/javascript">
	$("#pushUser").click(function(){
		var data = {
				username : $("#username").val(),
				password : $("#password").val(),
				sex 	 : $("#sex").val()
		};
		ajaxDo("/activemq-service/push/user",data);
	});
	
function ajaxDo(url,data){
	 $.ajax({
	        url:url ,
	        type: "post",
	        dataType: "json",
	        data: data,
	        success:function(result){
	           if(result.success){
	        	   var obj = JSON.stringify(result.data);
	        	   alert(obj);
	           }else{
	        	   alert(result.msg);
	           }
	        }
	    });
}	

</script>

</body>
</html>
```



## 5.2 客户端

### - xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:jms="http://www.springframework.org/schema/jms"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
     http://www.springframework.org/schema/context  
     http://www.springframework.org/schema/context/spring-context-3.0.xsd  
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
    http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms-3.0.xsd">  
     
    <!-- 内部统一使用的MQ地址 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://localhost:61616"/>
    </bean>
    <bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">  
        <property name="connectionFactory" ref="targetConnectionFactory"/>  
        <property name="maxConnections" value="50"/>
    </bean>
    <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">  
        <property name="connectionFactory" ref="connectionFactory"/>
    </bean>

   <!-- 推送给用户信息 -->
	<bean id="userPushListenerMQ" class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg>
			<value>user.service.queue</value>
		</constructor-arg>
	</bean>
	
	<!-- 用户接受推送 -->
    <bean id="userPushListenerConsumer"
          class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="userPushListenerMQ" />
        <!-- 消息监听器 -->
        <property name="messageListener" ref="userPushListener" />
    </bean>
</beans>
```



### - Listener

```java
@Component("userPushListener")
public class UserPushListener implements MessageListener {
	 protected static final Logger logger = Logger.getLogger(UserPushListener.class);
	@Override
	public void onMessage(Message message) {
		 logger.info("[UserPushListener.onMessage]:begin onMessage.");
	        TextMessage textMessage = (TextMessage) message;
	        try {
	            String jsonStr = textMessage.getText();
	            logger.info("[UserPushListener.onMessage]:receive message is,"+ jsonStr);
	            if (jsonStr != null) {
	                User info = JSON.parseObject(jsonStr, User.class);
	                System.out.println("==============================接受到的用户信息 开始====================================");
	                System.out.println(info.toString());
	                System.out.println("==============================接受到的用户信息 结束====================================");
	                WebsocketController.broadcast("user", jsonStr);
	            }
	        } catch (JMSException e) {
	            logger.error("[UserPushListener.onMessage]:receive message occured an exception",e);
	        }
	        logger.info("[UserPushListener.onMessage]:end onMessage.");
	    }
}


```



### - WebsocketController

```java
/**
 * 功能说明：websocket处理类, 使用J2EE7的标准
 * @author Administrator
 * @create 2016-8-11 下午4:08:35
 * @version 1.0
 */
@ServerEndpoint("/websocket/{myWebsocket}")
public class WebsocketController {
	private static final Logger logger = LoggerFactory.getLogger(WebsocketController.class);

    public static Map<String, Session> clients = new ConcurrentHashMap<String, Session>();

    /**
     * 打开连接时触发
     * @param myWebsocket
     * @param session
     */
    @OnOpen
    public void onOpen(@PathParam("myWebsocket") String myWebsocket, Session session){
        logger.info("Websocket Start Connecting:" + myWebsocket);
        System.out.println("进入："+myWebsocket);
        clients.put(myWebsocket, session);
    }

    /**
     * 收到客户端消息时触发
     * @param myWebsocket
     * @param message
     * @return
     */
    @OnMessage
    public String onMessage(@PathParam("myWebsocket") String myWebsocket, String message) {
        return "Got your message ("+ message +").Thanks !";
    }

    /**
     * 异常时触发
     * @param myWebsocket
     * @param throwable
     */
    @OnError
    public void onError(@PathParam("myWebsocket") String myWebsocket, Throwable throwable) {
        logger.info("Websocket Connection Exception:" + myWebsocket);
        logger.info(throwable.getMessage(), throwable);
        clients.remove(myWebsocket);
    }

    /**
     * 关闭连接时触发
     * @param myWebsocket
     */
    @OnClose
    public void onClose(@PathParam("myWebsocket") String myWebsocket) {
        logger.info("Websocket Close Connection:" + myWebsocket);
        System.out.println("Websocket Close Connection:" + myWebsocket);
        clients.remove(myWebsocket);
    }


    /**
     * 将数据传回客户端
     * 异步的方式
     * @param myWebsocket
     * @param message
     */
    public static void broadcast(String myWebsocket, String message) {
        if (clients.containsKey(myWebsocket)) {
            clients.get(myWebsocket).getAsyncRemote().sendText(message);
        } else {
            throw new NullPointerException("[" + myWebsocket +"]Connection does not exist");
        }
    }

}

```



### - 前端页面

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
          <script type="text/javascript" src="resources/jquery-1.8.3.min.js"></script>
     </head>   
<body>

 
<script type="text/javascript">
 //测试controller是否可以进入
// ajaxDo("/activemq-client/index",null);
 
// function ajaxDo(url,data){
// 	 $.ajax({
// 	        url:url ,
// 	        type: "post",
// 	        dataType: "json",
// 	        data: data,
// 	        success:function(result){
// 	           if(result.success){
// 	        	   alert(result.data);
// 	           }else{
// 	        	   alert(result.msg);
// 	           }
// 	        }
// 	    });
// }	


//--------------------------------- webSocket ----------------------------------------------
  initSocket("user");
  initSocket("news");
  initSocket("client");
  

function initSocket(myWebsocket) {
	
	var webSocket = null;
	
    window.onbeforeunload = function () {
        //离开页面时的其他操作
    };

    if (!window.WebSocket) {
        console("您的浏览器不支持websocket！");
        return false;
    }

    var target = 'ws://' + window.location.host + "/activemq-client/websocket/"+myWebsocket;  
		  
		if ('WebSocket' in window) {  
			webSocket = new WebSocket(target);  
		} else if ('MozWebSocket' in window) {  
			webSocket = new MozWebSocket(target);  
		} else {  
		    alert('WebSocket is not supported by this browser.');  
		    return;  
		}  
    
    
    // 收到服务端消息
    webSocket.onmessage = function (msg) {
            alert(msg.data);
        // 关闭连接
        webSocket.onclose();
        console.log(msg);
    };

    // 异常
    webSocket.onerror = function (event) {
        console.log(event);
    };

    // 建立连接
    webSocket.onopen = function (event) {
        console.log(event);
    };

    // 断线
    webSocket.onclose = function () {
		
        console.log("websocket断开连接");
    };
}



</script>
</body>
</html>
```

