### agent.py
主要定义了两个类：Agent和AgentPluginBase。

前者提供对agent管理扩展；后者提供对agent进行操作的rest API，包括对agent的CRUD操作，在agent_db.py中被AgentDbMixin类继承。
