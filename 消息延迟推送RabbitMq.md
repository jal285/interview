# 基于RabbitMq实现的消息延迟推送

<!--more-->

本文来自公众号 ["程序员修炼秘籍"](https://mp.weixin.qq.com/s?__biz=MzkwNTI1NjcxMQ==&mid=2247485821&idx=1&sn=82e775f26dc8b83569521371a9a44eef&chksm=c0fbc315f78c4a033249a65e4c3d13de6258edfc0d7d06d5c866bd23f4127ee9551848a3a6eb&mpshare=1&scene=23&srcid=0913qQsEGq7upmm2jtdAiddT&sharer_sharetime=1631511616571&sharer_shareid=8898e746b0f249ecdba4fe363392aada#rd)

RabbitMQ 3.6.x之前我们一般采用死信队列+TTL过期时间来实现延迟队列，我们这里不做过多介绍，可以参考之前文章来了解：[TTL、死信队列](https://www.cnblogs.com/haixiang/p/10905189.html)

RabbitMQ 3.6.x开始,RabbitMQ官方提供了延迟队列的插件,可以直接下载到RabbitMQ根目录下的plugins下,[延迟队列插件下载](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)

添加依赖

配置.propreties，开启 confirm 确认机制，开启 return 确认模式，设置 `mandatory`属性 为 true，当设置为 true 的时候，路由不到队列的消息不会被自动删除，从而才可以被 return 消息模式监听到。

```java
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
spring.rabbitmq.connection-timeout=15000

#开启 confirm 确认机制
spring.rabbitmq.publisher-confirms=true
#开启 return 确认机制
spring.rabbitmq.publisher-returns=true
#设置为 true 后 消费者在消息没有被路由到合适队列情况下会被return监听，而不会自动删除
spring.rabbitmq.template.mandatory=true
```

创建消息队列和交换机，此处不应该创建 ConnectionFactory 和 RabbitAdmin，应该在 application.properties 中设置用户名、密码、host、端口、虚拟主机即可。

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class MQConfig {

    public static final String LAZY_EXCHANGE = "Ex.LazyExchange";
    public static final String LAZY_QUEUE = "MQ.LazyQueue";
    public static final String LAZY_KEY = "lazy.#";

    @Bean
    public TopicExchange lazyExchange(){
        //Map<String, Object> pros = new HashMap<>();
        //设置交换机支持延迟消息推送
        //pros.put("x-delayed-message", "topic");
        TopicExchange exchange = new TopicExchange(LAZY_EXCHANGE, true, false, pros);
        exchange.setDelayed(true);
        return exchange;
    }

    @Bean
    public Queue lazyQueue(){
        return new Queue(LAZY_QUEUE, true);
    }

    @Bean
    public Binding lazyBinding(){
        return BindingBuilder.bind(lazyQueue()).to(lazyExchange()).with(LAZY_KEY);
    }
}
```

通过Exchange声明中设置exchange.setDelayed(true) 来开启延迟队列,也可以设置为以下内容传入交换机声明的方法中,

```
  //Map<String, Object> pros = new HashMap<>();
        //设置交换机支持延迟消息推送
        //pros.put("x-delayed-message", "topic");
        TopicExchange exchange = new TopicExchange(LAZY_EXCHANGE, true, false, pros);
```

发送消息时我们需要指定延迟推送的时间，我们这里在发送消息的方法中传入参数 `new MessagePostProcessor()` 是为了获得 `Message`对象，因为需要借助 `Message`对象的api 来设置延迟时

```java
import com.anqi.mq.config.MQConfig;
import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageDeliveryMode;
import org.springframework.amqp.core.MessagePostProcessor;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class MQSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    //confirmCallback returnCallback 代码省略，请参照上一篇

    public void sendLazy(Object message){
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setConfirmCallback(confirmCallback);
        rabbitTemplate.setReturnCallback(returnCallback);
        //id + 时间戳 全局唯一
        CorrelationData correlationData = new CorrelationData("12345678909"+new Date());

        //发送消息时指定 header 延迟时间
        rabbitTemplate.convertAndSend(MQConfig.LAZY_EXCHANGE, "lazy.boot", message,
                new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                //设置消息持久化
                message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                //message.getMessageProperties().setHeader("x-delay", "6000");
                message.getMessageProperties().setDelay(6000);
                return message;
            }
        }, correlationData);
    }
}
```

可以观察`setDelay(Integer i)`底层代码，也是在 header 中设置 x-delay。等同于我们手动设置 header

```
message.getMessageProperties().setHeader("x-delay", "6000");
```

```java
/**
 * Set the x-delay header.
 * @param delay the delay.
 * @since 1.6
 */
public void setDelay(Integer delay) {
    if (delay == null || delay < 0) {
        this.headers.remove(X_DELAY);
    }
    else {
        this.headers.put(X_DELAY, delay);
    }
}
```

消费端进行消费

```
import com.rabbitmq.client.Channel;
import org.springframework.amqp.rabbit.annotation.*;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Map;

@Component
public class MQReceiver {

    @RabbitListener(queues = "MQ.LazyQueue")
    @RabbitHandler
    public void onLazyMessage(Message msg, Channel channel) throws IOException{
        long deliveryTag = msg.getMessageProperties().getDeliveryTag();
        channel.basicAck(deliveryTag, true);
        System.out.println("lazy receive " + new String(msg.getBody()));

    }
```

测试结果

```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@SpringBootTest
@RunWith(SpringRunner.class)
public class MQSenderTest {

    @Autowired
    private MQSender mqSender;

    @Test
    public void sendLazy() throws  Exception {
        String msg = "hello spring boot";

        mqSender.sendLazy(msg + ":");
    }
}
```

6秒后收到消息"lazy receive hello spring boot"
