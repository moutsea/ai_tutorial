这是Python系列最后一篇文章，我们来聊聊Python当中的命令行参数工具argparse。



命令行参数工具是我们非常常用的工具，比如当我们做实验希望调节参数的时候，如果参数都是通过硬编码写在代码当中的话，我们每次修改参数都需要修改对应的代码和逻辑显然这不太方便。比较好的办法就是把必要的参数设置成**通过命令行传入**的形式，这样我们只需要在运行的时候修改参数就可以了。



## sys.argv



解析命令行传入参数最简单的办法就是通过**sys.argv**，`sys.argv`可以获取到我们通过命令行传入的参数。



```python
import sys

print(sys.argv)
```



用法很简单，只需要调用`sys.argv`即可。`argv`是一个数组，如果参数有多个，我们可以通过下标进行访问。但是有一点需要注意，`argv`当中存储的结果是从Python调用开始的。



我们来看一个例子，我们随意传入一些参数，`print sys.argv`之后是这样的。

```python
python test.py -a -c -d=222 
>>> ['test.py', '-a', '-c', '-d=222']
```



也就是说我们python运行test.py这个**文件名也当做参数之一**，所以我们要获取自定义参数的话需要从`argv[1]`开始。



`sys.argv`的好处是方便，我们只需要访问它就可以拿到传入的参数了。但是缺点也很明显，就是功能太少了。假如我们是看其他大神的代码，我们想要知道运行的时候需要传入什么参数，以及每个参数代表什么含义就做不到了。



为了解决这个问题，我们需要使用封装更多功能的工具，也是本篇文章的核心——**argparse**。





## 基本用法



`argparse`是Python当中的一个库，我们需要先`import`一下，这个库我没记错应该是Python自带的，也不需要安装，我们直接就可以使用。



在我们使用之前，我们需要先初始化这个parse，也就是一个参数解析器。



```python
# 这里ArgumentParser可以传入一个字符串，表示用途
parser = argparse.ArgumentParser()
parser.parse_args()
```



这个时候其实就已经有了一个解析器了，我们在运行的时候可以传入参数`-h`，表示help，也就是查看目前解析器当中定义的参数。由于我们现在什么也没有，所以能显示出来的就只有help。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtim589a2j318w052js7.jpg)



## 必选参数



首先我们来介绍**必选参数**，它的定义和函数当中的必填参数是一样的，也就是说我们运行程序必须要的参数。如果不传，那么程序不应该执行会进行报错并提示。



定义必选参数的方法非常简单，我们只需要通过`add_argument`传入参数的名称就可以了。



```python
import argparse

parser = argparse.ArgumentParser("For test the parser")
parser.add_argument('test')
args = parser.parse_args()

print(args.test)
```



这样我们就定义了一个名叫`test`的参数，我们可以通过`args.test`来访问它。



这个时候我们再运行`python test.py -h`就会发现提示的信息当中多了一行：



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtiu7gumxj318e086q41.jpg)



告诉我们必选参数当中有test，**必选参数直接传入，不需要加上前缀**。所以我们执行的时候直接`python test.py xxx`就可以了。



## 可选参数



有必选参数当然就有可选参数，可选参数由于可选可不选， 所以我们在使用的时候需要在参数前加上标识-或者--。比如我们参数名叫做test，可以定义成`-test`或者`--test`，这两种都可以，也可以这两种都定义。



```python
parser.add_argument('-test', '--test')
```



我们运行`-h`可以发现`optional arguments`当中多了`test`和`--test`。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtj00hkhfj3184060my4.jpg)



但是这个只print出来了参数名，并没有告诉我们这个参数究竟是做什么的，像是help参数后面就跟了show this help message and exit这个提示语。如果我们也希望help能够提示我们参数的作用怎么办呢？



我们可以通过help参数传入我们希望打印出来的提示语，这样方便使用者在使用的时候了解参数的情况。



比如我们把这行语句改成：



```python
parser.add_argument('-test', '--test', help='just for help')
```



这样当我们运行的时候，就会看到提示语了：



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtj2bewz8j31bc076t9t.jpg)



## 默认值



