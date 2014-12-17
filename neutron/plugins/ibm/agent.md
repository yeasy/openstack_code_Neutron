#### agent
sdnve_neutron_agent.py，该文件主要实现一个在计算节点和网络节点上的daemon，对本地的网桥进行实际操作。其主要过程代码为
```python
def main():
    eventlet.monkey_patch()
    cfg.CONF.register_opts(ip_lib.OPTS)
    cfg.CONF(project='neutron')
    logging_config.setup_logging(cfg.CONF)

    try:
        agent_config = create_agent_config_map(cfg.CONF)
    except ValueError as e:
        LOG.exception(_("%s Agent terminated!"), e)
        raise SystemExit(1)

    plugin = SdnveNeutronAgent(**agent_config)

    # Start everything.
    LOG.info(_("Agent initialized successfully, now running... "))
    plugin.daemon_loop()
```

其中，eventlet.monkey_patch()是使用eventlet的patch，将本地的一些python库进行绿化，使之支持协程。
```python
cfg.CONF.register_opts(ip_lib.OPTS)
cfg.CONF(project='neutron')
logging_config.setup_logging(cfg.CONF)
```

这三行则初始化配置信息。register_opts是注册感兴趣的关键字并设置它们的默认值，只有感兴趣的关键字才会被后面的步骤进行配置更新。

最关键的cfg.CONF(project='neutron')，这其实是个函数调用，实际上调用了cfg.ConfigOpts类的__call__方法，来解析project参数所指定的相关配置文件，并从中读取配置信息。需要注意的是，外部的sys.argv参数会传递给所import的cfg模块进行解析。因此，如果在启动agent的时候通过命令行给出了参数，则cfg.ConfigOpts类会解析这些命令行参数。否则，将默认去~/.${project}/、~/、/etc/${project}/、/etc/等地方搜索配置文件（默认为os.path.basename(sys.argv[0])）。如果不进行这一步，那么所有的关键字只带有默认的信息，配置文件中信息就不起作用了。
```python
try:
        agent_config = create_agent_config_map(cfg.CONF)
    except ValueError as e:
        LOG.exception(_("%s Agent terminated!"), e)
        raise SystemExit(1)
```

这部分则试图从全局配置库中读取agent相关的一些配置项。包括网桥、接口mapping、控制器IP等等。

后面部分是实例化一个SdnveNeutronAgent类，并调用它的daemon_loop()方法。
