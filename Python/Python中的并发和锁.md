这篇我们一起来聊聊多线程场景当中不可或缺的另外一个部分——**锁**。



如果你学过操作系统，那么对于锁应该不陌生。锁的含义是线程锁，可以用来指定某一个逻辑或者是资源**同一时刻只能有一个线程访问**。这个很好理解，就好像是有一个房间被一把锁锁住了，只有拿到钥匙的人才能进入。每一个人从房间门口拿到钥匙进入房间，出房间的时候会把钥匙再放回到门口。这样下一个到门口的人就可以拿到钥匙了。这里的房间就是某一个资源或者是一段逻辑，而拿取钥匙的人其实指的是一个线程。



## 加锁的原因



我们明白了锁的原理，不禁有了一个问题，我们为什么需要锁呢，它在哪些场景当中会用到呢？



其实它的使用场景非常广，我们举一个非常简单的例子，就是淘宝买东西。我们都知道商家的库存都是有限的，卖掉一个少一个。假如说当前某个商品库存只剩下一个，但当下却有两个人同时购买。两个人同时购买也就是有两个请求同时发起购买请求，如果我们不加锁的话，两个线程同时查询到商品的库存是1，大于0，进行购买逻辑之后，减一。由于两个线程同时执行，所以最后商品的库存会变成-1。



显然商品的库存不应该是一个负数，所以我们需要避免这种情况发生。通过**加锁**可以完美解决这个问题。我们规定一次只能有一个线程发起购买的请求，那么这样当一个线程将库存减到0的时候，第二个请求就无法修改了，就保证了数据的准确性。



## 代码实现



那么在Python当中，我们怎么样来实现这个锁呢？



其实很简单，`threading`库当中已经为我们提供了现成的工具，我们直接拿过来用就可以了。我们通过使用`threading`当中的`Lock`对象， 可以很轻易的实现方法加锁的功能。



```python
import threading

class PurchaseRequest:
    '''
    初始化库存与锁
    '''
    def __init__(self, initial_value = 0):
        self._value = initial_value
        self._lock = threading.Lock()

    def incr(self,delta=1):
        '''
       	加库存
        '''
        self._lock.acquire()
        self._value += delta
        self._lock.release()

    def decr(self,delta=1):
        '''
        减库存
        '''
        self._lock.acquire()
        self._value -= delta
        self._lock.release()
```



我们从代码当中就可以很轻易的看出`Lock`这个对象的使用方法，我们在进入**加锁区**（资源抢占区）之前，我们需要先使用`lock.acquire()`方法获取锁。`Lock`对象可以保证同一时刻只能有一个线程获取锁，只有获取了锁之后才会继续往下执行。当我们执行完成之后，我们需要把锁“放回门口”，所以需要再调用一下`release`方法，表示锁的释放。



这里有一个小问题是很多程序员在编程的时候总是会**忘记`release`，导致不必要的bug**，而且这种分布式场景当中的bug很难通过测试发现。因为测试的时候往往很难测试并发场景，code review的时候也很容易忽略，因此一旦泄露了还是挺难发现的。



为了解决这个问题，`Lock`还提供了一种改进的用法，就是**使用with语句**。with语句我们之前在使用文件的时候用到过，使用with可以替我们完成try catch以及资源回收等工作，我们只管用就完事了。这里也是一样，使用`with`之后我们就可以不用管锁的申请和释放了，直接写代码就行，所以上面的代码可以改写成这样：



```python
import threading

class PurchaseRequest:
    '''
    初始化库存与锁
    '''
    def __init__(self, initial_value = 0):
        self._value = initial_value
        self._lock = threading.Lock()

    def incr(self,delta=1):
        '''
       	加库存
        '''
		with self._lock:
	        self._value += delta

    def decr(self,delta=1):
        '''
        减库存
        '''
        with self._lock:
	        self._value -= delta
```



这样看起来是不是清爽很多？



## 可重入锁



上面介绍的只是最简单的锁，我们经常使用的往往是**可重入锁**。



什么叫可重入锁呢？简单解释一下，就是在一个线程已经持有了锁的情况下，它可以再次进入被加锁的区域。但是既然线程还持有锁没有释放，那么它不应该还是在加锁区域吗，怎么会有需要再次进入被加锁区域的情况呢？其实是有的，**道理也很简单，就是递归**。



我们把上面的例子稍微改一点点，就完全不一样了。



```python
import threading

class PurchaseRequest:
    '''
    初始化库存与锁
    '''
    def __init__(self, initial_value = 0):
        self._value = initial_value
        self._lock = threading.Lock()

    def incr(self,delta=1):
        '''
       	加库存
        '''
		with self._lock:
	        self._value += delta

    def decr(self,delta=1):
        '''
        减库存
        '''
        with self._lock:
	        self.incr(-delta)
```



我们关注一下上面的`decr`方法，我们用`incr`来代替了原本的逻辑实现了`decr`。但是有一个问题是decr也是一个加锁的方法，需要前一个锁释放了才能进入。但它已经持有了锁了，那么这种情况下就会发生**死锁**。



我们只需要把Lock换成可重入锁就可以解决这个问题，只需要修改一行代码。



```python
import threading

class PurchaseRequest:
    '''
    初始化库存与锁
    我们使用RLock代替了Lock，也可重入锁代替了普通锁
    '''
    def __init__(self, initial_value = 0):
        self._value = initial_value
        self._lock = threading.RLock()

    def incr(self,delta=1):
        '''
       	加库存
        '''
		with self._lock:
	        self._value += delta

    def decr(self,delta=1):
        '''
        减库存
        '''
        with self._lock:
	        self.incr(-delta)
```



## 总结



今天我们的文章介绍了Python当中锁的使用方法，以及可重入锁的概念。在并发场景下开发和调试都是一个比较困难的工作，稍微不小心就会踩到各种各样的坑，**死锁只是其中一种比较常见并且比较容易解决的问题**，除此之外还有很多其他各种各样的问题。



针对死锁的问题，Python还提供了其他的解决方案，我们放到下一篇文章当中再和大家分享。