**Odoo中，一切皆模型，连视图都是模型。Odoo将各种数据，如：权限数据、类数据、视图数据等，按照模型分表存储，然后在查看时，按照索引从各个表格读取信息，组合成我们看到的内容。**

# models

## BaseModel

BaseModel 是odoo模型的基础类
Odoo模型是通过继承创建的

- Model：常规模型，用于常规的数据库持久化模型
- TransientModel：瞬态模型，用于临时数据，存储在数据库中但经常自动清理
- AbstractModel：抽象模型，用于抽象的超类，该类应由多个继承模型共享

系统在每个数据库自动实例化每个模型一次。那些实例来自于每个数据库上的可用模型，并取决于该数据库上安装哪些模块。每个的实际类实例是从相应的模型生成。

每个模型实例都是一个“记录集”，即模型记录的有序集合。记录集通过下列等方法来返回。记录没有显式表示（*显示表示的含义*？），一个记录表示为一个记录的记录集。

- method：`〜.browse`
- method：`〜.search`
- 字段访问 (如many2one字段)



### 属性

模型属性：模型类可以使用一些属性来控制它们的一些行为

| 属性名           | 默认值      | 描述                                                         |
| ---------------- | ----------- | ------------------------------------------------------------ |
| _auto            | False       | 是否自动创建数据                                             |
| _register        | False       | 是否ORM中可见                                                |
| _abstract        | True        | 模型是否为抽象                                               |
| _transient       | False       | 模型是否为瞬态                                               |
| _name            | None        | 模型名                                                       |
| _description     | None        | 模型描述                                                     |
| _custom          | False       | 自定义模型时为True                                           |
| _inherit         | None        | 继承的模型，str or [str]                                     |
| _inherits        | {}          | 委托继承，{'parent_model': 'm2o_field'}                      |
| _constraints     | []          | Python 约束 (old API)                                        |
| _table           | None        | 数据库表名                                                   |
| _sequence        | None        | SQL sequence to use for ID field                             |
| _sql_constraints | []          | SQL约束，[(name, sql_def, message)]                          |
| _rec_name        | None        | 数据显示名称，默认为name字段                                 |
| _order           | 'id'        | 搜索结果的默认顺序, 默认为id字段                             |
| _parent_name     | 'parent_id' | 层次数据结构相关，the many2one field used as parent field    |
| _parent_store    | False       | 层次数据结构相关，set to True to compute MPTT (parent_left, parent_right) |
| _parent_order    | False       | 层次数据结构相关，order to use for siblings in MPTT          |
| _date_name       | 'date'      | 用于默认日历视图的字段                                       |
| _fold_name       | 'fold'      | 用于判断kanban视图的折叠组                                   |
| _needaction      | False       | whether the model supports "need actions" (see mail)         |
| _translate       | True        | 是否允许此模型的翻译导出                                     |
| _depends         | {}          | dependencies of models backed up by sql views ，{model_name: field_names, ...} |



### 方法

*调用和重写根据源码和本人建议，标记调用的方法也可重写，标记重写的方法同理，按需使用*

#### view_init(self, fields_list)

重写：该方法由`default_get`调用，在初始化默认值之前执行特定操作

#### default_get(self, fields_list)

重写：修改初始化记录的默认值。返回`fields_list`中字段的默认值。默认值由context，用户默认值和模型本身确定。

#### view_header_get(self, view_id=None, view_type='form')

重写：修改窗口标题，如果需要依赖于上下文的窗口标题请重写此方法

#### user_has_groups(self, groups)

调用：根据条件判断用户所属组返回True、False。通常用于解析视图和模型定义中的`groups`属性。

```python
# groups已','分开，'!'表示不具有此组
# eg：拥有base.group_user但无base.group_system的返回True
self.user_has_groups(groups='base.group_user,!base.group_system')
```

#### load_views(self, views, options=None)

重写：视图加载函数，控制视图行为。被`data_manager.js`调用，调用了`fields_view_get`。实测是触发窗口动作之后调用的该方法。

#### fields_view_get(self, view_id=None, view_type='form', toolbar=False, submenu=False)

重写：获取模型的视图，例如字段，模型，arch。

odoo的视图以xml存放在`ir.ui.view`表中，属于静态格式，设计之后就固定了。通过修改该方法可以在视图加载时动态修改视图的结构

eg：https://www.odoogo.com/post/87/

#### get_formview_action(self)

重写：form视图获取函数，返回特定的form视图的窗口动作。未找到实际使用场景，该函数仅被`form_relational_widgets.js`调用了，模板是many2one，推测是many2one打开相关form视图时调用。

#### search_count(self, args)

调用：传入`domain`返回符合条件的记录数。该函数直接调用`search(args, count=True)`，所以需要返回数量时使用哪种都可以

- `len(search(args))`效率低，查出所有结果之后再计算数量
- `search(args, count=True)`或`search_count(args)`直接使用`SELECT count(1)`返回结果

