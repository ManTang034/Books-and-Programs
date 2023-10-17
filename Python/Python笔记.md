# Python

## 代码规范

### 1.全局变量

> 避免使用全局变量，使用模块级的常量，例如MAX_HOLY_HANDGRENADE_COUNT=3，常量命名必须全部大写，用_分隔。
>
> 应在模块内声明全局变量，并在名称前使用_使其成为模块内部变量，外部访问必须通过模块级的公共函数。

```python
# my_module.py
_MAX_HOLY_HANDGRENADE_COUNT = 3

def use_holy_handgrenade():
    global _MAX_HOLY_HANDGRENADE_COUNT
    if _MAX_HOLY_HANDGRENADE_COUNT > 0:
        _MAX_HOLY_HANDGRENADE_COUNT -= 1
        print("Holy handgrenade used!")
    else:
        print("Out of holy handgrenade!")

def add_holy_handgrenade(count):
    global _MAX_HOLY_HANDGRENADE_COUNT
    _MAX_HOLY_HANDGRENADE_COUNT += count

def get_holy_handgrenade_count():
    return _MAX_HOLY_HANDGRENADE_COUNT
  
"""
在这个例子中，_MAX_HOLY_HANDGRENADE_COUNT是模块级的全局变量，只在my_module.py这个模块内部可见。外部访问这个变量只能通过模块级公共函数get_holy_handgrenade_count获取。add_holy_handgrenade和use_holy_handgrenade是操作这个变量的函数，也只能在模块内部使用。这种方式避免了直接暴露全局变量，同时方便了模块内部对状态的维护。
"""
```

### 2.嵌套/局部/内部类或函数

> 定义：类可以定义在方法，函数或者类中。函数可以定义在方法或函数中。封闭区间中定义的变量对嵌套函数是只读的。（即内嵌函数可以读外部函数中定义的变量，但是无法改写，除非使用nonlocal）
>
> 应该避免使用内嵌函数或类，若想对模块的用户隐藏某个函数，不要采用嵌套它来隐藏，应该在需要被隐藏的方法的模块级名称加_前缀，这样它仍然是可以被测试的。

```python
# 首先看类在方法中的定义：
class MyClass:
    def my_method(self):
        class MyInnerClass:
            def __init__(self):
                self.name = "inner"

        inner_obj = MyInnerClass()
        print(inner_obj.name)

my_obj = MyClass()
my_obj.my_method()  # 输出 "inner"
"""
在这个例子中，我们定义了一个外部类 MyClass，并在它的方法 my_method 中定义了一个内部类 MyInnerClass。在方法中我们创建了一个 MyInnerClass 的对象 inner_obj，并输出了它的属性 name。
"""

# 接下来看函数在方法中的定义：
class MyClass:
    def my_method(self):
        def my_inner_func():
            return "inner"

        print(my_inner_func())

my_obj = MyClass()
my_obj.my_method()  # 输出 "inner"
"""
在这个例子中，我们定义了一个外部类 MyClass，并在它的方法 my_method 中定义了一个内部函数 my_inner_func。在方法中我们调用了 my_inner_func 并输出了它的返回值。
"""

# 最后看封闭区间中定义的变量对嵌套函数的影响：
def outer_func():
    name = "outer"
    def inner_func():
        print(name)
    inner_func()

outer_func()  # 输出 "outer"
"""
在这个例子中，我们定义了一个外部函数 outer_func，并在它的内部定义了一个字符串变量 name。然后我们又定义了一个内部函数 inner_func，在它的输出语句中使用了变量 name。最后我们调用了 outer_func，输出了 name 的值。可以看到，在 Python 中，内嵌函数可以访问外部函数中的变量，但无法改写它。如果需要在内嵌函数中改写外部函数中的变量，可以使用 nonlocal 关键字。
"""

def outer_func():
    name = "outer"
    def inner_func():
        nonlocal name
        name = "inner"
        print(name)

    inner_func()
    print(name)

outer_func() # 输出inner,inner
"""
在这个例子中，我们定义了一个外部函数 outer_func，并在它的内部定义了一个字符串变量 name。然后我们又定义了一个内部函数 inner_func，在它的内部使用了 nonlocal 关键字来声明 name 是一个非局部变量。在内部函数中，我们将 name 的值修改为 "inner"，并输出了修改后的值。
"""
```

### 3.推导式&生成式

> 定义：列表、字典和集合的推导式&生成式提供了一种简洁高效的方式来创建容器和迭代器，而不必借助map()，filter()或者lambda。
>
> 结论：适用于简单情况；每个部分应该单独置于一行：映射表达式，for语句，过滤器表达式。禁止多重for语句或过滤器表达式。复杂情况下还是使用循环。

```python
result = [mapping_expr for value in iterable if filter_expr]
"""
mapping_expr：对可迭代对象中的每个元素执行的映射表达式。该表达式可以对元素进行操作或转换。
value：循环变量，表示可迭代对象中的当前元素。
iterable：可迭代对象，例如列表、元组、字符串等。
filter_expr：过滤条件表达式，用于过滤要包含在最终列表中的元素。
"""

result = [{'key': value} for value in iterable
						if a_long_filter_expression(value)] 

result = [complicated_transform(x) for x in iterable if predicate(x)]

descriptive_name = [
transform({'key': key, 'value': value}, color='black') for key, value in generate_iterable(some_input)
if complicated_condition_is_met(key, value)
]
result = []
for x in range(10):
for y in range(5): if x * y > 10:
            result.append((x, y))
return {x: complicated_transform(x)
for x in long_generator_function(parameter) if x is not None}
squares_generator = (x**2 for x in range(10))
unique_names = {user.name for user in users if user is not None}
eat(jelly_bean for jelly_bean in jelly_beans if jelly_bean.color == 'black')
```



## 代码的抽象三原则

软件开发是“抽象化”原则的一种体现。“抽象化”指从具体问题中，提取出具有共性的模式，再使用通用的解决方法加以处理。

> 一. DRY原则（Don't Repeat Yourself）
>
> 也称为“一次且仅一次原则”（Once and Only one）
>
> 系统的每一个功能都应该有唯一的实现。也就是说，如果多次遇到同样的问题，就应该抽象出一个共同的解决方法，不要重复开发同样的功能。

> 二. YAGNI原则（You aren't gonna need it）
>
> 这是“极限编程”提倡的原则，指的是你自以为有用的功能，实际上都是用不到的。因此，除了最核心的功能，其他功能一概不要部署，这样可以大大加快开发。
>
> 尽可能简单地让软件运行起来，do the simplest thing that could possibly work.

> 三. Rule of Three原则
>
> 指的是当某个功能第三次出现时，才进行“抽象化”。
>
> 1. 省事。如果一种功能只有一到两个地方会用到，就不需要在“抽象化“上面耗费时间。
> 2. 容易发现模式。”抽象化“需要找到问题的模式，问题出现的场合越多，就越容易看出模式，从而可以更准确地i”抽象化“
> 3. 防止过度冗余。如果一种功能同时有多个实现，管理起来非常麻烦，修改的时候需要修改多处。在实际工作中，重复实现最多可以容忍出现一次，再多就无法接受了。

