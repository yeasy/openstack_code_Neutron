## api-paste.ini
定义了WSGI应用和路由信息。利用Paste来实例化Neutron的APIRouter类，将资源（端口、网络、子网）映射到URL上，以及各个资源的控制器。
在neutron-server启动的时候，一般会指定参数--config-file neutron.conf --config-file xxx.ini。看neutron/server/__init__.py的代码: main()主程序中会调用config.parse(sys.argv[1:])来读取这些配置文件中的信息。而api-paste.ini信息中定义了neutron、neutronapi_v2_0、若干filter和两个app。
```
[composite:neutron]
use = egg:Paste#urlmap
/: neutronversions
/v2.0: neutronapi_v2_0

[composite:neutronapi_v2_0]
use = call:neutron.auth:pipeline_factory
noauth = request_id catch_errors extensions neutronapiapp_v2_0
keystone = request_id catch_errors authtoken keystonecontext extensions neutronapiapp_v2_0

[filter:request_id]
paste.filter_factory = oslo.middleware:RequestId.factory

[filter:catch_errors]
paste.filter_factory = oslo.middleware:CatchErrors.factory

[filter:keystonecontext]
paste.filter_factory = neutron.auth:NeutronKeystoneContext.factory

[filter:authtoken]
paste.filter_factory = keystonemiddleware.auth_token:filter_factory

[filter:extensions]
paste.filter_factory = neutron.api.extensions:plugin_aware_extension_middleware_factory

[app:neutronversions]
paste.app_factory = neutron.api.versions:Versions.factory

[app:neutronapiapp_v2_0]
paste.app_factory = neutron.api.v2.router:APIRouter.factory
```

neutron-server在读取完配置信息后，会执行neutron/common/config.py:load_paste_app(“neutron”)，即将neutron应用load进来。从api-paste.ini中可以看到，neutron实际上是一个composite，分别将URL“/”和“/v2.0”映射到neutronversions应用和neutronapi_v2_0（也是一个composite）。

前者实际上调用了 neutron.api.versions 模块中的 Versions.factory 来处理传入的请求。

后者则要复杂一些，首先调用 neutron.auth 模块中的pipeline_factory 处理。如果是 noauth，则传入参数为 request_id，catch_errors，extensions 这些 filter和 neutronapiapp_v2_0 应用；如果是 keystone，则多传入一个 authtoken filter，最后一个参数仍然是 neutronapiapp_v2_0 应用。来看 neutron.auth 模块中的 pipeline_factory 处理代码。

```
def pipeline_factory(loader, global_conf, **local_conf):
    """Create a paste pipeline based on the 'auth_strategy' config option."""
    pipeline = local_conf[cfg.CONF.auth_strategy]
    pipeline = pipeline.split()
    filters = [loader.get_filter(n) for n in pipeline[:-1]]
    app = loader.get_app(pipeline[-1])
    filters.reverse()
    for filter in filters:
        app = filter(app)
    return app
```

最终的代码入口是neutron.api.v2.router:APIRouter.factory。该方法主要代码为
```
class APIRouter(wsgi.Router):

    @classmethod
    def factory(cls, global_config, **local_config):
        return cls(**local_config)

    def __init__(self, **local_config):
        mapper = routes_mapper.Mapper()
        plugin = manager.NeutronManager.get_plugin()
        ext_mgr = extensions.PluginAwareExtensionManager.get_instance()
        ext_mgr.extend_resources("2.0", attributes.RESOURCE_ATTRIBUTE_MAP)

        col_kwargs = dict(collection_actions=COLLECTION_ACTIONS,
                          member_actions=MEMBER_ACTIONS)

        def _map_resource(collection, resource, params, parent=None):
            allow_bulk = cfg.CONF.allow_bulk
            allow_pagination = cfg.CONF.allow_pagination
            allow_sorting = cfg.CONF.allow_sorting
            controller = base.create_resource(
                collection, resource, plugin, params, allow_bulk=allow_bulk,
                parent=parent, allow_pagination=allow_pagination,
                allow_sorting=allow_sorting)
            path_prefix = None
            if parent:
                path_prefix = "/%s/{%s_id}/%s" % (parent['collection_name'],
                                                  parent['member_name'],
                                                  collection)
            mapper_kwargs = dict(controller=controller,
                                 requirements=REQUIREMENTS,
                                 path_prefix=path_prefix,
                                 **col_kwargs)
            return mapper.collection(collection, resource,
                                     **mapper_kwargs)

        mapper.connect('index', '/', controller=Index(RESOURCES))
        for resource in RESOURCES:
            _map_resource(RESOURCES[resource], resource,
                          attributes.RESOURCE_ATTRIBUTE_MAP.get(
                              RESOURCES[resource], dict()))

        for resource in SUB_RESOURCES:
            _map_resource(SUB_RESOURCES[resource]['collection_name'], resource,
                          attributes.RESOURCE_ATTRIBUTE_MAP.get(
                              SUB_RESOURCES[resource]['collection_name'],
                              dict()),
                          SUB_RESOURCES[resource]['parent'])
```

neutron server启动后，根据配置文件动态加载对应的core plugin 和 service plugin。neturon server 中会对收到的 rest api 请求进行解析，并最终转换成对该 plugin(core or service) 中相应方法的调用。

