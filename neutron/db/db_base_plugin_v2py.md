### db_base_plugin_v2.py
其中定义了NeutronDbPluginV2，是各个plugin的基础类。
继承自neutron.neutron_plugin_base_v2.NeutronPluginBaseV2类和neutron.common_db_mixin.CommonDbMixin类。

该类利用SQLAlchemy的模型实现了neutron plugin 的接口，主要包括对网络、子网、端口等资源的 CRUD 操作。
