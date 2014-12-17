## Plugin专题
plugin实现对REST API请求的后端支持。前端的rest框架会调用plugin的相应方法来实现rest api规定的语义。

plugin包括两种类型：核心plugin和服务plugin。

核心Plugin实现了对标准API的支持（对网络、端口和子网资源的相应操作），此外还可以选择性的支持一些扩展的API（_supported_extension_aliases属性中会指出）。
以neutron的openvswitch plugin为例，支持的扩展API包括
```python
_supported_extension_aliases = ["provider", "external-net", "router",
                                    "ext-gw-mode", "binding", "quotas",
                                    "security-group", "agent", "extraroute",
                                    "l3_agent_scheduler",
                                    "dhcp_agent_scheduler",
                                    "extra_dhcp_opt",
                                    "allowed-address-pairs"]
```
服务plugin则专门用于实现扩展API。目前，路由器、防火墙、vpn等都有相应的service plugin来专门实现。

neturon server启动时候，根据配置文件动态加载相应的核心plugin及服务plugins。
目前，neutron-server只能配置一个核心Plugin，但可以配置多个服务plugin（但每个extension的实现plugin不能超过一个）。

ML2 core plugin允许同时加载多个mechanism driver，可以达到同时使用不同的L2实现技术的效果。
