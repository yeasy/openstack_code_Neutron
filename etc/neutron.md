## neutron/

### plugins/
包括bigswitch、brocade、cisco、……等多种插件的配置文件（ini文件）。

### rootwrap.d
包括一系列的filter文件。包括debug.filters
rootwrap是实现让非特权用户以root权限去运行某些命令。这些命令就在filter中指定。
以neutron用户为例，在/etc/sudoers.d/neutron文件中有
```
neutron ALL = (root) NOPASSWD: SETENV: /usr/bin/neutron-rootwrap
```
使得neutron可以以root权限运行neutron-rootwrap。

而在/etc/neutron/rootwarp.conf中定义了
```
filters_path=/etc/neutron/rootwrap.d,/usr/share/neutron/rootwrap,/etc/quantum/rootwrap.d,/usr/share/quantum/rootwrap
```

这些目录中定义了命令的filter，也就是说匹配这些filter中定义的命令就可以用root权限执行了。这些filter中命令的典型格式为
```
cmd-name: filter-name, raw-command, user, args
```

例如/usr/share/neutron/rootwrap/dhcp.filters文件中的如下命令允许以root身份执行ovs-vsctl命令。
```
ovs-vsctl: CommandFilter, ovs-vsctl, root
```

需要注意/etc/neutron/rootwrap.d，/usr/neutron/nova/rootwrap必须是root权限才能修改。

