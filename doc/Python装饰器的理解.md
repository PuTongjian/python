在对Python装饰器进行说明之前，有必要先了解什么是闭包。

# 闭包的概念

我们尝试从概念上去理解一下闭包。
>在一些语言中，在函数中可以（嵌套）定义另一个函数时，如果内部的函数引用了外部的函数的变量，则可能产生闭包。闭包可以用来在一个函数与一组“私有”变量之间创建关联关系。在给定函数被多次调用的过程中，这些私有变量能够保持其持久性。—— 维基百科)

用比较容易懂的人话说，就是当某个函数被当成对象返回时，夹带了外部变量，就形成了一个闭包。看例子。
~~~python
def make_printer(msg):
    def printer():
        print msg  # 夹带私货（外部变量）
    return printer  # 返回的是函数，带私货的函数

printer = make_printer('Foo!')
printer()
~~~

支持将函数当成对象使用的编程语言，一般都支持闭包。比如Python, JavaScript。

# 如何理解闭包

- 闭包的作用是什么？  
**可以使用函数的内部变量，并且变量会被保存至内存中。**

~~~python
def tag(tag_name):
    def add_tag(content):
        return "<{0}>{1}</{0}>".format(tag_name, content)
    return add_tag

content = 'Hello'

add_tag = tag('a')
print add_tag(content)
# <a>Hello</a>

add_tag = tag('b')
print add_tag(content)
# <b>Hello</b>
~~~
在这个例子里，我们想要一个给content加tag的功能，但是具体的tag_name是什么样子的要根据实际需求来定，对外部调用的接口已经确定，就是add_tag(content)。如果按照面向接口方式实现，我们会先把add_tag写成接口，指定其参数和返回类型，然后分别去实现a和b的add_tag。  
但是在闭包的概念中，add_tag就是一个函数，它需要tag_name和content两个参数，只不过tag_name这个参数是打包带走的。所以一开始时就可以告诉我怎么打包，然后带走就行。  

# 装饰器
Python的装饰器本质就是一个函数，准确的来说，是一个闭包函数。**它可以让被装饰函数在不需要做任何代码变动的前提下增加额外的功能。**  
Python的装饰器是一个固定的函数接口形式。它要求你的装饰器函数（或装饰器类）必须接受一个函数并返回一个函数：

~~~python
# how to define
def wrapper(func1):  # 接受一个callable对象
    return func2  # 返回一个对象，一般为函数
    
# how to use
def target_func(args): # 目标函数
    pass

# 调用方式一，直接包裹
result = wrapper(target_func)(args)

# 调用方式二，使用@语法，等同于方式一
@wrapper
def target_func(args):
    pass

result = target_func()
~~~

那么如果你的装饰器如果带参数呢？那么你就需要在原来的装饰器上再包一层，用于接收这些参数。这些参数（私货）传递到内层的装饰器里后，闭包就形成了。所以说当你的装饰器需要自定义参数时，一般都会形成闭包。（类装饰器例外）
~~~python
def html_tags(tag_name):
    def wrapper_(func):
        def wrapper(*args, **kwargs):
            content = func(*args, **kwargs)
            return "<{tag}>{content}</{tag}>".format(tag=tag_name, content=content)
        return wrapper
    return wrapper_

@html_tags('b')
def hello(name='Toby'):
    return 'Hello {}!'.format(name)

# 不用@的写法如下
# hello = html_tag('b')(hello)
# html_tag('b') 是一个闭包，它接受一个函数，并返回一个函数

print hello()  # <b>Hello Toby!</b>
print hello('world')  # <b>Hello world!</b>
~~~

下面让我们来了解一下闭包的包到底长什么样子。其实闭包函数相对与普通函数会多出一个__closure__的属性，里面定义了一个元组用于存放所有的cell对象，每个cell对象一一保存了这个闭包中所有的外部变量。

~~~python
>>> def make_printer(msg1, msg2):
    def printer():
        print msg1, msg2
    return printer
>>> printer = make_printer('Foo', 'Bar')  # 形成闭包

>>> printer.__closure__   # 返回cell元组
(<cell at 0x03A10930: str object at 0x039DA218>, <cell at 0x03A10910: str object at 0x039DA488>)

>>> printer.__closure__[0].cell_contents  # 第一个外部变量
'Foo'
>>> printer.__closure__[1].cell_contents  # 第二个外部变量
'Bar'
~~~
