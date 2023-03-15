今天的这篇文章和大家聊聊Python当中的排序，和很多高级语言一样，Python封装了成熟的排序函数。我们只需要调用内部的`sorted`函数，就可以完成排序。但是实际场景当中，排序的应用往往比较复杂，比如对象类型，当中有多个字段，我们希望按照指定字段排序，或者是希望按照多关键字排序，这个时候就不能简单的函数调用来解决了。

## 字典排序

我们先来看下最常见的字典排序的场景，假设我们有一个字典的数组，字典内有多个字段。我们希望能够根据字典当中的某一个字段来进行排序，我们用实际数据来举个例子：

```python
kids = [
    {'name': 'xiaoming', 'score': 99, 'age': 12},
    {'name': 'xiaohong', 'score': 75, 'age': 13},
    {'name': 'xiaowang', 'score': 88, 'age': 15}
]
```

这里的`kids`是一个`dict`类型的数组，`dict`当中拥有`name`，`score`和`age`三个字段。假设我们当下希望能够按照`score`来排序，应该怎么办呢？

对于这个问题，解决的方案有很多，首先，我们可以使用上一篇文章当中提到的匿名函数来指定排序的。这里的用法和上篇文章优先队列的用法是一样的，我们直接来看代码：

```python
sorted(kids, key=lambda x: x['score'])
```

在匿名函数当中我们接收的`x`是`kids`当中的元素，也就是一个`dict`，所以我们想要指定我们希望的字段，需要用`dict`访问元素的方法，也就是用中括号来查找对应字段的值。

假如我们希望按照多关键字排序呢？

首先介绍一下多关键字排序，还是用上面的数据打比方。在上面的例子当中，各个`kid`的`score`都不一样，所以排序的结果是确定的。但如果存在两个人的`score`相等，我希望年龄小的排在前面，那么应该怎么办呢？我们分析一下可以发现，原本是按照分数从小到大排序，但有可能会出现分数相等的情况。这个时候，我们希望能够按照在分数相等的情况下来比较年龄，也就是说我们希望根据两个关键字来排序，第一个关键字是分数，第二个关键字是年龄。

由于Python当中支持`tuple`和`list`类型的排序，也就是说我们可以直接比较`[1, 3]`和`[1, 2]`的大小关系，Python会自动一次比较两个数组当中的元素的大小。如果相等就自动往后比较，直到出现不等或者结束为止。

明白了这点，其实就很好办了。我们只要在匿名函数当中稍稍修改，让它返回的结果增加一个字段即可。

```python
sorted(kids, key=lambda x: (x['score'], x['age']))
```

## itemgetter

除了匿名函数，Python也有自带的库可以解决这个问题。用法和匿名函数非常接近，使用起来稍稍容易一些。

它就是`operator`库当中的`itemgetter`函数，我们直接来看代码：

```python
from operator import itemgetter

sorted(kids, key=itemgetter('score'))
```

如果是多关键字也可以，传入多个key即可：

```python
sorted(kids, key=itemgetter('score', 'age'))
```

## 对象排序

我们接下来看一下对象的自定义排序，我们首先把上面的`dict`写成对象：

```python
class Kid:
    def __init__(self, name, score, age):
        self.name = name
        self.score = score
        self.age = age

    def __repr__(self):
        return 'Kid, name: {}, score: {}, age:{}'.format(self.name, self.score, self.age)
```

为了方便观察打印结果，我们重载了`__repr__`方法，可以简单地将它当做是Java当中的`toString`方法，这样我们可以指定在`print`它的时候的输出结果。

同样，`operator`当中也提供了对象的排序因子函数，用法上和`itemgetter`一样，只是名字不同。

```python
from operator import attrgetter

kids = [Kid('xiaoming', 99, 12), Kid('xiaohong', 75, 13), Kid('xiaowang', 88, 15)]

sorted(kids, key=attrgetter('score'))
```

