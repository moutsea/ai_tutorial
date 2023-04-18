---
title: orm-db
date: 2023-04-18 08:17:38
categories: 
- Python
tags:
- orm
- 面向对象

---

今天这篇文章我们一起来用元类实现一个简易的ORM数据库框架。



本文主要是受到了廖雪峰老师Python3入门教程的启发，不过廖老师的博客有些精简，一些小白可能看起来比较吃力。我在他的基础上做了一些**补充和注释**，尽量写得浅显一些。



## ORM框架是什么



如果是没有做过后端的小伙伴上来估计会有点蒙，这个ORM框架究竟是什么？ORM框架是后端工程师常用的一个框架，它的英文全称是Object Relational Mapping，即**对象-关系映射**框架。顾名思义就是把关系转化成对象的框架，关系这个词我们在哪里用的最多呢？



显然应该是数据库。之前我们在分布式的文章介绍关系型数据库和非关系型数据库的时候就着重介绍过关系的含义。我们常用的MySQL就是经典的关系型数据库，它存储的形式是表，但是**表承载的数据其实是两个实体之间的"关系"**。比如学生上课这个场景，学生和课程是两个主体（entity），我们要记录的是这两个主体之间的关系，也就是学生上课这件事。



而ORM框架做的事情是将这些关系映射成类，这样我们可以将这张表当中增删改查的功能抽象成类当中的方法。这样我们就可以通过调用类的方式来操作数据库了，从而达到**高度抽象业务逻辑、降低用户使用难度**的目的。



比如Java后端工程师常用的`hibernate`和`ibatis`都是用来做这件事情的，明确了框架的功能之后，我们先来设想一下最后的成果。假设我们现在开发出来了这么一套框架，那么它用起来的感觉应该是怎样的？



我们来看下廖老师博客里给的例子：



```python
class User(Model):
    # 定义类的属性到列的映射：
    id = IntegerField('id')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')
```



User类代表了数据库当中的一张表，它有4个字段：`id, name, email`和`password`，我们在定义字段的同时也通过类别指定了它们的类型。这个应该不难理解，上面的这个类等价于我们在数据库当中执行了这么一段建表的SQL：



```sql
create table if not exists user (
	id int,
    name string,
    email string,
    password string
)
```



我们定义了表字段之后，接下来要做的就是**根据字段创建数据**了，其实也就是根据类创建实例。我们希望**User类型的实例就对应User表当中的一条记录**，并且我们可以通过调用实例当中的方法，来操作这张表进行增删改查。



```python
# 创建一个实例：
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
# 保存到数据库：
u.save()
```



那么，我们怎样可以实现这样的功能呢？



## 功能实现



我们先从简单的功能开始实现，首先是`Field`类，**Field类表示数据库表当中一个字段的类型**。这里的逻辑很容易理清楚，我们需要定义多种类型，比如`IntegerField`和`StringField`。我们可以对这些`field`类抽象出一个父类来：



```python
class Field(object):
    def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type
        
    def __str__(self):
        return '<{}:{}>'.format(self.__class__.__name__, self.name)
```



`__str__`方法当中打印出来的两个字段，分别是类别的名称和字段的名称，这段代码应该不难理解。



接着，我们实现它的两个子类，分别是`IntegerField`和`StringField`：



```python
class StringField(Field):
    def __init__(self, name):
        super(StringField, self).__init__(name, 'varchar(100)')
        
        
class IntegerField(Field):
    def __init__(self, name):
        super(IntegerField, self).__init__(name, 'bigint')
```



这里也不难理解，只是一个简单的继承应用而已。



接下来就到了最关键的部分，也就是`Model`类的实现。我们先来分析一下我们希望`Model`这个类拥有的功能，由于它是我们定义出来的每一张表的父类，所以它应该**能够获取子类当中的字段**，并且将它存放在一个容器当中。由于我们需要存储的是字段名和类型的映射，所以将它存储在`dict`当中比较合理。



