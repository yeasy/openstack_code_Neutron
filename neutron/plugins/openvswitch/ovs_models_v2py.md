#### ovs_models_v2.py
定义了继承自model_base.BASEV2的四个类。

* NetworkBinding代表虚拟网和物理网的绑定。
* TunnelAllocation代表隧道id的分配状态。
* TunnelEndpoint代表隧道的一个端点。
* VlanAllocation代表物理网上的vlan id的分配状态。

这些模型类供ovs_db_v2.py进行使用。
