# compare_flask_django_request
比较flask和django的请求过程

##django的一次请求
1： 首先会去调用setting中WSGI_APPLICATION配置的调用函数。

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```
