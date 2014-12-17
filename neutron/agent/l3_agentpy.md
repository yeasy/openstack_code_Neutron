### l3_agent.py
提供L3层服务的agent，包括L3NATAgent类、继承自它的L3NATAgentWithStateReport类（作为manager）、继承自n_rpc.RpcProxy类的L3PluginApi类（作为agent调用plugin一侧的api）和RouterInfo类。

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

也是标准的 service 流程，启动一个管理neutron-l3-agent执行程序的服务，该服务将监听topic为topics.L3_AGENT的rpc消息队列。

#### L3NATAgent类
继承自firewall_l3_agent.FWaaSL3AgentRpcCallback和manager.Manager两个类。
前者是由于现在的FWaaS设计都是挂载到router上的，因此，在创建router的时候，需要把对应的firewall添加上。不得不说这是个十分不合理的临时方案。

而Manager作为一个进行rpc调用管理和执行周期性任务的基础类。
初始化中根据配置信息，导入driver，获取admin的上下文，获取L3PluginApi，然后定期执行self._rpc_loop()方法。该方法根据数据库中的信息来同步本地的router。

调用self._process_routers()方法和self._process_router_delete()方法，这两个方法会进一步对本地的iptables进行操作，完成router的添加或删除。

#### L3PluginApi类
继承自neutron.common.rpc.RpcProxy类，是一个进行rpc调用的代理。
被L3NATAgent类来调用，负责向L3的Plugin发出rpc消息（主题为topics.L3PLUGIN），这些消息到达plugin，最终被plugin的父类neutron.db.l3_rpc_base.L3RpcCallbackMixin类中的对应方法来处理，这些方法进一步调用父类neutron.db.l3_db.L3_NAT_db_mixin类中的对应方法跟数据库进行交互。

目前定义了三个方法：get_external_network_id()通过rpc调用external_network_id()来获取外部网络的id；get_routers()通过rpc调用sync_routers()来获取所有的router的信息；update_floatingip_statuses()通过rpc调用update_floatingip_statuses()来更新flaoting ip的状态。

#### L3NATAgentWithStateReport类
该类是L3 agent资源service的manager，其继承自L3NATAgent，并添加了rpc.PluginReportStateAPI类来进行周期性状态汇报。该类会以topic.PLUGIN向rpc队列中写入report_state()方法，并携带agent的状态信息作为参数。这些消息会被各个plugin收到。
