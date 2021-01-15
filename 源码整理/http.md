# http

https://www.odoo.com/documentation/10.0/reference/http.html

## route(route=None, **kw)

该方法作为装饰器将修饰的方法标记为请求的处理程序。被修饰的方法必须是`Controller`子类的一部分

### 参数

- route：字符串或字符串数组。路由部分将确定哪些http请求将与被修饰的方法匹配。有关路由表达式的格式，请参阅werkzeug的路由文档（http://werkzeug.pocoo.org/docs/routing/）
- type：请求的类型，http或json
- auth：身份验证方法的类型：
  - user：用户必须通过身份校验并且当前请求将使用用户的权限执行
  - public：用户通不通过身份校验均可，若不通过则当前请求使用共享的public用户执行
  - none：任何人均可访问。该方法始终处于活动状态，即使没有数据库也是如此。主要由框架和身份验证模块使用。请求代码将不具有访问数据库的任何功能，也不会具有指示当前数据库或当前用户的任何配置。
- method：此路由适用的一系列http方法。如果未指定，则允许所有方法。e.g. ['get', 'post']
- cors：Access-Control-Allow-Origin cors指令值。
- csrf：是否为该路由启用CSRF保护。默认为True

警告：csrf保护

​		CSRF保护默认情况下处于启用状态，并且适用于`rfc:7231`定义的* UNSAFE * HTTP方法（除`GET`，`HEAD`，`TRACE`和`OPTIONS`之外的所有方法）

​		CSRF保护是通过使用不安全的方法检查请求中是否存在称为csrf_token的值（作为请求的表单数据的一部分）来实现的。该值将从表单中删除，作为验证的一部分，您自己的表单处理不必考虑该值。

当为不安全的方法添加新的控制器时（通常是POST，例如表单）：

- 如果表单是用Python生成的，则可以通过以下方法使用csrf令牌：`request.csrf_token（）<odoo.http.WebRequest.csrf_token`。`〜odoo.http.request`对象默认可用在QWeb（python）模板中。如果您不使用QWeb，则可能必须显式添加它。
- 如果表单是用Javascript生成的，则默认情况下，CSRF令牌以`csrf_token`的形式添加到QWeb（js）上下文中；或者可以在`web.core`模块中以`csrf_token`的形式使用（`require('web.core').csrf_token`）
- 如果路由方法是外部（非Odoo）调用，例如他是REST API或webhook，必须在路由方法上禁用CSRF保护。如果可能，可能需要实现其他请求验证方法（以确保不被无关的第三方调用）



## WebRequest(object)

所有Odoo Web请求类型的父类，主要处理请求对象的初始化和设置（调度本身必须由子类处理）

### 参数

#### httprequest

包装好的werkzeug Request对象，class:`werkzeug.wrappers.BaseRequest`

### 方法

#### cr

为当前方法调用初始化了游标cr。当请求使用`none`身份验证时访问游标将引发异常。

#### uid

返回当前用户的id

#### context

返回当前请求的上下文

#### env

返回当前请求的env。class:`~odoo.api.Environment`

#### lang

从当前请求的上下文中找到语言`lang`并返回

#### session

保留当前http session的HTTP session数据。class:`OpenERPSession`

#### debug

标示当前请求是否在debug模式

#### registry

注册到与该请求链接的数据库。如果当前请求使用`none`身份验证，则可以为None

#### db

与该请求链接的数据库。如果当前请求使用`none`身份验证，则可以为None

#### csrf_token

注册并返回当前会话的CSRF token

- time_limit=3600：CSRF令牌仅应在指定的持续时间内有效（以秒为单位），默认情况下为1h。若为None可使令牌在当前用户会话有效期间一直有效。



## JsonRequest(WebRequest)

处理JSON-RPC 2请求

### 参数

#### method

被忽略

#### params

必须是JSON对象（不是数组），并作为关键字参数传递给处理程序方法

```python
Sucessful request::
  --> {"jsonrpc": "2.0",
       "method": "call",
       "params": {"context": {},"arg1": "val1" },
       "id": null}

  <-- {"jsonrpc": "2.0",
       "result": { "res1": "val1" },
       "id": null}

Request producing a error::
  --> {"jsonrpc": "2.0",
       "method": "call",
       "params": {"context": {},"arg1": "val1" },
       "id": null}

  <-- {"jsonrpc": "2.0",
       "error": {"code": 1,
                 "message": "End user error message.",
                 "data": {"code": "codestring", "debug": "traceback" } },
       "id": null}
```



