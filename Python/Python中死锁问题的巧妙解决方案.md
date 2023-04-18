---
title: Python中死锁问题巧妙解决办法
date: 2023-04-18 09:18:03
categories: 
- Python
tags:
- 多线程
- 多进程
- 并发

---

这篇文章，我们一起来聊聊多线程开发当中死锁的问题。



## 死锁



死锁的原理非常简单，用一句话就可以描述完。就是当多线程访问多个锁的时候，**不同的锁被不同的线程持有**，它们都在等待其他线程释放出锁来，于是便陷入了永久等待。比如A线程持有1号锁，等待2号锁，B线程持有2号锁等待1号锁，那么它们永远也等不到执行的那天，这种情况就叫做死锁。



关于死锁有一个著名的问题叫做**哲学家就餐**问题，有5个哲学家围坐在一起，他们每个人需要拿到两个叉子才可以吃饭。如果他们同时拿起自己左手边的叉子，那么就会永远等待右手边的叉子释放出来。这样就陷入了永久等待，于是这些哲学家都会饿死。



![img](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghdbrswecrj30sg0tie4y.jpg)



这是一个很形象的模型，因为在计算机并发场景当中，一些**资源的数量往往是有限的**。很有可能出现多个线程抢占的情况，如果处理不好就会发生大家都获取了一部分资源，然后在等待另外的资源的情况。



对于死锁的问题有多种解决方法，这里我们介绍比较简单的一种，就是对这些锁进行编号。我们规定当一个线程需要同时持有多个锁的时候，**必须要按照序号升序的顺序对这些锁进行访问**。通过上下文管理器我们可以很容易实现这一点。



## 上下文管理器



首先我们来简单介绍一下上下文管理器，上下文管理器我们其实经常使用，比如我们经常使用的**with语句**就是一个上下文管理器的经典使用。当我们通过`with`语句打开文件的时候，它会自动替我们处理好文件读取之后的关闭以及抛出异常的处理，可以节约我们大量的代码。



同样我们也可以自己定义一个上下文处理器，其实很简单，我们只需要实现`__enter__`和`__exit__`这两个函数即可。`__enter__`函数用来实现进入资源之前的操作和处理，那么显然`__exit__`函数对应的就是使用资源结束之后或者是出现异常的处理逻辑。有了这两个函数之后，我们就有了自己的上下文处理类了。



我们来看一个样例：



```python
class Sample:
    def __enter__(self):
        print('enter resources')
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print('exit')
        # print(exc_type)
        # print(exc_val)
        # print(exc_tb)

    def doSomething(self):
        a = 1/1
        return a

def getSample():
    return Sample()

if __name__ == '__main__':
    with getSample() as sample:
        print('do something')
        sample.doSomething()
```



当我们运行这段代码的时候，屏幕上打印的结果和我们的预期是一致的。



![image-20200803091558632](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ghdcveh1o3j30mr025aa5.jpg)



我们观察一下`__exit__`函数，会发现它的参数有4个，**后面的三个参数对应的是抛出异常的情况**。`type`对应异常的类型，`val`对应异常时的输出值，`trace`对应异常抛出时的运行堆栈。这些信息都是我们排查异常的时候经常需要用到的信息，通过这三个字段，我们可以根据我们的需要对可能出现的异常进行自定义的处理。



实现上下文管理器并不一定要通过类实现，Python当中也提供了上下文管理的注解，通过使用注解我们可以很方便地实现上下文管理。我们同样也来看一个例子：



```python
import time
from contextlib import contextmanager

@contextmanager
def timethis(label):
    start = time.time()
    try:
        yield
    finally:
        end = time.time()
        print('{}: {}'.format(label, end - start))
        
        
with timethis('timer'):
    pass
```



在这个方法当中yield之前的部分相当于`__enter__`函数，`yield`之后的部分相当于`__exit__`。如果出现异常会在try语句当中抛出，那么我们编写except对异常进行处理即可。



