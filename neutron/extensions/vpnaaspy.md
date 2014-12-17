### vpnaas.py
Vpnaas扩展类，继承自extensions.ExtensionDescriptor，提供安全组扩展支持。

VPNPluginBase抽象基础类，继承自service_base.ServicePluginBase，定义对vpn服务进行管理操作的接口。这个类被vpn_db.py中的VPNPluginDb类继承。
