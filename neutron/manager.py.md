## manager.py
定义了类Manager和类NeutronManager。

前者继承自neutron.common.rpc.RpcCallback类和neutron.openstack.common.periodic_task.PeriodicTasks类。该类会定义运行任务。

后者负责解析配置文件并初始化neutron的plugin。
