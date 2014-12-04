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
* init.d/neutron-server：neutron-server系统服务脚本，支持start、stop、restart和status操作。
* neutron/: 核心的库代码。
* plugins/: 各种厂商的plugin相关的配置文件（*.ini），其中被注释掉的行表明了（不指明情况下的）默认值。
* rootwrap.d/: 一些filters文件，用来限定各个模块执行命令的权限。
各种ini和conf文件，包括api-paste.ini、dhcp_agent.ini、l3_agent.ini、fwaas_driver.ini、lbaas_agent.ini、metadata_agent.ini、metering_agent.ini，以及neutron.conf、policy.json、rootwrap.conf、services.conf等。
基本上neutron相关的各个组件的配置信息都在这里了。

## neutron
核心的代码实现都在这个目录下。
可以通过下面的命令来统计主要实现的核心代码量。
```
find neutron  -name "*.py" | xargs cat | wc -l
```
目前版本，约为226k行。

## tools
一些相关的代码格式化检测、环境安装的脚本。
