# 实战rabbit读后笔记

###### rabbitmq 框架介绍

```mermaid
graph LR
A[producer]-->|Exchange&Routing| B[broker] --> C[consumer]
```

> rabbit关联名词的

* Exchange 交换器  

* RoutingKey 路由key

* BindingKey绑定key
* QueueName 队列名称

> rabbitmq类型

+ fanout 

  - 使用此类型 则producer将会将消息发送到交换器的所有队列中
+ direct

  - 使用此类型 则producer将会把消息通过[^Exchange]根据[^RoutingKey]与[^BindingKey]向对应的队列中 。
+ topic
  - 通过*号匹配一个单词 #号匹配零个或多个单词 将消息通过[^Exchange]以[^RoutingKey]与[^BindingKey]的匹配 
    - 例如 [^RoutingKey]为com.test [^BindingKey] 为com.*  即可匹配到相应的队列中
+ headers
  - 通过 消息体中的headers 与队列绑定的



######   安装erlang

  安装到`/opt/erlang`目录下

```   shell
tar zxvf otp_src_19.3.tar.gz
cd otp_src_19.3
./configure --prefix=/opt/erlang
yum install ***（安装相应包）
make
make install

# 修改/etc/profile
ERLANG_HOME=/opt/erlang
export PATH=$PATH:$ERLANG_HOME/bin
export ERLANG_HOME
source /etc/profile
erl
```

###### 安装rabbitmq

``` shell
tar zxvf rabbit-server-generic-unix-3.6.0.tar.gz -C /opt
cd /opt
mv rabbit_server-3.6.0 rabbitmq

#修改/etc/profile
export PATH=$PATH:/opt/rabbitmq/sbin
export RABBITMQ_HOME=/opt/rabbitmq
source /etc/profile

rabbitmq-server -detached #rabbitmq以守护进程方式在后台运行
rabbitmqctl status #查看rabbit启动状态
rabbitmqctl cluster_status #查看集群状态
```

###### 添加rabbitmq用户

​     rabbitmq中默认用户名和密码都为“**guset**”  只能在本地登录 不可远程查看

```  shell
rabbitmqctl add_user root root123 #添加用户名为root 密码为root123的用户到rabbitmq
rabbitmqctl set_permissions -p / root ".*" ".*" ".*"  #为root 用户设置所有权限
rabbitmqctl set_user_tags root adminstrator #设置root用户为管理员
```

###### 服务端代码

``` java

import com.rabbitmq.client.Channel; 
import com.rabbitmq.client.Connection ; 
import com.rabbitmq.client.ConnectionFactory
import com.rabbitmq.client.MessagePropert es
import java io IOExcept 0口
import java . util.concurrent.TimeoutException; 
public class Rabb tProducer { 
    private stat final Stri EXCHANGE NAME ="exchange_demo"; 
    private stat final String ROUTING KEY = "routingkey_demo "; 
    private static final String QUEUE NAME = "queue_demo"; 
    private static final String IP ADDRESS = "192.168.0.2"; 
    private static final int PORT= 5672;//RabbitMQ 服务端默认端口号为 5672

    public static void main(String[] args) throws IOException ,
    TimeoutException , InterruptedException { 
        ConnectionFactory factory= new ConnectionFactory();
        factory.setHost(IP ADDRESS); 
        factory.setPort (PORT) ; 
        factory.setUsername ("root") ; 
        factory.setPassword ("rootl23") ; 
        Connection connection= factory.newConnection();／／创建连接
            Channel channel= connection.createChannel(); ／／创建信道
            ／／创建一个 type direct 、持久化的、非自动删除的交换器
            channel.exchangeDeclare(EXCHANGE_NAME,"direct", true , false , null); 
            ／／创建一个持久化、非排他的、非自动删除的队列
            channel.queueDeclare(QUEUE_NAME, true , false , false , null); 
            ／／将交换器与队列通过路由键绑定
            channel.queueBind (QUEUE_NAME,EXCHANGE_NAME,ROUTING KEY);
            ／／发送一条持久 的消息 hello worl d ! 
            String message = "Hello World !"; 
            channel.basicPublish(EXCHANGE_NAME , ROUTING_KEY , 
                             MessageProperties.PERSISTENT_TEXT_PLAIN , 
                             message.getBytes()); 
        ／／关闭资源
            channel.close();
            connection.close();
          }
 }
```

###### 客户端代码

```java
private static final String IP_ADDRESS = " 192.168 . 0.2 "; 
private static final int PORT = 5672 ; 
publ static void main(String[] args) throws IOException , 
TimeoutException , InterruptedException { 
    Address[] addresses= new Address[] { 
        new Address(IP ADDRESS , PORT) 
    };
    ConnectionFactory factory = new ConnectionFactory() ; 
    factory . setUsername (“ root“ ) ; 
    factory . setPassword (" rootl23 "); 
    ／／这里的连接方式与生产者的 demo 略有不同，注意辨别区别
        Connection connection= factory.newConnection(addresses) ; ／／创建连接
        final Channel channel= connection.createChannel （） ／／创建信道
        channel basicQos 64 ／／设置客户端最多接收未被 ack 的消息的个数
        Consumer consumer= new Defaul tConsumer (channel) { 
        @Override 
        public void handleDelivery(String consumerTag, 
                                   Envelope envelope , 
                                   AMQP . BasicProperties properties, 
                                   byte [] body) 
            throws IOException { 
            System . out . println (" recv message : "+ new String(body)) ; 
            try { 
                TimeUnit.SECONDS .sleep(l); 
            } catch (InterruptedException e) { 
                e . printStackTrace () ; 
                channel . basicAck(envelope . getDeliveryTag() , false) ; 
                channel . basicConsume(QUEUE NAME , consumer) ; 
                ／／等待回调函数执行完毕之后 关闭资源
                    Time Un t.SECONDS sleep(S);
                channel . close(); 
                connection . close();
}
}
```





 



### 名词注释

---------

[^Exchange]:交换器
[^RoutingKey]: 路由key
[^BindingKey]:绑定key
[^消息label]: 路由key + 绑定key