如果参数很多的时候，我们有时候可能不希望每一个都指定一个值，而是希望可以在不填的时候有一个默认值。这个想法非常正常，想要做到这点也很简单，我们可以通过**default参数**来指定。



```python
import argparse

parser = argparse.ArgumentParser("For test the parser")
parser.add_argument('-test', '--test', default=1, help='just for help')
args = parser.parse_args()

print(args.test)
```



比如这样我们在代码当中把test参数的默认值设置成了1，当我们运行的时候，如果不填test这个参数的话，那么程序就会使用它的默认值也就是1。



但有一点**默认值的信息并不会print在help当中**，所以我们需要自己在提示语当中告知使用者默认值是多少。



## type



我们可以定义参数的默认值，当然也可以定义它的类型。



因为**命令行传入的参数默认都是字符串**，如果我们要进行数学上的计算，使用`str`还需要自己转换，这就很不方便。我们可以在传入参数的时候就完成类型的匹配，这样如果传入参数的类型不对， 那么直接报错，不往下运行。



想要做到这点也很简单，通过`type`参数就可以实现。



```python
parser.add_argument('-test', '--test', default=1, type=int, help='just for help')
```



比如当我们定义了一个`int`型的参数，而传入的是类型不匹配的话，那么就会引起报错：



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtjk48uj6j31bo03c3za.jpg)



报错信息当中写得很清楚，我们得到了一个无效的`int`的值，它是abc。



## 可选值



它同样还支持可选值，可选值很好理解，就是我们希望**限定传入参数的范围仅仅在几个值当中**。比如说我们希望传入的值不是0就是1，或者是在某几个具体的值当中，这个时候我们可以通过`choices`参数来实现这一点。



`choices`参数传入的是一个list，也就是我们的限定范围，只有在这个范围当中的值才被允许。



```python
parser.add_argument('-test', '--test', default=1, choices=[2, 3, 4], type=int, help='just for help')
```



如果我们运行传入`test=1`，那么就会引起报错，告诉我们传入的值不在`choices`范围当中。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtjpcsyrij317g02y0tj.jpg)



这是一个挺有意思的例子，仔细看会发现我们默认值设置成了1，但是可选值当中并没有1。这也是允许的，**默认值可以不在可选值范围内**，但是当我们传入1就会触发可选值校验。



## action



`action`是一个很神奇也很有用的操作，可以**指定参数的处理方式**。我们默认的方式是store，也就是存储的意思，这个我们都能理解。除此之外，还有`store_true`，它表示出现则是true，否则是false。



```python
parser.add_argument('-test', '--test', action='store_true', help='just for help')
```



当我们把test参数的定义改成这样之后，我们来对比一下运行的结果就明白了。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtq5deajdj31bs0443zf.jpg)



除了`store_true`之外还有`store_const`，也就是说出现就指定为一个固定值。



```python
parser.add_argument('-test', '--test', action='store_const', const=23, help='just for help')
```



这样当我们指定`-test`参数之后，它会自动被赋值成23。



除了这两个之外，另外一个很常用的参数是**append**，可以将多次出现的同一个参数自动存入一个list当中。



```python
parser.add_argument('-test', '--test', action='append', type=int, help='just for help')
```



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtl70htzwj31be01w74n.jpg)



## nargs



`nargs`也是一个非常有用的参数，可以对参数进行一些花式操作。nargs的传入参数有以下几种，首先是N，也就是一个整数。代表可以**接收N个参数值**，这N个值会被存入一个list当中。



```python
parser.add_argument('-test', '--test', nargs=2, type=int, help='just for help')
```



另外一种传入的参数是`'+'`或者是`'*'`，它可以将任意多个值存入一个list当中。



```python
parser.add_argument('-test', '--test', nargs='*', type=int, help='just for help')
```



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghtlno7hoej31cg01umxk.jpg)



## 总结



有了parser之后，我们在Python当中处理命令行参数会变得非常简单，我们可以做各种各样的定制化操作。除了我们上面介绍的之外，还有一些其他的做法，相对来说不是非常常用，所以就不一一穷尽了，感兴趣的同学可以自行了解一下。