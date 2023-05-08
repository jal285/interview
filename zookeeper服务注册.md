# Zookeeper 服务注册与发现

> zookeeper作为一个分布式协调服务 

## 服务注册与发现 demo

![](D:\program\writing\images\2022-05-20-15-45-44-image.png)

- 将产品服务的信息注册到zookeeper的节点上
- 然后获取到节点上的信息并存储起来（本文存到List）
- Watcher机制监控List里数据的变化并更新数据 （假如产品服务2挂了通过监听机制将其移出）
- 利用轮询或者hash等算法去获取List里的数据供订单服务调用（负载均衡）

![](D:\program\writing\images\2022-05-20-16-54-36-image.png)

### 需要注意点

> **注意** ：节点创建需要父节点存在
> 
> 例如创建 /service/product 
> 
> 需要存在/service 

## 关于注册中心

> CAP 和BASE理论 成了指导分布式系统及互联网应用构建的关键原则之一

注册中心最本质的功能可以看成是一个Query函数 `Si = F(service-name)`，以 `service-name` 为查询参数，`service-name` 对应的服务的可用的 `endpoints (ip:port)` 列表为返回值.
