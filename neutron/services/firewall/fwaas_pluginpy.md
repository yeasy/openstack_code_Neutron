#### fwaas_plugin.py
FirewallCallbacks类，继承自rpc.RpcCallback，agent利用rpc来调用该类的方法实现设置防火墙的状态、通知防火墙被删除、获取tenant所拥有的防火墙和规则、获取防火墙所属的租户们的信息。

FirewallAgentApi类，继承自rpc.RpcProxy类，是plugin向agent端发送rpc调用时候，为plugin端使用的方法，包括对防火墙的创建、更新和删除。

FirewallPlugin类，继承自firewall_db.Firewall_db_mixin类，是防火墙这个服务plugin的实现类，其中定义了一系列对防火墙进行操作的方法，包括create_firewall、update_firewall、delete_db_firewall_object、delete_firewall、update_firewall_policy、update_firewall_rule、insert_rule、remove_rule等。
plugin和agent之间的调用关系如下面图表所示。

![防火墙服务plugin和agent之间的rpc消息](../../_images/neutron.services.firewall.png)
