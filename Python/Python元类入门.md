今天我们来继续聊聊Python当中的元类。



在之前的文章当中我们介绍了type元类的用法，在上一篇文章当中我们介绍了`__new__`函数与`__init__`函数的区别，以及它在一些设计模式当中的运用。这篇文章我们来看看`metacalss`与元类，以及`__new__`函数在元类当中的使用。



**那文章非常重要，是这一篇的基础**，如果错过了上篇文章，推荐回顾一下。



## metaclass



`metaclass`的英文直译过来就是元类，这既是一个概念也可以认为是Python当中的一个关键字，不管怎么理解，对它的内核含义并没有什么影响。我们可以不必纠结，就认为它是类的类的意思即可。在这个用法当中，**支持我们自己定义一个类，使得它是后面某一个类的元类。**



之前使用type动态创建类的时候，我们传入了类名，和父类的`tuple`以及属性的`dict`。在`metaclass`用法当中，其实核心相差不大，只是表现形式有所区别。我们来看一个例子即可：



```python
class AddInfo(type):
    def __new__(cls, name, bases, attr):
        attr['info'] = 'add by metaclass'
        return super().__new__(cls, name, bases, attr)
        
        
class Test(metaclass=AddInfo):
    pass
```



在这个例子当中，我们首先创建了一个类叫做`AddInfo`，这是我们定义的一个元类。由于我们希望通过它来实现元类的功能，所以我们**需要它继承type类**。我们在之前的文章当中说过，在Python面向对象当中，所有的类的根本来源就是`type`。也就是说Python当中的每一个类都是`type`的实例。



我们在这个类当中重载了`__new__`方法，我们在`__new__`方法当中传入了四个参数。眼尖一点的小伙伴一定已经看出来了，**这个函数的四个参数，正是我们调用type创建类的时候传入的参数**。其实我们调用`type`的方法来创建类的时候，就是调用的`__new__`这个函数完成的，这两种写法对应的逻辑是完全一样的。



我们之后又创建了一个新的类叫做`Test`，这个当中没有任何逻辑，直接pass。但是我们在创建类的时候指定了一个参数`metaclass=AddInfo`，这里**这个参数其实就是指定的这个类的元类**，也就是指定这个类的创建逻辑。虽然我们用代码写了类的定义，但是在实际执行的时候，这个类是以`metaclass`为元类创建的。



根据上面的逻辑，我们可以知道，Test类在创建的时候就被赋予了类属性`info`。我们可以验证一下：



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gfxdt5zn6ej30rc086weq.jpg)



## 拓展类功能



上面这段就是元类的基本用法了，其实本质上和我们之前介绍的`type`的动态类创建是一样的，只不过展现的形式不同。那么我们就有一个问题要问了，我们使用元类究竟能够做什么呢？



这里有一个经典的例子，我们都知道Python原生的list是没有`add`这个方法的。假设我们习惯了Java当中list的使用，习惯用add来为它添加元素。我们希望创建一个新的类，在这个新的类当中，我们可以通过add来添加函数。通过元类可以很方便地使用这一点。



```python
class ListMeta(type):
    def __new__(cls, name, bases, attrs):
        # 在类属性当中添加了add函数
        # 通过匿名函数映射到append函数上
        attrs['add'] = lambda self, value: self.append(value)
        return super().__new__(cls, name, bases, attrs)
    
    
class MyList(list, metaclass=ListMeta):
    pass
```



我们首先是定义了一个叫做`ListMeta`的元类，在这个元类当中我们给类添加了一个属性叫做`add`。它只是包装了一下而已，**底层是通过`append`方法实现的**。我们来实验一下：



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gfxdt5gcc2j30q806yaa5.jpg)



从结果来看也没什么问题，我们成功通过调用`add`方法往list当中插入了元素。这里藏着一个小细节，我们在`ListMeta`当中为`attrs`添加了一个名叫`add`的属性。**这个属性是添加给类的**，而不是类初始化出来的实例的。所以如果我们print出`MyList`这个类当中的所有属性，也能看到`add`的存在。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gfxdt4y8okj31cc07g0ts.jpg)



