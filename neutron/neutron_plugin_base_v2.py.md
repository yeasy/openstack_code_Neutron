## neutron_plugin_base_v2.py
Neutron plugin的抽象基础类，是实现plugin的参考和基础，它定义了实现一个neutron plugin所需的基本接口。
包括下面的方法：

属性 | Create | Delete | Read | Update
--- | --- | --- | --- | ---
port | Y | Y | Y | Y
ports | | | Y |
ports_count | | | Y |
network  | Y | Y | Y | Y
networks  | | | Y |
networks_count | | | Y |
subnet | Y | Y | Y | Y
subnets | | | Y |
subnet_count | | | Y |
