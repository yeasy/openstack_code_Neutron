# 整体结构

源代码主要分为5个目录和若干文件：
bin，doc，etc，neutron和tools。
除了这 5 个目录外，还包括一些说明文档、安装需求说明文件等。

## bin
主要包括 neutron-rootwrap、neutron-rootwrap-xen-dom0 两个文件。
提供一些可执行命令。

## doc
包括文档生成的相关源码。

## etc
跟服务和配置相关的文件，基本上该目录中内容在安装时会被复制到系统的/etc/ 目录下。
* init.d/neutron-server：neutron-server 系统服务脚本，支持 start、stop、restart 和 status 操作。
* neutron/: 核心的库代码。
* plugins/: 各种厂商的 plugin 相关的配置文件（*.ini），其中被注释掉的行表明了默认值。
* rootwrap.d/: 一些 filters 文件，用来限定各个模块执行命令的权限。
各种 ini 和 conf 文件，包括 api-paste.ini、dhcp_agent.ini、l3_agent.ini、fwaas_driver.ini、lbaas_agent.ini、metadata_agent.ini、metering_agent.ini，以及 neutron.conf、policy.json、rootwrap.conf、services.conf 等。
基本上 neutron 相关的各个组件的配置信息都在这里了。

## neutron
核心的代码实现都在这个目录下。
可以通过下面的命令来统计主要实现的核心代码量。
```
find neutron  -name "*.py" | xargs cat | wc -l
```
目前版本，约为226k行。

## tools
一些相关的代码格式化检测、环境安装的脚本。

## 其它文档
* README.rst：介绍了项目的情况和连接。
* TESTING.rst：介绍如何进行开发后的测试。官方配置的 jenkins 当 gerrit 上有代码提交 review 的时候会触发 tox 测试。实际上，OpenStack 中的项目使用 [tox](http://tox.readthedocs.org/en/latest/) 来管理测试的虚拟环境，使用 [testr](https://wiki.openstack.org/wiki/Testr) 来管理运行测试案例的顺序。
* run_tests.sh：自动创建一个虚拟环境并进行测试。命令为 `./run_tests -V`。
