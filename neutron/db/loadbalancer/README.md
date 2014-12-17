#### loadbalancer
loadbalancer_db.py中定义了跟负载均衡服务相关的几个数据库资源和操作类。
资源类都继承自model_base.BASE2类，此外还根据需求继承了其他几个类增添属性。
* SessionPersistence类表示数据库中一条session的持久化类型；
* PoolStatistics类表示数据库中一个pool的一些统计信息；
* Vip类表示数据库中一个VIP记录；
* Member类表示一个负载均衡的成员；
* Pool类表示一个负载均衡池资源；
* HealthMonitor类表示一个负载均衡的健康状态监视器资源；
* PoolMonitorAssociation类表示Pool到HealthMonitor的关联关系。
* LoadBalancerPluginDb则继承自loadbalancer.LoadBalancerPluginBase和base_db.CommonDbMixin，代表对负载均衡相关的数据资源进行操作实现。

![](../../_images/neutron.db.loadbalancer.loadbalancer_db.png)
