### l3_agent.py
提供 L3 层服务的 agent，包括 L3NATAgent 类、继承自它的 L3NATAgentWithStateReport 类（作为manager）、继承自 n_rpc.RpcProxy 类的 L3PluginApi 类（作为 agent 调用 plugin 一侧的 api）和 RouterInfo 类。

主过程为
```
def main(manager='neutron.agent.l3_agent.L3NATAgentWithStateReport'):
    _register_opts(cfg.CONF)
    common_config.init(sys.argv[1:])
    config.setup_logging()
    server = neutron_service.Service.create(
        binary='neutron-l3-agent',
        topic=topics.L3_AGENT,
        report_interval=cfg.CONF.AGENT.report_interval,
        manager=manager)
    service.launch(server).wait()
```

也是标准的 service 流程。

从 conf.CONF 中注册各个系统常量，包括 L3 相关 agent 的参数、接口驱动、命名空间等等。

启动一个管理 neutron-l3-agent 执行程序的服务，该服务将监听 topic 为 topics.L3_AGENT 的 RPC 消息队列，管理类为 L3NATAgentWithStateReport。

#### L3NATAgent 类
继承自 firewall_l3_agent.FWaaSL3AgentRpcCallback、l3_ha_agent.AgentMixin 和 manager.Manager。

前者是由于现在的 FWaaS 设计都是挂载到 router 上的，因此，在创建router 的时候，需要把对应的 firewall 添加上。

而 Manager 作为一个进行rpc调用管理和执行周期性任务的基础类。
初始化中根据配置信息，导入 driver，获取 admin 的上下文，获取 L3PluginApi，然后定期执行 self._rpc_loop() 方法。该方法根据数据库中的信息来同步本地的 router。

调用 self._process_routers() 方法和 self._process_router_delete() 方法，这两个方法会进一步对本地的 iptables 进行操作，完成 router 的添加或删除。

#### L3NATAgentWithStateReport类
该类是 L3 agent 资源 service 的 manager，其继承自 L3NATAgent，并添加了 rpc.PluginReportStateAPI 类来进行周期性状态汇报。

主要添加了两个方法。
* `_report_state()`：定期的汇报自己的状态，以 `topic.PLUGIN` 作为主题向 rpc 队列中写入 agent 的状态信息消息。这些消息会被各个 plugin 收到。
* `agent_updated()`：收到 agent_updated 消息的处理。

#### L3PluginApi 类
继承自 `neutron.common.rpc.RpcProxy` 类，是一个进行 rpc 调用的代理。

被 L3NATAgent 类来调用，负责向 L3 的 Plugin 发出 rpc 消息（主题为 `topics.L3PLUGIN`），这些消息到达 plugin，最终被 plugin 的父类 neutron.db.l3_rpc_base.L3RpcCallbackMixin 类中的对应方法来处理，这些方法进一步调用父类 `neutron.db.l3_db.L3_NAT_db_mixin` 类中的对应方法跟数据库进行交互。

目前定义了下面几个方法：
* `get_routers()` 通过 rpc 调用 L3 Plugin 的 sync_routers() 方法来来获取外部网络的id。
* `get_external_network_id()` 通过 rpc 调用 external_network_id() 来获取外部网络的id。
* `update_floatingip_statuses()` 通过 rpc 调用 update_floatingip_statuses() 来更新 floating ip 的状态。
* `get_ports_by_subnet()` 通过 rpc 调用 get_ports_by_subnet() 来获取对应 subnet 中的端口信息。
* `get_service_plugin_list()` 通过 rpc 调用 get_service_plugin_list() 来获取 L3 Plugin 中激活服务的列表。

