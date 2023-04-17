上一篇文章当中我们介绍了Python装饰器的定义和基本的用法，这篇文章我们一起来学习一下Python装饰器的一些进阶使用方法。对装饰器不太熟悉，可以回顾一下上一篇文章。

之前的文章当中我们从前到后仔细推到了一下装饰器的本质和用途，也学会了它的基本用法，已经足够**应付80%的场景**了。但是总有20%的场景使用基本的方法解决不了，这个时候就需要我们学习更多、更全的其他用法。

比如我想要**通过一个参数控制装饰器的功能**，这个问题其实很常见。就拿记录时间来说，我们都知道时间可以记录成很多种格式，比如可以记成`2020-05-04`也可以记录成`20200504`，还可以记录成`04/05/2020`，如果是后端还会记录时间的时间戳。比如说我们现在实现了一个记录日志的装饰器，用来给我们的方法打上日志，现在我们想要控制记录日志的时候打印出来的时间格式，这个需求使用最简单的装饰器就没有办法解决了。

这个时候，如果想要解决问题，就必须引入参数，也就是说我们必须要在装饰器当中加入参数才行。但问题来了，这个参数怎么加，加在哪里呢？

## 定义装饰器参数

在我们介绍具体的用法之前，我们先来回顾一下装饰器的代码：

```python
def mydec(func):
    @wraps(func)
    def mywrap(*args, **kw):
        print('hello this is decorator1')
        func(*args, **kw)
    return mywrap
    
@mydec
def helloWorld():
    print('hello, world')
```

这个就是我们上次讲的最简单的那种装饰器，假如说我们这个时候希望传入一个参数type，可以控制装饰器的输出结果。就像这样：

```python
@mydec(type_='test')
def helloWorld():
    print('hello, world')
```

我们可能会想是不是应该在mydec这个方法的参数里面加上一个type_，但是如果你试一下就好发现这样是不行的，会得到一个error：

![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gegpjq8jryj313808gdhf.jpg)

Error错误的字面意思很好理解，但是原因却令人费解。这个Error是说函数`mydec`**少了一个必选参数func**，这个`func`就是我们要包装的函数，但是这个不是自动传入的吗，怎么会提示我们少了这个参数呢？

如果这个问题的本质不能理解的话，那么装饰器就很难大成了，因为只有理解清楚了这一点，才能理解后面装饰器各种稀奇古怪的进阶用法。但是很坑爹的是，很多资料当中都只是简单地介绍了怎么用，很少会探究其中背后的原因，这会让初学者在学习的时候陷入费解。我在学习的时候也花了很多心思，才终于搞明白，说穿了很简单，但是想通不容易。

其实这样会报错的主要原因是注解当中有参数和没有参数的装饰器是完全不同的。

我们来回顾一下不加参数的装饰器的用法，比如：

```python
@mydec
def hello_world():
    pass
```

我们执行hello\_world()的时候，等价于执行`mydec(hello\_world)()`。看明白了吗，我们把这行代码展开，它其实是下面这两行代码共同执行的结果：

```python
cur = mydec(hello_world)
cur()
```

如果`hello_world`这个函数带上参数呢？

```python
@mydec
def hello_world(*args, **kw):
    pass
```

那么执行的时候它其实是这样的：

```python
cur = mydec(hello_world)
cur(*args, **kw)
```

这个理解了之后，我们继续往下，现在我们想要将一个参数传给装饰器，按照我们的想法下面这两段代码应该是一样的。

```python
@mydec(type_='test')
def helloWorld():
    print('hello, world')


cur = mydec(hello_world, type_)
cur()
```

但是很遗憾的是，Python解释器当中并不是这么设计的。它对加上了参数的装饰器**多做了一层封装**，也就是说上面传入参数的`hello_world`函数执行的时候等价于下面这段代码：

```python
cur1 = mydec(type_)
cur2 = cur1(hello_world)
cur2()
```

正是因为额外多封装了一层，所以函数和装饰器的参数传入装饰器的顺序是不同的，顺序也是不一样的。明白了这点之后就简单很多了，既然Python解释器在解释装饰器参数的时候多增加了一层，那么如果我们想要实现带参数的装饰器，只需要也在装饰器当中多封装一层就可以了。比如可以写成这样：

