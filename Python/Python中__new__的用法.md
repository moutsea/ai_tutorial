这篇文章我们来聊聊Python当中一个新的默认函数`__new__`。



上一篇当中我们讲了如何使用type函数来动态创建Python当中的类，除了type可以完成这一点之外，还有另外一种用法叫做`metaclass`。原本这一篇应该是继续元类的内容，讲解`metaclass`的使用。但是`metaclass`当中用到了一个**新的默认函数`__new__`**，关于这个函数大家可能会比较陌生，所以在我们研究`metaclass`之前，我们先来看看`__new__`这个函数的用法。



## 真假构造函数



如果你去面试Python工程师的岗位，面试官问你，请问Python当中的类的**构造函数**是什么？



你不假思索，当然是`__init__`啦！如果你这么回答，很有可能你就和offer无缘了。因为在Python当中`__init__`并不是构造函数，`__new__`才是。是不是有点蒙，多西得（日语：为什么）？我们不是一直将`__init__`方法当做构造函数来用的吗？怎么又冒出来一个`__new__，`如果`__new__`才是构造函数，那么**为什么我们创建类的时候从来不用它呢？**



别着急，我们慢慢来看。首先我们回顾一下`__init__`的用法，我们随便写一段代码：



```python
class Student:
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender
```



我们一直都是这么用的，对不对，毫无问题。但是我们换一个问题，我们在Python当中怎么实现单例(Singleton)的设计模式呢？怎么样实现工厂呢？



从这个问题出发，你会发现只使用`__init__`函数是不可能完成的，因为`__init__`并不是构造函数，它只是**初始化方法**。也就是说在调用`__init__`之前，我们的实例就已经被创建好了，`__init__`只是为这个实例赋上了一些值。如果我们把创建实例的过程比喻成做一个蛋糕，`__init__`方法并不是烘焙蛋糕的，只是点缀蛋糕的。那么显然，在点缀之前必须先烘焙出一个蛋糕来才行，那么这个烘焙蛋糕的函数就是`__new__`。



## `__new__`函数



我们来看下`__new__`这个函数的定义，我们在使用Python面向对象的时候，**一般都不会重构这个函数**，而是使用Python提供的默认构造函数，Python默认构造函数的逻辑大概是这样的：



```python
def __new__(cls, *args, **kwargs):
    return super().__new__(cls, *args, **kwargs)
```



从代码可以看得出来，函数当中基本上什么也没做，就原封不动地调用了父类的构造函数。这里隐藏着Python当中类的创建逻辑，是根据继承关系一级一级创建的。根据逻辑关系，我们可以知道，当我们创建一个实例的时候，实际上是**先调用的`__new__`函数创建实例，然后再调用`__init__`对实例进行的初始化**。我们可以简单做个实验：



```python
class Test:
    def __new__(cls):
        print('__new__')
        return object().__new__(cls)
    def __init__(self):
        print('__init__')
```



当我们创建Test这个类的时候，通过输出的顺序就可以知道Python内部的调用顺序。

![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gfl6w59vwkj30b8028jr7.jpg)



从结果上来看，和我们的推测完全一样。



## 单例模式



那么我们重载`__new__`函数可以做什么呢？一般都是用来完成`__init__`无法完成的事情，比如前面说的**单例模式**，通过`__new__`函数就可以实现。我们来简单实现一下：



```python
class SingletonObject:
    def __new__(cls, *args, **kwargs):
        if not hasattr(SingletonObject, "_instance"):
            SingletonObject._instance = object.__new__(cls)
        return SingletonObject._instance
    
    def __init__(self):
        pass
```



当然，如果是在并发场景当中使用，还需要加上线程锁防止并发问题，但逻辑是一样的。



除了可以实现一些功能之外，还可以**控制实例的创建**。因为Python当中是先调用的`__new__`再调用的`__init__`，所以如果当调用`__new__`的时候返回了`None`，那么最后得到的结果也是`None`。通过这个特性，我们可以控制类的创建。比如设置条件，只有在满足条件的时候才能正确创建实例，否则会返回一个`None`。



比如我们想要创建一个类，它是一个`int`，但是不能为0值，我们就可以利用`__new__`的这个特性来实现：



```python
class NonZero(int):
    def __new__(cls, value):
        return super().__new__(cls, value) if value != 0 else None
```



那么当我们用0值来创建它的时候就会得到一个`None`，而不是一个实例。



## 工厂模式



理解了`__new__`函数的特性之后，我们就可以灵活运用了。我们可以用它来实现许多其他的设计模式，比如大名鼎鼎经常使用的**工厂模式**。



所谓的工厂模式是指通过一个接口，**根据参数的取值来创建不同的实例**。创建过程的逻辑对外封闭，用户不必关系实现的逻辑。就好比一个工厂可以生产多种零件，用户并不关心生产的过程，只需要告知需要零件的种类。也因此称为工厂模式。



比如说我们来创建一系列游戏的类：



```python
class Last_of_us:
    def play(self):
        print('the Last Of Us is really funny')
        
        
class Uncharted:
    def play(self):
        print('the Uncharted is really funny')
        

class PSGame:
    def play(self):
        print('PS has many games')
```



然后这个时候我们希望可以通过一个接口根据参数的不同返回不同的游戏，如果不通过__new__，这段逻辑就只能写成函数而不能通过面向对象来实现。通过重载`__new__`我们就可以很方便地用参数来获取不同类的实例：



```python
class GameFactory:
    games = {'last_of_us': Last_Of_us, 'uncharted': Uncharted}
    def __new__(cls, name):
        if name in cls.games:
            return cls.games[name]()
        else:
            return PSGame()
        

uncharted = GameFactory('uncharted')
last_of_us = GameFactory('last_of_us')
```



## 总结



相信看到这里，关于`__new__`这个函数的用法应该都能理解了。一般情况下我们是用不到这个函数的，只会在一些特殊的场景下使用。虽然如此，我们学会它并不只是用来实现设计模式，更重要的是可以加深我们对于Python面向对象的理解。



除此之外，另一个经常使用`__new__`场景是元类。所以今天的这篇文章其实也是为了后面介绍元类的其他用法打基础。