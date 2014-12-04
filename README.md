OpenStack Neutron 源码分析
============
[Neutron](https://wiki.openstack.org/wiki/Neutron) 是 OpenStack 项目中负责提供网络服务的组件，它基于软件定义网络的思想，实现了网络虚拟化下的资源管理。

本书将剖析 Neutron 组件的代码。

最新版本在线阅读：[GitBook](https://www.gitbook.io/book/yeasy/openstack_code_Neutron)。

本书源码在 Github 上维护，欢迎参与： [https://github.com/yeasy/openstack_code_Neutron](https://github.com/yeasy/openstack_code_Neutron)。

感谢所有的 [贡献者](https://github.com/yeasy/openstack_code_Neutron/graphs/contributors)。

## 更新历史:
* V0.71: 2014-08-07
    * 添加更多细节分析，添加对ML2的分析。
* V0.7: 2014-07-18
    * 完成对cmd、common和db部分的分析；
    * 整体代码框架分析完毕。
* V0.6: 2014-07-11
	* 完成对api部分的分析；
    * 增加目录；
	* 增加新的一章，集中从专题角度剖析代码。
* V0.5: 2014-07-07
	* 完成对agent部分的补充修订。
* V0.4: 2014-05-19
	* 完成对OpenvSwitch plugin的分析。
* V0.3: 2014-05-12
	* 完成对IBM 的SDN-VE plugin的分析。
*V0.2: 2014-05-06
	* 完成配置文件（etc/）相关分析。
* V0.1: 2014-04-14
	* 完成代码基本结构。


## 参加步骤
* 在 GitHub 上 `fork` 到自己的仓库，如 `user/openstack_code_Neutron`，然后 `clone` 到本地，并设置用户信息。
```
$ git clone git@github.com:user/openstack_code_Neutron.git
$ cd openstack_code_Neutron
$ git config user.name "User"
$ git config user.email user@email.com
```

* 修改代码后提交，并推送到自己的仓库。
```
$ #do some change on the content
$ git commit -am "Fix issue #1: change helo to hello"
$ git push
```

* 在 GitHub 网站上提交 pull request。
* 定期使用项目仓库内容更新自己仓库内容。
```
$ git remote add upstream https://github.com/yeasy/openstack_code_Neutron
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
$ git push -f origin master
```
