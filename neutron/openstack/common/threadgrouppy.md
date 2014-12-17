#### threadgroup.py
定义了类Thread和ThreadGroup。

前者是对greenthread的封装，拥有一个到threadgroup的引用，当线程结束时候通知threadgroup将自身移除。

后者对greenthread进行管理，可以为greenthread添加一个计时器。
