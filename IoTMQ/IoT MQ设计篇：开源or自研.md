## IoT MQ设计篇：开源or自研，系统复杂度分析

### 概述

上一篇介绍了IoT MQ的一些基本知识以及与Kafka这类“系统级别”的MQ的区别，同时简单介绍了使用最广的两种物联网通信协议coap与mqtt并最终决定使用mqtt作为基础协议，本篇主要介绍IoT MQ在进行设计时考虑到的一些问题：

1. IoT MQ需求及系统复杂度分析
2. 开源or自研

### 需求及系统复杂度分析

因为初始阶段对于IoT MQ并不是很了解，也从未涉及过物联网领域，所以初步从两个方面考虑：功能方面与系统方面。

#### 功能层面

1. 该IoT MQ应该能对接入的客户端进行管理，包括统计，监控流量，强制下线，数据传输等
2. 能对集群，broker主机进行监控等
3. 该IoT MQ应该能上云进行分析，主要是通过桥接RocketMQ等上行到其它存储平台进行数据分析

#### 系统层面

1. 服务高可用：不能有单点故障等问题，所以该IoT MQ必须要能集群部署，数据必须有一定可靠性，不能出现数据大量丢失的情况
2. 高连接高请求：也就是高性能，不过要求单机（4c16g）要支持十万级以上的连接，并且消息TPS要高，同时响应时间低
3. 高可扩展性：架构总是演进的，需求也一直在变，才开始的IoT MQ只满足了最低的需求，所以应该是高可扩展性的。
4. 安全：物联网一定是公网的，所以需要考虑传输加密以及权限验证等安全，要实现TLS/SSL以及权限验证
5. 规模：初始规模一般预估为真实需求的2到5倍，我们初始评估长连接设备数20W，消息大小为200byte的消息的TPS至少达到10000，响应时间在100ms内
6. 成本：集群部署为了节省主机成本所以单机性能有一定要求，人员方面技能储备以java为主

### 开源or自研

在对功能和系统进行分析后，我们有了两种基本的方案：1.基于开源项目二次开发，2.完全自研。我们先对比一下这两种选择的优劣势：

**优势：**

* 开源项目一般已经有人维护，如果社区活跃的也有很多解决方案，技术成本较低，开发者的压力也较低

* 不用重复造轮子，重新走老路，时间成本等也降低了

**劣势：**

* 开源项目一般是针对“通用”的场景，大改起来比较麻烦，特别是设计不好的开源项目
* 开源项目的可靠性，可用性等需要考量，安全，灵活性等也降低了

