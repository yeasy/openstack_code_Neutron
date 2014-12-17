### rpc
#### agentnotifiers
主要负责发出一些rpc的通知给agent，包括三个文件：dhcp_rpc_agent_api.py、l3_rpc_agent_api.py、metering_rpc_agent_api.py。
分别实现向dhcp、l3或者metering的agent发出通知消息。

以dhcp_rpc_agent_api.py为例，定义了DhcpAgentNotifyAPI类，该类继承自neutron.common.rpc.RpcProxy。

首先定义允许对agent操作的资源和方法。
```python
VALID_RESOURCES = ['network', 'subnet', 'port']
    VALID_METHOD_NAMES = ['network.create.end',
                          'network.update.end',
                          'network.delete.end',
                          'subnet.create.end',
                          'subnet.update.end',
                          'subnet.delete.end',
                          'port.create.end',
                          'port.update.end',
                          'port.delete.end']
```
实现的方法包括agent_updated()、network_added_to_agent()、network_removed_from_agent()，分别cast一条rpc消息到dhcp agent，调用对应方法。
```python
 def _cast_message(self, context, method, payload, host,
                      topic=topics.DHCP_AGENT):
        """Cast the payload to the dhcp agent running on the host."""
        self.cast(
            context, self.make_msg(method, payload=payload),topic='%s.%s' % (topic, host))

def network_removed_from_agent(self, context, network_id, host):
        self._cast_message(context, 'network_delete_end',
                           {'network_id': network_id}, host)

def network_added_to_agent(self, context, network_id, host):
        self._cast_message(context, 'network_create_end',
                           {'network': {'id': network_id}}, host)

def agent_updated(self, context, admin_state_up, host):
        self._cast_message(context, 'agent_updated',
                           {'admin_state_up': admin_state_up}, host)
```

另外，实现notify()方法，可以调用所允许的方法。
neutron的api中会直接调用notify()方法。
```python
def notify(self, context, data, method_name):
        # data is {'key' : 'value'} with only one key
        if method_name not in self.VALID_METHOD_NAMES:
            return
        obj_type = data.keys()[0]
        if obj_type not in self.VALID_RESOURCES:
            return
        obj_value = data[obj_type]
        network_id = None
        if obj_type == 'network' and 'id' in obj_value:
            network_id = obj_value['id']
        elif obj_type in ['port', 'subnet'] and 'network_id' in obj_value:
            network_id = obj_value['network_id']
        if not network_id:
            return
        method_name = method_name.replace(".", "_")
        if method_name.endswith("_delete_end"):
            if 'id' in obj_value:
                self._notify_agents(context, method_name,
                                    {obj_type + '_id': obj_value['id']},
                                    network_id)
        else:
            self._notify_agents(context, method_name, data, network_id)
```

整体API的agent 通知调用的主要调用结构如下图所示。

![](../../_images/neutron.api.rpc.agentnotifiers.png)

#### handler
包括dvr_rpc.py文件。
其中定义了对dvr服务两端的rpc的api和callback类，包括DVRServerRpcApiMixin、DVRServerRpcCallbackMixin、DVRAgentRpcApiMixin、DVRAgentRpcCallbackMixin。
