#### sdnve_api.py
封装sdnve控制器所支持的操作为一些API。
RequestHandler类，处理与sdnve控制器的请求和响应消息的基本类。提供get、post、put、delete等请求。对HTTP消息处理的实现通过其内部的httplib2.Http成员来进行。
Client类，继承自RequestHandler类。提供对sdnve中各种网络资源（网络，子网，端口，租户，路由器，浮动IP）的CRUD操作的API和对应实现。
KeystongClient类，主要是获取系统中的租户信息。
