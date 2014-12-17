### config.py
对配置进行管理。
定义了core_opts的属性和默认值，包括绑定的主机地址、端口、配置文件默认位置、策略文件位置、VIF的起始Mac地址、DNS数量、子网的主机路由限制、DHCP释放时间、nova的配置信息等。

定义了core_cli_opts的属性和默认值，包括状态文件的路径。
注册上面定义的配置项。

主要包括
load_paste_app(app_name)方法，从默认的paste config文件来读取配置，生成并返回WSGI应用。最关键的逻辑实现是
```
app = deploy.loadapp("config:%s" % config_path, name=app_name)
```

init(args)方法，读入配置文件，调用rpc的初始化函数，并检查base_mac参数是否合法。
setup_logging(conf)方法，配置logging模块，导入配置信息。