如果我们直接去通过`MyList`去访问`add`方法的话会引起报错，因为我们实现`add`这个方法逻辑的匿名函数**限制了需要传入两个参数**。第一个参数是实例的对象`self`，第二个参数才是添加的元素`value`。如果我们通过`MyList`的类属性去访问它的话会触发一个错误，因为缺少了一个参数。因为类当中的属性实例也是可以调用的，并且Python会在参数前面自动添加self这个参数，就刚好满足了要求。



搞明白了这些我们只是解决了可能性问题，我们明白了元类可以实现这样的操作，但没有解决我们为什么必须要使用元类呢？就拿刚才的例子来说，我们完全可以继承list这个类，然后在其中再开发我们想要的方法，为什么一定要使用元类呢？



就刚才这个场景来说，的确，我们是找不出任何理由的。完全没有理由不使用继承，而非要用元类。但是在有些场景和有些问题当中，我们必须要使用元类不可。就是**涉及类属性变更和类创建的时候**，我们来看下面这个例子。





## 控制实例的创建



还记得我们上篇文章介绍的工厂设计模式的例子吗？就是我们可以通过参数来得到不同类的实例。



我们创建了三种游戏的类和一个工厂类，我们重载了工厂类的`__new__`函数。使得我们可以根据实例化时传入的参数返回不同类型的实例。



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
        
        
class GameFactory:
    games = {'last_of_us': Last_of_us, 'uncharted': Uncharted}
    def __new__(cls, name):
        if name in cls.games:
            return cls.games[name]()
        else:
            return PSGame()
        

uncharted = GameFactory('uncharted')
last_of_us = GameFactory('last_of_us')
```



假设这个需求完成得很好顺利上线了，但是运行了一段时间之后我们**发现下游有的时候为了偷懒会不通过工厂类来创建实例**，而是直接对需要的类做实例化。原本这没有问题，但是现在产品想要在工厂类当中加上一些埋点，统计出访问我们工厂的访问量。所以我们需要**限制这些游戏类不能直接实例化，必须要通过工厂返回实例**。



那么这个功能我们怎么实现呢？



我们分析一下问题就会发现，这一次不是需要我们在创建实例的时候做动态的添加，而是直接限制一些类不允许直接调用进行创建。限制的方法比较常用的一种就是抛出异常，所以我们希望可以给这些类加上一个逻辑，**实例化类的时候传入一个参数，表明是否是通过工厂类进行的，如果不是，则抛出异常**。



这里，我们需要用到另外一个默认函数，叫做`__call__`，它是**允许将类实例当做函数调用**。我们通过类名来实例化，其实也是一个调用逻辑。这个`__call__`的逻辑并不难写，我们随手就来：



```python
def __call__(self, *args, **kwargs):
    if len(args) == 0 or args[0] != 'factory':
        raise TypeError("Can't instantiate directly")
```



但问题是这个`__call__`函数**并不能直接加在类当中，因为它的应用范围是实例**，而不是类。而我们希望的是在创建实例的时候进行限制，而不是对调用实例的时候进行限制，所以这段逻辑**只能通过元类实现**。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gfxdt7e4pkj310q09igna.jpg)



我们直接创建类的时候就会触发异常，因为不是通过工厂创建的。我们这里判断是否是工厂创建的逻辑简化掉了，只是通过一个简单的字符串来进行的判断，实际上会用一些更加复杂的逻辑，这不是本文的重点，我们了解即可。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gfxdt6j702j314i0hqacv.jpg)



整体运行的逻辑和我们设想的一样，说明这样实现是正确的。



## 总结



我们日常开发当中用到元类的情况非常罕见，一般都是在一些**高端开发的场景**当中。比如说开发一些框架或者是中间件，为了方便下游的使用，需要创建一些关于类属性的动态逻辑，才会用到元类。对于普通开发者而言，如果你无法理解元类的含义以及应用，也没有关系，使用频率非常低。



另外，**元类的概念和动态类、动态语言的概念有关**，Python语言的动态特性很多正是通过这一点体现的。所以随着我们对于Python动态特性理解的加深，理解元类也会变得越来越容易，同样也会理解越来越深刻。如果我们把Python的元类和装饰器做一个类比的话，会发现**两者的核心逻辑是很类似的**。本质上都是在原有的逻辑之外封装新的逻辑，只不过装饰器针对的是一段逻辑，而元类针对的是类的属性和创建过程。



仔细思考，我相信一定会有灵光乍现的感觉。