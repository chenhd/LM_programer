#greenlet: Lightweight concurrent programming

link：    [greenlet](http://greenlet.readthedocs.org/en/latest/)

##Usage

###Introduction

`greenlet`是一个小的，独立的虚假线程。把它当成一个小的栈帧（在C语言里，从调用函数开始到调用函数结束，所有的帧合起来就是一个栈帧）；栈帧在内存上是从小到大扩大，底部（内存地址最小的帧）便是最初的函数调用，顶部（内存地址最大的帧）便是当前`greenlet`暂停的地方（当前运行的位置，像打断点似的停留在一个位置）。**你使用`greenlet`就相当于创建一定数量的这样的栈帧然后在它们之间交换执行。****交换执行必定是显式操作**：一个`greenlet`必须选择另外一个`greenlet`交换执行。这种交换执行的操作称为"switching"(应该与yield语义有关)。

当你创建一个`greenlet`，便会得到一个初始化的空栈；一但你向它switch，就相当于开始执行一个指定的函数，它也能调用其他函数，或者switch向其它`greenlet`等。当最外层的函数（一开始执行的那个函数）执行完毕，`greenlet`的栈就再次变成空栈，然后它就"死"了。它也会死于未捕获的异常。

```python
from greenlet import greenlet

def test1():
    print 12
    gr2.switch()
    print 34

def test2():
    print 56
    gr1.switch()
    print 78

gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()
print 'over'


output:
12
56
34
over
```

###Parents

每一个`greenlet`都有一个`父greenlet`。在`greenlet`创建的时候如果没有指定的话会自动初始化一个`父greenlet`（**相当于将parent指定为greenlet.getcurrent()**， 该参数可以在任何时候更改），创建的位置所属的`greenlet`即是它的`父greenlet`。`父greenlet`的作用是当`greenlet`执行完毕"死"的时候会继续执行（相当于switch向`父greenlet`）。

`greenlet`组织成树的形状，最顶层的代码并不是运行在一个自定义的`greenlet`中而是在一个隐式的main `greenlet`中运行，它是整棵树的"root"。

在刚才的例子中，gr1和gr2的父`greenlet`都是main `greenlet`，当它们执行完毕后便会switch回"main"。

```python
from greenlet import greenlet

def test1():
    print 12
    gr2.switch()
    print 34

def test2():
    print 56
    gr1.switch()
    print 78

gr2 = greenlet(test2)
gr1 = greenlet(test1, parent=gr2)
gr1.switch()
print 'over'


output:
12
56
34
78
over
```

未捕获的异常也会传播到上一层。例如，如果一开始test2()含有一个排印错误，它会触发一个``NameError``然后杀掉gr2，然后异常直接返回给"main"。trackback会显示test2而不是test1（如果发生异常了，异常栈顺序与第一次switch执行指定函数的调用栈顺序一致）。记住，switch不是调用，但会在平行的"stack containers"中交替运行，还有"parent"参数定义依据栈的逻辑下次调用哪个。

###Instantiation(实例化)
`greenlet.greenlet`是一个greenlet类型，它支持以下操作：

* `greenlet(run=None, parent=None)`：创建一个新的greenlet对象（但并不允许它）。`run`是用来回调调用的，`parent`是用来指定`父greenlet`的，默认值是当前的greenlet对象。
* `greenlet.getcurrent()`：返回当前greenlet对象（eg：谁调用的当前函数）。
* `greenlet.GreenletExit`：该异常比较特殊并不会传递到`父greenlet`（？？）；它可以用来杀死一个单一的greenlet。


###Switching
当某个`greenlet`的switch（）被调用的时候，greenlets直接会进行切换，在这种情况下，会执行该`greenlet`，如果该`greenlet`已经死亡，那么便会继续切换到它的`父greenlet`。在一次切换之中，可以将一个对象或者一个异常作为switch（）参数传递给目标`greenlet`；这对`greenlet`之间交换信息来说是一种方便的方式。例如：
```python
from greenlet import greenlet 

def test1(x, y):
    z = gr2.switch(x+y)
    print z

def test2(u):
    print u
    gr1.switch(42)

gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch("hello", " world")
print 'over'


output:
hello world
42
over
```
注意test1和test2的参数并不是在它们创建时就传递给它们，而是在第一次切换到它的时候传递给它。

发送对象的精确规则：
* `g.switch(*args, **kwargs)`：切换到`greenlet`g，并将所带参数传递给它。作为特例，如果g还未启动，则开始启动它
* **Dying greenlet**：如果一个`greenlet`的`run()`已经结束，它（已死的`greenlet`）的返回值会传给它`父greenlet`的对象（即直接将参数传给它的`父greenlet`）。如果`run()`是异常导致的终止，那么异常将会传给它的`父greenlet`（除了`greenlet.GreenletExit`，在该情况下异常会被捕捉然后返回给它的`父greenlet`？？）。

除了上面描述的情况之外，目标`greenlet`一般将之前另外的`greenlet`调用`switch()`然后停止时传入的参数作为返回值接收。的确，虽然调用`switch()`并不会立刻返回，它将会在未来某个时刻有其他`greenlet`切换回来时返回。当这种情况发生时，从它停止的地方恢复执行，然后`switch()`就返回其他地方发送的对象。这就意味着`x = g.switch(y)`将会把`y`发送给`g`，稍后将其他`greenlet`切换回来时传入的对象从`switch()`返回给`x`。

注意，任何尝试切换到已经死的`greenlet`的实际都会切换到它们的`父greenlet`，以此类推，最后是`main greenlet`。(The final parent is the "main" greenlet, which is never dead.)

###Greenlets and Python threads
`greenlet`可以与Python的线程结合到一起；在这种情况下，每个线程包含一个独立的`mian greenlet`以及整个树的`子greenlet`，不可能将属于不同线程的`greenlet`混合或在它们之间切换。
