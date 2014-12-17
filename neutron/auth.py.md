## auth.py
定义了类NeutronKeystoneContext，可以利用keystone的头部信息来生成一次请求的上下文。
定义了方法pipeline_factory，是一个流水线处理，对给定的auth_strategy（例如filter1 filter2 filter3 ... app），逆序调用各个过滤器对app进行处理，并返回最终结果。
