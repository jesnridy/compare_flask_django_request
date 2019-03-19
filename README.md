# compare_flask_django_request
比较flask和django的请求过程

## django的一次请求

1： 首先会去调用setting中WSGI_APPLICATION配置的调用方法。
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

2：调用方法返回类WSGIHandle，他继承了base.BaseHandler， 看了下这个基类中几个有意思的方法。

    2.1：`load_middleware`： 导入setting.MIDDLEWARE的模块，。
    2.2：`make_view_atomic`：事务化视图方法，就是针对数据库做了atomic操作。
        
        def make_view_atomic(self, view):
            non_atomic_requests = getattr(view, '_non_atomic_requests', set())
            # connectons.all()引用了db.utils中的类ConnectionHandler(object)
            for db in connections.all():
                if db.settings_dict['ATOMIC_REQUESTS'] and db.alias not in non_atomic_requests:
                    view = transaction.atomic(using=db.alias)(view)
            return view
    2.3：`get_exception_response`：获取response异常信息。
    2.4：`process_exception_by_middleware`：用中间件中引入的exception_middleware处理异常信息。
    2.5：`handle_uncaught_exception`：可以重写这个方法处理未被捕获的异常。
 
 3：WSGIHandle先做了load_middleware操作这个操作写在构造方法中，只调用一次。
    并且定义了一个类变量`request_class = WSGIRequest`。
    ```
    class WSGIHandler(base.BaseHandler):
    request_class = WSGIRequest

    def __init__(self, *args, **kwargs):
        super(WSGIHandler, self).__init__(*args, **kwargs)
        self.load_middleware()

    def __call__(self, environ, start_response):
        set_script_prefix(get_script_name(environ))
        signals.request_started.send(sender=self.__class__, environ=environ)
        request = self.request_class(environ)
        response = self.get_response(request)

        response._handler_class = self.__class__

        status = '%d %s' % (response.status_code, response.reason_phrase)
        response_headers = [(str(k), str(v)) for k, v in response.items()]
        for c in response.cookies.values():
            response_headers.append((str('Set-Cookie'), str(c.output(header=''))))
        start_response(force_str(status), response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
    
    ```
