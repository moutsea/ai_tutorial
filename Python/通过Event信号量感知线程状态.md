今天这篇文章我们继续多线程的话题。



上篇文章当中我们简单介绍了线程和进程的概念，以及在Python当中如何在主线程之外创建其他线程，并且还了解了**用户级线程和后台线程**的区别以及使用方法。今天我们来看看线程的其他使用，比如如何停止一个线程，线程之间的Event用法等等。



## 停止线程





利用`Threading`库我们可以很方便地创建线程，让它按照我们的想法执行我们想让它执行的事情，从而加快程序运行的效率。然而有一点坑爹的是，线程创建之后，就交给了操作系统执行，**我们无法直接结束一个线程，也无法给它发送信号，无法调整它的调度**，也没有其他高级操作。如果想要相关的功能，只能自己开发。



怎么开发呢？



我们创建线程的时候指定了`target`等于一个我们想让它执行的函数，**这个函数并不一定是全局函数，实际上也可以是一个对象中的函数**。如果是对象中的函数，那么我们就可以在这个函数当中获取到对象中的其他信息，我们可以利用这一点来实现手动控制线程的停止。



说起来好像不太好理解，但是看下代码真的非常简单：



```python
import time
from threading import Thread

class TaskWithSwitch:
    def __init__(self):
        self._running = True
        
    def terminate(self):
        self._running = False

    def run(self, n):
        while self._running and n > 0:
            print('Running {}'.format(n))
            n -= 1
            time.sleep(1)

c = TaskWithSwitch()
t = Thread(target=c.run, args=(10, ))
t.start()
c.terminate()
t.join()
```



如果你运行这段代码，会发现屏幕上只输出了10，因为我们将`_running`这个字段置为`False`之后，下次循环的时候不再满足循环条件，它就会自己退出了。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ggh3b5y6tmj30mx011mx6.jpg)



如果我们想要用多线程来读取`IO`，由于**IO可能存在堵塞**，所以可能会出现线程一直无法返回的情况。也就是说我们在循环内部卡死了，这个时候单纯用`_running`来判断还是不够的，我们需要**在线程内部设置计时器**，防止循环内部的卡死。



```python
class IOTask:
    def __init__(self):
        self._running = True
        
    def terminate(self):
        self._running = False

    def run(self, sock):
        # 在socket中设置计时器
        sock.settimeout(10)
        while self._running:
            try:
                # 由于设置了计时器，所以这里不会永久等待
                data = sock.recv(1024)
                break
            except socket.timeout:
                continue
        return
```





## 线程信号的传递



我们之所以如此费劲才能控制线程的运行，主要原因是**线程的状态是不可知的**，并且我们无法直接操作它，因为它是被操作系统管理的。我们运行的主线程和创建出来的线程是独立的，两者之间并没有从属关系，所以想要实现对线程的状态进行控制，往往需要我们通过其他手段来实现。



我们来思考一个场景，假设我们有一个任务，需要在另外一个线程运行结束之后才能开始执行。要想要实现这一点，就**必须对线程的状态有所感知**，需要其他线程传递出信号来才行。我们可以使用`threading`中的`Event`工具来实现这一点。`Event`工具就是可以用来传递信号的，就好像是一个开关，当一个线程执行完成之后，会去启动这个开关。而这个开关控制着另外一段逻辑的运行。



我们来看下样例代码：



```python
import time
from threading import Thread, Event

def run_in_thread():
    time.sleep(1)
    print('Thread is running')

t = Thread(target=run_in_thread)
t.start()

print('Main thread print')

```



我们在线程里面就只做了输出一行提示符，没有其他任何逻辑。由于我们在`run_in_thread`函数当中沉睡了`1s`，所以一定是先输出`Main thread print`再输出的`Thread is running`。假设这个线程是一个很重要的任务，我们希望主线程能够等待它运行到一个阶段再往下执行，我们应该怎么办呢？



注意，**这里说的是运行到一个阶段，并不是运行结束**。运行结束我们很好处理，可以通过`join`来完成。但如果不是运行结束，而是运行完成了某一个阶段，当然通过`join`也可以，但是会损害整体的效率。这个时候我们就必须要用上`Event`了。加上`Event`之后，我们再来看下代码：



```python
import time
from threading import Thread, Event

def run_in_thread(event):
    time.sleep(1)
    print('Thread is running')
    # set一下event，这样外面wait的部分就会被启动
    event.set()

# 初始化Event
event = Event()
t = Thread(target=run_in_thread, args=(event, ))
t.start()

# event等待set
event.wait()
print('Main thread print')
```



整体的逻辑没有太多的修改，主要的是增加了几行关于`Event`的使用代码。



我们**如果要用到`Event`，最好在代码当中只使用一次**。当然通过`Event`中的`clear`方法我们可以重置`Event`的值，但问题是我们没办法保证重置的这个逻辑会在`wait`之前执行。如果是在之后执行的，那么就会问题，并且在`debug`的时候会异常痛苦，因为`bug`不是必现的，而是有时候会出现有时候不会出现。这种情况往往都是因为多线程的使用问题。



所以如果要多次使用开关和信号的话，不要使用`Event`，可以使用信号量。



## 信号量



`Event`的问题在于如果多个线程在等待`Event`的发生，当它一旦被`set`的时候，那么这些线程都会同时执行。但有时候我们并不希望这样，我们**希望可以控制这些线程一个一个地运行**。如果想要做到这一点，`Event`就无法满足了，而需要使用信号量。



信号量和`Event`的使用方法类似，不同的是，**信号量可以保证每次只会启动一个线程**。因为这两者的底层逻辑不太一致，对于`Event`来说，它更像是一个开关。一旦开关启动，所有和这个开关关联的逻辑都会同时执行。而信号量则像是许可证，只有拿到许可证的线程才能执行工作，并且许可证一次只发一张。



想要使用信号量并不需要自己开发，`thread`库当中为我们提供了现成的工具——`Semaphore`，我们来看它的使用代码：



```python
# 工作线程
def worker(n, sema):
    # 等待信号量
    sema.acquire()
    print('Working', n)

# 初始化
sema = threading.Semaphore(0)
nworkers = 10
for n in range(nworkers):
    t = threading.Thread(target=worker, args=(n, sema,))
    t.start()
```



在上面的代码当中我们创建了10个线程，虽然这些线程都被启动了，但是都不会执行逻辑，因为**`sema.acquire`是一个阻塞方法**，没有监听到信号量是会一直挂起等待。



![](https://moutsea-blog.oss-cn-hangzhou.aliyuncs.com/007S8ZIlgy1ggh3b57xvoj30ga03kdfq.jpg)



当我们释放信号量之后，线程被启动，才开始了执行。我们每释放一个信号，则会多启动一个线程。这里面的逻辑应该不难理解。



## 总结





在并发场景当中，多线程的使用**绝不是多启动几个线程做不同的任务**而已，我们需要线程间协作，需要同步、获取它们的状态，这是非常不容易的。一不小心就会出现幽灵`bug`，时显时隐，这也是并发问题让人头疼的主要原因。



这篇文章当中我们只是简单介绍了线程间通信的基本方法，针对这个问题，**还有更好的解决方案**。我们将在后续的文章当中继续讨论这个问题，敬请期待。