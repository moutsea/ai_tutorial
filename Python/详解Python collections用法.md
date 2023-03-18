这篇文章为大家介绍Python当中一个很好用也是很基础的工具库，叫做`collections`。

collection在英文当中有容器的意思，所以顾名思义，这是一个容器的集合。这个库当中的容器很多，有一些不是很常用，本篇文章选择了其中最常用的几个，一起介绍给大家。

## defaultdict

`defaultdict`可以说是这个库当中使用最简单的一个，并且它的定义也很简单，我们从名称基本上就能看得出来。它解决的是我们使用`dict`当中最常见的问题，就是`key`为空的情况。

在正常情况下，我们在`dict`中获取元素的时候，都需要考虑`key`为空的情况。如果不考虑这点，那么当我们获取了一个不存在的`key`，会导致系统抛出异常。我们当然可以在每次get之前写一个if判断，但是这很麻烦，比如：

```python
if key in dict:
    return dict[key]
else:
    return None
```

当然，这是最笨的方法，`dict`当中为我们提供了带默认值的get方法。比如，我们可以写成：

```python
return dict.get(key, None)
```

这样，当`key`不在`dict`当中的时候，会自动返回我们设置的默认值。这个省去了很多麻烦的判断，但是在一些特殊情况下仍然存在一点问题。举个例子，比如当`key`存在重复，我们希望将`key`相同的`value`存进一个`list`当中，而不是只保留一个。这种情况下写成代码就会比较复杂：

```python
data = [(1, 3), (2, 1), (1, 4), (2, 5), (3, 7)]
d = {}
for k, v in data:
    if k in d:
        d[k].append(v)
    else:
        d[k] = [v]
```

由于`dict`的`value`是一个list，所以我们还是需要判断是否为空，不能直接使用默认值，间接操作当然可以，但是还是不够简单：

```python
for k, v in data:
    cur = d.get(k, [])
    cur.append(v)
    d[k] = v
```

这和使用`if`区别并不大，为了完美解决这个问题，我们可以使用`collections`当中的`defaultdict`：

```python
from collections import defaultdict
d = defaultdict(list)

for k, v in data:
    d[k].append(v)
```

使用`defaultdict`之后，如果`key`不存在，容器会自动返回我们预先设置的默认值。需要注意的是`defaultdict`传入的默认值可以是一个类型也可以是一个方法。如果我们传入`int`，那么默认值会被设置成`int()`的结果，也就是0，如果我们想要自定义或者修改，我们可以传入一个方法，比如：

```python
d = defaultdict(lambda: 3)

for k, v in data:
    d[k] += v
```

## Counter

这是一个非常常用和非常强大的工具，我们经常用到。

在我们实际的编程当中，我们经常遇到一个问题，就是数数和排序。比如说我们在分析文本的时候，会得到一堆单词。其中可能有大量的长尾词，在整个文本当中可能只出现过寥寥几次。于是我们希望计算一下这些单词出现过的数量，只保留出现次数最高的若干个。

这个需求让我们自己实现当然也不困难，我们完全可以创建一个`dict`，然后对这些单词一个一个遍历。原本我们还需要考虑单词之前没有出现过的情况，如果我们上面说的`defaultdict`，又要简单许多。但是我们还是少不了计数然后排序的步骤，如果使用Counter这个步骤会缩减成一行代码。

举个例子：

```python
words = ['apple', 'apple', 'pear', 'watermelon', 'pear', 'peach']
from collections import Counter
counter = Counter(words)

print(counter)

Counter({'apple': 2, 'pear': 2, 'watermelon': 1, 'peach': 1})
```

我们直接将一个`list`传入`Counter`中作为参数，它会自动为我们替当中的每个元素计数。

如果我们要筛选`topK`，也非常简单，它为我们提供了`most_common`方法，我们只需要传入需要求的K即可：

```python
counter.most_common(1)

[('apple', 2)]
```

除此之外，它的构造函数还接收`dict`类型。我们可以直接通过一个`value`是`int`类型的`dict`来初始化一个`Counter`，比如：

