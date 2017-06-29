# Flask源码分析

## 前言

Flask是一个用Python语言编写的Web框架. <br/>
Flask最大的特点就是"微", Flask号称"微框架", 相比于Django、Ruby on rails等框架, Flask本身的功能少的可怜, 只包含核心的请求、路由、响应处理等功能, 其余大多数功能通过扩展实现. Flask微的特性使得Flask的代码规模较小, 适合初学者阅读学习. <br/>
Flask源码分析系列文章会从Flask的整体架构入手, 结合Flask的源码分析Flask的实现. 希望读者阅读完成后可以编写一个简单的Web框架.

## 获取Flask源码

Flask项目开源在github上, 源码可以使用git获取:

```shell
$ git clone https://github.com/pallets/flask.git
```
### 给vim用户的小贴士　

对于使用vim阅读源码的读者, 建议使用**ctags**工具, 方便快速查找定位函数定义

```shell
$ cd flask/
$ ctags -R
```

会在当前源码目录下生成tags文件, 在vim中使用```Ctrl-]```就可以快速移动到函数定义处.
<br/>

Python是一门自省的语言, 使用```help()```函数可以查看文档, 如果想在vim中查看各个对象的说明文档, 可以安装[**jedi-vim**](https://github.com/davidhalter/jedi-vim); 安装后, 在vim中使用```Shift-K```就可以查看文档, 十分方便.
<br/>

IDE用户上面两个功能应该是标配.


## Step0: ```__init__.py```

```__init__.py```文件在Python包中具有重要功能, ```flask/__init__.py```定义了Flask框架的公共API(也就是Flask提供的功能接口), 我们就从这个文件开始Flask源码之旅! <br/>

```__init__.py```文件首先定义了Flask的版本号

```python
__version__ = '0.13-dev'
```

接下来导入了一系列的Public API, 可以分为3类

+ 从依赖包(Werkzeug, Jinja2)中导入的公共接口

```python
from werkzeug.exceptions import abort
from werkzeug.utils import redirect
from jinja2 import Markup, escape
```

从注释中可以看到Flask本身并未使用这些函数, 这是专门给框架的使用者用的. <br/>

+ Flask自己实现的功能接口

```python
from .app import Flask, Request, Response
from .config import Config
from .helpers import url_for, flash, send_file, send_from_directory, \
     get_flashed_messages, get_template_attribute, make_response, safe_join, \
     stream_with_context
from .globals import current_app, g, request, session, _request_ctx_stack, \
     _app_ctx_stack
from .ctx import has_request_context, has_app_context, \
     after_this_request, copy_current_request_context
from .blueprints import Blueprint
from .templating import render_template, render_template_string

# the signals
from .signals import signals_available, template_rendered, request_started, \
     request_finished, got_request_exception, request_tearing_down, \
     appcontext_tearing_down, appcontext_pushed, \
     appcontext_popped, message_flashed, before_render_template

# We're not exposing the actual json module but a convenient wrapper around
# it.
from . import json
jsonify = json.jsonify
```

这些函数/类都是在Flask源码中实现的, 是Flask提供的主要功能接口. 而且Flask还定制了自己的json模块. <br/>

+ 旧版本兼容模块

```python
# backwards compat, goes away in 1.0
from .sessions import SecureCookieSession as Session
json_available = True
```

<hr/>

在接下来的系列文章中, 我会遵从Public API的导入顺序来分析Flask源码, 从依赖库(Werkzeug, Jinja2)到Flask的核心--Flask类. 一步步理清Flask的架构, 了解Flask的实现!

## CopyRight

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
