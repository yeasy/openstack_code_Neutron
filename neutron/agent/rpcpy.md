### rpc.py
定义了create_consumer()方法，设置agent 进行RPC时候的消费者。

定义了PluginApi类和PluginReportStateAPI类。两者都是继承自rpc.RpcProxy类。
前者代表rpc API在agent一侧部分，用于agent调用plugin的方法。后者是agent汇报自身状态，用于向plugin汇报状态信息。

PluginApi类包括四个方法：get_device_details()、tunnel_sync()、update_device_down()和update_device_up()。

PluginReportStateAPI类只提供一个方法：report_state，将agent获取的本地的状态信息以rpc消息的方式发出去。
