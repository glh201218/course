# 安装
1. 安装mq前要安装Erlang
    ```txt
    https://www.rabbitmq.com/which-erlang.html
    查看对应版本
    ```
2. 下载mq
    ```txt
    https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.8.3/rabbitmq-server-3.8.3.exe
    ```
3. 下载对应Erlang
    ```txt
    http://erlang.org/download/otp_win64_22.2.exe
    ```
4. 打开文件安装(一直下一步)
5. 检查
    * 进入`E:\Software\RabbitMQ\rabbitmq_server-3.8.3\sbin`,mq安装目录运行cmd,输入`rabbitmq-plugins enable rabbitmq_management`
    * 执行成功后打开浏览器输入`http://localhost:15672`进入登录界面
    * 登录界面账号密码默认是`guest`
    * 进入管理后台
6. 登录后点击Admin点击Add user输入账号密码选择角色
    * 创建成功后,上面会出现创建后的账户,`virtual hosts`相当于数据库
    * ![avatar][base64adduser]
    * 现在还没有虚拟数据库,可以点击右侧的`virtual hosts`进行添加,以`/`开头
    * ![avatar][base64adddb]
7. 数据库对用户进行授权
    1. 点击创建的数据库
    2. 进行授权
        * ![avatar][base64permissions]
    3. 查看是否成功
        * ![avatar][base64setpermissions]
