## extensions
对现有neutron API的扩展。某些plugin可能还支持额外的资源或操作，可以以extension的方式实现。包括firewall、vpnaas、l3、lbaas等。

这些扩展资源类，大部分都继承自neutron.api.extensions.ExtensionDescriptor类，一般都实现了如下的类方法。
get_name、get_alias、get_description、get_namespace、get_updated、get_extended_resources。
