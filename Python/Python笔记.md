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



### 4.生成器

> 生成器函数就是每当它执行一次生成（yield）语句，它就返回一个迭代器，这个迭代器生成一个值。生成值后，生成器函数的运行状态将被挂起，直到下一次生成。

```python
def my_generator():
    for i in range(5):
        yield i

# 使用生成器
my_gen = my_generator()
for num in my_gen:
    print(num)
"""
当执行 my_gen = my_generator()时，函数 my_generator() 并不会立即执行，而是返回一个生成器对象 my_gen。

第一次迭代时，调用 next(my_gen) 或直接使用 for 循环开始执行生成器。程序进入 my_generator() 函数，并执行第一次循环。在循环体内，遇到 yield 0，它将数字 0 返回给迭代器的调用者，并暂停函数的执行。此时，生成器的状态被保留，记住了程序的位置和局部变量的值。

迭代器获取到了数字 0，打印出来，并继续请求下一个值。程序重新进入 my_generator() 函数，并从上次暂停的地方继续执行。这次循环中，遇到 yield 1，将数字 1 返回给迭代器，并再次暂停。

迭代器获取到数字 1，打印出来，并继续请求下一个值。程序再次从上次暂停的地方继续执行，依次类推，直到生成器完成所有循环。

最终生成器完成迭代，没有更多的值可供产生，迭代结束。

总结来说，生成器通过 yield 的使用，将函数的执行过程分成多个阶段，每次返回一个值并暂停，等待下一次请求。这使得我们可以逐步获取结果，而不需要一次性生成或存储所有的值。
"""
```

生成器执行方式的好处：

- **节省内存**：生成器一次只生成一个值，并在需要时返回。这意味着生成器不会一次性生成或存储整个结果集，从而减少内存占用。特别适用于处理大型数据集或无限序列。
- **惰性计算**：生成器使用惰性计算，也就是在需要时才进行计算。这样可以延迟计算复杂或耗时的操作，提高程序的性能和效率。
- **可迭代性**：生成器对象是可迭代的，可以像列表或其他容器一样使用。我们可以使用for循环一次迭代生成器的值，或者使用next()函数逐个获取生成器的下一个值。
- **状态保留**：生成器在每次遇到yield时暂停，并保存当前的状态。当再次调用生成器时，它可以从上次离开的地方继续执行，保留了函数内的局部变量和程序位置。这使得生成器非常灵活，可以处理需要记住先前状态的任务。
- **简洁性**：相比手动实现迭代器的复杂性，生成器提供了一种更直观、简洁的编程方式。通过使用yield语句，我们可以将生成器视为一个特殊类型的函数，而不必编写完整的迭代器类。

### 5.lambda函数

> lambda arguments: expression
>
> 适用于单行函数，如果代码超过60-80个字符，最好还是定义成常规（嵌套）函数

```python
add = lambda x,y:x+y
res = add(3,4)
print(res)
```

lambda函数通常与高阶函数结合使用，例如map()、filter()、reduce()等函数，以提供更简洁的代码实现

```python
# map()函数中使用lambda函数对列表中的每个元素进行操作
numbers=[1,2,3,4,5]
squared_numbers=list(map(lambda x:x**2,numbers))
print(squared_numbers) # 输出[1,4,9,16,25]

# filter()函数中使用lambda函数筛选列表中的元素
numbers=[1,2,3,4,5]
even_numbers=list(filter(lambda x:x%2==0,numbers))
print(even_numbers) # 输出[2,4]

# 在排序函数中使用lambda函数指定自定义的排序逻辑
names = ["Alice", "Bob", "Charlie", "Dave"]
sorted_names = sorted(names, key=lambda x: len(x))
print(sorted_names)  # 输出: ['Bob', 'Dave', 'Alice', 'Charlie']

# 在需要传递简单函数作为参数的场景中，使用lambda函数代替命名函数
def apply_function(func,x):
    return func(x)
res = apply_function(lambda x:x**2,3)
print(res) # 输出：9
```



### 6.特性

> python中数据的属性和处理数据的方法统称属性（attribute），而在不改变类接口的前提下用来修改数据属性的存取方法我们称为特性（property）。
>



### 7.True/False的求值

> python中所有的“空”值都被认为是false，因此0，None，[]，{}都被认为是false
>
> 1.**对于** **None** **等单例对象测试时,** **使用** **is** **或者** **is not**. **当你要测试一个默认值是** **None** **的变量或参**
>
> **数是否被设为其它值.** **这个值在布尔语义下可能是** **false!**
>
> 2. 永远不要用 == 将一个布尔量与 false 相比较. 使用 if not x: 代替. 如果你需要区分 false 和
>
> None, 你应该用像 if not x and x is not None: 这样的语句.
>
> 3. 对于序列 (字符串, 列表, 元组), 要注意空序列是 false. 因此 if not seq: 或者 if seq: 比 if
>
> len(seq): 或 if not len(seq): 要更好.
>
> 4. 处理整数时, 使用隐式 false 可能会得不偿失 (即不小心将 None 当做 0 来处理). 你可以将一个已知
>
> 是整型 (且不是 len() 的返回结果) 的值与 0 比较.

```python
if not users:
    print('no users')
    
if foo==0:
    self.handle_zero()
    
if i%10==0:
    self.handle_multiple_of_ten()
    
def f(x=None):
    if x is None:
        x=[]
```



### 8.注释

**类**

```python
class SampleClass(object):
    """Summary of class here.
    
    Longer class information....
    Longer class information....
    
    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """
    
    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0
        
    def public_method(self):
    	"""Performs operation blah."""
```

**块注释和行注释**

最需要写注释的是代码中那些技巧性的部分。如果你在下次代码审查的时候必须解释一下，那么你应该现在就给它写注释。对于复杂的操作，应该在其操作开始前写上若干行注释。对于不是一目了然的代码，应在其行尾添加注释。（为了提高可读性，注释应该至少离开代码2个空格）

```python
# We use a weighted dictionary search to find out where i is in
# the array. We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.

if i & (i-1) == 0:  # True if i is 0 or a power of 2.
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

