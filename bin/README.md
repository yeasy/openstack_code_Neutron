# bin
* neutron-rootwrap文件，python可执行文件
* neutron-rootwrap-xen-dom0文件，python可执行文件。
提供利用root权限执行命令时候的操作接口，通过检查，可以配置不同用户利用管理员身份执行命令的权限。其主要实现是利用了oslo.rootwrap 包中的cmd模块。