前期对于时间成本，技术学习成本都比较看重，所以找个合适的开源项目进行二次开发也是很好的选择，[这里有所有的mqtt server](https://github.com/mqtt/mqtt.github.io/wiki/brokers)

下面是一些mqtt服务的对比：

| Server                                                       | QoS 0 | QoS 1 | QoS 2 | auth | [bridge](https://github.com/mqtt/mqtt.github.io/wiki/bridge_protocol) | [$SYS](https://github.com/mqtt/mqtt.github.io/wiki/conventions#%24sys) | SSL  | [dynamic topics](https://github.com/mqtt/mqtt.github.io/wiki/are_topics_dynamic) | cluster | websockets | plugin system |
| ------------------------------------------------------------ | ----- | ----- | ----- | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- | ------------------------------------------------------------ | ------- | ---------- | ------------- |
| [2lemetry](http://2lemetry.com/platform/)                    | ✔     | ✔     | ✔     | ✔    | ✔                                                            | §                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✘             |
| [Jmqtt](https://github.com/Cicizz/jmqtt)                     | ✔     | ✔     | ✔     | ✔    | ✔                                                            | §                                                            | §    | ✔                                                            | §       | ✔          | ✔             |
| [Apache ActiveMQ](http://activemq.apache.org/)               | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✔             |
| [Apache ActiveMQ Artemis](http://activemq.apache.org/artemis) | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✔             |
| [Bevywise IoT Platform](https://www.bevywise.com/iot-platform/) | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✔    | ✔                                                            | ✔       | ✔          | **rm**        |
| [emitter](https://github.com/emitter-io/emitter)             | ✔     | §     | ✘     | ✔    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✘             |
| [emqttd](http://emqtt.io/)                                   | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✔             |
| [flespi](https://flespi.com/mqtt-broker)                     | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✘             |
| [GnatMQ](https://github.com/ppatierno/gnatmq)                | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✘                                                            | ✘    | ✔                                                            | ✘       | ✘          | ✘             |
| [HBMQTT](https://github.com/beerfactory/hbmqtt)              | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✔                                                            | ✔    | ✔                                                            | ✘       | ✔          | ✔             |
| [HiveMQ](http://www.hivemq.com/)                             | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✔             |
| [IBM MessageSight](http://www-03.ibm.com/software/products/en/messagesight/) | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✔                                                            | ✔    | ✔                                                            | §       | ✔          | ✘             |
| [JoramMQ](http://mqtt.jorammq.com/)                          | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✔             |
| [Mongoose](https://github.com/cesanta/mongoose)              | ✔     | ✔     | ?     | ?    | ?                                                            | ?                                                            | ?    | ?                                                            | ?       | ?          | ?             |
| [moquette](https://github.com/andsel/moquette)               | ✔     | ✔     | ✔     | ✔    | ?                                                            | ?                                                            | ✔    | ?                                                            | **rm**  | ✔          | ✘             |
| [mosca](https://github.com/mqtt/mqtt.github.io/wiki/mosca)   | ✔     | ✔     | ✘     | ✔    | ?                                                            | ?                                                            | ?    | ?                                                            | ✘       | ✔          | ✘             |
| [mosquitto](https://github.com/mqtt/mqtt.github.io/wiki/mosquitto_message_broker) | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✔    | ✔                                                            | §       | ✔          | ✔             |
| [MQTT.js](https://github.com/mqttjs/MQTT.js)                 | ✔     | ✔     | ✔     | §    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ✘       | ✔          | ✘             |
| [MqttWk](https://github.com/Wizzercn/MqttWk)                 | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ?                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✘             |
| [RabbitMQ](http://www.rabbitmq.com/blog/2012/09/12/mqtt-adapter/) | ✔     | ✔     | ✘     | ✔    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ?       | ?          | ?             |
| [RSMB](https://github.com/mqtt/mqtt.github.io/wiki/Really-Small-Message-Broker) | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✘    | ✔                                                            | ✘       | ✘          | ?             |
| [Software AG Universal Messaging](http://um.terracotta.org/#page/%2Fum.terracotta.org%2Funiversal-messaging-webhelp%2Fto-mqttoverview.html%23) | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ✔       | rm         | ✘             |
| [Solace](http://dev.solacesystems.com/tech)                  | ✔     | ✔     | ✘     | ✔    | §                                                            | ✔                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✘             |
| [SwiftMQ](http://www.swiftmq.com/landing/router/index.html)  | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✘                                                            | ✔    | ✔                                                            | ✔       | ✘          | ✔             |
| [Trafero Tstack](https://github.com/trafero/tstack)          | ✔     | ✔     | ✔     | ✔    | ✘                                                            | ✘                                                            | ✔    | ✔                                                            | ✘       | ✘          | ✘             |
| [VerneMQ](https://verne.mq/)                                 | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✔    | ✔                                                            | ✔       | ✔          | ✔             |
| [WebSphere MQ](http://www-03.ibm.com/software/products/en/wmq/) | ✔     | ✔     | ✔     | ✔    | ✔                                                            | ✔                                                            | ✔    | ✔                                                            | ?       | ?          | ?             |

我们可以看到目前已经有很多mqtt的broker了，因为技术储备是影响我们决策最重要的一个因素，所以我们选择了java开发的Moquette为研究对象，其中基于erlang语言的**EMQTT**，基于cpp的**Mosquitto**也是经常使用的两种mqtt broker.