# Fields

odoo/fields.py

## Field类

​		所有字段都继承自该类, 实例化字段时可以提供以下属性

### 基础属性

| 名称     | 类型        | 描述                                                         |
| -------- | ----------- | ------------------------------------------------------------ |
| string   | str         | 用户看到的字段的标签 , 如果未设置 , 则使用大写的字段名       |
| help     | str         | 用户看到的提示(tooltip的方式)                                |
| readonly | bool        | 该字段是否只读 , 默认为False                                 |
| required | bool        | 该字段是否必需 , 默认为False                                 |
| index    | bool        | 该字段是否在数据库中建立索引 , 默认为False                   |
| default  | 静态值/函数 | 函数可以是一个通过记录集返回值的函数 eg: default=lambda self: self.env.user.name |
| states   | dict        | {state: [(属性, 条件)]} , 可能的属性是：`readonly`，`required`，`invisible`。注意：任何基于状态的条件都要求`state`字段值在客户端UI上可用。通常是通过将其包含在相关视图中来完成的，如果与最终用户无关，则可能不可见。 |
| groups   | str         | 用逗号分隔的groups的xml ID, 将字段访问限制为给定组的用户     |
| copy     | bool        | 复制记录时是否复制该字段, 除了one2many和计算字段默认为False , 其余默认为True |
| oldname  | str         | 该字段的曾用名, 以便ORM在迁移时自动重命名                    |

### 计算属性

​		定义一个字段, 其值是计算出来的, 而不是简单地从数据库中读取 , 要定义计算字段只需要为`compute`属性提供一个值, 该值为函数名

| 名称         | 类型 | 描述                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| compute      | str  | 该字段是计算字段，只读，通过对记录进行一些操作返回处理后的值，每次调用都是最新值 |
| inverse      | str  | 设置该属性可以让计算字段可修改, 可选                         |
| search       | str  | 设置该属性可以给计算字段增加筛选条件, 可选                   |
| store        | bool | 计算字段是否存储在数据库中 , 默认为False                     |
| compute_sudo | bool | 计算字段是否已superuser的权限来计算, 以绕过权限控制, 默认为False |

为compute，inverse和search提供的值是模型方法。eg：

```python
upper = fields.Char(compute='_compute_upper',
					inverse='_inverse_upper',
					search='_search_upper')

@api.depends('name')
def _compute_upper(self):
    for rec in self:
    	rec.upper = rec.name.upper() if rec.name else False

def _inverse_upper(self):
	for rec in self:
		rec.name = rec.upper.lower() if rec.upper else False

def _search_upper(self, operator, value):
	if operator == 'like':
		operator = 'ilike'
    return [('name', operator, value)]
```

​		计算方法将被作用到这个模型的每一个记录中（原文: The compute method has to assign the field on all records of the invoked recordset 直译: 计算方法必须在调用的记录集的所有记录上分配该字段，不知道怎么翻译合适）， `@api.depends`必须指定计算方法依赖的字段，这些依赖关系用于确定何时重新计算该字段，为保证缓存/数据库的一致性, 重新计算是自动执行的。同一方法可以用于多个字段，只需给这些字段设置属性`compute='方法名'`即可，该方法会针对这些字段被调用一次。

​		默认情况下，计算字段不会存储到数据库中，而是即时进行计算。设置属性`store = True`可以将字段的值存储在数据库中。存储计算字段的优点是，该字段的搜索由数据库完成（还可以设置搜索视图）。缺点是，当必须重新计算字段时，它需要更新数据库。

​		顾名思义，`inverse`方法与计算方法相反：使得计算字段可以被修改，为了计算得到修改后的值，会修改计算字段所依赖的字段（为了该计算字段，修改依赖的字段）。注意，默认情况下，不带`inverse`方法的计算字段为只读。

​		在对模型进行实际搜索之前，在处理域时会调用`search`方法。用新domain替换旧domain

### 关系属性

​		通过本模型中的关系型字段，取该关系型字段的值。

| 名称    | 类型 | 描述               |
| ------- | ---- | ------------------ |
| related | str  | 关系型字段名称顺序 |

​		如果未重新定义某些字段属性，则会从源字段自动复制它们。所有无语义的属性都从源字段复制。

​		和计算字段一样，默认情况下，关系字段的值不会存储在数据库中，添加属性`store=True`可以使其存储。依赖的字段值发生变化时，也会重新计算。

eg:

```python
partner_id = fields.Many2one('res.partner',string='学习伙伴')
partner_email = fields.Char(related='partner_id.email',readonly=True,string='伙伴邮箱')
```

### 公司属性

​		让该字段根据公司看到不同值，取代了已淘汰的Property字段类型

| 名称              | 类型 | 描述                       |
| ----------------- | ---- | -------------------------- |
| company_dependent | bool | 让该字段根据公司看到不同值 |

### 属性继承

