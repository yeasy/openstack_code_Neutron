## agent
在 neutron 的架构中，各种 agent 运行在计算节点和网络节点上，接收来自 neutron-server 的 plugin 的指令，对所管理的网桥进行实际的操作，属于“直接干活”的部分。plugin 和 agent 之间进行双向交互，一般的，每个 plugin 会创建一个 RPC server 来监听 agent 的请求。

agent 可以大致分为 core agent、dhcp、l3 和其它（metadata等）。
本部分代码实现各种 agent 所需要的操作接口和库函数。
