### neutron-server的rpc
这个rpc服务端主要通过neutron.server中主函数中代码执行
```python
neutron_rpc = service.serve_rpc()
```
方法的实现代码如下
```python
def serve_rpc():
    plugin = manager.NeutronManager.get_plugin()

    # If 0 < rpc_workers then start_rpc_listeners would be called in a
    # subprocess and we cannot simply catch the NotImplementedError.  It is
    # simpler to check this up front by testing whether the plugin supports
    # multiple RPC workers.
    if not plugin.rpc_workers_supported():
        LOG.debug(_("Active plugin doesn't implement start_rpc_listeners"))
        if 0 < cfg.CONF.rpc_workers:
            msg = _("'rpc_workers = %d' ignored because start_rpc_listeners "
                    "is not implemented.")
            LOG.error(msg, cfg.CONF.rpc_workers)
        raise NotImplementedError

    try:
        rpc = RpcWorker(plugin)

        if cfg.CONF.rpc_workers < 1:
            rpc.start()
            return rpc
        else:
            launcher = common_service.ProcessLauncher(wait_interval=1.0)
            launcher.launch_service(rpc, workers=cfg.CONF.rpc_workers)
            return launcher
    except Exception:
        with excutils.save_and_reraise_exception():
            LOG.exception(_('Unrecoverable error: please check log '
                            'for details.'))
```
其中，RpcWorker(plugin)主要通过调用plugin的方法来创建rpc服务端。
```python
self._servers = self._plugin.start_rpc_listeners()
```

该方法在大多数plugin中并未被实现，目前ml2支持该方法。

在neutron.plugin.ml2.plugin.ML2Plugin类中，该方法创建了一个topic为topics.PLUGIN的消费rpc。

```python
def start_rpc_listeners(self):
        self.endpoints = [rpc.RpcCallbacks(self.notifier, self.type_manager),
                          agents_db.AgentExtRpcCallback()]
        self.topic = topics.PLUGIN
        self.conn = n_rpc.create_connection(new=True)
        self.conn.create_consumer(self.topic, self.endpoints,
                                  fanout=False)
        return self.conn.consume_in_threads()
```
