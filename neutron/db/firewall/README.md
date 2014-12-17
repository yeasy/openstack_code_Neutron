#### firewall
firewall_db.py中定义了跟防火墙数据库相关的几个类，包括FirewallRule、Firewall、FirewallPolicy和Firewall_db_mixin。
* FirewallRule表示数据库中一条防火墙规则记录；
* Firewall表示数据库中一个防火墙资源记录；
* FirewallPolicy表示数据库中一条防火墙策略记录；
* Firewall_db_mixin表示数据库中防火墙相关操作的实现类，包括创建、删除、更新和获取各种防火墙资源等操作；

![](../../_images/neutron.db.firewall.firewall_db.png)
