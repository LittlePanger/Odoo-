# api

该模块提供了用于管理两种不同API样式的方法。

在传统样式中，数据库游标，用户id，上下文字典和记录id（通常表示为`cr`,`uid`,`context`,`ids`）等参数是显示传递给所有方法。

在记录样式中，这些参数被隐藏在模型实例中，从而使其更具有面向对象的感觉。

e.g.

```python
# 传统样式
model = self.pool.get(MODEL)
ids = model.search(cr, uid, DOMAIN, context=context)
for rec in model.browse(cr, uid, ids, context=context):
	print rec.name
model.write(cr, uid, ids, VALUES, context=context)

# 记录样式
env = Environment(cr, uid, context) # cr, uid, context被包裹在env中
model = env[MODEL]                  # 使用env激活MODEL实例
recs = model.search(DOMAIN)         # 搜索并返回记录集
for rec in recs:                    # 遍历记录集
	print rec.name
recs.write(VALUES)                  # 更新记录集中所有记录
```



## 装饰器

#### constrains(*args)

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

#### onchange(*args)

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

#### depends(*args)

装饰`_compute_func`方法，参数为字段名称

e.g.

```python
pname = fields.Char(compute='_compute_pname')

@api.one
@api.depends('partner_id.name', 'partner_id.is_company')
def _compute_pname(self):
    if self.partner_id.is_company:
        self.pname = (self.partner_id.name or "").upper()
    else:
        self.pname = self.partner_id.name
```

#### returns(model, downgrade=None, upgrade=None)

假定返回一个model的记录集

- model：模型名，为self时为当前model

```python
@model
@returns('res.partner')
def find_partner(self, arg):
    ...     # return some record

    # output depends on call style: traditional vs record style
    partner_id = model.find_partner(cr, uid, arg, context=context)

    # recs = model.browse(cr, uid, ids, context)
    partner_record = recs.find_partner(arg)
```

#### model(method)

装饰一个内容不明确但是模型明确的方法。

```python
@api.model
def method(self, args):
    ...

may be called in both record and traditional styles, like::
# recs = model.browse(cr, uid, ids, context)
recs.method(args)
model.method(cr, uid, args, context=context)

Notice that no ``ids`` are passed to the method in the traditional style.
```

#### multi(method)

装饰一个对记录集操作的方法。

```python
@api.multi
def method(self, args):
	...

may be called in both record and traditional styles, like::
# recs = model.browse(cr, uid, ids, context)
recs.method(args)

model.method(cr, uid, ids, args, context=context)
```



## Environment

存储ORM记录数据的环境。

```
It provides access to the registry by implementing a mapping from model
names to new api models. It also holds a cache for records, and a data
structure to manage recomputations.
```

- cr：当前数据库游标
- uid：当前用户id
- context：当前上下文字典

#### ref(self, xml_id, raise_if_not_found=True)

返回与给定的xml_id对应的记录

#### user(self)

返回当前用户的记录集

#### lang(self)

返回当前语言代码



