#### sdnve_neutron_plugin.py
主要定义了 SdnvePluginV2 类，继承自如下几个基础类：
* db_base_plugin_v2.NeutronDbPluginV2：提供在数据库中对网络、子网、端口的CRUD操作API；
* external_net_db.External_net_db_mixin：为db_base_plugin_v2添加对外部网络的操作方法；
* portbindings_db.PortBindingMixin：端口绑定相关的操作；
* l3_gwmode_db.L3_NAT_db_mixin：添加可配置的网关模式，为端口和网络提供字典风格的扩展函数。
* agents_db.AgentDbMixin：为db_base_plugin_v2添加agent扩展，对agent的创建、删除、获取等。

SdnvePluginV2类实现了neutron中定义的API，实现基于SDN-VE对上提供网络抽象的支持。包括对网络、子网、端口、路由器等资源的CRUD操作。
