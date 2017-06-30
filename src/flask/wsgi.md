# Werkzeug与WSGI

## WSGI

WSGI的全称是**Web Server Gateway Interface**, 翻译成中文就是Web服务网关接口. 既然是接口, 必然涉及到接口对接的各方, WSGI涉及三大类应用: **实现WSGI接口的服务器**, **实现WSGI接口的application/framework**, **实现WSGI接口的中间件(middleware)**. <br/>

### WSGI的起源

在WSGI出现前, Python已经拥有许多不同类型的web框架, 以及用于部署web应用的web服务器. 由于框架作者和web服务器作者间没有统一的接口, 所以开发者对框架的选择往往会限制部署的选择, 反之也是如此.
![wsgi1](https://user-images.githubusercontent.com/10671733/27729129-2137c5b6-5db7-11e7-8498-542b3c1f8c98.png) <br/>

定义WSGI接口的目的就是让**实现WSGI接口的web应用可以部署在任何实现WSGI接口的web服务器上.**
![wsgi2](https://user-images.githubusercontent.com/10671733/27729789-a241797a-5db9-11e7-9bc4-0bdb6d494062.png) <br/>
Python官方在[PEP333](https://www.python.org/dev/peps/pep-333/)中正式定义了WSGI, 并在[PEP3333](https://www.python.org/dev/peps/pep-3333/)中添加了Python3.x修正. <br/>

### WSGI接口详解
接下来从WSGI接口的三方:应用、服务器和中间件详解WSGI.

**WSGI Application**: <br/>
符合WSGI接口的应用需满足已下3个条件:

1. 可被WSGI服务器调用
2. 调用WSGI服务器端提供的start_response回调
3. 返回可迭代的http body

比如: PEP333上的WSGI App示例代码:

```python
class AppClass:
    """Produce the same output, but using a class

    (Note: 'AppClass' is the "application" here, so calling it
    returns an instance of 'AppClass', which is then the iterable
    return value of the "application callable" as required by
    the spec.

    If we wanted to use *instances* of 'AppClass' as application
    objects instead, we would have to implement a '__call__'
    method, which would be invoked to execute the application,
    and we would need to create an instance for use by the
    server or gateway.
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield HELLO_WORLD
```

这里, AppClass类就是一个WSGI Object. 可见, WSGI对Object的要求不一定是类对象, 可调用的函数(实现```__call__```方法)、类都可以是WSGI Object. 这里AppClass为了被服务器调用处理, 实现了```__iter__```方法.<br/>
```environ```、```start_response```是WSGI服务器端提供的. ```environ```是一个环境变量字典,主要包含系统环境变量, 以及特定的WSGI环境变量; ```start_response```是服务器端提供的回调函数, WSGI Application调用start_response回调, 将http状态码(status)、http响应头部(response_headers)传递给服务器; 最后WSGI App返回可迭代的Http Body. <br/>
由此可以看出, Http响应的主要部分(状态码、头部、Body)都是在WSGI Application中处理的. 这是Web服务很流行的一种风格, Web服务器只用于建立和路由Http请求到特定的App, 从而使Web服务器从繁琐的业务工作中解放出来, 将精力放在建立更多的连接上. <br/>

**WSGI Server**<br/>
符合WSGI接口的Web服务器需要满足以下个条件:

1. 接收Http请求后调用WSGI Application
2. 向WSGI Application提供environ环境字典和start_response回调
3. 构造并返回Http响应

还是以PEP333上的WSGI Server代码为例:

```python
def run_with_cgi(application):
    """a cgi gateway function"""
    # 构造environ环境变量字典
    ## 系统环境变量
    environ = {k: unicode_to_wsgi(v) for k,v in os.environ.items()}
    ## WSGI环境变量
    environ['wsgi.input']        = sys.stdin.buffer
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    def write(data):
        # 将WSGI App返回的Http status、Http Headers、Http Body构造成Http Response
        out = sys.stdout.buffer

        if not headers_set:
             raise AssertionError("write() before start_response()")

        elif not headers_sent:
             # Before the first output, send the stored headers
             """bytes
             Status: <status>\r\n
             <Header> : <Value>\r\n
             <Header> : <Value>\r\n
             <Header> : <Value>\r\n
             \r\n
             """
             status, response_headers = headers_sent[:] = headers_set
             out.write(wsgi_to_bytes('Status: %s\r\n' % status))
             for header in response_headers:
                 out.write(wsgi_to_bytes('%s: %s\r\n' % header))
             out.write(wsgi_to_bytes('\r\n'))

        out.write(data)
        out.flush()

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[1].with_traceback(exc_info[2])
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]

        return write

    # 调用WSGI App
    result = application(environ, start_response)
    try:
        # 处理Http Body
        for data in result: # http_body in a list
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```
这里省略了部分细节代码, 比如编码转换, ```Http响应是Bytes String```, 而```Python3默认一切皆为Unicode```, 所以在```write```构造Http响应时需要调用```wsgi_to_bytes```将WSGI native string编码为Bytes String.

```python
def wsgi_to_bytes(s):
    return s.encode('iso-8859-1')
```

从下面这张图中可以更清晰的理解WSGI Server、Application实现之间的关系:

![wsgi3](https://user-images.githubusercontent.com/10671733/27740452-a95d8d5e-5de4-11e7-844e-88b5d19926e1.png)
<br/>
**WSGI MiddleWare**<br/>

中间件, 顾名思义位于WSGI服务器和WSGI应用的"中间". 这里的中间指的是请求调用顺序, WSGI服务器在将Http 请求传递给WSGI App前先经过中间件处理, 因而中间件可以用于实现请求过滤、请求路由、更改Http头部等功能. <br/>
考虑中间件的特殊位置, 对于服务器来说, 中间件就像是WSGI App, 可以被服务器调用, 处理服务器传入的请求; 而对于WSGI App来说, 中间件就像是服务器, 可以调用应用并向其传递请求.

```python
WSGI_Server(WSGI_MiddleWare(WSGI_App))
```

WSGI服务器、中间件、应用形成了一个线性的调用链.多个MiddleWare构成一个中间件调用栈:

```python
WSGI_Server(WSGI_MiddleWare1(WSGI_MiddleWare2(...WSGI_MiddleWareN(WSGI_App)...))
```

中间件依次执行各自的功能.<br/>
对于Python, 装饰器是天然的中间件语法!
　
```python
@app.use
async middleware1(ctx, next): # --> 2
    # do some thing
    await next(ctx)           # --> 4

@app.use
async middleware2(ctx, next): # --> 3
    # do some thing
    await next(ctx)           # --> 5

@a_middleware                 # --> 1
async def main(ctx):          # --> 6
    # pass
```

上面这段代码是我设计的一个基于中间件的异步Web框架的API(是不是很像Koa；). Py_Web_Lab实验一就希望实现这个Web框架!!! <br/>

下面给出一个MiddleWare的代码示例:

```python
class SimpleMiddleWare:
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        # 实现__call__方法, 可以被WSGI服务器调用
        print(‘something you want done in every http request’)
        return self.app(environ, start_response)

app = App()
app = SimpleMiddleWare(app)
```

## Werkzeug
Werkzeug是大名鼎鼎的[pocoo团队](http://www.pocoo.org/)开发的一款WSGI工具集. WSGI毕竟只是一个Web框架和服务器间的低层次接口, 并没有提供高级API供Web应用开发者使用(WSGI也不推荐开发者直接在WSGI上开发应用). 而Werkzeug则填补了低层次接口与高级Web框架之间的空白.你可以基于Werkzeug开发自己的框架或应用, 比如Flask的很多功能都是Werkzeug实现的,在前言里```__init__.py```分析中可以略窥一二. <br/>
这里给出[Werkzeug官网](http://werkzeug.pocoo.org/)的一个示例代码:

```python
from werkzeug.wrappers import Request, Response

@Request.application
def application(request):
    return Response('Hello World!')

if __name__ == '__main__':
    from werkzeug.serving import run_simple
    run_simple('localhost', 4000, application)
```

可以看到,Werkzeug很好的隐藏了WSGI接口的实现细节, 通过提供高层次的API, 使得编写Python Web应用变得如此简单! <br/>

Werkzeug是Flask的重要组成部分之一, 接下来的文章会介绍Flask使用的另一个库: Jinja2.

## CopyRight

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.