#### search(self, args, offset=0, limit=None, order=None, count=False)

调用：根据domain搜索匹配的记录，如果用户尝试绕过访问规则调用函数会AccessError

- args：domain，使用空列表来查询所有记录
- offset：跳过查询记录条数
- limit：查询条数
- order：查询记录排序方式
- count：为True时返回匹配记录条数

#### name_get(self)

重写：定义了该模型的记录在被关联，搜索时显示的名字，默认按照`_rec_name`指定的字段的值。

返回`[(id, text_repr), ...]`

#### name_create(self, name)

调用：提供一个`name`创建一条记录并返回`(id, name)`。内部调用了`create`，作用大概是初始化一条记录，感觉没啥用，直接create就行了。

#### name_search(self, name='', args=None, operator='ilike', limit=100)

重写：定义了记录在被关联，被搜索时的查找逻辑。调用`_name_search`返回结果。

- name：被搜索的关键字
- args：domain
- operator：用于匹配`name`的domain搜索符
- limit：返回的最大记录条数

#### _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):

重写：name_search的私有方法，筛选结果后调用`name_get`返回`[(id, _rec_name), ...]`。允许在调用`name_get`时传递用户id已解决一些权限问题。

#### read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True)

重写：按照给定的`groupby`字段在列表视图中分组

- domain：search视图中定义的domain
- fields：tree视图中的字段
- groupby：search视图中设置的groupby
- offset：跳过的记录条数
- limit：返回最大记录条数
- orderby：排序，用于覆盖组的默认排序规则
- lazy：
  - True：将结果按照第一个groupby分组，剩余groupby放入`__context`中。即分组下面可有分组
  - False：所有groupby在一次调用中完成。每个组展开一次即可全部展开

返回一个`[{},...]`，每个组一个字典，每个字典包括：

- 由groupby参数中的字段的值
- 本组符合条件的数量
- __domain：指定搜索条件的元组列表，其中包括search视图中指定的和按本次分组生成的
- __context：参数字典，包括下一级分组的字段，eg：`{'group_by': ['字段A']}`

#### read(self, fields=None, load='_classic_read')

RPC调用：读取记录的请求字段。返回一个字典列表，每条记录一个字典

- fields：[字段A, 字段B]

#### workflow相关

- create_workflow：为给定的记录创建工作流实例
- delete_workflow：删除绑定到给定记录的工作流实例
- step_workflow：修改给定记录的工作流实例
- signal_workflow(self, signal)：给定记录发送工作流信号，并返回映射id到工作流结果的字典
- redirect_workflow(self, old_new_ids)：将绑定给旧记录的工作流实例重新绑定给新记录，`old_new_ids = [(old, new), ...]`

#### unlink(self)

重写：删除当前记录

#### write(self, vals)

重写：使用提供的值更新当前的所有记录

- vals：`{field: vals}`，键必须是字段名，值必须是符合键类型的值

其中many2one必须是 关联模型设置的数据库标示符，例如id

由于历史和兼容性原因，`date`和`datetime`字段使用字符串进行读和修改

`one2many`和`many2many`使用特殊的格式来操作关联记录集。格式是按顺序执行的三元组的列表。

| 三元组          | 描述                                                         | 条件                     |
| --------------- | ------------------------------------------------------------ | ------------------------ |
| (0, _, values)  | 根据values的字典创建一条新记录                               |                          |
| (1, id, values) | 根据values的字典更新指定id的记录                             | 不能用于create           |
| (2, id, _)      | 删除指定id记录的关联并在数据库中删除这条数据                 | 不能用于create           |
| (3, id, _)      | 删除指定id记录的关联并但不删除这条数据                       | 不能用于one2many和create |
| (4, id, _)      | 建立与指定id的记录的关联                                     | 不能用于one2many         |
| (5, _, _)       | 移除所有关联的记录，等于在每个记录上使用命令3                | 不能用于one2many和create |
| (6, _, ids)     | 使用ids列表中的记录替换原来的记录，相当于执行5之后为每个id执行4 | 不能用于one2many         |





### AbstractModel

AbstractModel = BaseModel；

AbstractModel 是一个抽象模型不会在数据库创建对应表，Model可以继 AbstractModel，AbstractModel为多个Model提供相同属性的统一声明。抽象模型作为可重用的功能集，利用Odoo的继承功能，混入到其他模型。

### Model

Model继承自AbstractModel，但是Model的 _auto=True ， _abstract = True ；
Model的模型对象在模块安装或升级的时候会自动在数据库中创建相应的数据表

### TransientModel

TransientModel继承自Model，但是TransientModel的_transient = True，TransientModel是一种特殊的Model，TransientModel对应的数据表中的数据系统会定时的清理；TransientModel的数据只能做临时数据使用，一般向导对象模型会声明成TransientModel

