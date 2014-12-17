### netns_cleanup_util.py
清理无用的网络名字空间。当neutron的agent非正常退出时可以通过该工具来清理环境。

主过程十分简单，第一步是获取可能的无用名字空间，第二步是sleep后清除这些名字空间。

```python
def main():
    """Main method for cleaning up network namespaces.

    This method will make two passes checking for namespaces to delete. The
    process will identify candidates, sleep, and call garbage collect. The
    garbage collection will re-verify that the namespace meets the criteria for
    deletion (ie it is empty). The period of sleep and the 2nd pass allow
    time for the namespace state to settle, so that the check prior deletion
    will re-confirm the namespace is empty.

    The utility is designed to clean-up after the forced or unexpected
    termination of Neutron agents.

    The --force flag should only be used as part of the cleanup of a devstack
    installation as it will blindly purge namespaces and their devices. This
    option also kills any lingering DHCP instances.
    """
    conf = setup_conf()
    conf()
    config.setup_logging()

    root_helper = agent_config.get_root_helper(conf)
    # Identify namespaces that are candidates for deletion.
    candidates = [ns for ns in
                  ip_lib.IPWrapper.get_namespaces(root_helper)
                  if eligible_for_deletion(conf, ns, conf.force)]

    if candidates:
        eventlet.sleep(2)

        for namespace in candidates:
            destroy_namespace(conf, namespace, conf.force)
```