我们也可以使用匿名函数lambda来实现：

```python
sorted(kids, key=lambda x: x.score)
```

## 自定义排序

到这里还没有结束，因为仍然存在一些问题解决不了。虽然我们实现了多关键字排序，但是还有一个问题解决不了，就是排序的顺序问题。

我们可以在`sorted`函数的参数当中传入`reverse=True`来控制是正序还是倒叙，但是如果我使用多关键字，想要按照某个关键字升序，某个关键字降序怎么办？举个例子，比如说我们想要按照分数降序，年龄升序就没办法通过`reverse`来解决了，这就是当前解决不了的问题。

那应该怎么办呢？

这个时候就需要终极排序杀器上场了，也就是标题当中所说的自定义排序。也就是说我们自己实现一个定义元素大小的函数，然后让`sorted`来调用我们这个函数来完成排序。这也是C++和Java等语言的用法。

自定义的函数并不难写，我们随手就来：

```python
def cmp(kid1, kid2):
    return kid1.age < kid2.age if kid1.score == kid2.score else kid1.score > kid2.score
```

如果看不明白，也没关系，我写成完整版：

```python
def cmp(kid1, kid2):
    if kid1.score == kid2.score:
        return kid1.age < kid2.age
    else:
        return kid1.score > kid2.score
```

写完了之后，还没有结束，这个函数是不能直接投入使用的，他和我们之前提到的`lambda`匿名函数是不一样的。之前的匿名函数只是用来指定字段的，所以我们不能直接将这个函数传递给`key`，还需要在外面包一层加工处理才可以。不过这一层处理函数Python也已经有现成的工具了，我们可以直接调用，它在`functools`里，我们来看代码：

```python
from functools import cmp_to_key

sorted(kids, key=cmp_to_key(cmp))
```

我们来看一下`cmp_to_key`函数里的源码：

```python
def cmp_to_key(mycmp):
    """Convert a cmp= function into a key= function"""
    class K(object):
        __slots__ = ['obj']
        def __init__(self, obj):
            self.obj = obj
        def __lt__(self, other):
            return mycmp(self.obj, other.obj) < 0
        def __gt__(self, other):
            return mycmp(self.obj, other.obj) > 0
        def __eq__(self, other):
            return mycmp(self.obj, other.obj) == 0
        def __le__(self, other):
            return mycmp(self.obj, other.obj) <= 0
        def __ge__(self, other):
            return mycmp(self.obj, other.obj) >= 0
        __hash__ = None
    return K
```

我们可以看到，在函数内部，它其实定义了一个类，然后在类当中重载了比较函数，最后返回的是一个重载了比较函数的新的对象。这些`__lt__, `_`_gt__`函数就是类当中重载的比较函数。比如`__lt__`是小于的判断函数，`__eq__`是相等的函数。那么问题来了，我们能不能直接在`Kid`类当中重载比较函数呢，这样就可以直接排序了。

答案是确定的，我们当然可以这么办，实际上这也是面向对象当中非常常用的做法。相比于自定义比较函数，我们往往更倾向于在类当中定义好优先级。Python当中实现的方法也很简单，就是我们手动实现一个`__lt__`函数，`sorted`默认会将小的元素排在前面，所以我们只用实现`__lt__`一个函数就够了。这个函数当中传入的参数是另一个对象，我们直接在函数里面写清楚比较逻辑就行了。返回`True`表示当前对象比`other`小，否则比`other`大。

我们附上完整代码：

```python
class Kid:
    def __init__(self, name, score, age):
        self.name = name
        self.score = score
        self.age = age

    def __repr__(self):
        return 'Kid, name: {}, score: {}, age:{}'.format(self.name, self.score, self.age)

    def __lt__(self, other):
        return self.score > other.score or (self.score == other.score and self.age < other.age)
```

实现了比较函数之后，我们直接调用`sorted`，不用任何其他传参就可以对它进行排序了。