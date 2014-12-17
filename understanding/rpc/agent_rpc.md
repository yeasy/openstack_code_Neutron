### agent端的rpc
在dhcp agent、l3 agent、firewall agent以及metering agent的main函数中都能找到类似的创建一个rpc 服务端的代码，以dhcp agent为例。
```python
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
最核心的，也是跟rpc相关的部分包括两部分，首先是创建rpc服务端。
```python
server = neutron_service.Service.create(
        binary='neutron-dhcp-agent',
        topic=topics.DHCP_AGENT,
        report_interval=cfg.CONF.AGENT.report_interval,
        manager='neutron.agent.dhcp_agent.DhcpAgentWithStateReport')
```
该代码实际上创建了一个rpc服务端，监听指定的topic并定期的运行manager上的定期任务。

create()方法返回一个neutron.service.Service对象，neutron.service.Service继承自neutron.common.rpc.Service类。

首先看neutron.common.rpc.Service类，该类定义了start方法，该方法主要完成两件事情：一件事情是将manager添加到endpoints中；一件是创建了三个rpc的consumer，分别监听topic、topic.host和fanout的队列消息。

而在neutron.service.Service类中，初始化中生成了一个manager实例（即neutron.agent.dhcp_agent.DhcpAgentWithStateReport）；并为start方法添加了周期性执行report_state方法和periodic_tasks方法。report_state方法实际上什么都没做，periodic_tasks方法则调用manager的periodic_tasks方法。

manager实例（即neutron.agent.dhcp_agent.DhcpAgentWithStateReport）在初始化的时候首先创建一个rpc的client端，通过代码
```python
        self.state_rpc = agent_rpc.PluginReportStateAPI(topics.PLUGIN)
```
该client端实际上定义了report_state方法，可以状态以rpc消息的方式发送给plugin。

manager在初始化后，还会指定周期性运行_report_state方法，实际上就是调用client端的report_state方法。

至此，对rpc服务端的创建算是完成了，之后执行代码。
```python
service.launch(server).wait()
```

service.launch(server)方法首先会将server放到协程组中，并调用server的start方法来启动server。