字段被定义为模型类的类属性。如果扩展了模型，则还可以通过在子类上重新定义具有相同名称和相同类型的字段来扩展字段属性。在这种情况下，该字段的属性取自父类，并被子类中给定的属性覆盖。

eg:

```python
class First(models.Model):
    _name = 'foo'
    state = fields.Selection([...], required=True)

class Second(models.Model):
    _inherit = 'foo'
    state = fields.Selection(help="Blah blah blah")
```



## 字段

- Boolean 布尔
- Integer 整数
  - group_operator : 分组计算符，在使用groupby进行分组时，该字段采取的计算方式。默认为sum，可以为`avg`,`sum`,`min`,`max`等
- Float 浮点
  - digits : tuple(总长度, 小数精度)
  - group_operator : 分组计算符，同Integer
- Monetary 货币
  - currency_field : `res.currency.id`
  - group_operator : 分组计算符，同Integer
- Char 单行字符串
  - size : int 该字段存储值的最大值
  - translate : True or 函数, 使得这个field的值可以被翻译
- Text 多行字符串
  - translate : 同Char
- Html html页面
  - translate : 同Char
  - sanitize : 用于去除包含不安全标签的内容，使用它会对输入进行全局清理。如果需要更精细的控制，可以使用一些关键字，仅在启用sanitize时生效：
    - sanitize_tags=True删除白名单列表以外的标签（默认项）
    - sanitize_attributes=True删除白名单列表以外的标签属性
    - sanitize_style=True删除白名单列表以外的样式属性
    - strip_style=True删除所有样式元素
    - strip_class=True删除所有class属性
- Date 日期(%Y-%m-%d)
  - today() : 返回当前日期的字符串
  - context_today(record, timestamp=None) : 返回字符串时间
    - record: 记录, 用于获取记录中的时区, 默认为当前用户的时区
    - timestamp: datetime类型 若传入该参数则返回传入值的字符串, 默认为当前日期
  - from_string(value) : 传入时间字符串("%Y-%m-%d"), 返回date对象
  - to_string(value) : 传入date对象返回字符串("%Y-%m-%d")
- Datetime 日期(%Y-%m-%d %H:%M:%S)
  - now() : 返回当前日期的字符串
  - context_timestamp(record, timestamp) :　返回datetime时间
    - record: 记录, 用于获取记录中的时区, 默认为当前用户的时区
    - timestamp: datetime类型 返回传入值的字符串
  - from_string(value) : 传入时间字符串(date或datetime, 若为date会补00:00:00), 返回datetime对象
  - to_string(value) : 传入datetime对象返回字符串
- Binary 二进制
  - attachment : 是否存储在附件中
- Selection 选择
  - selection : [('value', 'string')]
  - selection_add : [('value', 'string')] , 继承模型覆盖字段时扩展selection
- Reference 选择模型, 与选择字段一样
  - selection : [('model', 'string')] , tuple的第一个参数必须是模型名, 即存储在ir.model中
  - selection_add : [('model', 'string')] , 同selection
- Many2one 多对一, 整型, 该字段的值是大小为0(无记录) 或 1(单个记录)的记录集的id
  - comodel_name : 目标模型的name(string类型), 若无相关字段或字段扩展名，则必填
  - domain : 在客户端候选值上设置可选域(domain or string)
  - context : 处理该字段时在客户端使用的context(dict)
  - ondelete : 删除引用记录时本条记录的操作 ('set null', 'restrict', 'cascade'), 默认为`set null`
  - auto_join: 将允许ORM在数据查询是使用SQL的join(拼接，级联)功能。 默认为False
  - delegate: set it to ``True`` to make fields of the target model accessible from the current model (corresponds to ``_inherits``)
- One2many 一对多, 该字段的值是`comodel_name`中所有记录的记录集
  - comodel_name : 目标模型的name(string类型), 必填
  - inverse_name : `comodel_name`中对应的Many2one字段(string类型), 等于当前记录, 必填
  - domain : 在客户端候选值上设置可选域(domain or string)
  - context : 处理该字段时在客户端使用的context (dict)
  - auto_join: 将允许ORM在数据查询是使用SQL的join(拼接，级联)功能。 默认为False
  - limit: optional limit to use upon read (integer) 源码中没有使用到
  - copy : 默认为False，默认不复制
- Many2many 多对多, 该字段的值是记录集
  - comodel_name : 目标模型的name(string类型), 必填
  - relation：数据库中存储关系的表名(string类型),可选, 默认为两模型名加_rel
  - column1 : 关系表本模型的字段名, 默认为本模型名加_id
  - column2 : 关系表目标模型的字段名, 默认目标模型名加_id
  - domain : 在客户端候选值上设置可选域(domain or string)
  - context : 处理该字段时在客户端使用的context (dict)
  - limit: optional limit to use upon read (integer) 源码中没有使用到
- Serialized 序列化, 源码中未用, 猜测是用来存储json
- Id 特殊字段, 存储id, 默认readonly