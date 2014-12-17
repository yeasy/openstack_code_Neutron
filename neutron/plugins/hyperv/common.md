#### common
这里面的文件主要是定义一些常量。

config.py定义了配置选项（关键词）和默认值等，包括sdnve_opts和sdnve_agent_opts两个配置组，并且将这些配置项导入到全局的cfg.CONF中。只要导入该模块，相应的配置组和配置选项就会被认可合法，从而可以通过解析配置文件中这些关键词，而为这些配置选项赋值；

constants.py则分别定义了一些固定的常量；
exceptions.py中定义了一些异常类型。
