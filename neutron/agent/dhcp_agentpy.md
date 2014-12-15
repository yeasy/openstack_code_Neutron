### dhcp_agent.py
dhcp 服务的 agent 端，负责实现 dhcp 的分配等。

主要包括 DhcpAgent()类、继承自它的 DhcpAgentWithStateReport 类和继承自 RpcProxy 的 DhcpPluginApi 类。

主函数为
```
def main():
    register_options()
    common_config.init(sys.argv[1:])
    config.setup_logging(cfg.CONF)
    server = neutron_service.Service.create(
        binary='neutron-dhcp-agent',
        topic=topics.DHCP_AGENT,
        report_interval=cfg.CONF.AGENT.report_interval,
        manager='neutron.agent.dhcp_agent.DhcpAgentWithStateReport')
    service.launch(server).wait()
```
读取和注册相关配置（包括dhcpagent、interface_driver、use_namespace等）。
然后创建一个neutron_service。绑定的主题是DHCP_AGENT，默认驱动是Dnsmasq，默认的管理器是DhcpAgentWithStateReport类
然后启动这个service。

dhcp agent的任务包括：汇报状态、处理来自plugin的RPC调用 API、管理dhcp信息。

plugin端的rpc调用方法（一般由neutron.api.v2.base.py发出通知）在neutron.api.rpc.agentnoftifiers.DhcpAgentNotifyAPI()类中实现，其中发出notification消息，会调用agent中对应的方法，包括（其中点符号替换为下划线符号）
```
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

#### DhcpAgent类
继承自manager.Manager类。

manager.Manager类继承自n_rpc.RpcCallback类和periodic_task.PeriodicTasks类，提供周期性运行任务的方法。

初始化方法会首先从配置中导入driver类信息，然后获取admin的上下文。之后创建一个DhcpPluginApi类作为向plugin发出rpc消息的handler。

after_start()方法会调用run()方法，执行将neutron中状态同步到本地和孵化一个新的协程来周期性同步状态。

```
def after_start(self):
        self.run()
        LOG.info(_("DHCP agent started"))

def run(self):
        """Activate the DHCP agent."""
        self.sync_state()
        self.periodic_resync()
```

其中sync_state()会发出rpc消息给plugin，获取最新的网络状态，然后更新本地信息，调用dnsmasq进程使之生效。该方法在启动后运行一次。
periodic_resync()方法则孵化一个协程来运行_periodic_resync_helper()方法，该函数是一个无限循环，它周期性的调用sync_state()。


#### DhcpPluginApi类
提供从agent往plugin一侧进行rpc调用的api。

#### DhcpAgentWithStateReport类
该类继承自DhcpAgent，主要添加了状态汇报。
汇报状态主要是DhcpAgentWithStateReport初始化中指定了一个agent_rpc.PluginReportStateAPI(topics.PLUGIN)类作为状态汇报rpc消息的处理handler。

```
if report_interval:
            self.heartbeat = loopingcall.FixedIntervalLoopingCall(self._report_state)
            self.heartbeat.start(interval=report_interval)
```

这些代码会让_report_state()定期执行来汇报自身状态。
其中_report_state()方法主要代码为：

```
self.agent_state.get('configurations').update(
                self.cache.get_state())
            ctx = context.get_admin_context_without_session()
            self.state_rpc.report_state(ctx, self.agent_state, self.use_call)
```
