### plugin端的rpc
以openvswitch的plugin为例进行分析。

neutron.plugin.openvswitch.ovs_neutron_plugin.OVSNeutronPluginV2类在初始化的时候调用了self.setup_rpc方法。

其代码为
```python
def setup_rpc(self):
        # RPC support
        self.service_topics = {svc_constants.CORE: topics.PLUGIN,
                               svc_constants.L3_ROUTER_NAT: topics.L3PLUGIN}
        self.conn = n_rpc.create_connection(new=True)
        self.notifier = AgentNotifierApi(topics.AGENT)
        self.agent_notifiers[q_const.AGENT_TYPE_DHCP] = (
            dhcp_rpc_agent_api.DhcpAgentNotifyAPI()
        )
        self.agent_notifiers[q_const.AGENT_TYPE_L3] = (
            l3_rpc_agent_api.L3AgentNotifyAPI()
        )
        self.endpoints = [OVSRpcCallbacks(self.notifier, self.tunnel_type),
                          agents_db.AgentExtRpcCallback()]
        for svc_topic in self.service_topics.values():
            self.conn.create_consumer(svc_topic, self.endpoints, fanout=False)
        # Consume from all consumers in threads
        self.conn.consume_in_threads()
```

创建一个通知rpc的客户端，用于向l2的agent发出通知。所有plugin都需要有这样一个发出通知消息的客户端。

分别创建了一个dhcp agent和l3 agent的通知rpc客户端。

之后，创建两个跟service agent相关的consumer，分别监听topics.PLUGIN和topics.L3PLUGIN两个主题。

![plugin端的rpc](../../_images/openvswitch_plugin_rpc.png)
