## service.py
定义了相关的配置信息，包括periodic_interval，api_workers，rcp_workers，periodic_fuzzy_delay。

实现neutron中跟服务相关的类。

包括NeutronApiService，RpcWorker，继承自n_rpc.Service的Service和WsgiService。

Service是各个服务的基类。

WsgiService是实现基于WSGI的服务的基础类。

NeutronApiService继承自WsgiService，添加了create()方法，配置log相关的选项，并返回类实体。

#### Service类
Service是很重要的一个概念，各个服务的组件都以Service类的方式来进行交互。此处Service类继承自rpc中的Service，整体的继承关系为
neutron.openstack.common.service.Service类-->neutron.common.rpc.Service类-->neutron.service.Service类。

其中neutron.openstack.common.service.Service类定义了简单的reset()、start()、stop()和wait()方法。该类初始化后会维护一个线程组。

neutron.common.rpc.Service类中进一步丰富了start()和stop()方法，并在初始化中引入了host、topic、manager和serializer参数。

start()增加创建了Connection对象，之后创建了三个consumer，分别监听主题为参数传入的topic（fanout分别为True和False），以及主题为topic.host。然后调用manager的初始化。最后作为server启动所有的consumer。

neutron.service.Service类的初始化中更进一步的增加了binary、report_interval、periodic_interval、periodic_fuzzy_delay等参数。除丰富了start()、stop()和wait()方法外，还增加了create()类方法、kill()、periodic_tasks()和report_state()。

start()增加了周期性执行report_state()和periodic_tasks()，并且调用manager的init_host()和after_start()方法。

create()方法是类方法，它根据传入的参数binary参数获取真实的程序名，并在未给定参数的情况下尝试从配置文件中解析manager和report_interval、periodic_interval、periodic_fuzzy_delay等参数。最后是返回生成的Service类对象。

report_state()方法仅定义了接口。

periodic_tasks()则首先获取admin的上下文，然后调用manager的periodic_tasks()方法执行。

总结一下，neutron.service.Service类，初始化会处理传入参数，并解析配置文件。start()方法则创建并启动三个consumer，监听传入的topic和topic.host。初始化manager并周期性运行它的periodic_tasks()和report_state()方法。
