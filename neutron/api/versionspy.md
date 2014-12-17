### versions.py
当rest api请求是版本号时候，调用该模块中的类Versions进行处理。
绑定也是在api-paste.ini文件中。
```
/: neutronversions
[app:neutronversions]
paste.app_factory = neutron.api.versions:Versions.factory
```

调用的是Versions类中的factory()方法，该方法返回一个类的实体。该类是一个callable对象，主要函数就是`__call__()`方法。
```python
@webob.dec.wsgify(RequestClass=wsgi.Request)
    def __call__(self, req):
        """Respond to a request for all Neutron API versions."""
        version_objs = [
            {
                "id": "v2.0",
                "status": "CURRENT",
            },
        ]

        if req.path != '/':
            language = req.best_match_language()
            msg = _('Unknown API version specified')
            msg = gettextutils.translate(msg, language)
            return webob.exc.HTTPNotFound(explanation=msg)

        builder = versions_view.get_view_builder(req)
        versions = [builder.build(version) for version in version_objs]
        response = dict(versions=versions)
        metadata = {
            "application/xml": {
                "attributes": {
                    "version": ["status", "id"],
                    "link": ["rel", "href"],
                }
            }
        }

        content_type = req.best_match_content_type()
        body = (wsgi.Serializer(metadata=metadata).
                serialize(response, content_type))

        response = webob.Response()
        response.content_type = content_type
        response.body = body

        return response
```
