## RabbitMQ在Spring boot中的实践（Topic模式）

### 一、依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

### 二、发送方配置

+ 创建exchange
+ 对RabbitTemplate配置Jackson2JsonMessageConverter编码

```java
@Configuration
public class RabbitMQConfiguration {

    @Bean
    public TopicExchange topicExchange(@Value("${topicExchangeName}") final String exchangeName){
        return new TopicExchange(exchangeName);
    }

    @Bean
    public RabbitTemplate rabbitTemplate(final ConnectionFactory connectionFactory){
        final RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(producerJackson2MessageConverter());
        return rabbitTemplate;
    }

    @Bean
    public Jackson2JsonMessageConverter producerJackson2MessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

}
```

### 三、接收方配置

+ 创建相同的exchange
+ 创建队列Queue
+ 将队列绑定到交换，并赋予接收规则的key
+ 同样需要修改编码方式，在RabbitListenerEndpointRegistrar中设置

```java
@Configuration
public class RabbitMQConfiguration implements RabbitListenerConfigurer {
    @Bean
    public TopicExchange topicExchange(@Value("${topicExchangeName}") final String exchangeName){
        return new TopicExchange(exchangeName);
    }

    @Bean
    public Queue inferQueue(){
        return new Queue("infer", true);
    }

    @Bean
    public Queue impressionQueue(){
        return new Queue("impression",true);
    }

    @Bean
    public Queue interviewQueue(){
        return new Queue("interview", true);
    }

    @Bean
    public Binding bindinginfer(Queue inferQueue, TopicExchange exchange){
        return BindingBuilder.bind(inferQueue).to(exchange).with("*.infer");
    }

    @Bean
    public Binding bindingimpression(Queue impressionQueue, TopicExchange exchange){
        return BindingBuilder.bind(impressionQueue).to(exchange).with("*.impression");
    }

    @Bean
    public Binding bindinginterview(Queue interviewQueue, TopicExchange exchange){
        return BindingBuilder.bind(interviewQueue).to(exchange).with("*.interview");
    }

    @Bean
    public MappingJackson2MessageConverter consumerJackson2MessageConverter() {
        return new MappingJackson2MessageConverter();
    }

    @Bean
    public DefaultMessageHandlerMethodFactory messageHandlerMethodFactory() {
        DefaultMessageHandlerMethodFactory factory = new DefaultMessageHandlerMethodFactory();
        factory.setMessageConverter(consumerJackson2MessageConverter());
        return factory;
    }

    @Override
    public void configureRabbitListeners(final RabbitListenerEndpointRegistrar registrar) {
        registrar.setMessageHandlerMethodFactory(messageHandlerMethodFactory());
    }
}
```

### 四、发送方发送消息类

+ 使用rabbitTemplate.convertAndSend(topicExchange, keys[1], impression)方法，参数分别为exchange名、发送的key、对象

```java
@Component
public class EventDispatcher {
    private RabbitTemplate rabbitTemplate;
    private String topicExchange;
    private final String[] keys = {"publish.infer", "publish.impression", "publish.interview"};

    @Autowired
    EventDispatcher(final RabbitTemplate rabbitTemplate,
                    @Value("${topicExchangeName}") String topicExchange){
        this.rabbitTemplate = rabbitTemplate;
        this.topicExchange = topicExchange;
    }

    public void sendInfer(final Infer infer){
        rabbitTemplate.convertAndSend(topicExchange, keys[0], infer);
    }

    public void sendImpression(final Impression impression){
        rabbitTemplate.convertAndSend(topicExchange, keys[1], impression);
    }

    public void sendInterview(final Interview interview){
        rabbitTemplate.convertAndSend(topicExchange, keys[2], interview);
    }
}
```

### 五、接收方接收消息类

+ 在相应的方法上加@RabbitListener注解，并注明Queue名

```java
@Component
public class EventHandler {
    private ImpressionService impressionService;
    private InferService inferService;
    private InterviewService interviewService;

    EventHandler(final ImpressionService impressionService,
                 final InferService inferService,
                 final InterviewService interviewService){
        this.impressionService = impressionService;
        this.inferService = inferService;
        this.interviewService = interviewService;
    }

    @RabbitListener(queues = "infer")
    void handleInfer(final Infer infer){
        inferService.handleInfer(infer);
    }

    @RabbitListener(queues = "impression")
    void handleImpression(final Impression impression){
        impressionService.handleImpression(impression);
    }

    @RabbitListener(queues = "interview")
    void handleInterview(final Interview interview){
        interviewService.handleInterview(interview);
    }
}
```

### 六、注意的地方

+ 发送对象需要实现Serializable接口，如果需要自己实现对象编码成byte[]，看这篇[博文](https://blog.csdn.net/east123321/article/details/78900791)

### 七、其他资料

+ 交换机有四种类型：Direct, topic, Headers and Fanout

  此[文章](https://juejin.im/post/5cefc04251882510eb758630)介绍了其中3种模式的基础用法

+ 上述只列举了RabbitMQ的初级用法，更多高级用法与高级配置见[这里](https://www.cnblogs.com/sw008/p/11054293.html)

+ [官网](https://www.rabbitmq.com/tutorials/tutorial-six-spring-amqp.html)的例子也不错，给出了除了spring领域的其他用法

+ 最后给两个rabbitmq的全面教程，可以拿来全面学习

  [教程1](https://blog.csdn.net/hellozpc/article/details/81436980)，[教程2](https://www.cnblogs.com/yihuihui/p/9127300.html)