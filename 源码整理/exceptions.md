# exceptions

Odoo核心异常。该模块定义了一些异常类型（均为class）。 RPC层可以使用这些类型。任何其他异常类型冒泡到RPC层都将被视为“服务器错误”。如果您考虑引入新的异常，请查看test_exceptions插件。

### RedirectWarning(Exception)

重定向警告，可能将页面重定向，而不是简单的显示警告信息。

- action_id：重定向action的id
- button_text：触发重定向按钮的文本

### AccessDenied(Exception)

登录名/密码错误。没有消息，没有回溯。示例：当您尝试使用错误的密码登录时。

### AccessError(except_orm)

访问权限错误。示例：当您尝试读取不允许的记录时。

### MissingError(except_orm)

缺少记录。示例：当您尝试写入已删除的记录时

### ValidationError(except_orm)

违反python约束。示例：当您尝试使用数据库中已经存在的登录名创建新用户时。

### DeferredException(Exception)

Exception object holding a traceback for asynchronous reporting.

    Some RPC calls (database creation and report generation) happen with
    an initial request followed by multiple, polling requests. This class
    is used to store the possible exception occuring in the thread serving
    the first request, and is then sent to a polling request.
    
    ('Traceback' is misleading, this is really a exc_info() triple.)