```python
c = Counter({'apple': 5, 'pear': 4})
c = Counter(apple=4, pear=3)
```

并且，它还支持加减法的操作，比如我们可以将两个`Counter`相加，它会自动将两个`Counter`合并，相同的`key`对应的`value`累加。相减也是同理，会将能对应的`value`做减法，被减的`key`对应不上的会保留，而减数中对应不上的`key`则会被丢弃。并且需要注意，`Counter`支持`value`为负数。

## deque

我们都知道`queue`是队列，`deque`也是队列，不过稍稍特殊一些，是双端队列。对于`queue`来说，只允许在队尾插入元素，在队首弹出元素。而`deque`既然称为双端队列，那么说明它的队首和队尾都支持元素的插入和弹出。相比于普通的队列，要更加灵活一些。

除了常用的`clear、copy、count、extend`等api之外，`deque`当中最常用也是最核心的api主要是`append、pop、appendleft`和`popleft`。从名字上我们就看得出来，`append`和`pop`和`list`的`append`和`pop`一样，而`appendleft`和`popleft`则是在队列左侧，也就是头部进行`pop`和`append`的操作。非常容易理解。

在日常的使用当中，真正用到双端队列的算法其实不太多。大多数情况下我们使用`deque`主要有两个原因，第一个原因是`deque`受到GIL的管理，它是线程安全的。而`list`则没有GIL锁，因此不是线程安全的。也就是说在并发场景下，`list`可能会导致一致性问题，而`deque`不会。另一个原因是`deque`支持固定长度，当长度满了之后，当我们继续`append`时，它会自动弹出最老插入的数据。

比如说当我们拥有海量的数据，我们不知道它的数量，但是想要保留最后出现的指定数量的数据的时候，就可以使用`deque`。

```python
from collections import deque
dque = deque(maxlen=10)
# 假设我们想要从文件当中获取最后10条数据
for i in f.read():
    dque.append(i)
```

## namedtuple

`namedtuple`很特殊，它涉及到元编程的概念。简单介绍一下元编程的概念，我们不做过多的深入。简而言之，就是在常见的面向对象当中。我们都是定义类，然后通过类的构造函数来创建实例。而元编程指的是我们定义元类，根据元类创建出来的并不是一个实例，而是一个类。如果用模具和成品来分别比喻类和实例的话，元类相当于是模具的模具。

`namedtuple`是一个非常简单的元类，通过它我们可以非常方便地定义我们想要的类。

它的用法很简单，我们直接来看例子。比如如果我们想要定义一个学生类，这个类当中有name、score、age这三个字段，那么这个类会写成：

```python
class Student:
    def __init__(self, name=None, score=None, age=None):
        self.name = name
        self.score = score
        self.age = age
```

这还只是粗略的写法，如果考虑规范，还需要定义`property`等注解，又需要很多代码。如果我们使用`namedtuple`可以简化这个工作，我们来看代码：

```python
from collections import namedtuple
# 这个是类，columns也可以写成'name score age'，即用空格分开
Student = namedtuple('Student', ['name', 'score', 'age'])

# 这个是实例
student = Student(name='xiaoming', score=99, age=10)
print(student.name)
```

通过使用`namedtuple`，我们只需要一行就定义了一个类，但是这样定义的类是没有缺失值的，但是`namedtuple`很强大，我们可以通过传入`defaults`参数来定义缺失值。

```python
Student = namedtuple('Student', ['name', 'score', 'age'], defaults=(0, 0))
```

可以注意到，虽然我们定义了三个字段，但是我们只设置了两个缺失值。在这种情况下，`namedtuple`会自动将缺失值匹配上`score`和`age`两个字段。因为在Python的规范当中，必选参数一定在可选参数前面。

细数一下，我们今天的文章当中介绍了`defaultdict、Counter、deque`和`namedtuple`这四种数据结构的用法。除了这四个之外，`collections`库当中还有一些其他的工具类，只是我们用的频率稍稍低一些，加上由于篇幅的原因，这里就不多做赘述了。感兴趣的同学可以自行查看相关的api和文档。

