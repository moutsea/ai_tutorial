我们都知道Python是一个非常灵活的语言，以至于如果它不是你的第一门语言，你会发现它总能给你各种各样的惊喜，让你忍不住惊叹：woc，还有这种操作。比如我在系统地学习Python之前是Java后端出身，所以对此感受尤其明显，每一阶段几乎都会让我觉得打开了新世界的大门。今天就和大家介绍一个最基础，非常好用，但是很多人不知道的操作。

## 解压变量

我们都知道，Python允许进行多个变量的赋值操作，比如著名的交换两个元素，如果是在C++或者Java语言当中，如果不通过函数实现，必须要引入第三个变量，比如：

```python
# swap a, b
c = a
a = b
b = c
```

我们要交换a和b必须要引入c，这是因为当我们赋值b给a的时候，a原本的值会丢失，所以我们必须要先”缓存“下来。但是由于Python支持多变量赋值的操作，所以大可不必引入其他变量就可以完成，所以交换两个元素在Python当中只有一行就可以搞定：

```python
a, b = b, a
```

Python的解释器会直接计算后边的值然后覆盖左边，赋值是同时进行的，所以不需要引入其他变量，而且看起来也非常geek。

除此之外，Python还支持`tuple`和`list`的解压。

举个例子，假设我们有一个二元数组：[1, 2]，我们希望用两个变量分别获取它的第0位和第一位，我们当然可以写成这样：

```python
l = [1, 2]
a, b = l[0], l[1]
```

其实并不用这么麻烦，因为当Python检测到等号左边是多个变量，右边是`list`或者是`tuple`之后，会自动执行`list`和`tuple`的解压，将它依次赋值给对应的元素，所以上面的代码可以简化成：

```python
l = [1, 2]
a, b = l
```

那如果l是一个二维数组，我们希望遍历它呢？同样可以在循环当中使用：

```python
l = [[1, 2], [3, 4], [5, 6]]
for i, j in l:
    print(i, j)
```

即使是在变量的组合当中也可以生效：

```python
a, b, c = 1, 3, (4, 5)
print(c)
```

当我们执行这段代码，屏幕上会输出什么呢？是会报错吗？还是会解压`(4, 5)`这个`tuple`然后将`4`赋值给`c`呢？

都不对，输出的结果是`(4, 5)`，也就是说Python发现变量数量对不上之后，会自动将`tuple`当做一个整体进行赋值。不但如此，即使是下面这种情况，Python也能自动识别：

```python
a, b, (c, d), e = 1, 3, (4, 5), 7
print(c, d)
```

在下面的赋值当中，既有`tuple`又有普通元素，并且我们的变量也组合成了`tuple`，这时Python同样会识别出`(4, 5)`应该赋值给`(c, d)`这个整体，也就是说4和5分别赋值给`c`和`d`。

## 缺省元素

在有的时候，我们在获取元素的时候，源数据当中有我们不需要的字段。虽然Python自动解压非常方便，但是我们还是要为我们不需要的数据设置变量。在一些情况下这会导致内存的浪费，并且这也不符合我们编程的规范，即所有变量都应该派上用场。为了解决这个问题，Python提供缺省元素的方法。我们可以使用\_来代表一个缺省值，\_对应的数据不会被存储下来，只是为了方便我们”凑齐“元素。

举个例子，还用上面的例子举例，假设源数据的格式是这样：`1, 3, (4, 5), 7`，但是我们只需要中间的元组，我们就可以这样去接收：

```python
_, _, (c, d), _ = 1, 3, (4, 5), 7
```

再比如，当我们遍历`dict`的时候，有可能我们并不关注`dict`的key，只希望获得它的`value`，这个时候也可以使用缺省符号：

```python
a = {}
for _, v in a.items():
    print(v)
```

## 压缩变量

既然变量可以解压，那么自然也可以压缩。想象一个场景，比如有一批衡量工厂零件的数据，这个数据当中除了零件的尺寸之外还包含了零件的名称，生产日期和工厂名称等等其他的属性。假设我们当下希望解析这份数据，并且将零件的尺寸用数组存储，这个时候应该怎么办呢？

比如，零件的数据的规格长这样：`wheel, factory1, 3, 4, 5, 6, 2020-02-02`

Python同样针对这个问题提供了解决方法，就是变量压缩符\*，针对上面那个问题，我们可以写成：

```python
data = ['wheel', 'factory1', 3, 4, 5, 6, '2020-02-02']
name, factory, *inch, date = data
print(inch)
```

最后我们打印出来的`inch`是`[3, 4, 5, 6]`，也就是说通过使用\*，我们成功地将中间表示零件尺寸的数据赋值进了一个数组当中。这个操作非常重要，因为有可能不同零件尺寸的数量是不同的，如果我们自己写解析的话就很难处理这个问题。而使用Python当中的 \*操作符，我们可以很好地解决这个问题。

## 联合使用

到这里，我们介绍了缺省符号的用法，介绍了压缩符号的用法，问题来了，我们能不能将这两个符号组合使用，获取数据当中任意个缺省值呢？

当然是可以的，还是刚才的问题，假设我们现在不关心零件的尺寸，想要过滤掉它们，我们只要对上面的代码稍作改动即可：

```python
data = ['wheel', 'factory1', 3, 4, 5, 6, '2020-02-02']
name, factory, *_, date = data
```

如此我们就过滤掉了中间若干个尺寸信息，仅仅保留了头尾其他的信息。

## 其他用途

到这里还没结束，不知道大家在看到 `*` 这个操作符号的时候有没有什么联想，如果稍稍了解过Python的话，应该会想起Python当中，如果我们想让一个函数接收任何参数的话，我们可以写成：

```python
def func(*args, **kw):
    pass
```

其中`args`其实代表一个数组，`kw`代表一个`dict`，这些我们都是知道的。但是前面的 `*` 和 `**` 呢，又代表什么呢？

`*`代表解压数组，`**`自然就代表解压`dict`。我们来看个例子：

```python
a = [1, 3, 5]
```

请问`print(a)`和`print(*a)`有什么区别？如果你试一下就会发现，直接打印`a`，出来的结果是`[1, 3, 5]`，如果你打印 `*a`，得到的结果是`1, 3, 5`。也就是说前者是将`a`当成一个数组输出，是一个变量，后者则是将`a`解压了，当成了3个变量输出。那么同样的道理，`**kw`，也是将作为`dict`的`kw`解压，以`key: value`的形式展开。不过如果你直接调用 `**kw`会得到一个报错，这个操作只能在函数传递参数的时候使用。

所以到这里，我们就明白了，`*args`和`**kw`为什么能够代表所有参数了。因为前者代表了直接传递的必选参数，后者呢，代表提供了默认值的默认参数。这也是为什么Python限定了默认参数必须放在必选参数后面的原因，一方面是为了消除歧义，另一方面也是为了能够用`*args`, `**kw`来统一表示。