8. 重新登录创建的账号
# 操作
1. 创建maven项目(myrabbitmq)
2. 导入依赖
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
        <project xmlns="http://maven.apache.org/POM/4.0.0"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
            <modelVersion>4.0.0</modelVersion>
            <groupId>cn.glh</groupId>
            <artifactId>myrabbitmq</artifactId>
            <version>1.0-SNAPSHOT</version>
            <dependencies>
                <dependency>
                    <groupId>com.rabbitmq</groupId>
                    <artifactId>amqp-client</artifactId>
                    <version>4.0.2</version>
                </dependency>
                <dependency>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                    <version>1.7.10</version>
                </dependency>
                <dependency>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                    <version>1.2.17</version>
                </dependency>
                <dependency>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                    <version>4.11</version>
                </dependency>
            </dependencies>
        </project>
    ```
## 简单队列
* 模型
![avatar](https://upload-images.jianshu.io/upload_images/6597489-d043f6d1a896e14d?imageMogr2/auto-orient/strip|imageView2/2/w/432/format/webp)
    * p : Producer(生产者)
    * c : consumer(消费者)
    * 红色方块 : 队列
    * 功能 ：一个生产者P发送消息到队列Q,一个消费者C接收

* 搭建
    1. 编写工具类
    ```java
    package cn.glh.rabbitmq.util;
    import com.rabbitmq.client.Connection;
    import com.rabbitmq.client.ConnectionFactory;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class ConnectionUtils {
        public static Connection getConnectionUtils() throws IOException, TimeoutException {
            //定义一个连接工厂
            ConnectionFactory factory = new ConnectionFactory();
            //设置服务地址
            factory.setHost("127.0.0.1");
            //AMQP 5672
            factory.setPort(5672);
            //vhost
            factory.setVirtualHost("/vhost_glh");
            //用户名
            factory.setUsername("user_glh");
            //密码
            factory.setPassword("glh1218glh");
            //获取连接
            Connection connection = factory.newConnection();
            return connection;
        }
    }
    ```
    2. 生产者
    ```java
    package cn.glh.rabbitmq.simple;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.Connection;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Send {
        //队列名称
        private static final String QUEUE_NAME = "test_simple_queue";
        public static void main(String[] args) throws IOException, TimeoutException {
            //获取一个连接
            Connection connection = ConnectionUtils.getConnectionUtils();
            //从连接中获取一个通道
            Channel channel = connection.createChannel();
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            String msg = "hello simple !";
            channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
            System.out.println("---send msg:"+msg);
            channel.close();
            connection.close();
        }
    }
    ```
    3. 消费者
    ```java
    package cn.glh.rabbitmq.simple;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    /**
    * 消费者获取消息
    */
    public class Recv {
        //队列名称
        private static final String QUEUE_NAME = "test_simple_queue";
        public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
            //oldapi();
            //获取连接
            Connection connection = ConnectionUtils.getConnectionUtils();
            //创建频道
            Channel channel = connection.createChannel();
            //队列声明
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            //定义消费者
            DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
                //获取到达的消息
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    String s = new String(body, "utf-8");
                    System.out.println("[new api recv:]:" + s);
                }
            };
            //监听队列
            channel.basicConsume(QUEUE_NAME,true,defaultConsumer);
        }
        public static void oldapi() throws IOException, TimeoutException, InterruptedException {
            //获取连接
            Connection connection = ConnectionUtils.getConnectionUtils();
            //创建频道
            Channel channel = connection.createChannel();
            //定义队列的消费者     老的api
            QueueingConsumer consumer = new QueueingConsumer(channel);
            //监听队列
            channel.basicConsume(QUEUE_NAME,true,consumer);
            while (true){
                QueueingConsumer.Delivery delivery = consumer.nextDelivery();
                String s = new String(delivery.getBody());
                System.out.println("[recv msg:]:"+s);
            }
        }
    }
    ```
* 测试

    运行消费者,再运行生产者,消费者监测到新的队列进行获取
* 简单队列的不足

    耦合度高,生产者一一对应消费者(如果我想有多个消费者消费队列中的消息,这时候就不行了),队列名变更,这时候就要同时变更
## 工作队列
* 模型
![avatar](https://img-blog.csdn.net/20170120111818310?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQxNjU4OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    * p : Producer(生产者)
    * c : consumer(消费者)
    * 红色方块 : 队列
    * 功能 ：一个生产者P发送消息到队列Q,多个个消费者C接收
* 为什么出现工作队列

    Simple队列是一一对应的,而且我们实际开发中生产者发送消息是毫不费力的,而消费者一般是要跟业务相结合的,消费者接收到消息后就需要处理,可能需要花费时间,这时候队列会积压了更多消息
* 搭建
    1. 生产者
    ```java
    package cn.glh.rabbitmq.work;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.Connection;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Send {
        //队列名称
        private static final String QUEUE_NAME = "test_work_queue";
        public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
            //获取连接
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            //获取channel
            Channel channel = connectionUtils.createChannel();
            //声明队列
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            for (int i = 0; i < 50; i++) {
                String msg = "hello "+i;
                channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
                Thread.sleep(i*20);
                System.out.println("[work msg]"+msg);
            }
            channel.close();
            connectionUtils.close();
        }
    }
    ```
    2. 消费者1
    ```java
    package cn.glh.rabbitmq.work;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Recv1 {
        //队列名称
        private static final String QUEUE_NAME = "test_work_queue";
        public static void main(String[] args) throws IOException, TimeoutException {
            //获取连接
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            //声明队列
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            //定义一个消费者
            DefaultConsumer Consumer = new DefaultConsumer(channel) {
                //消息到达,触发方法
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    String s = new String(body, "utf-8");
                    System.out.println("[1] Recv msg:"+s);
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        System.out.println("[1] done");
                    }
                }
            };
            //这里auto=false表示打开应答机制
            boolean autoAck=true;
            channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
        }
    }
    ```
    3. 消费者2
    ```java
    package cn.glh.rabbitmq.work;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Recv2 {
        //队列名称
        private static final String QUEUE_NAME = "test_work_queue";
        public static void main(String[] args) throws IOException, TimeoutException {
            //获取连接
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            //声明队列
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            //定义一个消费者
            DefaultConsumer Consumer = new DefaultConsumer(channel) {
                //消息到达,触发方法
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    String s = new String(body, "utf-8");
                    System.out.println("[2] Recv msg:"+s);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        System.out.println("[2] done");
                    }
                }
            };
            //这里auto=false表示打开应答机制
            boolean autoAck=true;
            channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
        }
    }
    ```
* 测试

    先启动两个消费者,再启动生产者
### 轮询分发

    出现一个现象消费者1和消费者2处理的消息是一样多的
    这种方式叫做轮询分发(round-robin)结果就是不管谁忙谁闲都不会多给一个消息
### 公平分发

使用公平分发必须关闭自动应答,autoAck改为手动

* 修改生产者
```java
package cn.glh.rabbitmq.workfair;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Send {
    //队列名称
    private static final String QUEUE_NAME = "test_work_queue";
    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        //获取连接
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        //获取channel
        Channel channel = connectionUtils.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        /**
        * 每个消费者发送确认消息之前,消息队列不发送下一个消息到消费者,一次只处理一个消息
        * 限制发送给同一个消费者不得超过一个
        */
        int prefetchCount=1;
        channel.basicQos(prefetchCount);
        for (int i = 0; i < 50; i++) {
            String msg = "hello "+i;
            channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
            Thread.sleep(i*20);
            System.out.println("[work msg]"+msg);
        }
        channel.close();
        connectionUtils.close();
    }
}
```
* 修改消费者1
```java
package cn.glh.rabbitmq.workfair;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Recv1 {
    //队列名称
    private static final String QUEUE_NAME = "test_work_queue";
    public static void main(String[] args) throws IOException, TimeoutException {
        //获取连接
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        final Channel channel = connectionUtils.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        //保证只发送一个
        channel.basicQos(1 );
        //定义一个消费者
        DefaultConsumer Consumer = new DefaultConsumer(channel) {
            //消息到达,触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String s = new String(body, "utf-8");
                System.out.println("[1] Recv msg:"+s);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    System.out.println("[1] done");
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        boolean autoAck=false;
        channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
    }
}
```
* 修改消费者2
```java
package cn.glh.rabbitmq.workfair;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Recv2 {
    //队列名称
    private static final String QUEUE_NAME = "test_work_queue";
    public static void main(String[] args) throws IOException, TimeoutException {
        //获取连接
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        final Channel channel = connectionUtils.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        //保证只发送一个
        channel.basicQos(1 );
        //定义一个消费者
        DefaultConsumer Consumer = new DefaultConsumer(channel) {
            //消息到达,触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String s = new String(body, "utf-8");
                System.out.println("[2] Recv msg:"+s);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    System.out.println("[2] done");
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        //这里auto=false表示打开应答机制
        boolean autoAck=false;
        channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
    }
}
```
### 消息应答与消息持久化
* 消息应答

    boolean autoAck=false
    channel.basicConsume(QUEUE_NAME,autoAck,consumer)

    boolean autoAck=true;(自动确认模式)一旦rabbitmq将消息分发给消费者,就会从内存中删除.这种情况下如果杀死正在执行的消费者,就会丢失掉正在处理的消息

    Boolean autoAck=false(手动模式),如果一个消费者挂掉,就会交付给其他消费者,rabbitmq支持消息应答,消费者发送一个消息应答,告诉rabbitmq这个消息我已经处理完成了你可以删除了.

    消息应答默认是打开的(false)

* 消息持久化

    Boolean durable=false;

    channel.queueDeclare(Queue_NAME,durable,false,false,null);

    我们将程序中的Boolean durable = false ;改为true;是不可以的,尽管代码是正确的,他也不会运行成功,因为我们已经定义了一个叫test_work_queue这个queue是未持久化,rabbitmq不准许重新定义(不同参数)一个已经存在的队列


## 订阅模式
* 模型
![avatar](https://img-blog.csdn.net/20180310222045849?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzk4NTY2NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
    * p : Producer(生产者)
    * c : consumer(消费者)
    * x : exchange(交换机)
    * 红色方块 : 队列
    * 功能 ：一个生产者P发送消息到交换机,交换机发送消息到队列里,消费者监听队列(一个消费者一个队列)
* 搭建
    1. 创建生产者
    ```java
    package cn.glh.rabbitmq.ps;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.Connection;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Send {
        //队列名称
        private static final String EXCHANGE_NAME = "test_exchange_fanout";
        public static void main(String[] args) throws IOException, TimeoutException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            //声明交换机 fanout:分发类型
            channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
            //发送消息
            String msg="hello ps";
            channel.basicPublish(EXCHANGE_NAME,"",null,msg.getBytes());
            System.out.println("send :"+msg);
            channel.close();
            connectionUtils.close();
        }
    }
    ```
    2. 消费者1
    ```java
    package cn.glh.rabbitmq.ps;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Recv1 {
        //队列名称
        private static final String QUEUE_NAME = "test_queue_fanout_email";
        //交换机名称
        private static final String EXCHANGE_NAME = "test_exchange_fanout";
        public static void main(String[] args) throws IOException, TimeoutException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            //声明队列
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            //绑定队列到交换机
            channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"");
            //保证一次只分发一个
            channel.basicQos(1);
            Consumer Consumer = new DefaultConsumer(channel) {
                //消息到达,触发方法
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    String s = new String(body, "utf-8");
                    System.out.println("[1] Recv msg:"+s);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        System.out.println("[1] done");
                        //手动回执
                        channel.basicAck(envelope.getDeliveryTag(),false);
                    }
                }
            };
            //这里auto=false表示打开应答机制
            boolean autoAck=false;
            channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
        }
    }
    ```
    3. 消费者2
    ```java
    package cn.glh.rabbitmq.ps;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Recv2 {
        //队列名称
        private static final String QUEUE_NAME = "test_queue_fanout_sms";
        private static final String EXCHANGE_NAME = "test_exchange_fanout";
        public static void main(String[] args) throws IOException, TimeoutException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            //声明队列
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            //绑定队列到交换机
            channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"");
            //保证一次只分发一个
            channel.basicQos(1);
            Consumer Consumer = new DefaultConsumer(channel) {
                //消息到达,触发方法
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    String s = new String(body, "utf-8");
                    System.out.println("[2] Recv msg:"+s);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        System.out.println("[2] done");
                        //手动回执
                        channel.basicAck(envelope.getDeliveryTag(),false);
                    }
                }
            };
            //这里auto=false表示打开应答机制
            boolean autoAck=false;
            channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
        }
    }
    ```
### Exchange(交换机)
* `channel.basicPublish(EXCHANGE_NAME,"",null,msg.getBytes());`中的`""`是匿名的意思
* `channel.exchangeDeclare(EXCHANGE_NAME,"fanout");`
    * `fanout` : 是不处理路由键
    
    * `Direct` : 处理路由键
## 路由模式
* 模型
![avatar](https://www.rabbitmq.com/img/tutorials/python-four.png)
### 搭建
1. 生产者
```java
package cn.glh.rabbitmq.routing;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Send {
    private static final String EXCHANGE_NAME = "test_exchange_direct";
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        Channel channel = connectionUtils.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");
        String msg="hello direct";
        String routingKey="info";
        channel.basicPublish(EXCHANGE_NAME,routingKey,null,msg.getBytes());
        System.out.println("send:"+msg);
        channel.close();
        connectionUtils.close();
    }
}
```
2. 消费者1
```java
package cn.glh.rabbitmq.routing;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Recv1 {
    //队列名称
    private static final String QUEUE_NAME = "test_queue_direct_1";
    private static final String EXCHANGE_NAME = "test_exchange_direct";
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        Channel channel = connectionUtils.createChannel();
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        channel.basicQos(1);
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"error");
        Consumer Consumer = new DefaultConsumer(channel) {
            //消息到达,触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String s = new String(body, "utf-8");
                System.out.println("[1] Recv msg:"+s);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    System.out.println("[1] done");
                    //手动回执
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        //这里auto=false表示打开应答机制
        boolean autoAck=false;
        channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
    }
}
```
3. 消费者2
```java
package cn.glh.rabbitmq.routing;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Recv2 {
    //队列名称
    private static final String QUEUE_NAME = "test_queue_direct_2";
    private static final String EXCHANGE_NAME = "test_exchange_direct";
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        Channel channel = connectionUtils.createChannel();
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        channel.basicQos(1);
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"error");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"info");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"warning");
        Consumer Consumer = new DefaultConsumer(channel) {
            //消息到达,触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String s = new String(body, "utf-8");
                System.out.println("[2] Recv msg:"+s);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    System.out.println("[2] done");
                    //手动回执
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        //这里auto=false表示打开应答机制
        boolean autoAck=false;
        channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
    }
}
```
## Topic 
* 模型
![avatar](https://www.rabbitmq.com/img/tutorials/python-five.png)

* 搭建
1. 生产者
```java
package cn.glh.rabbitmq.topic;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Send {
    private static final String EXCHANGE_NAME = "test_exchange_topic";
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        Channel channel = connectionUtils.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME,"topic");
        String msg="商品.....";
        String routingKey="goods.delete";
        channel.basicPublish(EXCHANGE_NAME,routingKey,null,msg.getBytes());
        System.out.println("send:"+msg);
        channel.close();
        connectionUtils.close();
    }
}
```
2. 消费者1
```java
package cn.glh.rabbitmq.topic;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Recv1 {
    //队列名称
    private static final String QUEUE_NAME = "test_queue_topic_1";
    private static final String EXCHANGE_NAME = "test_exchange_topic";
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        Channel channel = connectionUtils.createChannel();
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        channel.basicQos(1);
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"goods.add");
        Consumer Consumer = new DefaultConsumer(channel) {
            //消息到达,触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String s = new String(body, "utf-8");
                System.out.println("[1] Recv msg:"+s);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    System.out.println("[1] done");
                    //手动回执
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        //这里auto=false表示打开应答机制
        boolean autoAck=false;
        channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
    }
}
```
3. 消费者2
```java
package cn.glh.rabbitmq.topic;
import cn.glh.rabbitmq.util.ConnectionUtils;
import com.rabbitmq.client.*;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
public class Recv2 {
    //队列名称
    private static final String QUEUE_NAME = "test_queue_topic_2";
    private static final String EXCHANGE_NAME = "test_exchange_topic";
    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connectionUtils = ConnectionUtils.getConnectionUtils();
        Channel channel = connectionUtils.createChannel();
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        channel.basicQos(1);
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"goods.#");
        Consumer Consumer = new DefaultConsumer(channel) {
            //消息到达,触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String s = new String(body, "utf-8");
                System.out.println("[2] Recv msg:"+s);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    System.out.println("[2] done");
                    //手动回执
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        //这里auto=false表示打开应答机制
        boolean autoAck=false;
        channel.basicConsume(QUEUE_NAME,autoAck,Consumer);
    }
}
```
## Rabbitmq的消息确认机制(事务+confirm)
在rabbitmq中我们可以通过持久化数据解决rabbitmq服务器异常的数据丢失问题

问题:生产者将消息发送出去之后,消息到底有没有到达rabbitmq服务器,默认的情况是不知道的

解决:两种方式
1. AMQP实现了事务机制
2. Confirm模式
### 事务机制
txSelect:用于将当前channel设置成transation模式
txCommit:提交事务
txRollback:回滚事务
* 搭建
    1. 生产者
    ```java
    package cn.glh.rabbitmq.tx;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.Connection;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class TxSend {
        //队列名称
        private static final String QUEUE_NAME = "test_queue_tx";
        public static void main(String[] args) throws IOException, TimeoutException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            String msg="hello tx message";
            try {
                channel.txSelect();
                channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
                channel.txCommit();
            }catch (Exception e){
                channel.txRollback();
                System.out.println("send meg rollcabk");
            }finally {
                channel.close();
                connectionUtils.close();
            }
        }
    }
    ```
    2. 消费者
    ```java
    package cn.glh.rabbitmq.tx;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class TxRecv {
        //队列名称
        private static final String QUEUE_NAME = "test_queue_tx";
        public static void main(String[] args) throws IOException, TimeoutException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            channel.basicConsume(QUEUE_NAME,true,new DefaultConsumer(channel){
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("recv msg:"+new String(body,"utf-8"));
                }
            });
        }
    }
    ```
* 注意:这种模式是很耗时的,降低了rabbitmq的吞吐量

### Confirm模式
    生产者将信息设置成confirm模式,一旦信道进入confirm模式,所有在该信道上面发布的消息都会被指派一个唯一的id(从1开始),这就使得生产者知道消息已经正确到达目的队列了,如果消息和队列是可持久化的,那么确认消息会将消息写入磁盘之后发出,broker回传给生产者的确认消息中deliver-tag域,表示到这个序列号之前的所有消息都已经得到了处理.
`Confirm模式最大的好处在于他是异步的`
1. 普通模式(发一条)`waitForConfirm`
    * 生产者
    ```java
    package cn.glh.rabbitmq.confirm;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.Connection;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    /**
    * 普通模式
    */
    public class Send1 {
        //队列名称
        private static final String QUEUE_NAME = "test_simple_confirm1";
        public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            //生产者调用confirmSelect将channel设为confirm模式
            channel.confirmSelect();
            String msg = "hello confirm1 !";
            channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
            if (channel.waitForConfirms()){
                System.out.println("mesage send ok");
            }else{
                System.out.println("mesage send error");
            }
            channel.close();
            connectionUtils.close();
        }
    }
    ```
    * 消费者
    ```java
    package cn.glh.rabbitmq.confirm;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Recv {
        //队列名称
        private static final String QUEUE_NAME = "test_simple_confirm1";
        public static void main(String[] args) throws IOException, TimeoutException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            channel.basicConsume(QUEUE_NAME,true,new DefaultConsumer(channel){
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("recv confirm1 msg:"+new String(body,"utf-8"));
                }
            });
        }
    }
    ```
2. 批量模式(发一批)`waitForConfirms`
    * 消费者
    ```java
    package cn.glh.rabbitmq.confirm;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.*;
    import java.io.IOException;
    import java.util.concurrent.TimeoutException;
    public class Recv {
        //队列名称
        private static final String QUEUE_NAME = "test_simple_confirm1";
        public static void main(String[] args) throws IOException, TimeoutException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            channel.queueDeclare(QUEUE_NAME,false,false,false,null);
            channel.basicConsume(QUEUE_NAME,true,new DefaultConsumer(channel){
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("recv confirm1 msg:"+new String(body,"utf-8"));
                }
            });
        }
    }
    ```
3. 异步模式 提供一个回调模式
    * 生产者
    ```java
    package cn.glh.rabbitmq.confirm;
    import cn.glh.rabbitmq.util.ConnectionUtils;
    import com.rabbitmq.client.Channel;
    import com.rabbitmq.client.ConfirmListener;
    import com.rabbitmq.client.Connection;
    import java.io.IOException;
    import java.util.Collections;
    import java.util.SortedSet;
    import java.util.TreeSet;
    import java.util.concurrent.TimeoutException;
    /**
    * 普通模式
    */
    public class Send3 {
        //队列名称
        private static final String QUEUE_NAME = "test_simple_confirm3";
        public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
            Connection connectionUtils = ConnectionUtils.getConnectionUtils();
            Channel channel = connectionUtils.createChannel();
            //生产者调用confirmSelect将channel设置成confirm模式
            channel.confirmSelect();
            //未确认的消息
            final SortedSet<Long> confirmSet = Collections.synchronizedSortedSet(new TreeSet<Long>());
            //通道添加监听
            channel.addConfirmListener(new ConfirmListener() {
                //没有问题的handleAck
                @Override
                public void handleAck(long l, boolean b) throws IOException {
                    if (b){
                        System.out.println("------handleAck------multiple");
                        confirmSet.headSet(l+1).clear();
                    }else {
                        System.out.println("------handleAck------multiple false");
                        confirmSet.remove(l);
                    }
                }
                @Override
                public void handleNack(long l, boolean b) throws IOException {
                    if (b){
                        System.out.println("----handleNack-------multiple");
                        confirmSet.headSet(l+1).clear();
                    }else {
                        System.out.println("----handleNack-------multiple false");
                        confirmSet.remove(l);
                    }
                }
            });
            String msg="ssssss";
            while (true){
                long seqNo = channel.getNextPublishSeqNo();
                channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
                confirmSet.add(seqNo);
            }
        }
    }
    ```


















[base64setpermissions]:data:image/png;base64,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

[base64permissions]:data:image/png;base64,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

[base64adddb]:data:image/png;base64,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

[base64adduser]:data:image/png;base64,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