## HttpRequest(WebRequest)

处理http请求

匹配的路由参数，查询字符串参数，form参数和文件将作为关键字参数传递给处理程序方法。

在名称冲突的情况下，路由参数具有优先权。

结果可能是：

- 一个伪造值，这种情况下HTTP响应将为`HTTP 204`
- werkzeug响应对象按原样返回
- `str`或`unicode`，将被解释为HTML并包装在response对象中

### 方法

#### make_response(self, data, headers=None, cookies=None)

​		帮助返回一个非HTML响应（图片，文件等）或带有自定义响应头/cookie的HTML响应。

​		尽管返回非HTML数据时，处理程序仅可以返回要作为字符串发送的页面的HTML标记，但它们需要创建一个完整的响应对象，否则客户端将无法正确解释返回的数据。

- data：响应体，字符串
- headers：http响应头，`[(name, value)]`
- cookies：客户端上设置的cookie，dict

#### render(self, template, qcontext=None, lazy=True, **kw)

渲染QWeb模板

The actual rendering of the given template will occur at then end of the dispatching. Meanwhile, the template and/or qcontext can be altered or even replaced by a static response.

- template：要渲染的模板名

- qcontext：要传入模板中的数据，dict

- lazy：是否将模板渲染推迟到最后一个可能的时刻

  ```python
  response = Response(template=template, qcontext=qcontext, **kw)
  if not lazy:
  	return response.render()
  return response
  ```

- kw：转发给werkzeug的Response对象

#### not_found(self, description=None)

返回一个404



## Controller

控制器是通过继承创建的。控制器提供了独立于模型的扩展机制，但非常相似。e.g.:

```python
class MyController(odoo.http.Controller):
    @route('/some_url', auth='public')
    def handler(self):
        return stuff()
```

要重写控制器，请从其类继承并重写相关方法

```python
class Extension(MyController):
    @route()
    def handler(self):
        do_before()
        return super(Extension, self).handler()
```

- 使用route装饰器是保持方法（和路由）生效的必要条件：如果在不使用的情况下重新定义方法，则该方法将“unpublished”

- 所有方法的装饰器都是组合的，如果重写方法的装饰器没有参数，则保留所有以前的装饰器，任何提供的参数都将覆盖以前定义的装饰器中的参数，例如：

  ```python
  # 将改变/some_url 从public认证到user认证
  class Restrict(MyController):
      @route(auth='user')
      def handler(self):
          return super(Restrict, self).handler()
  ```

 

## OpenERPSession(werkzeug.contrib.sessions.Session)

session相关

## Response(werkzeug.wrappers.Response)

Response object passed through controller route chain.

除了werkzeug.wrappers.Response参数外，此类的构造函数还可以为QWeb Lazy Rendering使用以下附加参数。

- template：要渲染的模板名
- qcontext：要传入模板中的数据，dict
- uid：用于`ir.ui.view`渲染调用的用户id，默认为None，此时使用发起请求的用户

这些属性可用作Response对象上的参数，并且可以在渲染之前随时更改



## send_file

send_file(filepath_or_fp, mimetype=None, as_attachment=False, filename=None, mtime=None,  add_etags=True, cache_timeout=STATIC_CACHE, conditional=True)

这是Flask的send_file()的修改版本。将文件的内容发送到客户端。默认情况下，它将尝试使用WSGI服务器的file_wrapper支持。

- filepath_or_fp：文件地址或文件句柄
- mimetype：文件类型，未提供会自动检测
- as_attachment：为True时header增加`Content-Disposition: attachment`。作为附件下载
- filename：文件名
- mtime：用于条件响应的最后修改时间。
- add_etags：set to `False` to disable attaching of etags.
- conditional：set to `False` to disable conditional responses.
- cache_timeout：header的超时时间（以秒为单位）

## content_disposition(filename)

传入一个文件名，将文件名解码后根据浏览器版本返回`"attachment; filename=%s" % filename`

设置header时使用，e.g. `('Content-Disposition', content_disposition(filename))`



