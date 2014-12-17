### model_base.py
定义了继承自 oslo.db.sqlalchemy.models.ModelBase的NeutronBase 和继承自其的 NeutronBaseV2 类。

![](../../_images/neutron.db.model_base.png)

实际上，oslo.db.sqlalchemy.models.ModelBase是sqlalchemy的模型基础类，因此继承自其的类自动完成映射，可以对数据库进行操作。

最后，还定义了BASEV2作为其他地方可以继承的模型类。
```
BASEV2 = declarative.declarative_base(cls=NeutronBaseV2)
```
