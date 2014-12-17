## Agent专题
agent一般作为plugin的后端，接收远端plugin的命令来具体实施对本地网络资源的操作，并同时汇报需要的信息给远端的plugin。

agent可以分为四种：核心agent、dhcp agent、l3 agent、其他agent。
核心agent用于服务核心plugin，提供L2功能，所以又被称为L2 agent。该agent在所有的计算和网络节点均存在。

dhcp agent用于提供对虚机的dhcp管理和分配。核心plugin在实现上通过继承DhcpAgentSchedulerDbMixin来实现对dhcp agent的操作。dhcp agent可以安装在任何节点上，并支持可以配置多个实例，还支持配置的的调度策略。
l3 agent用于提供对路由器的管理（配置iptables规则）。l3 agent也可以安装在任何节点上，并支持可以配置多个实例，还支持配置的的调度策略。

其它 agent，用于实现其它的网络服务等，包括metadata、metering等。
从agent的结构上来看，主要包括两部分：跟plugin之间的rpc消息处理，以及本地的操作。前者主要用于跟plugin进行交互，接收plugin命令、调用plugin方法、汇报状态等；后者用于对本地资源进行实际的操作，包括配置iptables表项，配置dhcp服务等。