另外一个功能是我们希望它**能够提供增删改查的接口**，能够根据子类当中定义的字段自动生成相应的SQL语句去调用数据库。这个也是ORM框架的意义所在。



第二个功能容易实现，只要第一个功能搞定了，做一下字符串处理即可。但是第一个功能有些麻烦，它也是元类的意义所在。因为**父类当中的方法是无法获取子类中定义的类属性的**，只能通过元类，在构建类的时候可以拿到属性的信息。



所以我们已经很明确了，我们实现元类的目的就是为了实现这个功能。理清楚了之后，再来写代码就不难了。我们先来实现这个元类：



```python
class ModelMetaclass(type):

    def __new__(cls, name, bases, attrs):
        # 创建model类的时候不做任何处理
        if name=='Model':
            return type.__new__(cls, name, bases, attrs)
        # 打印表名的信息
        print('Found model: %s' % name)
        # mappings用来存储字段的信息
        mappings = dict()
        for k, v in attrs.items():
            # 判断v的类型，只有是Field的子类才会存储起来
            if isinstance(v, Field):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v
        # 将mappings当中的数据从类属性当中移除，防止关键字冲突
        for k in mappings.keys():
            attrs.pop(k)
        attrs['__mappings__'] = mappings # 保存属性和列的映射关系
        attrs['__table__'] = name # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)
```



如果你看过之前的文章，对元类已经很熟悉了，那么这段代码对你来说应该不难理解。元类搞定了，剩下的`Model`就更简单了。按照规范，我们需要实现增删改查四个函数，但是这里我们只是为了展示，所以就只实现其中一个作为例子，其他几个都可以如法炮制。



```python
class Model(dict, metaclass=ModelMetaclass):
    def __init__(self, **kw):
        # 由于Model的基类是dict，所以创造Model的字段会被解析成dict的构造参数
        # 也就是说字段名和字段值的映射会存储在dict当中
        super(Model, self).__init__(**kw)
        
    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Model' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

    def save(self):
        fields = []
        params = []
        args = []
        for k, v in self.__mappings__.items():
            # fields存储字段名
            fields.append(v.name)
            # params填充问号
            params.append('?')
            # 获取字段的值
            args.append(getattr(self, k, None))
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(params))
        print('SQL: %s' % sql)
        print('ARGS: %s' % str(args))
```



Model当中的save方法不难看懂，但是前面的几个方法看起来有些多余。但实际上它们也很重要，这里有一个关键信息是**Model类的父类是`dict`**，我们在构建`Model`的时候传入的参数会被用来初始化一个`dict`。所以我们创建数据实例的时候数据的名称和数据值的映射会被存储在`dict`当中，所以我们在`save`方法当中才会从`self`的`attr`当中获取字段的值。并且我们**在初始化`User`的时候，也必须要填写每个字段的名称**，原因就在这里。



最后我们来运行一下：



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gg0uhfmqhhj30jg03lq36.jpg)



从结果上来看，我们输出了User这个类的插入SQL以及它的字段的值。只需要链接一下数据库，我们的这个ORM框架就可以真正投入使用了。

## 总结



在整个ORM框架实现的过程当中，最重要的是我们对Model这个类创建了元类，但是**真正应用的地方却是在Model的子类**。实际上在实际创建User类的时候，解释器会先搜索User内部是否定义了元类，如果没有，会上一层去往User的父类也就是Model类搜索元类，如果找到了元类，就会使用元类来创建User。**相当于元类被隐形地继承了下来**，但是我们在使用子类的时候却感知不到。



对于框架的使用者来说，也的确不需要了解框架内部的实现机制，只需要明白使用方法，照着使用就行了。虽然元类的实现和理解很复杂，但是使用起来却很简单，这也是它的一个显著特点。



最后，本文的代码示例源于廖雪峰老师的博客，**向廖雪峰老师致敬**。