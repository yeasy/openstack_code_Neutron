## REST API 专题
neutron-server（相当于REST API Server）的核心作用之一就是要实现REST API。

在Neutron中，neutron-server（相当于REST API Server）负责将收到的REST API请求交由Plugin来进行相关处理。可以看出，这其实就是一个web服务器要完成的事情，将http请求转化为对资源的操作（通过plugin的方法调用），并返回响应。
REST API可以分为两类：标准API和扩展API。

前者主要是由原先的 nova-network项目沿用而来，实现对网络二层的支持，核心资源只有网络、端口和子网；后者则通过进行扩展，提供更多的网络服务。目前已有的扩展有L3（router）,L4(TPC/udp firewall)及L7(http/https load balancer)。随着neutron项目的不断成熟，扩展API会演化为标准API。

这个过程的主要涉及的模块包括
neutron.server、service模块，以及定义了neutron API的neutron.api包。

![REST API 实现逻辑](../_images/restapi.png)

以neutron.api.v2包为例，其中base.py中定义了Controller类，是实现URL到plugin的api进行mapping的核心。

其主要方法包括create、delete、index、show、update。

顾名思义，create用于创建资源，delete用于删除资源，show是获取资源、update是更新资源，index是返回请求资源的列表。

以create方法为例来看主要过程。

向network的notifier发送一个资源create.start的通知；然后进行policy的检查；之后调用plugin中相应的方法进行处理，最后向network的notifier发出一个资源create.end的通知。
