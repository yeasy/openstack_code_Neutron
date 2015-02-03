## Agent专题
agent 一般作为 plugin 的后端，接收远端 plugin 的命令来具体实施对本地网络资源的操作，并同时反馈信息给远端的 plugin。

agent 可以分为四种：核心 agent、dhcp agent、l3 agent、其它 agent。
核心 agent 用于服务核心plugin，提供 L2 功能，所以又被称为 L2 agent。该 agent 在所有的计算和网络节点均存在。

dhcp agent 用于提供对虚机的 dhcp 管理和分配。核心 plugin 在实现上通过继承 DhcpAgentSchedulerDbMixin 来实现对 dhcp agent 的操作。dhcp agent 可以安装在任何节点上，并支持可以配置多个实例，还支持配置的的调度策略。

l3 agent 用于提供对路由器的管理（配置 iptables规则）。l3 agent 也可以安装在任何节点上，并支持可以配置多个实例，还支持配置的的调度策略。

其它 agent，用于实现其它的网络服务等，包括 metadata、metering 等。
从 agent 的结构上来看，主要包括两部分：跟 plugin 之间的 rpc 消息处理，以及本地的操作。前者主要用于跟 plugin 进行交互，接收 plugin 命令、调用 plugin 方法、汇报状态等；后者用于对本地资源进行实际的操作，包括配置 iptables 表项，配置 dhcp 服务等。
