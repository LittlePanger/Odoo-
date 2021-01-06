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

*调用和重写根据源码和本人建议，标记调用的方法也可重写*

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



















### AbstractModel

AbstractModel = BaseModel；

AbstractModel 是一个抽象模型不会在数据库创建对应表，Model可以继 AbstractModel，AbstractModel为多个Model提供相同属性的统一声明。抽象模型作为可重用的功能集，利用Odoo的继承功能，混入到其他模型。

### Model

Model继承自AbstractModel，但是Model的 _auto=True ， _abstract = True ；
Model的模型对象在模块安装或升级的时候会自动在数据库中创建相应的数据表

### TransientModel

TransientModel继承自Model，但是TransientModel的_transient = True，TransientModel是一种特殊的Model，TransientModel对应的数据表中的数据系统会定时的清理；TransientModel的数据只能做临时数据使用，一般向导对象模型会声明成TransientModel

