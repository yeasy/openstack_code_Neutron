#### agent
主要包括xenapi目录（xen相关）、ovs_neutron_agent.py和ovs_dvr_neutron_agent.py文件（运行在各个节点上的对网桥进行操作的代理）。
ovs_neutron_agent.py文件main函数主要过程如下：
```python
def main():
    cfg.CONF.register_opts(ip_lib.OPTS)
    common_config.init(sys.argv[1:])
    common_config.setup_logging(cfg.CONF)
    logging_config.setup_logging(cfg.CONF)
    q_utils.log_opt_values(LOG)

    try:
        agent_config = create_agent_config_map(cfg.CONF)
    except ValueError as e:
        LOG.error(_('%s Agent terminated!'), e)
        sys.exit(1)

    is_xen_compute_host = 'rootwrap-xen-dom0' in agent_config['root_helper']
    if is_xen_compute_host:
        # Force ip_lib to always use the root helper to ensure that ip
        # commands target xen dom0 rather than domU.
        cfg.CONF.set_default('ip_lib_force_root', True)

    agent = OVSNeutronAgent(**agent_config)
    signal.signal(signal.SIGTERM, handle_sigterm)

    # Start everything.
    LOG.info(_("Agent initialized successfully, now running... "))
    agent.daemon_loop()
    sys.exit(0)
```

首先是读取各种配置信息，然后提取agent相关的属性。

然后生成一个agent实例（OVSNeutronAgent类），并调用其daemon_loop()函数，该函数进一步执行rpc_loop()。

OVSNeutronAgent类初始化的时候，会依次调用setup_rpc()、setup_integration_br()和setup_physical_bridges()。

ovs_dvr_neutron_agent.py文件是neutron实现分布式路由器设计时的agent。

##### setup_rpc()
setup_rpc()首先创建了两个rpc句柄，分别是
```python
self.plugin_rpc = OVSPluginApi(topics.PLUGIN)
self.state_rpc = agent_rpc.PluginReportStateAPI(topics.PLUGIN)
```

其中，前者是与neutron-server（准确的说是ovs plugin）进行通信，通过rpc消息调用plugin的方法。这些消息在neutron.agent.rpc.PluginApi类中定义，包括get_device_details、get_devices_details_list、update_device_down、update_device_up、tunnel_sync、security_group_rules_for_devices等。

后者是agent定期将自身的状态上报给neutron-server。

之后，创建dispatcher和所关注的消息主题：
```python
self.dispatcher = self.create_rpc_dispatcher()
        # Define the listening consumers for the agent
        consumers = [[topics.PORT, topics.UPDATE],
                     [topics.NETWORK, topics.DELETE],
                     [constants.TUNNEL, topics.UPDATE],
                     [topics.SECURITY_GROUP, topics.UPDATE]]
```
这样，neutron-server发到这四个主题的消息，会被agent接收到。agent会检查端口是否在本地，如果在本地则进行对应动作。
创建消费者rpc连接来接收rpc消息：
```python
self.connection = agent_rpc.create_consumers(self.dispatcher,
                                                     self.topic,
                                                     consumers)
```
最后，创建heartbeat，定期的调用self._report_state()，通过state_rpc来汇报本地状态。
图表 3展示了这一过程，是一个很好的理解agent端的rpc实现的例子。

![Openvswitch中agent端rpc的实现](../../_images/neutron.plugin.openvswitch.agent.ovs_neutron_agent.OVSNeutronAgent.png)

##### setup_integration_br()
清除integration网桥上的int_peer_patch_port端口和流表，添加一条normal流。

##### setup_physical_bridges()
创建准备挂载物理网卡的网桥，添加一条normal流，然后创建veth对，连接到integration网桥，添加drop流规则，禁止未经转换的流量经过veth对。
