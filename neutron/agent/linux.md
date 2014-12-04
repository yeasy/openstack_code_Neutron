### linux/
主要包括跟 Linux 环境相关的一些函数实现，为各种 agent 调用系统命令进行包装，例如对 iptables 操作，ovs 操作等等。

#### async_process.py
实现了AsyncProcess类，对异步进程进行管理。

#### daemon.py
实现一个通用的Daemon基类。一个daemon意味着一个后台进程，可以通过对应的pid文件对其进行跟踪。

#### dhcp.py
实现了Dnsmasq类、DhcpBase类、DhcpLocalProcess类、DeviceManager类、DicModel类、NetModel类。

对 Linux 环境下 dhcp 相关的分配和维护实现进行管理。
通过调用 dnsmasq 工具来管理 dhcp 的分配。

#### external_process.py
定义了ProcessManager类，对neutron孵化出的进程进行管理（可以通过跟踪pid文件进行激活和禁用等）。

#### interface.py
提供对网桥上的接口进行管理的一系列驱动。

定义了常见的配置信息，包括网桥名称，用户和密码等。

定义了几个不同类型网桥的接口驱动类，包括LinuxInterfaceDriver元类和由它派生出来的MetaInterfaceDriver、BridgeInterfaceDriver、IVSInterfaceDriver、MidonetInterfaceDriver、NullDriver和OVSInterfaceDriver等。

其中LinuxInterfaceDriver元类定义了plug()和unplug()两个抽象方法，需要继承类自己来实现。init_l3()方法则提供对接口进行IP地址的配置。

#### ip_lib.py
对ip相关的命令进行封装，包括一些操作类。例如IpAddrCommand、IpLinkCommand、IpNetnsCommand、IpNeighCommand、IpRouteCommand、IpRule等。基本上需要对linux上ip相关的命令进行操作都可以通过这个库提供的接口进行。

#### iptables_firewall.py
利用iptables的规则实现的防火墙驱动，主要包括两个防火墙驱动类。
IptablesFirewallDriver，继承自firewall.FirewallDriver，默认通过iptables规则启用了security group功能，包括添加一条sg-chain链，为每个端口添加两条链（physdev-out和physdev-in）。

OVSHybridIptablesFirewallDriver，继承自IptablesFirewallDriver，基本代码没动，修改了两个获取名字函数的实现：_port_chain_name()和_get_deice_name()。

#### iptables_manager.py
对iptables规则、表资源进行封装，提供操作接口。

定义了IptablesManager类、IptablesRule类、IptablesTable类。

其中IptablesManager对iptables工具进行包装。首先，创建neutron-filter-top链，加载到FORWARD和OUTPUT两条链开头。默认的INPUT、OUTPUT、FORWARD链会被包装起来，即通过原始的链跳转到一个包装后的链。此外，neutron-filter-top链中有一条规则可以跳转到一条包装后的local链。

#### ovs_lib.py
提供对OVS网桥的操作支持，包括一个VifPort，BaseOVS类和继承自它的OVSBridge类。提供对网桥、端口等资源的添加、删除，执行ovs-vsctl命令等。

#### ovsdb_monitor.py
提供对ovsdb的监视器。包括一个OvsdbMonitor类（继承自neutron.agent.linux.async_process.AsyncProcess）和SimpleInterfaceMonitor类（继承自前者）。

#### polling.py
监视ovsdb来决定何时进行polling。包括一个BasePollingManager和继承自它的InterfacePollingMinimizer类等。

#### utils.py
一些辅助函数，包括create_process通过创建一个进程来执行命令、get_interface_mac、replace_file等。
