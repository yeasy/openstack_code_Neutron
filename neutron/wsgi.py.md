## wsgi.py
WorkerService类，封装被ProcessLauncher处理的一个工作者。
* Server 类，管理多个WSGI socket和应用。
* Middleware 类，封装基本的WSGI中间件。
* Request 类，继承自webob.Request类，代表请求信息。
* ActionDispatcher 类，通过行动名将方法名映射到本地方法。
* Application 类，是WSGI应用的封装基础类。
* Debug 类，用于进行debug的中间件。
* Router 类，WSGI中间件，将到达的请求发送给应用。
* Resource类，继承自Application类，是wsgi应用，用于处理序列化和控制器分发。
* Controller类，是一个WSGI的应用，根据请求，调用响应的方法。
