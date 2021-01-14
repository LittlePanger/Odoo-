# import_xml.rng

## convert_xml_import

解析xml文件的代码在odoo/tools/convert.py中

```python
def convert_xml_import(cr, module, xmlfile, idref=None, mode='init', noupdate=False, report=None):
    doc = etree.parse(xmlfile)
    relaxng = etree.RelaxNG(
        etree.parse(os.path.join(config['root_path'],'import_xml.rng' )))
    try:
        relaxng.assert_(doc)
    except Exception:
        _logger.info('The XML file does not fit the required schema !', exc_info=True)
        _logger.info(ustr(relaxng.error_log.last_error))
        raise

    if idref is None:
        idref={}
    if isinstance(xmlfile, file):
        xml_filename = xmlfile.name
    else:
        xml_filename = xmlfile
    obj = xml_import(cr, module, idref, mode, report=report, noupdate=noupdate, xml_filename=xml_filename)
    obj.parse(doc.getroot(), mode=mode)
    return True
```



## Relax NG

Relax NG 是 REgular LAnguage for XML Next Generation的缩写，即”[可扩展标记语言](http://baike.baidu.com/view/159832.htm)的下一代正规语言”，是一种基于语法的[可扩展标记语言](http://baike.baidu.com/view/159832.htm)模式语言，可用于描述、定义和限制 [可扩展标记语言](http://baike.baidu.com/view/159832.htm)（[标准通用标记语言](http://baike.baidu.com/view/5286041.htm)的子集）词汇表。简单地说 Relax NG是解释XML如何被定义的一套XML。Odoo就是通过定义了一套rng文件定义了xml框架结构，在模块被安装或者升级的时候将其解析成与之相对应的内置对象，存储在数据库中。关于Relax NG的语法规则，可以参考Relax NG的官网。

根据该文件简单推测一下Relax NG的语法

- `rng:start`：文件开始
- `rng:ref`：引用节点
- `rng:define`：定义结点
- `rng:element`：标签
- `rng:choice`：从`choice`标签中选择
- `rng:optional`：可选的
- `rng:attribute`：属性
- `rng:zeroOrMore`：0个或多个

## 常见节点标签名及其属性

https://www.odoo.com/documentation/10.0/reference/data.html

### odoo，openerp，data

- noupdate
- context

### menuitem

- id 必填，必填为源码注释约束
- name 选填
- parent 选填
- action 选填
- sequence 选填
- groups 选填
- load_xmlid 选填，默认为True
- icon 选填
- web_icon 选填
- web_icon_hover 选填
- string 选填，未在任何地方使用

### record

#### 属性

- id 选填
- forcecreate 选填
- model 必填
- context 选填

#### 标签

- field 零个或多个

### template

#### 属性

- id 选填
- t-name 选填
- name 选填
- forcecreate 选填
- context 选填
- priority 选填
- key 选填
- website_id 选填
- inherit_id 选填
- primary 选填，默认为True
- groups 选填
- active 选填
- customize_show 选填
- page 选填，默认为True

#### 标签

- any 零个或多个

### delete

- model 必填
- id 选填
- search 选填

### act_window

- id 必填
- name 必填
- res_model 必填
- domain 选填
- src_model 选填
- context 选填
- view_id 选填
- view_type 选填
- view_model 选填
- multi 选填
- target 选填
- key2 选填
- groups 选填
- limit 选填
- usage 选填
- auto_refresh 选填

### url

- id 必填
- name 必填
- url 必填
- target 选填

### assert

#### 属性

- model 必填
- search 选填
- count 选填
- string 选填
- id 选填
- context 选填
- severity 选填

#### 标签

- test
  - 属性 expr 必填

### report

- id 选填
- string 必填
- model 必填
- name 必填
- report_type 选填
- multi 选填
- menu 选填
- keyword 选填
- rml 选填
- file 选填
- sxw 选填
- xml 选填
- xsl 选填
- parser 选填
- auto 选填
- header 选填
- webkit_header 选填
- attachment 选填
- attachment_use 选填
- groups 选填
- paperformat 选填
- usage 选填

### workflow

#### 属性

- model 必填
- action 必填
- uid 选填
- context 选填
- ref 和value标签任选其一

#### 标签

- value 和ref属性任选其一

### function

#### 属性

- model 必填
- name 必填
- id 选填
- context 选填
- eval

#### 标签

- value
- function

### ir_set

#### 标签

- field 零个或多个

### field

#### 属性

- name 必填
- 另一个属性任选一组
  - type等于base64, char, file时。可添加属性file
  - type等于int时。可设值为int类型
  - type等于float时。可设值为float类型
  - type等于list, tuple时。可添加value标签零个或多个
  - type等于html, xml时。可添加any标签零个或多个
  - ref属性
  - eval属性时，可选model属性
  - search属性时，可选model, use属性

### value

- model 选填
- 任选一个
  - search属性，必填
  - eval属性，必填
  - type属性，选填
    - file属性，必填



## 校验自定义视图架构

在拓展视图后，需要对增加的属性标签进行校验。可以通过重写校验xml的方法，先查找自定义的rng文件，如果未找到再去上一级的rng文件查找。

校验views.xml的rng文件在odoo/addons/base/rng/view.rng