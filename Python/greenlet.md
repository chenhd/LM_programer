#greenlet: Lightweight concurrent programming

##Usage

###Introduction

"greenlet"是一个小的，独立的虚假线程。把它当成一个小的栈帧（在C语言里，从调用函数开始到调用函数结束，所有的帧合起来就是一个栈帧）；栈帧在内存上是从小到大扩大，底部（内存地址最小的帧）便是最初的函数调用，顶部（内存地址最大的帧）便是当前"greenlet"暂停的地方（当前运行的位置，像打断点似的停留在一个位置）。**你使用"greenlet"就相当于创建一定数量的这样的栈帧然后在它们之间交换执行。****交换执行必定是显式操作**：一个"greenlet"必须选择另外一个"greenlet"交换执行。这种交换执行的操作称为"switching"(应该与yield语义有关)。

当你创建一个"greenlet"，便会得到一个初始化的空栈；一但你向它switch，就相当于开始执行一个指定的函数，它也能调用其他函数，或者switch向其它"greenlet"等。当最外层的函数（一开始执行的那个函数）执行完毕，"greenlet"的栈就再次变成空栈，然后它就"死"了。它也会死于未捕获的异常。

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

#Parents
每一个"greenlet"都有一个"父greenlet"。在"greenlet"创建的时候如果没有指定的话会自动初始化一个"父greenlet"（**相当于将parent指定为greenlet.getcurrent()**， 该参数可以在任何时候更改），创建的位置所属的"greenlet"即是它的"父greenlet"。"父greenlet"的作用是当"greenlet"执行完毕"死"的时候会继续执行（相当于switch向"父greenlet"）。

"greenlet"组织成树的形状，最顶层的代码并不是运行在一个自定义的"greenlet"中而是在一个隐式的main "greenlet"中运行，它是整棵树的"root"。

在刚才的例子中，gr1和gr2的父"greenlet"都是main "greenlet"，当它们执行完毕后便会switch回"main"。
