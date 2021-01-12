# api

```
This module provides the elements for managing two different API styles,
    namely the "traditional" and "record" styles.

    In the "traditional" style, parameters like the database cursor, user id,
    context dictionary and record ids (usually denoted as ``cr``, ``uid``,
    ``context``, ``ids``) are passed explicitly to all methods. In the "record"
    style, those parameters are hidden into model instances, which gives it a
    more object-oriented feel.

    For instance, the statements::

        model = self.pool.get(MODEL)
        ids = model.search(cr, uid, DOMAIN, context=context)
        for rec in model.browse(cr, uid, ids, context=context):
            print rec.name
        model.write(cr, uid, ids, VALUES, context=context)

    may also be written as::

        env = Environment(cr, uid, context) # cr, uid, context wrapped in env
        model = env[MODEL]                  # retrieve an instance of MODEL
        recs = model.search(DOMAIN)         # search returns a recordset
        for rec in recs:                    # iterate over the records
            print rec.name
        recs.write(VALUES)                  # update all records in recs

    Methods written in the "traditional" style are automatically decorated,
    following some heuristics based on parameter names.
```



## 装饰器

#### constrains

检查约束。每个参数必须是检查中使用的字段名称

e.g.

```python
@api.one
@api.constrains('name', 'description')
def _check_description(self):
	if self.name == self.description:
		raise ValidationError("Fields name and description must be different")
```
警告：

- 只支持简单字段名称，点分名称（关系字段的字段，e.g. `partner_id.customer`）将被忽略
- 装饰器声明的字段必须在`create`或`write`中调用时才会触发`@constrains`。这意味着在记录创建或修改时，视图中不存在的字段将不会触发调用。必须通过覆盖`create`以确保是否能够触发约束（测试是否少值）

#### onchange

监听参数中的字段在视图中的变化，方法中的字段分配将返回给客户端。每个参数必须是字段名称

e.g.

```python
@api.onchange('partner_id')
def _onchange_partner(self):
    # 当视图中partner_id变化时，给message赋值
	self.message = "Dear %s" % (self.partner_id.name or "")
```

该方法还可以返回更改字段域的字典或弹出警告消息

e.g.

```python
return {
    'domain': {'other_id': [('partner_id', '=', partner_id)]},
    'warning': {'title': "Warning", 'message': "What is this?"},
}
```

警告：

- 只支持简单字段名称，点分名称（关系字段的字段，e.g. `partner_id.customer`）将被忽略



















