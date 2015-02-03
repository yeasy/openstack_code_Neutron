#### agent
包括 agent.py、agent_api.py、agent_device_driver.py
agent_manager.py 等模块，实现 LBaaS 服务的 agent 部分。

##### agent.py
按照标准流程，启动了一个 LbaasAgentService 服务，主代码如下。
```python
def main():
    cfg.CONF.register_opts(OPTS)
    cfg.CONF.register_opts(manager.OPTS)
    # import interface options just in case the driver uses namespaces
    cfg.CONF.register_opts(interface.OPTS)
    config.register_interface_driver_opts_helper(cfg.CONF)
    config.register_agent_state_opts_helper(cfg.CONF)
    config.register_root_helper(cfg.CONF)

    common_config.init(sys.argv[1:])
    config.setup_logging()

    mgr = manager.LbaasAgentManager(cfg.CONF)
    svc = LbaasAgentService(
        host=cfg.CONF.host,
        topic=topics.LOADBALANCER_AGENT,
        manager=mgr
    )
    service.launch(svc).wait()
```
其中，管理服务类为 agent_manager 中的 LbaasAgentManager 类。

LbaasAgentService 类是一个 rpc 服务类，定期执行管理服务类中的周期性任务。
```python
class LbaasAgentService(n_rpc.Service):
    def start(self):
        super(LbaasAgentService, self).start()
        self.tg.add_timer(
            cfg.CONF.periodic_interval,
            self.manager.run_periodic_tasks,
            None,
            None
        )
```

##### agent_manager.py
主要就是定义了 LbaasAgentManager 类，继承自 n_rpc.RpcCallback 和 periodic_task.PeriodicTasks。

作为 agent 中的管理类。它实现了 LBaaS 中定义的各种操作 API，包括：
* agent_updated
* collect_stats
* create_member
* create_pool
* create_pool_health_monitor
* create_vip
* delete_member
* delete_pool
* delete_pool_health_monitor
* delete_vip
* initialize_service_hook
* periodic_resync
* remove_orphans
* sync_state
* update_member
* update_pool
* update_pool_health_monitor
* update_vip
这些操作又是通过调用更下一层的 driver 来实现。

同时，它定义了一个 rpc api 调用成员，向 topics.LOADBALANCER_PLUGIN 主题发送消息。
```python
self.plugin_rpc = agent_api.LbaasAgentApi(
            topics.LOADBALANCER_PLUGIN,
            self.context,
            self.conf.host
        )
```

##### agent_api.py
主要定义了 LbaasAgentApi 类，继承自 n_rpc.RpcProxy，也是一个典型的 RPC 代理类。

这个类在 LbaasAgentManager 类中使用，用来负责替 agent 向指定的主题发送消息，调用监听者的对应方法。主要包括：
* get_logical_device
* get_ready_devices
* plug_vip_port
* pool_deployed
* pool_destroyed
* unplug_vip_port
* update_pool_stats
* update_status

##### agent_device_driver.py
主要定义了 AgentDeviceDriver 类，是一个抽象类，抽象出来支持 LBaaS agent 所需要 API 的一个驱动类。这些 API 包括对 pool、member 等资源的 CRUD 等。

