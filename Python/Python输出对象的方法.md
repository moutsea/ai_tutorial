这篇文章我们来聊聊Python当中的类。

## 打印实例

我们先从类和对象当中最简单的打印输出开始讲起，打印一个实例是一个非常不起眼的应用，但是在实际的编程当中却非常重要。原因也很简单，因为我们debug的时候往往会想看下某个类当中的内容是不是符合我们的预期。但是我们直接`print`输出的话，只会得到一个地址。

我们来看一个例子：

```python
class point:
    def __init__(self, x, y):
        self.x = x
        self.y = y


if __name__ == "__main__":
    p = point(3, 4)
    print(p)
```

在这段代码当中我们定义了一个简单的类，它当中有x和y两个元素，但是如果我们直接运行的话，屏幕上会输出这样一个结果：

```bash
<__main__.point object at 0x10a18c210>
```

这个是解释器在执行的时候这个实例的一些相关信息，但是对于我们来说几乎没有参考意义，我们想要的是这个实例当中具体的值，而不是一个内存当中的地址。

想要实现这个功能，我们有很多方法，下面我们一一来看。

## \_\_str\_\_方法

`__str__`方法大家应该都不陌生，它类似于Java当中的`toString`方法，可以根据我们的需要返回实例转化成字符串之后的结果。

比如，我们可以在类当中重载这个方法，就可以根据我们的需要输出结果了：

```python
class point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return 'x: %s, y: %s' % (self.x, self.y)
```

当我们运行它，得到的结果会是：

```bash
x: 3, y: 4
```

`__str__`和`__init__`, `__len__`很多函数一样是Python中的特殊函数，在我们创建类的时候，系统会我们隐式创造许多这样的特殊函数。我们可以根据需要重载其中的一部分完成我们想要的功能。比如如果我们写的是一棵二叉树的类，我们还可以在`__str__`函数当中进行递归遍历所有的节点，打印出完整的树来。

## \_\_repr\_\_方法

你也许可能也听说过`__repr__`函数，它也可以实现根据我们的需要自定义输出的功能。比如我们把上面的代码改下函数名，也可以得到一样的结果。

```python
class point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'x: %s, y: %s' % (self.x, self.y)
```

我们运行它，同样会得到：

```bash
x: 3, y: 4
```

这是为什么呢，难道`__repr__`和`__str__`是一样的吗？如果是一样的，Python的设计者干嘛要保留两个完全相同的函数呢，为什么不去掉其中一个呢？

在分析原因之前，我们先来做一个实验，如果我们两个函数都重载，那么当我们输出的时候，程序执行的是哪一个呢？为了做好区分，我们把`__repr__`当中的输出的格式稍微修改一下。

```python
class point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return 'x: %s, y: %s' % (self.x, self.y)

    def __repr__(self):
        return '<point x: %s, y: %s>' % (self.x, self.y)
```

我们运行之后，会发现输出的结果还是：

```bash
x: 3, y: 4
```

先别着急下结论，我们再把这段代码拷贝到jupyter当中，我们这次不通过打印输出，而通过jupyter自带的交互框输出交互结果，我们再来看下：

![IMAGE](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/F0DF91A9085CFCBBF35109399138A91C.jpg)

奇怪，怎么结果就变成了`__repr__`的结果了呢？

其实这正是反应了两者的区别，如果简单理解，这两个函数都是将一个实例转成字符串。但是不同的是，两者的使用场景不同，其中`__str__`更加侧重展示。所以当我们`print`输出给用户或者使用`str`函数进行类型转化的时候，Python都会默认优先调用`__str__`函数。而`__repr__`更侧重于这个实例的报告，除了实例当中的内容之外，我们往往还会附上它的类相关的信息，因为这些内容是给开发者看的。所以当我们在交互式窗口输出的时候，它会优先调用`__repr__`。

理论上来说，对于一个合格的`__repr__`函数要能够做到:

```python
eval(repr(obj)) == obj
```

也就是说我们通过`__repr__`输出的内容执行之后可以再还原得到这个实例本身，当然在一些场景下这个非常难以实现，所以我们退而求其次，保证`__repr__`当中输出类和对象足够多的信息，方便开发者调试和使用即可。

另外多说一句，repr是report的缩写，所以它有一个报告的意思在里面，而`str`就只是转化成字符串而已。这两者还是有一定区别的。

## format

Python当中最常用的输出函数除了上面两个之外，还有一个就是format。

比较简单的用法就是通过{}代表变量，然后按照顺序依次输入：

![IMAGE](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/259DE04CA4CD1C228F877AA839942A0F.jpg)

除此之外，我们还可以进一步写明花括号里的变量名称，进一步增加可读性：

![IMAGE](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/4611B780C0BF25F39E2183CA7F83A7C5.jpg)

format的功能远不止如此，它还支持许多参数，类似于C语言当中的`printf`，可以通过不同的参数做到各种各样的输出。比如控制小数点后面保留的位数，或者是转化成百分数、科学记数法、左右对齐等功能。这里不一一列举了，大家用到的时候再查询即可。

我们当然可以使用format重新`__repr__`和`__str__`当中的逻辑，但这并不能体现它的强大。因为在Python当中，也为类提供了__format__这个特殊函数，通过重写`__format__`和使用format，我们可以做到更牛的功能。

## format联合\_\_format\_\_

我们可以在类当中重载`__format__`函数，这样我们就可以在外部直接通过format函数来调用对象，输出我们想要的结果。

我们来看代码：

```python
class point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return 'x: %s, y: %s' % (self.x, self.y)

    def __format__(self, code):
        return 'x: {x}, y: {y}'.format(x = self.x, y = self.y)
```

我们把刚才的__repr__改成了__format__，但是需要注意一个细节，我们多加了一个参数`code`，这是由于format当中支持通过参数来对处理逻辑进行配置的功能，所以我们必须要在接口处多加一个参数。加好了以后，我们就可以直接调用`format(p)`了。

![IMAGE](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gfndypuhdpj30qi03ut95.jpg)

到这里还没有结束，在有些场景当中，对于同一个对象我们可能有多种输出的格式。比如点，在有些场景下我们可能希望输出`(x, y)`，有时候我们又希望输出`x: 3, y: 4`，可能还有些场景当中，我们希望输出`<x, y>`。

我们针对这么多场景，如果各自实现不同的接口会非常麻烦。这个时候利用`__format__`当中的这个参数，就可以大大简化这个过程，我们来看代码：

```python
formats = {
    'normal': 'x: {p.x}, y: {p.y}',
    'point' : '({p.x}, {p.y})',
    'prot': '<{p.x}, {p.y}>'
}

class point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return 'x: %s, y: %s' % (self.x, self.y)

    def __format__(self, code):
        return formats[code].format(p=self)
```

我们在调用的时候就可以通过参数来控制我们究竟使用哪一种格式来格式化对象了：

![IMAGE](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/C7FD3E3389121B55668BADE246513F51.jpg)


也就是说通过重载`__format__`方法，我们把原本固定的格式化的逻辑做成了可配置的。这样大大增加了我们使用过程当中的灵活性，这种灵活性在一些问题场景当中可以大大简化和简洁我们的代码。对于Python这门语言来说，我个人感觉实现功能只是其中很小的一个部分，把代码写得简洁美观，才是其中的大头。这也是为什么很多人都说Python是一门易学难精的语言的原因。