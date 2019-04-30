## WeEvent

关于`WeEvent`使用的一些常见问题和答案都会整理归档到这里。

简单的接口调用异常一般从错误码中就可以得到答案[WeEvent错误码](../protocal/errorcode.html) 。有任何意见和建议，欢迎你参与`WeEvent`项目讨论[WeEvent Issues](https://github.com/WeBankFinTech/WeEvent/issues) 。

- 怎么部署`WeEvent`和`FISCO-BCOS`？

  建议将`WeEvent`和`FISCO-BCOS`节点部署在同一网段/逻辑区，比如`DMZ`区。

- `WeEvent`的服务访问授权机制是怎么样的？

  `WeEvent`访问权限控制基于`HTTPS` + `IP`白名单，`STOMP`和`MQTT`协议还支持协议定义的账号/密码机制。

- 如何选择各种协议，`JsonRPC`、`RESTful`、`STOMP`还是`MQTT`？
  - Java程序
    - 如果是`Spring`服务，有内置的`org.springframework.boot:spring-boot-starter-websocket`支持，推荐使用`MQTT`。
    - 如果是其他`Java`程序，建议使用`WeEvent`提供的`Java SDK`。
  - 其他语言
    - 生产者`Producer`建议使用`RESTful`/`JsonRPC` ，简单方便。
    - 消费者`Consumer`因为涉及到事件的持续`Push`，建议使用`STOMP`协议接入。
  - `MQTT`主要面向物联网`IoT`设备的接入。

- `WeEvent`服务的高可用性方案是怎么样的？

  `WeEvent`除了订阅之外的功能，都是无状态的请求，通过多实例的负载均衡来保证高可用性。订阅功能按是否为长连接分成两类：

    - 使用长连接的协议，比如`STOMP`
        这部分功能基于长连接，订阅上下文以连接为中心，连接关闭则订阅会自动关闭。
        这种属于集群`Cluster`方案。`WeEvent`通过`Nginx`的负载均衡`Load balance`实现。  
    - 使用短链接的协议，比如`JsonRPC`/`RESTful`/`MQTT`
        这部分功能基于短连接，事件订阅的上下文不依赖于连接，连接关闭后订阅仍然存在。
        这种属于主备`Replication`方案。`WeEvent`通过`Zookeeper`实现，如果没有为`WeEvent`服务配置`Zookeeper`服务，则无法使用这部分功能。

- `WeEvent`怎么处理发布事件的去重？

  `WeEvent`不处理发布事件的去重。

  当服务超时或者机器宕机时，一般的逻辑是重试，这种策略很容易出现重复请求。但是整个业务系统内，不只在`WeEevnt`服务的边界会出现这种情况，任何一个服务边界都会出现。 建议业务统一处理，比如在事件内容里带一个唯一ID`UUID`，消费的时候使用`UUID`字段来判断该事件是否已经处理过。

  注意：`EventID`不是用来去重的，同一事件每成功发布一次，都会生成不同的`EventID`。

- 服务状态检查脚本`check-service.sh`出现`"deploy contract failed"`
  - 检查`WeEvent`到`FISCO-BCOS`的连接及其配置。
  - 检查`FISCO-BCOS`节点是否正常出块。

- `WeEvent`事件内容支持的最大长度是多少？

  事件内容最大长度为10 K字节 `Byte`， 字符集编码为`UTF8`。

- `WeEvent`到区块链`FISCO-BCOS`节点之间的网络故障怎么处理？

  建议为`WeEvent`配置2个属于不同网段的`FISCO-BCOS`节点。这样`WeEvent`服务会随机选用一个可用的节点使用，做到一定程度的网络容灾。
