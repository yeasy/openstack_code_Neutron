### metadata/

#### agent.py
主要包括MetadataProxyHandler、UnixDomainHttpProtocol、WorkerService、UnixDomainWSGIServer、UnixDomainMetadataProxy几个类和一个main函数。
该文件的主逻辑代码为：
```
cfg.CONF.register_opts(UnixDomainMetadataProxy.OPTS)
    cfg.CONF.register_opts(MetadataProxyHandler.OPTS)
    cache.register_oslo_configs(cfg.CONF)
    cfg.CONF.set_default(name='cache_url', default='memory://?default_ttl=5')
    agent_conf.register_agent_state_opts_helper(cfg.CONF)
    config.init(sys.argv[1:])
    config.setup_logging(cfg.CONF)
    utils.log_opt_values(LOG)
    proxy = UnixDomainMetadataProxy(cfg.CONF)
    proxy.run()
```

在读取相关配置完成后，则实例化一个UnixDomainMetadataProxy，并调用其run函数。run函数则进一步创建一个`server = UnixDomainWSGIServer('neutron-metadata-agent')` 对象，并调用其start()和wait()函数。

run函数会将应用绑定到MetadataProxyHandler()类，该类包括一个`__call__`函数，调用_proxy_request()对传入的HTTP请求进行处理。

#### namespace_proxy.py
定义了UnixDomainHTTPConnection、NetworkMetadataProxyHandler、ProxyDaemon三个类和主函数。主函数代码为

```
    eventlet.monkey_patch()
    opts = [
        cfg.StrOpt('network_id',
                   help=_('Network that will have instance metadata '
                          'proxied.')),
        cfg.StrOpt('router_id',
                   help=_('Router that will have connected instances\' '
                          'metadata proxied.')),
        cfg.StrOpt('pid_file',
                   help=_('Location of pid file of this process.')),
        cfg.BoolOpt('daemonize',
                    default=True,
                    help=_('Run as daemon.')),
        cfg.IntOpt('metadata_port',
                   default=9697,
                   help=_("TCP Port to listen for metadata server "
                          "requests.")),
        cfg.StrOpt('metadata_proxy_socket',
                   default='$state_path/metadata_proxy',
                   help=_('Location of Metadata Proxy UNIX domain '
                          'socket'))
    ]

    cfg.CONF.register_cli_opts(opts)
    # Don't get the default configuration file
    cfg.CONF(project='neutron', default_config_files=[])
    config.setup_logging(cfg.CONF)
    utils.log_opt_values(LOG)
    proxy = ProxyDaemon(cfg.CONF.pid_file,
                        cfg.CONF.metadata_port,
                        network_id=cfg.CONF.network_id,
                        router_id=cfg.CONF.router_id)

    if cfg.CONF.daemonize:
        proxy.start()
    else:
        proxy.run()
```

其基本过程也是读取完成相关的配置信息，然后启动一个ProxyDaemon实例，以daemon或run方法来运行。run方法则创建一个wsgi服务器，然后运行。最终绑定的应用为NetworkMetadataProxyHandler。

```
proxy = wsgi.Server('neutron-network-metadata-proxy')
proxy.start(handler, self.port)
proxy.wait()
```