```python
def mydec(type_=None):
    def decorate(func):
        @wraps(func)
        def mywrap():
            if type_ is not None:
                print(type_)
            func()
        return mywrap
    return decorate
```

这样我们再执行就可以了：

![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gegplhtdsej30uo07sjrz.jpg)

## 默认参数怎么办

到这里看似一切都很完美，但其实**有一个很大的问题**被我们忽略了。

这个问题就是默认参数问题，在前面我们定义装饰器的时候，将`type_`这个参数设置成了可选的。这也很符合我们实际情况，如非必要，参数能省略就省略。但是这就导致了一个问题，对于不用加上参数的装饰器，有些人习惯写成`mydec()`，有些人习惯写成`mydec`。如果我们试一下`mydec`，就会发现这样写会报错：

![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gegplrsfozj313206qq47.jpg)

这个报错和上面的报错一模一样，出现的原因也是一样的，都是少了`func`参数。但是很奇怪啊，为什么会少了`func`呢？

原因很简单，因为我们把括号去掉，装饰器又回到了之前的两层结构！

```python
cur = mydec(hello_world)
cur(*args, **kw)
```

这就很坑爹了，我们装饰器的结构肯定是不能改变的，如果使用两层结构就没办法传入参数了，但是如果不传参的时候怎么办，难道就只能强制程序员统一风格全部加上括号吗？这当然也是一个办法，那还有没有更好的办法呢？有没有办法统一这两种逻辑呢？

当然是有的，为了解决这个问题，我们需要用到一个新的工具，叫做**偏函数**。

偏函数很好理解，它本意也是一个高阶函数，其实就是**闭包**。偏函数的使用场景针对多参数的函数，通过使用偏函数，可以固定若干个参数的传值，从而起到简化函数传参的作用。我们来看一个例子，我们创建一个pow函数，用来计算x的n次方：

```python
import math
def pow(x, n):
    return math.pow(x, n)
```

这个函数需要传入x和n两个参数，如果我们当前只需要计算平方，我们可以使用闭包，**固定其中的参数n**，生成一个新的函数来做到这点。比如：

```python
def mypow(n):
    def func(x):
        return pow(x, n) 
    return func
```

![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1gegpqwiym0j30os04caa9.jpg)

偏函数的本质就是这样一个闭包，只不过它简化了我们的代码而已：

```python
from functools import partial

pow2 = partial(pow, n=2)
pow2(6)
```

使用偏函数我们只需要传入待加工的原函数，以及固定的参数值即可。我们把偏函数用在装饰器当中，就可以解决刚才的问题。回忆一下，不带参数的装饰器是两层函数嵌套，而带上参数的是三层嵌套。那么我们使用`partial`，专门为带上参数的情况额外增加一层嵌套即可：

```python
def mydec(func=None, type_=None):
    # 不带参数的话，func会是None，这时候我们固定参数即可
    if func is None:
        return partial(mydec, type_=type_)
    
    @wraps(func)
    def mywrap():
        if type_ is not None:
            print(type_)
        func()
    return mywrap
```

我们来看下这其中的细节，当我们不传入参数的时候，我们其实执行的是`cur = mydec(func)`，这个时候`func`不为空，那么不会触发`if`中的语句，所以会直接返回`mywrap`。如果传入参数，这时候`func`是`None`，会触发`if`中的`partial`。注意这里我们在**partial当中传入的函数依然是mydec**，也就是说我们固定了type_这个参数，调用的话依然返回的是`mywrap`，相当于我们通过`partial`额外在两层结构当中专门为带参数的情况增加了一层，统一了逻辑。

## 结尾

今天的概念比之前的装饰器要复杂很多，**一时可能并不好理解**，其实这是非常正常的。这不仅仅是装饰器的问题，也不仅是Python的问题，归根结底这是函数式编程的特性导致的。函数式编程的优点就是高度灵活，使用非常方便，但缺点也很明显，**代码难以维护，阅读难度高，理解起来也不简单**。典型的初学简单，精深非常难的典型。所以如果大家觉得一时理解不了，这并不是你们的问题，一方面我们需要培养自己函数传编程的思维，另一方面我们也需要熟悉Python中装饰器的使用方法。



