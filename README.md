# compare_flask_django_request
比较flask和django的请求过程

## django的一次请求

1： 首先会去调用setting中WSGI_APPLICATION配置的调用函数。

```
def get_wsgi_application():
    """
    The public interface to Django's WSGI support. Should return a WSGI
    callable.

    Allows us to avoid making django.core.handlers.WSGIHandler public API, in
    case the internal WSGI implementation changes or moves in the future.
    """
    django.setup(set_prefix=False)
    return WSGIHandler()
```