## 避免死锁



了解了上下文管理器之后，我们要做的就是**在lock的外面包装一层**，使得我们在获取和释放锁的时候可以根据我们的需要，对锁进行排序，按照升序的顺序进行持有。



这段代码源于Python的著名进阶书籍《Python cookbook》，非常经典：



```python
from contextlib import contextmanager

# 用来存储local的数据
_local = threading.local()

@contextmanager
def acquire(*locks):
	# 对锁按照id进行排序
    locks = sorted(locks, key=lambda x: id(x))

    # 如果已经持有锁当中的序号有比当前更大的，说明策略失败
    acquired = getattr(_local,'acquired',[])
    if acquired and max(id(lock) for lock in acquired) >= id(locks[0]):
        raise RuntimeError('Lock Order Violation')

    # 获取所有锁
    acquired.extend(locks)
    _local.acquired = acquired

    try:
        for lock in locks:
            lock.acquire()
        yield
    finally:
        # 倒叙释放
        for lock in reversed(locks):
            lock.release()
        del acquired[-len(locks):]

```



这段代码写得非常漂亮，可读性很高，逻辑我们都应该能看懂，但是有一个小问题是这里用到了**`threading.local`**这个组件。



它是一个多线程场景当中的**共享变量**，虽然说是共享的，但是对于每个线程来说读取到的值都是独立的。听起来有些难以理解，其实我们可以将它理解成一个`dict`，`dict`的key是每一个线程的id，value是一个存储数据的dict。每个线程在访问local变量的时候，都相当于先通过线程id获取了一个独立的dict，再对这个dict进行的操作。



看起来我们在使用的时候直接使用了`_local`，这是因为通过线程id先进行查询的步骤在其中封装了。不明就里的话可能会觉得有些难以理解。



我们再来看下这个acquire的使用：



```python
x_lock = threading.Lock()
y_lock = threading.Lock()

def thread_1():
    while True:
        with acquire(x_lock, y_lock):
            print('Thread-1')

def thread_2():
    while True:
        with acquire(y_lock, x_lock):
            print('Thread-2')

t1 = threading.Thread(target=thread_1)
t1.start()

t2 = threading.Thread(target=thread_2)
t2.start()
```



运行一下会发现没有出现死锁的情况，但如果我们把代码稍加调整，写成这样，那么就会触发异常了。



```python
def thread_1():
    while True:
        with acquire(x_lock):
            with acquire(y_lock):
	            print('Thread-1')

def thread_2():
    while True:
        with acquire(y_lock):
            with acquire(x_lock):
	            print('Thread-1')
```



因为我们把锁写成了层次结构，这样就没办法进行排序保证持有的有序性了，那么就会触发我们代码当中定义的异常。



最后我们再来看下哲学家就餐问题，通过我们自己实现的acquire函数我们可以非常方便地解决他们死锁吃不了饭的问题。



```python
import threading

def philosopher(left, right):
    while True:
        with acquire(left, right):
             print(threading.currentThread(), 'eating')

# 叉子的数量
NSTICKS = 5
chopsticks = [threading.Lock() for n in range(NSTICKS)]

for n in range(NSTICKS):
    t = threading.Thread(target=philosopher,
                         args=(chopsticks[n],chopsticks[(n+1) % NSTICKS]))
    t.start()
```



## 总结



关于死锁的问题，对锁进行排序**只是其中的一种解决方案**，除此之外还有很多解决死锁的模型。比如我们可以让线程在尝试持有新的锁失败的时候主动放弃所有目前已经持有的锁，比如我们可以设置机制检测死锁的发生并对其进行处理等等。发散出去其实有很多种方法，这些方法起作用的原理各不相同，其中涉及大量操作系统的基础概念和知识，感兴趣的同学可以深入研究一下这个部分，一定会对操作系统以及锁的使用有一个深刻的认识。



