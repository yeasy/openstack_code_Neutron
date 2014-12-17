# server
实现neutron-server的主进程。

__init__.py文件中包括一个main()函数，是WSGI服务器开始的模块，并且通过调用serve_wsgi来创建一个NeutronApiService的实例。然后通过eventlet的greenpool来运行WSGI的应用程序，响应来自客户端的请求。
主要过程为：
```python
   eventlet.monkey_patch()
```
绿化各个模块为支持协程（通过打补丁的方式让本地导入的库都支持协程）。
```python
config.parse(sys.argv[1:])
if not cfg.CONF.config_file:
        sys.exit(_("ERROR: Unable to find configuration file via the default"
                   " search paths (~/.neutron/, ~/, /etc/neutron/, /etc/) and"
                   " the '--config-file' option!"))
```
通过解析命令行传入的参数，获取配置文件所在。
```python
pool = eventlet.GreenPool()
```
创建基于协程的线程池。
```python
neutron_api = service.serve_wsgi(service.NeutronApiService)
api_thread = pool.spawn(neutron_api.wait)
```
serve_wsgi方法创建NeutronApiService实例（作为一个WsgiService），并调用其的start()来启动socket服务器端。
```python
#neutron.service
def serve_wsgi(cls):
    try:
        service = cls.create()
        service.start()
    except Exception:
        with excutils.save_and_reraise_exception():
            LOG.exception(_('Unrecoverable error: please check log '
                            'for details.'))
    return service
```
neutron.service.NeutronApiService类继承自neutron.service.WsgiService，其create方法返回一个appname默认为“neutron”的WsgiService对象；start方法则调用_run_wsgi方法。
```python
def start(self):
        self.wsgi_app = _run_wsgi(self.app_name)
```

_run_wsgi方法主要是从api-paste.ini文件中读取应用（最后是利用neutron.api.v2.router:APIRouter.factory来构造应用），然后为应用创建一个wsgi的服务端，并启动应用，主要代码为。
```python
def _run_wsgi(app_name):
    app = config.load_paste_app(app_name)
    if not app:
        LOG.error(_('No known API applications configured.'))
        return
    server = wsgi.Server("Neutron")
    server.start(app, cfg.CONF.bind_port, cfg.CONF.bind_host,
                 workers=cfg.CONF.api_workers)
    # Dump all option values here after all options are parsed
    cfg.CONF.log_opt_values(LOG, std_logging.DEBUG)
    LOG.info(_("Neutron service started, listening on %(host)s:%(port)s"),
             {'host': cfg.CONF.bind_host,
              'port': cfg.CONF.bind_port})
    return server
```

至此，neutron server启动完成，之后，需要创建rpc服务端。
```python
try:
            neutron_rpc = service.serve_rpc()
except NotImplementedError:
            LOG.info(_("RPC was already started in parent process by plugin."))
else:
            rpc_thread = pool.spawn(neutron_rpc.wait)
            rpc_thread.link(lambda gt: api_thread.kill())
            api_thread.link(lambda gt: rpc_thread.kill())
```

这些代码创建plugin的rpc服务端，并将api和rpc的生存绑定到一起，一个死掉，则另外一个也死掉。
```python
pool.waitall()
```
最后是后台不断等待。

下面的图表总结了neutron-server的核心启动过程。

neutron.server|neutron.service#serve_wsgi() | neutron.service.NeutronApiService|neutron.service.WsgiService |neutron.service#_run_wsgi()
--- | --- | --- | ---
neutron_api = service.serve_wsgi(service.NeutronApiService)	|service = cls.create(); service.start()|service = cls(app_name=’neutron’) |self.wsgi_app = _run_wsgi(self.app_name)| app = config.load_paste_app(app_name); server = wsgi.Server("Neutron"); server.start(app, cfg.CONF.bind_port, cfg.CONF.bind_host,            workers=cfg.CONF.api_workers)
