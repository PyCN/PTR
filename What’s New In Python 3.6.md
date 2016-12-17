原文：[What’s New In Python 3.6][1]

# What's New In Python 3.6¶

Release:| 3.6.0  
---|---  
Date:| December 15, 2016  
Editors:| Elvis Pranskevichus
&lt;[elvis@magic.io][2]&gt;, Yury Selivanov
&lt;[yury@magic.io][3]&gt;  

This article explains the new features in Python 3.6, compared to 3.5.

See also

[**PEP 494**][4] \- Python 3.6 Release
Schedule

## 摘要 - 发布亮点¶

新的语法特性：

  * PEP 498, 格式化字符串字面量
  * PEP 515, 数字字面量中的下划线
  * PEP 526, 变量注解中的语法
  * PEP 525, 异步生成器
  * PEP 520: 异步解析式

新的库模块

  * [`secrets`][5]: PEP 506 - 在标准库中添加了Secrets模块

CPython实现的改进：

  * 重新实现了[字典(dict)][6]类型，以便能像[PyPy的字典类型][7]一样使用更紧凑的表达方式。与Python 3.5相比，这使字典的内存用量减少了20%到25%。
  * 用新协定优化了类的自定义建立。
  * 类属性定义顺序(class attribute definition order)现在被保留了
  * \*\*kwargs内的元素顺序现在对应于将关键字(保留字）参数传递给函数的顺序
  * 新增了对DTrace和SystemTap probing的支持。
  * 新PYTHONMALLOC环境变量现在可用于调试解释器内存分配与访问错误。

标准库的重大改进：

  * 为[asyncio][8]模块开发了新功能、显著的可用性、性能优化，以及大量的错误修复。 从Python 3.6开始，asyncio模块不再是临时的了，其API也进入了稳定状态。
  * 实现了用于支持[类路径对象(path-like objects)][9]的新文件系统路径协议。 所有在路径(path)上使用的标准库函数都已更新，以便适应于新协议。
  * [datetime][10]模块已获得对本地时间消歧(Local Time Disambiguation)的支持。
  * 针对[typing][11]模块的一些改进，使其不再是临时模块。
  * [tracemalloc][12]模块已重大改进，现用于为ResourceWarning提供更好的输出，并为内存分配错误提供更好的诊断。 欲知详情，请参阅PYTHONMALLOC部分。

安全相关的改进:

  * 新[secrets模块][13]被用于简化那些适用于管理密文的密码学安全伪随机数生成器(cryptographically strong pseudo-random numbers)的生成过程，如认证、token等。
  * 在Linux上，现将[os.urandom()][14]改成了阻塞模式，直到系统的urandom的熵池(entropy pool)的初始化具有更高的安全性。 解释请参见[**PEP 524**][15]。
  * [`hashlib`][16]和[`ssl`][17]模块现已支持OpenSSL 1.1.0。
  * 改进了[`ssl模块`][18]的默认设置和特性集。
  * 新增了[`hashlib模块`][19]对BLAKE2、SHA-3、SHAKE哈希算法以及[`scrypt()`][20]密钥导出函数的支持。

Windows上的改进:

  * PEP 528与PEP 529，Windows文件系统和控制台的编码已更改为UTF-8。
  * 当用户没有指定版本（通过命令行参数或配置文件）时，py.exe启动器以交互方式使用时，不再以Python 2优先于Python 3。 处理shebang行的方式保持不变 - 此处的“python”依旧指Python 2。
  * python.exe和pythonw.exe已标记为长路径敏感(long-path aware)，这意味着260字符路径限制可能不再适用。 有关详细信息，请参阅[删除MAX\_PATH限制][21]。
  * 可以添加`._pth`文件以强制隔离模式(isolated mode)并完全指定所有搜索路径，以避免注册表查找和环境查找。 有关详细信息，请参阅[文档][22]。
  * 一个python36.zip文件现可用作一个地标(landmark)以臆指[`PYTHONHOME`][23]。 有关详细信息，请参阅[文档][24]。

## 新特性

### PEP 498: 格式化字符串

[**PEP 498**][25]引入了一种新的字符串：\_f-strings\_, 或者[格式化字符串][26]。

格式化字符串带`'f'`前缀，类似于[`str.format()`][27]接受的格式字符串。它们包含了由花括号括起来的替换字段。替换字段是表达式，它们会在运行时计算，然后使用[`format()`][28]协议进行格式化：

```

    >>> name = "Fred"
    >>> f"He said his name is {name}."
    'He said his name is Fred.'
    >>> width = 10
    >>> precision = 4
    >>> value = decimal.Decimal("12.34567")
    >>> f"result: {value:{width}.{precision}}"  # nested fields
    'result:      12.35'

```

又见

[**PEP 498**][29] - 字符串插值。

	PEP由Eric V. Smith编写和实现。

[特性文档][30]。

### PEP 526: 变量注释语法

[**PEP 484**][31]引入了函数参数的类型注释的标准，又名类型提示。这个PEP添加了用来注释变量（包括类变量和实例变量）类型的语法：

```

    primes: List[int] = []

    captain: str  # Note: no initial value!

    class Starship:
        stats: Dict[str, int] = {}

```

正如函数注释，Python解释器不附加任何特殊意义到变量注释上，只是将它们存储在一个类或者模块的`__annotations__`属性中。

与静态类型语言中的变量声明相比，注释语法的目的在于提供一种简单的方式，通过抽象语法树和`__annotations__`属性，来为第三方工具和库指定结构化类型元数据。

又见

[**PEP 526**][32] - 变量注释语法。

	PEP由Ryan Gonzalez, Philip House, Ivan Levkivskyi, Lisa Roach, 和Guido van Rossum编写。由Ivan Levkivskyi实现。

使用或将要使用这个新语法的工具：[mypy][33], [pytype][34], PyCharm等等。

### PEP 515: 数值文字中的下划线

[**PEP 515**][35]添加了在数值文字中使用下划线的能力，以提高可读性。例如：

```

    >>> 1_000_000_000_000_000
    1000000000000000
    >>> 0x_FF_FF_FF_FF
    4294967295

```

数字之间和任何基本符号之后允许单个下划线。不允许前置、后置或者多个连续的下划线。

[字符串格式化][36]语言现在还支持`'_'`选项，该选项用来通知对浮点表示类型和整型表示类型`'d'`，会把下划线当成千位分隔符使用。对于整型表示类型`'b'`, `'o'`, `'x'`, 和`'X'`, 下划线将会被插入到每4个数字之间：

```

    >>> '{:_}'.format(1000000)
    '1_000_000'
    >>> '{:_x}'.format(0xFFFFFFFF)
    'ffff_ffff'

```

又见

[**PEP 515**][37] - 数值文字中的下划线

	PEP由Georg Brandl和Serhiy Storchaka编写。

### PEP 525: 异步生成器¶

[**PEP 492**][38] 引入支持原生协程和`async` /`await`的语法到Python 3.5。 在Python 3.5实现里的一个值得注意的
局限性就在于它不可能使用`await`和`yield'在同一个函数体中。 而在Python 3.6中，这个限制
已解除，这使得定义_异步生成器_成为可能：

```

    async def ticker(delay, to):
        """Yield numbers from 0 to *to* every *delay* seconds."""
        for i in range(to):
            yield i
            await asyncio.sleep(delay)

```
新的语法允许更快更简洁的代码。

参见
[**PEP 525**][39] - 异步生成器

	由Yury Selivanov撰写并实现的PEP。

### PEP 530: 异步解析式¶

[**PEP 530**][40] 添加了对`async for`在list、set、dict解析式以及generator表达式中的使用支持：

```

    result = [i async for i in aiter() if i % 2]
    
```

此外，所有解析式都支持“await”表达式：

```

    result = [await fun() for fun in funcs if await condition()]
    
```

参见
[**PEP 530**][41] - 异步解析式

	由Yury Selivanov撰写并实现的PEP。

### PEP 487: 用于建立类的更简单的自定义¶

现在可以在不使用元类的情况下自定义子类。每当创建一个新的子类时，新的`__init_subclass__`类方法将在基类上被调用，：

```

    class PluginBase:
        subclasses = []
    
        def __init_subclass__(cls, **kwargs):
            super().__init_subclass__(**kwargs)
            cls.subclasses.append(cls)
    
    class Plugin1(PluginBase):
        pass
    
    class Plugin2(PluginBase):
        pass
    
```
为了允许零参数
[`super（）`] [42]
从[`_init_subclass __（）`](https://docs.python.org/3
.6/reference/datamodel.html#object.\_\_init\_subclass\_\_
"object.\_\_init\_subclass\_\_" )实现中被正确的调用并工作，自定义元类必须确保
新的`__classcell__`命名空间输入传递到`type .__ new__`
（如[创建类
对象](https://docs.python.org/3.6/reference/datamodel.html#class-object-
creation))

参见

[**PEP 487**][43] - 用于建立类的更简单的自定义

	由Martin Teichmann撰写并实现的PEP。
	
[功能文档](https://docs.python.org/3.6/reference/datamodel.html#class-customization)

### PEP 487: 描述符协议增强¶

[**PEP 487**][44] 扩展描述符协议必须包括新的可选的[`__set_name __（）`](https://docs.python.org/3.6/reference/datamodel.html#object.\_\_set\_name\_\_ "object.\_\_set\_name\_\_" )方法。 每当定义一个新类时，新方法将会调用定义中所有的描述符，并给它们提供定义类的引用，以及类命名空间中给予描述符的名字。 换句话说，描述符的实例现在可以获知所有者类的属性名：
```

    class IntField:
        def __get__(self, instance, owner):
            return instance.__dict__[self.name]

        def __set__(self, instance, value):
            if not isinstance(value, int):
                raise ValueError(f'expecting integer in {self.name}')
            instance.__dict__[self.name] = value

        # this is the new initializer:
        def __set_name__(self, owner, name):
            self.name = name

    class Model:
        int_field = IntField()

```

参见

[**PEP 487**][45] - 用于建立类的更简单的自定义

	由Yury Selivanov撰写并实现的PEP。

[功能文档](https://docs.python.org/3.6/reference/datamodel.html#d
escriptors)

### PEP 519: 添加文件系统路径协议¶

文件系统路径过去被表示为[`str`][46]或[`bytes`][47]对象。这会导致那些编写操作文件系统路径代码的人，假定这些对象只能是这两种类型之一(一个代表着文件描述符的[`int`][48]对象将不被计入即它不是一个文件路径)。
Unfortunately that assumption prevents alternative object representations of
file system paths like
[`pathlib`](https://docs.python.org/3.6/library/pathlib.html#module-pathlib
"pathlib: Object-oriented filesystem paths" ) from working with pre-existing
code, including Python's standard library.

To fix this situation, a new interface represented by
[`os.PathLike`](https://docs.python.org/3.6/library/os.html#os.PathLike
"os.PathLike" ) has been defined. By implementing the [`__fspath__()`](https:/
/docs.python.org/3.6/library/os.html#os.PathLike.\_\_fspath\_\_
"os.PathLike.\_\_fspath\_\_" ) method, an object signals that it represents a
path. An object can then provide a low-level representation of a file system
path as a [`str`](https://docs.python.org/3.6/library/stdtypes.html#str "str"
) or [`bytes`](https://docs.python.org/3.6/library/functions.html#bytes
"bytes" ) object. This means an object is considered [path-
like](https://docs.python.org/3.6/glossary.html#term-path-like-object) if it
implements
[`os.PathLike`](https://docs.python.org/3.6/library/os.html#os.PathLike
"os.PathLike" ) or is a
[`str`][49] or
[`bytes`][50]
object which represents a file system path. Code can use
[`os.fspath()`](https://docs.python.org/3.6/library/os.html#os.fspath
"os.fspath" ),
[`os.fsdecode()`](https://docs.python.org/3.6/library/os.html#os.fsdecode
"os.fsdecode" ), or
[`os.fsencode()`](https://docs.python.org/3.6/library/os.html#os.fsencode
"os.fsencode" ) to explicitly get a
[`str`][51] and/or
[`bytes`][52]
representation of a path-like object.

The built-in
[`open()`][53]
function has been updated to accept
[`os.PathLike`](https://docs.python.org/3.6/library/os.html#os.PathLike
"os.PathLike" ) objects, as have all relevant functions in the
[`os`][54] and
[`os.path`](https://docs.python.org/3.6/library/os.path.html#module-os.path
"os.path: Operations on pathnames." ) modules, and most other functions and
classes in the standard library. The
[`os.DirEntry`](https://docs.python.org/3.6/library/os.html#os.DirEntry
"os.DirEntry" ) class and relevant classes in
[`pathlib`](https://docs.python.org/3.6/library/pathlib.html#module-pathlib
"pathlib: Object-oriented filesystem paths" ) have also been updated to
implement
[`os.PathLike`](https://docs.python.org/3.6/library/os.html#os.PathLike
"os.PathLike" ).

The hope is that updating the fundamental functions for operating on file
system paths will lead to third-party code to implicitly support all [path-
like objects](https://docs.python.org/3.6/glossary.html#term-path-like-object)
without any code changes, or at least very minimal ones (e.g. calling
[`os.fspath()`](https://docs.python.org/3.6/library/os.html#os.fspath
"os.fspath" ) at the beginning of code before operating on a path-like
object).

Here are some examples of how the new interface allows for
[`pathlib.Path`](https://docs.python.org/3.6/library/pathlib.html#pathlib.Path
"pathlib.Path" ) to be used more easily and transparently with pre-existing
code:

```

    >>> import pathlib
    >>> with open(pathlib.Path("README")) as f:
    ...     contents = f.read()
    ...
    >>> import os.path
    >>> os.path.splitext(pathlib.Path("some_file.txt"))
    ('some_file', '.txt')
    >>> os.path.join("/a/b", pathlib.Path("c"))
    '/a/b/c'
    >>> import os
    >>> os.fspath(pathlib.Path("some_file.txt"))
    'some_file.txt'

```

(Implemented by Brett Cannon, Ethan Furman, Dusty Phillips, and Jelle
Zijlstra.)

See also

[**PEP 519**][55] - Adding a file system
path protocol

	PEP written by Brett Cannon and Koos Zevenhoven.

### PEP 495: Local Time Disambiguation¶

In most world locations, there have been and will be times when local clocks
are moved back. In those times, intervals are introduced in which local clocks
show the same time twice in the same day. In these situations, the information
displayed on a local clock (or stored in a Python datetime instance) is
insufficient to identify a particular moment in time.

[**PEP 495**][56] adds the new _fold_
attribute to instances of [`datetime.datetime`](https://docs.python.org/3.6/li
brary/datetime.html#datetime.datetime "datetime.datetime" ) and [\`datetime.tim
e\`](https://docs.python.org/3.6/library/datetime.html#datetime.time
"datetime.time" ) classes to differentiate between two moments in time for
which local times are the same:

```

    >>> u0 = datetime(2016, 11, 6, 4, tzinfo=timezone.utc)
    >>> for i in range(4):
    ...     u = u0 + i*HOUR
    ...     t = u.astimezone(Eastern)
    ...     print(u.time(), 'UTC =', t.time(), t.tzname(), t.fold)
    ...
    04:00:00 UTC = 00:00:00 EDT 0
    05:00:00 UTC = 01:00:00 EDT 0
    06:00:00 UTC = 01:00:00 EST 1
    07:00:00 UTC = 02:00:00 EST 0

```

The values of the [`fold`](https://docs.python.org/3.6/library/datetime.html#d
atetime.datetime.fold "datetime.datetime.fold" ) attribute have the value `0`
for all instances except those that represent the second (chronologically)
moment in time in an ambiguous case.

See also

[**PEP 495**][57] - Local Time
Disambiguation

	PEP written by Alexander Belopolsky and Tim Peters, implementation by Alexander Belopolsky.

### PEP 529: 更改windows下文件系统编码格式为UTF-8¶

使用str (Unicode) 表示文件系统路径比bytes能获得更好的效果。尽管如此，在某些情况下bytes也足以胜任并且也是正确的。

在3.6之前,使用bytes路径可能导致数据丢失。改进后, windows下现在支持使用bytes表示路径,这些bytes将以[`sys.getfilesy
stemencoding()`](https://docs.python.org/3.6/library/sys.html#sys.getfilesystemencoding "sys.getfilesystemencoding" )的方式编码，默认编码格式为`'utf-8'`。

不使用str方式表示路径的应用程序应当使用[`os.fsencode()`](https://docs.python.org/3.6/library/os.html#os.fsencode "os.fsencode" )和[`os.fsdecode()`](https://docs.python.org/3.6/library/os.html#os.fsdecode "os.fsdecode" ) 以确保他们的bytes被正确编码。要回复到之前的状态, 设置 [`PYTHONLEGACYWINDOWSFSENCODING`](https://docs.python.org/3.6/using/cmdline.html#envvar-PYTHONLEGACYWINDOWSFSENCODING) 或者调用 [`sys._enablelegacywindowsfsencoding()`](https://docs.python.org/3.6/library/sys.html#sys._enablelegacywindowsfsencoding "sys._enablelegacywindowsfsencoding" )。

查看 [**PEP 529**](https://www.python.org/dev/peps/pep-0529)以获取更多信息并讨论可能需要变更的代码。 

### PEP 528: 更改windows控制台编码为UTF-8¶

windows下的默认控制台现在支持所有的Unicode字符并可以正确读取Python代码中的str对象。 `sys.stdin`, `sys.stdout`
以及 `sys.stderr` 现在的默认使用utf-8编码。

这一变化仅适用于使用交互控制台之时，而非重定向文件或者管道。如果要使用之前的交互控制台, 需设置 [`PYTHONLEGACYWINDOWSIOENCODING`](https://docs.python.org/3.6/using/cmdline.html#envvar-PYTHONLEGACYWINDOWSIOENCODING)。

另请参阅

[**PEP 528**](https://www.python.org/dev/peps/pep-0528) - 修改windows控制台编码为UTF-8

    PEP 由Steve Dower编写和实现。

### PEP 520: 保存类属性定义顺序¶

类的定义体中的属性有一个自然顺序：即源码中属性名出现的顺序。 这个顺序现在保存在新的类[`__dict__`](https://docs.python.org/3.6/library/stdtypes.html#object.__dict__ "object.__dict__" ) 的属性中.

同样， 有效的缺省类和执行空间 (从[type.__prepare__()](https://docs.python.org/3.6/reference/datamodel.html#prepare)返回)是一个保存插入顺序的映射。

另请参阅

[**PEP 520**](https://www.python.org/dev/peps/pep-0520) - 保存类属性定义顺序

    该PEP由Eric Snow编写和实现。

### PEP 468: 保存关键字参数顺序¶

函数声明中的`**kwargs` 的顺序现在被保证是插入顺序的映射。

另请参阅

[**PEP 468**](https://www.python.org/dev/peps/pep-0468) - 保存关键字参数顺序

    该PEP由Eric Snow编写和实现。

### 新的 [字典dict](https://docs.python.org/3.6/library/stdtypes.html#typesmapping)类型的实现¶

[字典dict](https://docs.python.org/3.6/library/stdtypes.html#typesmapping)类型现在使用 [PyPy首创](https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-more.html)的 "紧凑" 表达方式。 新[`字典dict()`](https://docs.python.org/3.6/library/stdtypes.html#dict "dict" ) 的内存占用比Python3.5中减少20%到25%。

新的实现中保存顺序的功能被认为是不可过于依赖的(未来也许会改变，不过在将所有当前和未来的Python实现的语言规范转换为保证顺序的语法之前的几个版本中，新的dict有望被实现的; 这也能帮助保证对那些仍旧是随机迭代顺序的旧版本的向后兼容，比如Python 3.5)。

(由INADA Naoki在[issue 27350](https://bugs.python.org/issue27350)提供。 想法 [最初由Raymond Hettinger提出](https://mail.python.org/pipermail/python-dev/2012-December/123028.html).)

### PEP 523: Adding a frame evaluation API to CPython¶

While Python provides extensive support to customize how code executes, one
place it has not done so is in the evaluation of frame objects. If you wanted
some way to intercept frame evaluation in Python there really wasn't any way
without directly manipulating function pointers for defined functions.

[**PEP 523**][65] changes this by
providing an API to make frame evaluation pluggable at the C level. This will
allow for tools such as debuggers and JITs to intercept frame evaluation
before the execution of Python code begins. This enables the use of
alternative evaluation implementations for Python code, tracking frame
evaluation, etc.

This API is not part of the limited C API and is marked as private to signal
that usage of this API is expected to be limited and only applicable to very
select, low-level use-cases. Semantics of the API will change with Python as
necessary.

See also

[**PEP 523**][66] - Adding a frame
evaluation API to CPython

	PEP written by Brett Cannon and Dino Viehland.

### PYTHONMALLOC environment variable¶

The new [`PYTHONMALLOC`](https://docs.python.org/3.6/using/cmdline.html
# envvar-PYTHONMALLOC) environment variable allows setting the Python memory
allocators and installing debug hooks.

It is now possible to install debug hooks on Python memory allocators on
Python compiled in release mode using `PYTHONMALLOC=debug`. Effects of debug
hooks:

  * Newly allocated memory is filled with the byte `0xCB`
  * Freed memory is filled with the byte `0xDB`
  * Detect violations of the Python memory allocator API. For example, `PyObject_Free()` called on a memory block allocated by [`PyMem_Malloc()`][67].
  * Detect writes before the start of a buffer (buffer underflows)
  * Detect writes after the end of a buffer (buffer overflows)
  * Check that the [GIL][68] is held when allocator functions of [`PYMEM_DOMAIN_OBJ`][69] (ex: `PyObject_Malloc()`) and [`PYMEM_DOMAIN_MEM`][70] (ex: [`PyMem_Malloc()`][71]) domains are called.

Checking if the GIL is held is also a new feature of Python 3.6.

See the [`PyMem_SetupDebugHooks()`](https://docs.python.org/3.6/c-api/memory.h
tml#c.PyMem\_SetupDebugHooks "PyMem\_SetupDebugHooks" ) function for debug hooks
on Python memory allocators.

It is now also possible to force the usage of the `malloc()` allocator of the
C library for all Python memory allocations using `PYTHONMALLOC=malloc`. This
is helpful when using external memory debuggers like Valgrind on a Python
compiled in release mode.

On error, the debug hooks on Python memory allocators now use the
[`tracemalloc`](https://docs.python.org/3.6/library/tracemalloc.html#module-
tracemalloc "tracemalloc: Trace memory allocations." ) module to get the
traceback where a memory block was allocated.

Example of fatal error on buffer overflow using `python3.6 -X tracemalloc=5`
(store 5 frames in traces):

```

    Debug memory block at address p=0x7fbcd41666f8: API 'o'
        4 bytes originally requested
        The 7 pad bytes at p-7 are FORBIDDENBYTE, as expected.
        The 8 pad bytes at tail=0x7fbcd41666fc are not all FORBIDDENBYTE (0xfb):
            at tail+0: 0x02 *** OUCH
            at tail+1: 0xfb
            at tail+2: 0xfb
            at tail+3: 0xfb
            at tail+4: 0xfb
            at tail+5: 0xfb
            at tail+6: 0xfb
            at tail+7: 0xfb
        The block was made by call #1233329 to debug malloc/realloc.
        Data at p: 1a 2b 30 00

    Memory block allocated at (most recent call first):
      File "test/test_bytes.py", line 323
      File "unittest/case.py", line 600
      File "unittest/case.py", line 648
      File "unittest/suite.py", line 122
      File "unittest/suite.py", line 84

    Fatal Python error: bad trailing pad byte

    Current thread 0x00007fbcdbd32700 (most recent call first):
      File "test/test_bytes.py", line 323 in test_hex
      File "unittest/case.py", line 600 in run
      File "unittest/case.py", line 648 in __call__
      File "unittest/suite.py", line 122 in run
      File "unittest/suite.py", line 84 in __call__
      File "unittest/suite.py", line 122 in run
      File "unittest/suite.py", line 84 in __call__
      ...

```

(Contributed by Victor Stinner in [issue
26516](https://bugs.python.org/issue26516) and [issue
26564](https://bugs.python.org/issue26564).)

### DTrace and SystemTap probing support¶

Python can now be built `--with-dtrace` which enables static markers for the
following events in the interpreter:

  * function call/return
  * garbage collection started/finished
  * line of code executed.

This can be used to instrument running interpreters in production, without the
need to recompile specific debug builds or providing application-specific
profiling/debugging code.

More details in [Instrumenting CPython with DTrace and SystemTap](https://docs
.python.org/3.6/howto/instrumentation.html#instrumentation).

The current implementation is tested on Linux and macOS. Additional markers
may be added in the future.

(Contributed by Łukasz Langa in [issue
21590](https://bugs.python.org/issue21590), based on patches by Jesús Cea
Avión, David Malcolm, and Nikhil Benesch.)

## Other Language Changes¶

Some smaller changes made to the core Python language are:

  * A `global` or `nonlocal` statement must now textually appear before the first use of the affected name in the same scope. Previously this was a `SyntaxWarning`.
  * It is now possible to set a [special method][72] to `None` to indicate that the corresponding operation is not available. For example, if a class sets [`__iter__()`][73] to `None`, the class is not iterable. (Contributed by Andrew Barnert and Ivan Levkivskyi in [issue 25958][74].)
  * Long sequences of repeated traceback lines are now abbreviated as `"[Previous line repeated {count} more times]"` (see traceback for an example). (Contributed by Emanuel Barry in [issue 26823][75].)
  * Import now raises the new exception [`ModuleNotFoundError`][76] (subclass of [`ImportError`][77]) when it cannot find a module. Code that currently checks for ImportError (in try-except) will still work. (Contributed by Eric Snow in [issue 15767][78].)
  * Class methods relying on zero-argument `super()` will now work correctly when called from metaclass methods during class creation. (Contributed by Martin Teichmann in [issue 23722][79].)

## New Modules¶

### secrets¶

The main purpose of the new
[`secrets`](https://docs.python.org/3.6/library/secrets.html#module-secrets
"secrets: Generate secure random numbers for managing secrets." ) module is to
provide an obvious way to reliably generate cryptographically strong pseudo-
random values suitable for managing secrets, such as account authentication,
tokens, and similar.

Warning

Note that the pseudo-random generators in the
[`random`](https://docs.python.org/3.6/library/random.html#module-random
"random: Generate pseudo-random numbers with various common distributions." )
module should _NOT_ be used for security purposes. Use
[`secrets`](https://docs.python.org/3.6/library/secrets.html#module-secrets
"secrets: Generate secure random numbers for managing secrets." ) on Python
3.6+ and
[`os.urandom()`](https://docs.python.org/3.6/library/os.html#os.urandom
"os.urandom" ) on Python 3.5 and earlier.

See also

[**PEP 506**][80] - Adding A Secrets
Module To The Standard Library

	PEP written and implemented by Steven D'Aprano.

## Improved Modules¶

### array¶

Exhausted iterators of
[`array.array`](https://docs.python.org/3.6/library/array.html#array.array
"array.array" ) will now stay exhausted even if the iterated array is
extended. This is consistent with the behavior of other mutable sequences.

Contributed by Serhiy Storchaka in [issue
26492](https://bugs.python.org/issue26492).

### ast¶

The new `ast.Constant` AST node has been added. It can be used by external AST
optimizers for the purposes of constant folding.

Contributed by Victor Stinner in [issue
26146](https://bugs.python.org/issue26146).

### asyncio¶

Starting with Python 3.6 the `asyncio` module is no longer provisional and its
API is considered stable.

Notable changes in the
[`asyncio`](https://docs.python.org/3.6/library/asyncio.html#module-asyncio
"asyncio: Asynchronous I/O, event loop, coroutines and tasks." ) module since
Python 3.5.0 (all backported to 3.5.x due to the provisional status):

  * The [`get_event_loop()`][81] function has been changed to always return the currently running loop when called from couroutines and callbacks. (Contributed by Yury Selivanov in [issue 28613][82].)
  * The [`ensure_future()`][83] function and all functions that use it, such as `loop.run_until_complete()`, now accept all kinds of [awaitable objects][84]. (Contributed by Yury Selivanov.)
  * New [`run_coroutine_threadsafe()`][85] function to submit coroutines to event loops from other threads. (Contributed by Vincent Michel.)
  * New [`Transport.is_closing()`][86] method to check if the transport is closing or closed. (Contributed by Yury Selivanov.)
  * The `loop.create_server()` method can now accept a list of hosts. (Contributed by Yann Sionneau.)
  * New `loop.create_future()` method to create Future objects. This allows alternative event loop implementations, such as [uvloop][87], to provide a faster [`asyncio.Future`][88] implementation. (Contributed by Yury Selivanov in [issue 27041][89].)
  * New `loop.get_exception_handler()` method to get the current exception handler. (Contributed by Yury Selivanov in [issue 27040][90].)
  * New [`StreamReader.readuntil()`][91] method to read data from the stream until a separator bytes sequence appears. (Contributed by Mark Korenberg.)
  * The performance of [`StreamReader.readexactly()`][92] has been improved. (Contributed by Mark Korenberg in [issue 28370][93].)
  * The `loop.getaddrinfo()` method is optimized to avoid calling the system `getaddrinfo` function if the address is already resolved. (Contributed by A. Jesse Jiryu Davis.)
  * The `loop.stop()` method has been changed to stop the loop immediately after the current iteration. Any new callbacks scheduled as a result of the last iteration will be discarded. (Contributed by Guido van Rossum in [issue 25593][94].)
  * `Future.set_exception` will now raise [`TypeError`][95] when passed an instance of the [`StopIteration`][96] exception. (Contributed by Chris Angelico in [issue 26221][97].)
  * New [`loop.connect_accepted_socket()`][98] method to be used by servers that accept connections outside of asyncio, but that use asyncio to handle them. (Contributed by Jim Fulton in [issue 27392][99].)
  * `TCP_NODELAY` flag is now set for all TCP transports by default. (Contributed by Yury Selivanov in [issue 27456][100].)
  * New [`loop.shutdown_asyncgens()`][101] to properly close pending asynchronous generators before closing the loop. (Contributed by Yury Selivanov in [issue 28003][102].)
  * [`Future`][103] and [`Task`][104] classes now have an optimized C implementation which makes asyncio code up to 30% faster. (Contributed by Yury Selivanov and INADA Naoki in [issue 26081][105] and [issue 28544][106].)

### binascii¶

The [`b2a_base64()`](https://docs.python.org/3.6/library/binascii.html#binasci
i.b2a\_base64 "binascii.b2a\_base64" ) function now accepts an optional
_newline_ keyword argument to control whether the newline character is
appended to the return value. (Contributed by Victor Stinner in [issue
25357](https://bugs.python.org/issue25357).)

### cmath¶

The new [`cmath.tau`](https://docs.python.org/3.6/library/cmath.html#cmath.tau
"cmath.tau" ) (τ) constant has been added. (Contributed by Lisa Roach in
[issue 12345][107], see [\*\*PEP
628\*\*](https://www.python.org/dev/peps/pep-0628) for details.)

New constants:
[`cmath.inf`](https://docs.python.org/3.6/library/cmath.html#cmath.inf
"cmath.inf" ) and
[`cmath.nan`](https://docs.python.org/3.6/library/cmath.html#cmath.nan
"cmath.nan" ) to match
[`math.inf`](https://docs.python.org/3.6/library/math.html#math.inf "math.inf"
) and [`math.nan`](https://docs.python.org/3.6/library/math.html#math.nan
"math.nan" ), and also
[`cmath.infj`](https://docs.python.org/3.6/library/cmath.html#cmath.infj
"cmath.infj" ) and
[`cmath.nanj`](https://docs.python.org/3.6/library/cmath.html#cmath.nanj
"cmath.nanj" ) to match the format used by complex repr. (Contributed by Mark
Dickinson in [issue 23229][108].)

### collections¶

The new [`Collection`](https://docs.python.org/3.6/library/collections.abc.htm
l#collections.abc.Collection "collections.abc.Collection" ) abstract base
class has been added to represent sized iterable container classes.
(Contributed by Ivan Levkivskyi, docs by Neil Girdhar in [issue
27598](https://bugs.python.org/issue27598).)

The new [`Reversible`](https://docs.python.org/3.6/library/collections.abc.htm
l#collections.abc.Reversible "collections.abc.Reversible" ) abstract base
class represents iterable classes that also provide the [`__reversed__()`](htt
ps://docs.python.org/3.6/reference/datamodel.html#object.\_\_reversed\_\_
"object.\_\_reversed\_\_" ) method. (Contributed by Ivan Levkivskyi in [issue
25987](https://bugs.python.org/issue25987).)

The new [`AsyncGenerator`](https://docs.python.org/3.6/library/collections.abc
.html#collections.abc.AsyncGenerator "collections.abc.AsyncGenerator" )
abstract base class represents asynchronous generators. (Contributed by Yury
Selivanov in [issue 28720][109].)

The [`namedtuple()`](https://docs.python.org/3.6/library/collections.html#coll
ections.namedtuple "collections.namedtuple" ) function now accepts an optional
keyword argument _module_, which, when specified, is used for the `__module__`
attribute of the returned named tuple class. (Contributed by Raymond Hettinger
in [issue 17941][110].)

The _verbose_ and _rename_ arguments for [`namedtuple()`](https://docs.python.
org/3.6/library/collections.html#collections.namedtuple
"collections.namedtuple" ) are now keyword-only. (Contributed by Raymond
Hettinger in [issue 25628][111].)

Recursive [`collections.deque`](https://docs.python.org/3.6/library/collection
s.html#collections.deque "collections.deque" ) instances can now be pickled.
(Contributed by Serhiy Storchaka in [issue
26482](https://bugs.python.org/issue26482).)

### concurrent.futures¶

The [`ThreadPoolExecutor`](https://docs.python.org/3.6/library/concurrent.futu
res.html#concurrent.futures.ThreadPoolExecutor
"concurrent.futures.ThreadPoolExecutor" ) class constructor now accepts an
optional _thread\_name\_prefix_ argument to make it possible to customize the
names of the threads created by the pool. (Contributed by Gregory P. Smith in
[issue 27664][112].)

### contextlib¶

The [`contextlib.AbstractContextManager`](https://docs.python.org/3.6/library/
contextlib.html#contextlib.AbstractContextManager
"contextlib.AbstractContextManager" ) class has been added to provide an
abstract base class for context managers. It provides a sensible default
implementation for __enter__() which returns `self` and leaves __exit__() an
abstract method. A matching class has been added to the
[`typing`](https://docs.python.org/3.6/library/typing.html#module-typing
"typing: Support for type hints \(see PEP 484\)." ) module as [\`typing.Context
Manager\`](https://docs.python.org/3.6/library/typing.html#typing.ContextManage
r "typing.ContextManager" ). (Contributed by Brett Cannon in [issue
25609](https://bugs.python.org/issue25609).)

### datetime¶

The [`datetime`](https://docs.python.org/3.6/library/datetime.html#datetime.da
tetime "datetime.datetime" ) and
[`time`](https://docs.python.org/3.6/library/datetime.html#datetime.time
"datetime.time" ) classes have the new `fold` attribute used to disambiguate
local time when necessary. Many functions in the
[`datetime`](https://docs.python.org/3.6/library/datetime.html#module-datetime
"datetime: Basic date and time types." ) have been updated to support local
time disambiguation. See Local Time Disambiguation section for more
information. (Contributed by Alexander Belopolsky in [issue
24773](https://bugs.python.org/issue24773).)

The [`datetime.strftime()`](https://docs.python.org/3.6/library/datetime.html\#
datetime.datetime.strftime "datetime.datetime.strftime" ) and [\`date.strftime(
)\`](https://docs.python.org/3.6/library/datetime.html#datetime.date.strftime
"datetime.date.strftime" ) methods now support ISO 8601 date directives `%G`,
`%u` and `%V`. (Contributed by Ashley Anderson in [issue
12006](https://bugs.python.org/issue12006).)

The [`datetime.isoformat()`](https://docs.python.org/3.6/library/datetime.html
# datetime.datetime.isoformat "datetime.datetime.isoformat" ) function now
accepts an optional _timespec_ argument that specifies the number of
additional components of the time value to include. (Contributed by Alessandro
Cucci and Alexander Belopolsky in [issue
19475](https://bugs.python.org/issue19475).)

The [`datetime.combine()`](https://docs.python.org/3.6/library/datetime.html#d
atetime.datetime.combine "datetime.datetime.combine" ) now accepts an optional
_tzinfo_ argument. (Contributed by Alexander Belopolsky in [issue
27661](https://bugs.python.org/issue27661).)

### decimal¶

New [`Decimal.as_integer_ratio()`](https://docs.python.org/3.6/library/decimal
.html#decimal.Decimal.as\_integer\_ratio "decimal.Decimal.as\_integer\_ratio" )
method that returns a pair `(n, d)` of integers that represent the given
[`Decimal`](https://docs.python.org/3.6/library/decimal.html#decimal.Decimal
"decimal.Decimal" ) instance as a fraction, in lowest terms and with a
positive denominator:

```

    >>> Decimal('-3.14').as_integer_ratio()
    (-157, 50)

```

(Contributed by Stefan Krah amd Mark Dickinson in [issue
25928](https://bugs.python.org/issue25928).)

### distutils¶

The `default_format` attribute has been removed from
`distutils.command.sdist.sdist` and the `formats` attribute defaults to
`['gztar']`. Although not anticipated, any code relying on the presence of
`default_format` may need to be adapted. See [issue
27819](https://bugs.python.org/issue27819) for more details.

### email¶

The new email API, enabled via the _policy_ keyword to various constructors,
is no longer provisional. The
[`email`](https://docs.python.org/3.6/library/email.html#module-email "email:
Package supporting the parsing, manipulating, and generating email messages."
) documentation has been reorganized and rewritten to focus on the new API,
while retaining the old documentation for the legacy API. (Contributed by R.
David Murray in [issue 24277][113].)

The [`email.mime`](https://docs.python.org/3.6/library/email.mime.html#module-
email.mime "email.mime: Build MIME messages." ) classes now all accept an
optional _policy_ keyword. (Contributed by Berker Peksag in [issue
27331](https://bugs.python.org/issue27331).)

The [`DecodedGenerator`](https://docs.python.org/3.6/library/email.generator.h
tml#email.generator.DecodedGenerator "email.generator.DecodedGenerator" ) now
supports the _policy_ keyword.

There is a new
[`policy`](https://docs.python.org/3.6/library/email.policy.html#module-
email.policy "email.policy: Controlling the parsing and generating of
messages" ) attribute, [`message_factory`](https://docs.python.org/3.6/library
/email.policy.html#email.policy.Policy.message\_factory
"email.policy.Policy.message\_factory" ), that controls what class is used by
default when the parser creates new message objects. For the [\`email.policy.co
mpat32\`](https://docs.python.org/3.6/library/email.policy.html#email.policy.co
mpat32 "email.policy.compat32" ) policy this is [`Message`](https://docs.pytho
n.org/3.6/library/email.compat32-message.html#email.message.Message
"email.message.Message" ), for the new policies it is [`EmailMessage`](https:/
/docs.python.org/3.6/library/email.message.html#email.message.EmailMessage
"email.message.EmailMessage" ). (Contributed by R. David Murray in [issue
20476](https://bugs.python.org/issue20476).)

### encodings¶

On Windows, added the `'oem'` encoding to use `CP_OEMCP`, and the `'ansi'`
alias for the existing `'mbcs'` encoding, which uses the `CP_ACP` code page.
(Contributed by Steve Dower in [issue
27959](https://bugs.python.org/issue27959).)

### enum¶

Two new enumeration base classes have been added to the
[`enum`][114] module:
[`Flag`][115]
and `IntFlags`. Both are used to define constants that can be combined using
the bitwise operators. (Contributed by Ethan Furman in [issue
23591](https://bugs.python.org/issue23591).)

Many standard library modules have been updated to use the `IntFlags` class
for their constants.

The new [`enum.auto`](https://docs.python.org/3.6/library/enum.html#enum.auto
"enum.auto" ) value can be used to assign values to enum members
automatically:

```

    >>> from enum import Enum, auto
    >>> class Color(Enum):
    ...     red = auto()
    ...     blue = auto()
    ...     green = auto()
    ...
    >>> list(Color)
    [<Color.red: 1>, <Color.blue: 2>, <Color.green: 3>]

```

### faulthandler¶

On Windows, the
[`faulthandler`](https://docs.python.org/3.6/library/faulthandler.html#module-
faulthandler "faulthandler: Dump the Python traceback." ) module now installs
a handler for Windows exceptions: see [`faulthandler.enable()`](https://docs.p
ython.org/3.6/library/faulthandler.html#faulthandler.enable
"faulthandler.enable" ). (Contributed by Victor Stinner in [issue
23848](https://bugs.python.org/issue23848).)

### fileinput¶

[`hook_encoded()`](https://docs.python.org/3.6/library/fileinput.html#fileinpu
t.hook\_encoded "fileinput.hook\_encoded" ) now supports the _errors_ argument.
(Contributed by Joseph Hackman in [issue
25788](https://bugs.python.org/issue25788).)

### hashlib¶

[`hashlib`](https://docs.python.org/3.6/library/hashlib-blake2.html#module-
hashlib "hashlib: BLAKE2 hash function for Python" ) supports OpenSSL 1.1.0.
The minimum recommend version is 1.0.2. (Contributed by Christian Heimes in
[issue 26470][116].)

BLAKE2 hash functions were added to the module.
[`blake2b()`](https://docs.python.org/3.6/library/hashlib-
blake2.html#hashlib.blake2b "hashlib.blake2b" ) and
[`blake2s()`](https://docs.python.org/3.6/library/hashlib-
blake2.html#hashlib.blake2s "hashlib.blake2s" ) are always available and
support the full feature set of BLAKE2. (Contributed by Christian Heimes in
[issue 26798][117] based on code by Dmitry
Chestnykh and Samuel Neves. Documentation written by Dmitry Chestnykh.)

The SHA-3 hash functions `sha3_224()`, `sha3_256()`, `sha3_384()`,
`sha3_512()`, and SHAKE hash functions `shake_128()` and `shake_256()` were
added. (Contributed by Christian Heimes in [issue
16113](https://bugs.python.org/issue16113). Keccak Code Package by Guido
Bertoni, Joan Daemen, Michaël Peeters, Gilles Van Assche, and Ronny Van Keer.)

The password-based key derivation function
[`scrypt()`](https://docs.python.org/3.6/library/hashlib.html#hashlib.scrypt
"hashlib.scrypt" ) is now available with OpenSSL 1.1.0 and newer. (Contributed
by Christian Heimes in [issue 27928][118].)

### http.client¶

[`HTTPConnection.request()`](https://docs.python.org/3.6/library/http.client.h
tml#http.client.HTTPConnection.request "http.client.HTTPConnection.request" )
and [`endheaders()`](https://docs.python.org/3.6/library/http.client.html#http
.client.HTTPConnection.endheaders "http.client.HTTPConnection.endheaders" )
both now support chunked encoding request bodies. (Contributed by Demian
Brecht and Rolf Krahl in [issue 12319][119].)

### idlelib and IDLE¶

The idlelib package is being modernized and refactored to make IDLE look and
work better and to make the code easier to understand, test, and improve. Part
of making IDLE look better, especially on Linux and Mac, is using ttk widgets,
mostly in the dialogs. As a result, IDLE no longer runs with tcl/tk 8.4. It
now requires tcl/tk 8.5 or 8.6. We recommend running the latest release of
either.

'Modernizing' includes renaming and consolidation of idlelib modules. The
renaming of files with partial uppercase names is similar to the renaming of,
for instance, Tkinter and TkFont to tkinter and tkinter.font in 3.0. As a
result, imports of idlelib files that worked in 3.5 will usually not work in
3.6. At least a module name change will be needed (see idlelib/README.txt),
sometimes more. (Name changes contributed by Al Swiegart and Terry Reedy in
[issue 24225][120]. Most idlelib patches since
have been and will be part of the process.)

In compensation, the eventual result with be that some idlelib classes will be
easier to use, with better APIs and docstrings explaining them. Additional
useful information will be added to idlelib when available.

### importlib¶

Import now raises the new exception [`ModuleNotFoundError`](https://docs.pytho
n.org/3.6/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )
(subclass of [`ImportError`](https://docs.python.org/3.6/library/exceptions.ht
ml#ImportError "ImportError" )) when it cannot find a module. Code that
current checks for `ImportError` (in try-except) will still work. (Contributed
by Eric Snow in [issue 15767][121].)

[`importlib.util.LazyLoader`](https://docs.python.org/3.6/library/importlib.ht
ml#importlib.util.LazyLoader "importlib.util.LazyLoader" ) now calls [\`create\_
module()\`](https://docs.python.org/3.6/library/importlib.html#importlib.abc.Lo
ader.create\_module "importlib.abc.Loader.create\_module" ) on the wrapped
loader, removing the restriction that [`importlib.machinery.BuiltinImporter`](
https://docs.python.org/3.6/library/importlib.html#importlib.machinery.Builtin
Importer "importlib.machinery.BuiltinImporter" ) and [\`importlib.machinery.Ext
ensionFileLoader\`](https://docs.python.org/3.6/library/importlib.html#importli
b.machinery.ExtensionFileLoader "importlib.machinery.ExtensionFileLoader" )
couldn't be used with [`importlib.util.LazyLoader`](https://docs.python.org/3.
6/library/importlib.html#importlib.util.LazyLoader "importlib.util.LazyLoader"
).

[`importlib.util.cache_from_source()`](https://docs.python.org/3.6/library/imp
ortlib.html#importlib.util.cache\_from\_source
"importlib.util.cache\_from\_source" ), [`importlib.util.source_from_cache()`](h
ttps://docs.python.org/3.6/library/importlib.html#importlib.util.source\_from\_c
ache "importlib.util.source\_from\_cache" ), and [\`importlib.util.spec\_from\_file
\_location()\`](https://docs.python.org/3.6/library/importlib.html#importlib.uti
l.spec\_from\_file\_location "importlib.util.spec\_from\_file\_location" ) now
accept a [path-like object](https://docs.python.org/3.6/glossary.html#term-
path-like-object).

### inspect¶

The [`inspect.signature()`](https://docs.python.org/3.6/library/inspect.html#i
nspect.signature "inspect.signature" ) function now reports the implicit `.0`
parameters generated by the compiler for comprehension and generator
expression scopes as if they were positional-only parameters called
`implicit0`. (Contributed by Jelle Zijlstra in [issue
19611](https://bugs.python.org/issue19611).)

To reduce code churn when upgrading from Python 2.7 and the legacy [\`inspect.g
etargspec()\`](https://docs.python.org/3.6/library/inspect.html#inspect.getargs
pec "inspect.getargspec" ) API, the previously documented deprecation of [\`ins
pect.getfullargspec()\`](https://docs.python.org/3.6/library/inspect.html#inspe
ct.getfullargspec "inspect.getfullargspec" ) has been reversed. While this
function is convenient for single/source Python 2/3 code bases, the richer [\`i
nspect.signature()\`](https://docs.python.org/3.6/library/inspect.html#inspect.
signature "inspect.signature" ) interface remains the recommended approach for
new code. (Contributed by Nick Coghlan in [issue
27172](https://bugs.python.org/issue27172))

### json¶

[`json.load()`](https://docs.python.org/3.6/library/json.html#json.load
"json.load" ) and
[`json.loads()`](https://docs.python.org/3.6/library/json.html#json.loads
"json.loads" ) now support binary input. Encoded JSON should be represented
using either UTF-8, UTF-16, or UTF-32. (Contributed by Serhiy Storchaka in
[issue 17909][122].)

### logging¶

The new [`WatchedFileHandler.reopenIfNeeded()`](https://docs.python.org/3.6/li
brary/logging.handlers.html#logging.handlers.WatchedFileHandler.reopenIfNeeded
"logging.handlers.WatchedFileHandler.reopenIfNeeded" ) method has been added
to add the ability to check if the log file needs to be reopened. (Contributed
by Marian Horban in [issue 24884][123].)

### math¶

The tau (τ) constant has been added to the
[`math`][124] and
[`cmath`][125] modules. (Contributed by Lisa
Roach in [issue 12345][126], see [\*\*PEP
628\*\*](https://www.python.org/dev/peps/pep-0628) for details.)

### multiprocessing¶

[Proxy Objects](https://docs.python.org/3.6/library/multiprocessing.html
# multiprocessing-proxy-objects) returned by `multiprocessing.Manager()` can
now be nested. (Contributed by Davin Potts in [issue
6766](https://bugs.python.org/issue6766).)

### os¶

See the summary of PEP 519 for details on how the
[`os`][127] and
[`os.path`](https://docs.python.org/3.6/library/os.path.html#module-os.path
"os.path: Operations on pathnames." ) modules now support [path-like
objects](https://docs.python.org/3.6/glossary.html#term-path-like-object).

[`scandir()`](https://docs.python.org/3.6/library/os.html#os.scandir
"os.scandir" ) now supports
[`bytes`][128]
paths on Windows.

A new [`close()`](https://docs.python.org/3.6/library/os.html#os.scandir.close
"os.scandir.close" ) method allows explicitly closing a
[`scandir()`](https://docs.python.org/3.6/library/os.html#os.scandir
"os.scandir" ) iterator. The
[`scandir()`](https://docs.python.org/3.6/library/os.html#os.scandir
"os.scandir" ) iterator now supports the [context
manager](https://docs.python.org/3.6/glossary.html#term-context-manager)
protocol. If a `scandir()` iterator is neither exhausted nor explicitly closed
a [`ResourceWarning`](https://docs.python.org/3.6/library/exceptions.html#Reso
urceWarning "ResourceWarning" ) will be emitted in its destructor.
(Contributed by Serhiy Storchaka in [issue
25994](https://bugs.python.org/issue25994).)

On Linux,
[`os.urandom()`](https://docs.python.org/3.6/library/os.html#os.urandom
"os.urandom" ) now blocks until the system urandom entropy pool is initialized
to increase the security. See the [\*\*PEP
524\*\*](https://www.python.org/dev/peps/pep-0524) for the rationale.

The Linux `getrandom()` syscall (get random bytes) is now exposed as the new
[`os.getrandom()`](https://docs.python.org/3.6/library/os.html#os.getrandom
"os.getrandom" ) function. (Contributed by Victor Stinner, part of the [\*\*PEP
524\*\*](https://www.python.org/dev/peps/pep-0524))

### pathlib¶

[`pathlib`](https://docs.python.org/3.6/library/pathlib.html#module-pathlib
"pathlib: Object-oriented filesystem paths" ) now supports [path-like
objects](https://docs.python.org/3.6/glossary.html#term-path-like-object).
(Contributed by Brett Cannon in [issue
27186](https://bugs.python.org/issue27186).)

See the summary of PEP 519 for details.

### pdb¶

The [`Pdb`][129]
class constructor has a new optional _readrc_ argument to control whether
`.pdbrc` files should be read.

### pickle¶

Objects that need `__new__` called with keyword arguments can now be pickled
using [pickle protocols](https://docs.python.org/3.6/library/pickle.html
# pickle-protocols) older than protocol version 4. Protocol version 4 already
supports this case. (Contributed by Serhiy Storchaka in [issue
24164](https://bugs.python.org/issue24164).)

### pickletools¶

[`pickletools.dis()`](https://docs.python.org/3.6/library/pickletools.html#pic
kletools.dis "pickletools.dis" ) now outputs the implicit memo index for the
`MEMOIZE` opcode. (Contributed by Serhiy Storchaka in [issue
25382](https://bugs.python.org/issue25382).)

### pydoc¶

The [`pydoc`](https://docs.python.org/3.6/library/pydoc.html#module-pydoc
"pydoc: Documentation generator and online help system." ) module has learned
to respect the `MANPAGER` environment variable. (Contributed by Matthias Klose
in [issue 8637][130].)

[`help()`][131]
and [`pydoc`](https://docs.python.org/3.6/library/pydoc.html#module-pydoc
"pydoc: Documentation generator and online help system." ) can now list named
tuple fields in the order they were defined rather than alphabetically.
(Contributed by Raymond Hettinger in [issue
24879](https://bugs.python.org/issue24879).)

### random¶

The new
[`choices()`](https://docs.python.org/3.6/library/random.html#random.choices
"random.choices" ) function returns a list of elements of specified size from
the given population with optional weights. (Contributed by Raymond Hettinger
in [issue 18844][132].)

### re¶

Added support of modifier spans in regular expressions. Examples:
`'(?i:p)ython'` matches `'python'` and `'Python'`, but not `'PYTHON'`;
`'(?i)g(?-i:v)r'` matches `'GvR'` and `'gvr'`, but not `'GVR'`. (Contributed
by Serhiy Storchaka in [issue 433028][133].)

Match object groups can be accessed by `__getitem__`, which is equivalent to
`group()`. So `mo['name']` is now equivalent to `mo.group('name')`.
(Contributed by Eric Smith in [issue
24454](https://bugs.python.org/issue24454).)

`Match` objects now support [`index-like objects`](https://docs.python.org/3.6
/reference/datamodel.html#object.\_\_index\_\_ "object.\_\_index\_\_" ) as group
indices. (Contributed by Jeroen Demeyer and Xiang Zhang in [issue
27177](https://bugs.python.org/issue27177).)

### readline¶

Added [`set_auto_history()`](https://docs.python.org/3.6/library/readline.html
# readline.set\_auto\_history "readline.set\_auto\_history" ) to enable or disable
automatic addition of input to the history list. (Contributed by Tyler
Crompton in [issue 26870][134].)

### rlcompleter¶

Private and special attribute names now are omitted unless the prefix starts
with underscores. A space or a colon is added after some completed keywords.
(Contributed by Serhiy Storchaka in [issue
25011](https://bugs.python.org/issue25011) and [issue
25209](https://bugs.python.org/issue25209).)

### shlex¶

The [`shlex`](https://docs.python.org/3.6/library/shlex.html#shlex.shlex
"shlex.shlex" ) has much [improved shell
compatibility](https://docs.python.org/3.6/library/shlex.html#improved-shell-
compatibility) through the new _punctuation\_chars_ argument to control which
characters are treated as punctuation. (Contributed by Vinay Sajip in [issue
1521950](https://bugs.python.org/issue1521950).)

### site¶

When specifying paths to add to
[`sys.path`](https://docs.python.org/3.6/library/sys.html#sys.path "sys.path"
) in a .pth file, you may now specify file paths on top of directories (e.g.
zip files). (Contributed by Wolfgang Langner in [issue
26587](https://bugs.python.org/issue26587)).

### sqlite3¶

[`sqlite3.Cursor.lastrowid`](https://docs.python.org/3.6/library/sqlite3.html\#
sqlite3.Cursor.lastrowid "sqlite3.Cursor.lastrowid" ) 现在支持
`REPLACE` 声明. (贡献者： Alex LordThorsen 于 [issue
16864](https://bugs.python.org/issue16864).)

### socket¶

[`ioctl()`](https://docs.python.org/3.6/library/socket.html#socket.socket.
ioctl "socket.socket.ioctl" ) 函数现在支持 [\`SIO\_LOOPBACK\_FAST\_PA
TH\`](https://docs.python.org/3.6/library/socket.html#socket.SIO\_LOOPBACK\_FAST\_
PATH "socket.SIO\_LOOPBACK\_FAST\_PATH" ) 控制代码. (贡献者： Daniel
Stokes 于 [issue 26536][135].)

[`getsockopt()`](https://docs.python.org/3.6/library/socket.html#socket.so
cket.getsockopt "socket.socket.getsockopt" ) 常量 `SO_DOMAIN`,
`SO_PROTOCOL`, `SO_PEERSEC`, 和 `SO_PASSSEC` 现已被支持. (贡献者：Christian Heimes 于 [issue 26907][136].)

[`setsockopt()`](https://docs.python.org/3.6/library/socket.html#socket.so
cket.setsockopt "socket.socket.setsockopt" ) 现在支持
`setsockopt(level, optname, None, optlen: int)` 形式. (贡献者：
Christian Heimes 于 [issue 27744][137].)

socket 模块现在支持地址族
[`AF_ALG`](https://docs.python.org/3.6/library/socket.html#socket.AF\_ALG
"socket.AF\_ALG" ) 来和Linux Kernel crypto API连接. 添加了`ALG_*`,
`SOL_ALG` 和 [`sendmsg_afalg()`](https://docs.python.org/3.6/library/socket.h
tml#socket.socket.sendmsg\_afalg "socket.socket.sendmsg\_afalg" ).
(贡献者： Christian Heimes 于 [issue
27744](https://bugs.python.org/issue27744) with support from Victor Stinner.)

### socketserver¶

基于
[`socketserver`](https://docs.python.org/3.6/library/socketserver.html#module-
socketserver "socketserver: A framework for network servers." ) 模块,
包括了
[`http.server`](https://docs.python.org/3.6/library/http.server.html#module-
http.server "http.server: HTTP server and request handlers." ),
[`xmlrpc.server`](https://docs.python.org/3.6/library/xmlrpc.server.html#module-xmlrpc.server "xmlrpc.server: Basic XML-RPC server implementations." )
和 [`wsgiref.simple_server`](https://docs.python.org/3.6/library/wsgiref.html#module-wsgiref.simple_server)的Servers 现在支持 [context
manager](https://docs.python.org/3.6/glossary.html#term-context-manager)
protocol. (Contributed by Aviv Palivoda in [issue
26404](https://bugs.python.org/issue26404).)

[`StreamRequestHandler`](https://docs.python.org/3.6/
library/socketserver.html#socketserver.StreamRequestHandler
"socketserver.StreamRequestHandler" ) 类的 `wfile` 属性 现在实现 the [\`io.Buffered
IOBase\`](https://docs.python.org/3.6/library/io.html#io.BufferedIOBase
"io.BufferedIOBase" ) 可写接口. 特别地, 调用 [`write()`](h
ttps://docs.python.org/3.6/library/io.html#io.BufferedIOBase.write
"io.BufferedIOBase.write" ) 现在可以保证完全发送数据.
(贡献者： Martin Panter 于 [issue
26721](https://bugs.python.org/issue26721).)

### ssl¶

[`ssl`][139] 已支持 OpenSSL 1.1.0. 最低推荐版本号是 1.0.2. (贡献者： Christian Heimes 于 [issue
26470](https://bugs.python.org/issue26470).)

3DES 已从默认密码套件default cipher suites中删除
添加了密码套件ChaCha20 Poly1305. (贡献者： Christian Heimes 于 [issue
27850](https://bugs.python.org/issue27850) 和 [issue
27766](https://bugs.python.org/issue27766).)

[`SSLContext`](https://docs.python.org/3.6/library/ssl.html#ssl.SSLContext
"ssl.SSLContext" ) 有更好的默认配置选项和密码.
(贡献者： Christian Heimes 于 [issue
28043](https://bugs.python.org/issue28043).)

SSL session 可以使用新的 [`SSLSession`](https://docs.python.org/3.6/library/ssl.html#ssl.SSLSession
"ssl.SSLSession" ) 类从一个客户端连接复制到另一个客户端连接. TLS session resumption can speed up the initial
handshake, reduce latency and improve performance (Contributed by Christian
Heimes in [issue 19500][138] based on a draft
by Alex Warhawk.)
SSL会话可以使用新的SSLSession类从一个客户端连接复制到另一个客户端连接。 TLS会话恢复可以加速初始握手，减少延迟并提高性能

新的 [`get_ciphers()`](https://docs.python.org/3.6/library/ssl.html#ssl.SSL
Context.get\_ciphers "ssl.SSLContext.get\_ciphers" ) 方法可以用来得到一个
按密码优先级顺序的已启用密码列表.

所有的 constants and flags 已转换为
[`IntEnum`](https://docs.python.org/3.6/library/enum.html#enum.IntEnum
"enum.IntEnum" ) 和 `IntFlags`. (贡献者 ：Christian Heimes 于 [issue
28025](https://bugs.python.org/issue28025).)

Server 和 client-side 添加 [`SSLContext`](https://docs.python.org/3.6/library/ssl.html#ssl.SSLContext
"ssl.SSLContext" ) 特定 TLS 协议. (贡献者： Christian Heimes 于 [issue
28085](https://bugs.python.org/issue28085).)

### statistics¶

A new [`harmonic_mean()`](https://docs.python.org/3.6/library/statistics.html\#
statistics.harmonic\_mean "statistics.harmonic\_mean" ) function has been added.
(Contributed by Steven D'Aprano in [issue
27181](https://bugs.python.org/issue27181).)

### struct¶

[`struct`](https://docs.python.org/3.6/library/struct.html#module-struct
"struct: Interpret bytes as packed binary data." ) now supports IEEE 754 half-
precision floats via the `'e'` format specifier. (Contributed by Eli Stevens,
Mark Dickinson in [issue 11734][139].)

### subprocess¶

[`subprocess.Popen`](https://docs.python.org/3.6/library/subprocess.html#subpr
ocess.Popen "subprocess.Popen" ) destructor now emits a [`ResourceWarning`](ht
tps://docs.python.org/3.6/library/exceptions.html#ResourceWarning
"ResourceWarning" ) warning if the child process is still running. Use the
context manager protocol (`with proc: ...`) or explicitly call the [`wait()`](
https://docs.python.org/3.6/library/subprocess.html#subprocess.Popen.wait
"subprocess.Popen.wait" ) method to read the exit status of the child process.
(Contributed by Victor Stinner in [issue
26741](https://bugs.python.org/issue26741).)

The [`subprocess.Popen`](https://docs.python.org/3.6/library/subprocess.html#s
ubprocess.Popen "subprocess.Popen" ) constructor and all functions that pass
arguments through to it now accept _encoding_ and _errors_ arguments.
Specifying either of these will enable text mode for the _stdin_, _stdout_ and
_stderr_ streams. (Contributed by Steve Dower in [issue
6135](https://bugs.python.org/issue6135).)

### sys¶

The new [`getfilesystemencodeerrors()`](https://docs.python.org/3.6/library/sy
s.html#sys.getfilesystemencodeerrors "sys.getfilesystemencodeerrors" )
function returns the name of the error mode used to convert between Unicode
filenames and bytes filenames. (Contributed by Steve Dower in [issue
27781](https://bugs.python.org/issue27781).)

On Windows the return value of the [`getwindowsversion()`](https://docs.python
.org/3.6/library/sys.html#sys.getwindowsversion "sys.getwindowsversion" )
function now includes the _platform\_version_ field which contains the accurate
major version, minor version and build number of the current operating system,
rather than the version that is being emulated for the process (Contributed by
Steve Dower in [issue 27932][140].)

### telnetlib¶

[`Telnet`](https://docs.python.org/3.6/library/telnetlib.html#telnetlib.Telnet
"telnetlib.Telnet" ) is now a context manager (contributed by Stéphane Wirtel
in [issue 25485][141]).

### time¶

The
[`struct_time`](https://docs.python.org/3.6/library/time.html#time.struct\_time
"time.struct\_time" ) attributes `tm_gmtoff` and `tm_zone` are now available on
all platforms.

### timeit¶

The new [`Timer.autorange()`](https://docs.python.org/3.6/library/timeit.html\#
timeit.Timer.autorange "timeit.Timer.autorange" ) convenience method has been
added to call [`Timer.timeit()`](https://docs.python.org/3.6/library/timeit.ht
ml#timeit.Timer.timeit "timeit.Timer.timeit" ) repeatedly so that the total
run time is greater or equal to 200 milliseconds. (Contributed by Steven
D'Aprano in [issue 6422][142].)

[`timeit`](https://docs.python.org/3.6/library/timeit.html#module-timeit
"timeit: Measure the execution time of small code snippets." ) now warns when
there is substantial (4x) variance between best and worst times. (Contributed
by Serhiy Storchaka in [issue 23552][143].)

### tkinter¶

Added methods `trace_add()`, `trace_remove()` and `trace_info()` in the
`tkinter.Variable` class. They replace old methods `trace_variable()`,
`trace()`, `trace_vdelete()` and `trace_vinfo()` that use obsolete Tcl
commands and might not work in future versions of Tcl. (Contributed by Serhiy
Storchaka in [issue 22115][144]).

### traceback

traceback模块和解释器内置的异常展示现在都省略回溯中重复行的长串，如以下例子所示：

```

    >>> def f(): f()
    ...
    >>> f()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 1, in f
      File "<stdin>", line 1, in f
      File "<stdin>", line 1, in f
      [Previous line repeated 995 more times]
    RecursionError: maximum recursion depth exceeded

```

(由Emanuel Barry在[issue 26823](https://bugs.python.org/issue26823)中贡献。)

### tracemalloc

[`tracemalloc`](https://docs.python.org/3.6/library/tracemalloc.html#module-tracemalloc)模块限制支持跟踪多个不同地址空间内的内存分配。

已添加新的[`DomainFilter`](https://docs.python.org/3.6/library/tracemalloc.html#tracemalloc.DomainFilter "tracemalloc.DomainFilter" )过滤器类来根据地址空间（域）过滤块跟踪。

(由Victor Stinner在[issue 26588](https://bugs.python.org/issue26588)中贡献。)

### typing

从Python 3.6起，[`typing`](https://docs.python.org/3.6/library/typing.html#module-typing "typing: Support for type hints \(see PEP 484\)." )模块不再是临时的了，可以把它的API当成稳定版本使用。

由于[`typing`](https://docs.python.org/3.6/library/typing.html#module-typing "typing: Support for type hints \(see PEP 484\)." )模块在Python 3.5中是[临时的](https://docs.python.org/3.6/glossary.html#term-provisional-api)，因此Python 3.6中引入的所有变动也已向后移植到Python 3.5.x。

[`typing`](https://docs.python.org/3.6/library/typing.html#module-typing "typing: Support for type hints \(see PEP 484\)." )模块很大提高了对范型别名的支持。例如，`Dict[str, Tuple[S, T]]`现在是一个有效的类型注释了。(由Guido van Rossum在[Github #195](https://github.com/python/typing/pull/195)中贡献。)

已添加[`typing.ContextManager`](https://docs.python.org/3.6/library/typing.html#typing.ContextManager "typing.ContextManager" )类来展示[`contextlib.AbstractContextManager`](https://docs.python.org/3.6/library/contextlib.html#contextlib.AbstractContextManager "contextlib.AbstractContextManager" )。 (由Brett Cannon在[issue 25609](https://bugs.python.org/issue25609)中贡献。)

已添加[`typing.Collection`](https://docs.python.org/3.6/library/typing.html#typing.Collection "typing.Collection" )类来展示[`collections.abc.Collection`](https://docs.python.org/3.6/library/collections.abc.html#collections.abc.Collection "collections.abc.Collection" )。 (由Ivan Levkivskyi在[issue 27598](https://bugs.python.org/issue27598)中贡献。)

已添加[`typing.ClassVar`](https://docs.python.org/3.6/library/typing.html#typing.ClassVar "typing.ClassVar" )类型构造，来标识类变量。如[\*\*PEP 526\*\*](https://www.python.org/dev/peps/pep-0526)中所述，封装在ClassVar中的一个变量注释暗示着一个给定的属性打算作为一个类变量使用，并且不应该在那个类的实例上设置它。(由Ivan Levkivskyi在[Github #280](https://github.com/python/typing/issues/280)中贡献。)

一个新的[`TYPE_CHECKING`](https://docs.python.org/3.6/library/typing.html#typing.TYPE\_CHECKING "typing.TYPE\_CHECKING" )常量被静态类型检查器假设为`True`，但在运行时则为`False`。 (由Guido van Rossum在[Github #230](https://github.com/python/typing/issues/230)中贡献。)

已添加一个新的[`NewType()`](https://docs.python.org/3.6/library/typing.html#typing.NewType
"typing.NewType" )辅助函数来为注释创建轻量单值类型：

```

    from typing import NewType

    UserId = NewType('UserId', int)
    some_id = UserId(524313)

```

静态类型检查器将会把新的类型当成原始类型的一个子类。 (由Ivan Levkivskyi在[Github #189](https://github.com/python/typing/issues/189)中贡献。)

### unicodedata¶

The [`unicodedata`](https://docs.python.org/3.6/library/unicodedata.html
# module-unicodedata "unicodedata: Access the Unicode Database." ) module now
uses data from [Unicode 9.0.0][148].
(Contributed by Benjamin Peterson.)

### unittest.mock¶

The [`Mock`](https://docs.python.org/3.6/library/unittest.mock.html#unittest.m
ock.Mock "unittest.mock.Mock" ) class has the following improvements:

  * Two new methods, [`Mock.assert_called()`][149] and [`Mock.assert_called_once()`][150] to check if the mock object was called. (Contributed by Amit Saha in [issue 26323][151].)
  * The [`Mock.reset_mock()`][152] method now has two optional keyword only arguments: _return\_value_ and _side\_effect_. (Contributed by Kushal Das in [issue 21271][153].)

### urllib.request¶

If a HTTP request has a file or iterable body (other than a bytes object) but
no `Content-Length` header, rather than throwing an error,
`AbstractHTTPHandler` now falls back to use chunked transfer encoding.
(Contributed by Demian Brecht and Rolf Krahl in [issue
12319](https://bugs.python.org/issue12319).)

### urllib.robotparser¶

[`RobotFileParser`](https://docs.python.org/3.6/library/urllib.robotparser.htm
l#urllib.robotparser.RobotFileParser "urllib.robotparser.RobotFileParser" )
now supports the `Crawl-delay` and `Request-rate` extensions. (Contributed by
Nikolay Bogoychev in [issue 16099][154].)

### venv¶

[`venv`][155] accepts a new parameter `--prompt`. This
parameter provides an alternative prefix for the virtual environment.
(Proposed by Łukasz Balcerzak and ported to 3.6 by Stéphane Wirtel in [issue
22829](https://bugs.python.org/issue22829).)

### warnings¶

A new optional _source_ parameter has been added to the [\`warnings.warn\_explic
it()\`](https://docs.python.org/3.6/library/warnings.html#warnings.warn\_explici
t "warnings.warn\_explicit" ) function: the destroyed object which emitted a [\`
ResourceWarning\`](https://docs.python.org/3.6/library/exceptions.html#Resource
Warning "ResourceWarning" ). A _source_ attribute has also been added to
`warnings.WarningMessage` (contributed by Victor Stinner in [issue
26568](https://bugs.python.org/issue26568) and [issue
26567](https://bugs.python.org/issue26567)).

When a [`ResourceWarning`](https://docs.python.org/3.6/library/exceptions.html
# ResourceWarning "ResourceWarning" ) warning is logged, the
[`tracemalloc`](https://docs.python.org/3.6/library/tracemalloc.html#module-
tracemalloc "tracemalloc: Trace memory allocations." ) module is now used to
try to retrieve the traceback where the destroyed object was allocated.

Example with the script `example.py`:

```

    import warnings

    def func():
        return open(__file__)

    f = func()
    f = None

```

Output of the command `python3.6 -Wd -X tracemalloc=5 example.py`:

```

    example.py:7: ResourceWarning: unclosed file <_io.TextIOWrapper name='example.py' mode='r' encoding='UTF-8'>
      f = None
    Object allocated at (most recent call first):
      File "example.py", lineno 4
        return open(__file__)
      File "example.py", lineno 6
        f = func()

```

The "Object allocated at" traceback is new and is only displayed if
[`tracemalloc`](https://docs.python.org/3.6/library/tracemalloc.html#module-
tracemalloc "tracemalloc: Trace memory allocations." ) is tracing Python
memory allocations and if the
[`warnings`](https://docs.python.org/3.6/library/warnings.html#module-warnings
"warnings: Issue warning messages and control their disposition." ) module was
already imported.

### winreg¶

Added the 64-bit integer type
[`REG_QWORD`](https://docs.python.org/3.6/library/winreg.html#winreg.REG\_QWORD
"winreg.REG\_QWORD" ). (Contributed by Clement Rouault in [issue
23026](https://bugs.python.org/issue23026).)

### winsound¶

Allowed keyword arguments to be passed to
[`Beep`](https://docs.python.org/3.6/library/winsound.html#winsound.Beep
"winsound.Beep" ), [`MessageBeep`](https://docs.python.org/3.6/library/winsoun
d.html#winsound.MessageBeep "winsound.MessageBeep" ), and [`PlaySound`](https:
//docs.python.org/3.6/library/winsound.html#winsound.PlaySound
"winsound.PlaySound" ) ([issue 27982][156]).

### xmlrpc.client¶

The [`xmlrpc.client`](https://docs.python.org/3.6/library/xmlrpc.client.html
# module-xmlrpc.client "xmlrpc.client: XML-RPC client access." ) module now
supports unmarshalling additional data types used by the Apache XML-RPC
implementation for numerics and `None`. (Contributed by Serhiy Storchaka in
[issue 26885][157].)

### zipfile¶

A new [`ZipInfo.from_file()`](https://docs.python.org/3.6/library/zipfile.html
# zipfile.ZipInfo.from\_file "zipfile.ZipInfo.from\_file" ) class method allows
making a
[`ZipInfo`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipInfo
"zipfile.ZipInfo" ) instance from a filesystem file. A new [`ZipInfo.is_dir()`
](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipInfo.is\_dir
"zipfile.ZipInfo.is\_dir" ) method can be used to check if the
[`ZipInfo`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipInfo
"zipfile.ZipInfo" ) instance represents a directory. (Contributed by Thomas
Kluyver in [issue 26039][158].)

The [`ZipFile.open()`](https://docs.python.org/3.6/library/zipfile.html#zipfil
e.ZipFile.open "zipfile.ZipFile.open" ) method can now be used to write data
into a ZIP file, as well as for extracting data. (Contributed by Thomas
Kluyver in [issue 26039][159].)

### zlib¶

The [`compress()`](https://docs.python.org/3.6/library/zlib.html#zlib.compress
"zlib.compress" ) and
[`decompress()`](https://docs.python.org/3.6/library/zlib.html#zlib.decompress
"zlib.decompress" ) functions now accept keyword arguments. (Contributed by
Aviv Palivoda in [issue 26243][160] and Xiang
Zhang in [issue 16764][161] respectively.)

## 优化

  * Python解释器现在使用16位编码而不是字节码，这使得你可以实现多种操作码的优化。(由Demur Rumed，Serhiy Storchaka和Victor Stinner在[issue 26647][162] 以及 [issue 28050][163]中贡献。)
  * [`asyncio.Future`][164]类现在有一种优化的C实现。(由Yury Selivanov和INADA Naoki在[issue 26081][165]中贡献。)
  * [`asyncio.Task`][166]类现在有一种优化的C实现。(由Yury Selivanov在[issue 28544][167]中贡献。)
  * 在[`typing`][168]模块中通过各种改进的实施(如泛型类型的缓存)，使得其允许高达30倍的性能优化，并且减少了内存占用。
  * ASCII解码器的错误处理程序 `surrogateescape`, `ignore` 以及 `replace` 速度提高了60倍。(由Victor Stinner在[issue 24870][169]中贡献)。
  * ASCII和Latin1解码器的错误处理程序 `surrogateescape` 速度提高了3倍。(由Victor Stinner在[issue 25227][170]中贡献)。
  * UTF-8解码器的错误处理程序 `ignore`, `replace`, `surrogateescape`, `surrogatepass`速度提高了75倍。(由Victor Stinner在[issue 25267][171]中贡献)。
  * UTF-8解码器的错误处理程序`ignore`, `replace` and `surrogateescape`速度提高了15倍。由Victor Stinner在[issue 25301][172]中贡献)。
  * `bytes % args`现在速度可以提高到2倍。(由Victor Stinner在[issue 25349][173]中贡献)。
  * `bytearray % args` 现在速度是以前的2.5到5倍。(由Victor Stinner在[issue 25399][174]中贡献)。
  * 对[`bytes.fromhex()`][175]和[`bytearray.fromhex()`][176]进行优化:现在比以前快2x到3.5x倍。(由Victor Stinner在[issue 25401][177]中贡献)。
  * 对`bytes.replace(b'', b'.')`和`bytearray.replace(b'', b'.')`进行优化:速度可提高80%。 (由Josh Snider在[issue 26574][178]中贡献)。
  * 现在[`PyMem_Malloc()`][179]域([`PYMEM_DOMAIN_MEM`][180])的分配函数使用[pymalloc memory allocator][181]，而不是C库中的`malloc()`函数。 pymalloc分配器针对大小小于等于512字节、生命周期较短的对象进行了优化，而当需要占用较大内存块时则使用`malloc()`。 (由Victor Stinner在[issue 26249][182]中贡献)。
  * 当对大量小对象进行反序列化时，[`pickle.load()`][183]和[`pickle.loads()`][184]的速度可提高10%。(由Victor Stinner在[issue 27056][185]中贡献)。
  * 与传递[位置参数][187]相比，将[关键字参数][186]传递给函数会带来额外的开销。 现在在通过使用Argument Clinic实现的扩展功能中，这种开销将会显著的降低。(由Serhiy Storchaka在[issue 27574][188]中贡献)。
  *对[`glob`][191]模块中的[`glob()`][189]及[`iglob()`][190]进行优化;使得它们现在大概快了3-6倍。(由Serhiy Storchaka在[issue 25596][192]中贡献)。
  * 使用[`os.scandir()`][194]对[`pathlib`][193]中的glob进行优化;使它大概快了1.5-4倍。(由Serhiy Storchaka在[issue 26032][195]中贡献)。
  * [`xml.etree.ElementTree`][196]中解析、迭代和深拷贝的性能有了显著的提高。 (由Serhiy Storchaka在[issue 25638][197]、[issue 25873][198]及[issue 25869][199]中贡献。)
  * 由浮点数和小数创建[`fractions.Fraction`][200]实例的速度提高了2到3倍。(由Serhiy Storchaka在[issue 25971][201]中贡献。)

## 构建和C API变动

  * Python现在构建需要工具链中的一些C99支持。最值得注意的是，Python现在使用标准的整型类型和宏来替代诸如`PY_LONG_LONG`的自定义宏。欲知详情，见[**PEP 7**](https://www.python.org/dev/peps/pep-0007)和[issue 17884](https://bugs.python.org/issue17884)。
  * 带Android NDK和Android API level 21 (Android 5.0 Lollilop) 或者更高版本交叉编译CPython可以成功运行。虽然Android尚未是一个支持的平台，但是Python测试套件在Android模拟器上运行只有大约16个测试错误。见Android元问题[issue 26865](https://bugs.python.org/issue26865)。
  * 已添加`--enable-optimizations`配置标志。打开它将会激活昂贵的优化，例如PGO。(原始补丁由Intel的Alecsandru Patrascu在[issue 26539](https://bugs.python.org/issue26539)中提出。)
  * 当[`PYMEM_DOMAIN_OBJ`](https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_OBJ) (例如：`PyObject_Malloc()`)和[`PYMEM_DOMAIN_MEM`](https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_MEM) (例如：[`PyMem_Malloc()`](https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc))域的分配函数被调用的时候，现在必须使用[GIL](https://docs.python.org/3.6/glossary.html#term-global-interpreter-lock)。
  * 新的[`Py_FinalizeEx()`](https://docs.python.org/3.6/c-api/init.html#c.Py_FinalizeEx) API，它表示是否清除缓冲数据失败。 (由Martin Panter在[issue 5319](https://bugs.python.org/issue5319)中贡献。)
  * [`PyArg_ParseTupleAndKeywords()`](https://docs.python.org/3.6/c-api/arg.html#c.PyArg_ParseTupleAndKeywords)现在支持[仅位置参数](https://docs.python.org/3.6/glossary.html#positional-only-parameter)。仅位置参数是由空名称定义的。 (由Serhiy Storchaka在[issue 26282](https://bugs.python.org/issue26282)中贡献)。
  * `PyTraceback_Print`方法现在省略了重复行的长串，变成`"[Previous line repeated {count} more times]"`。 (由Emanuel Barry在[issue 26823](https://bugs.python.org/issue26823)中贡献。)
  * 新的[`PyErr_SetImportErrorSubclass()`](https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_SetImportErrorSubclass)函数指定要引发的[`ImportError`](https://docs.python.org/3.6/library/exceptions.html#ImportError)的子类。 (由Eric Snow在[issue 15767](https://bugs.python.org/issue15767)中贡献。)
  * 新的[`PyErr_ResourceWarning()`](https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_ResourceWarning)函数可以被用来生成一个[`ResourceWarning`](https://docs.python.org/3.6/library/exceptions.html#ResourceWarning)，提供资源分配的来源。(由Victor Stinner在[issue 26567](https://bugs.python.org/issue26567)中贡献。)
  * 新的[`PyOS_FSPath()`](https://docs.python.org/3.6/c-api/sys.html#c.PyOS_FSPath)函数返回一个[类路径(path-like)对象](https://docs.python.org/3.6/glossary.html#term-path-like-object)的文件系统表示。(由Brett Cannon在[issue 27186](https://bugs.python.org/issue27186)中贡献。)
  * [`PyUnicode_FSConverter()`](https://docs.python.org/3.6/c-api/unicode.html#c.PyUnicode_FSConverter)和[`PyUnicode_FSDecoder()`](https://docs.python.org/3.6/c-api/unicode.html#c.PyUnicode_FSDecoder)函数现在会接受[类路径(path-like)对象](https://docs.python.org/3.6/glossary.html#term-path-like-object)。

## 其他改进

  * 当使用[`--version`](https://docs.python.org/3.6/using/cmdline.html#cmdoption--version) (短格式：[`-V`](https://docs.python.org/3.6/using/cmdline.html#cmdoption-V))两次的时候，Python打印[`sys.version`](https://docs.python.org/3.6/library/sys.html#sys.version)以获得详细信息。
```     
$ ./python -VV

    Python 3.6.0b4+ (3.6:223967b49e49+, Nov 21 2016, 20:55:04)
    [GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.42.1)]

```

## Deprecated¶

### New Keywords¶

`async` and `await` are not recommended to be used as variable, class,
function or module names. Introduced by [\*\*PEP
492\*\*](https://www.python.org/dev/peps/pep-0492) in Python 3.5, they will
become proper keywords in Python 3.7. Starting in Python 3.6, the use of
`async` or `await` as names will generate a [`DeprecationWarning`](https://doc
s.python.org/3.6/library/exceptions.html#DeprecationWarning
"DeprecationWarning" ).

### Deprecated Python behavior¶

Raising the [`StopIteration`](https://docs.python.org/3.6/library/exceptions.h
tml#StopIteration "StopIteration" ) exception inside a generator will now
generate a [`DeprecationWarning`](https://docs.python.org/3.6/library/exceptio
ns.html#DeprecationWarning "DeprecationWarning" ), and will trigger a [\`Runtim
eError\`](https://docs.python.org/3.6/library/exceptions.html#RuntimeError
"RuntimeError" ) in Python 3.7. See [PEP 479: Change StopIteration handling
inside generators](https://docs.python.org/3.6/whatsnew/3.5.html#whatsnew-
pep-479) for details.

The [`__aiter__()`](https://docs.python.org/3.6/reference/datamodel.html#objec
t.\_\_aiter\_\_ "object.\_\_aiter\_\_" ) method is now expected to return an
asynchronous iterator directly instead of returning an awaitable as
previously. Doing the former will trigger a [`DeprecationWarning`](https://doc
s.python.org/3.6/library/exceptions.html#DeprecationWarning
"DeprecationWarning" ). Backward compatibility will be removed in Python 3.7.
(Contributed by Yury Selivanov in [issue
27243](https://bugs.python.org/issue27243).)

A backslash-character pair that is not a valid escape sequence now generates a
[`DeprecationWarning`](https://docs.python.org/3.6/library/exceptions.html#Dep
recationWarning "DeprecationWarning" ). Although this will eventually become a
[`SyntaxError`](https://docs.python.org/3.6/library/exceptions.html#SyntaxErro
r "SyntaxError" ), that will not be for several Python releases. (Contributed
by Emanuel Barry in [issue 27364][231].)

When performing a relative import, falling back on `__name__` and `__path__`
from the calling module when `__spec__` or `__package__` are not defined now
raises an [`ImportWarning`](https://docs.python.org/3.6/library/exceptions.htm
l#ImportWarning "ImportWarning" ). (Contributed by Rose Ames in [issue
25791](https://bugs.python.org/issue25791).)

### Deprecated Python modules, functions and methods¶

#### asynchat¶

The [`asynchat`](https://docs.python.org/3.6/library/asynchat.html#module-
asynchat "asynchat: Support for asynchronous command/response protocols." )
has been deprecated in favor of
[`asyncio`](https://docs.python.org/3.6/library/asyncio.html#module-asyncio
"asyncio: Asynchronous I/O, event loop, coroutines and tasks." ). (Contributed
by Mariatta in [issue 25002][232].)

#### asyncore¶

The [`asyncore`](https://docs.python.org/3.6/library/asyncore.html#module-
asyncore "asyncore: A base class for developing asynchronous socket handling
services." ) has been deprecated in favor of
[`asyncio`](https://docs.python.org/3.6/library/asyncio.html#module-asyncio
"asyncio: Asynchronous I/O, event loop, coroutines and tasks." ). (Contributed
by Mariatta in [issue 25002][233].)

#### dbm¶

Unlike other [`dbm`](https://docs.python.org/3.6/library/dbm.html#module-dbm
"dbm: Interfaces to various Unix "database" formats." ) implementations, the
[`dbm.dumb`](https://docs.python.org/3.6/library/dbm.html#module-dbm.dumb
"dbm.dumb: Portable implementation of the simple DBM interface." ) module
creates databases with the `'rw'` mode and allows modifying the database
opened with the `'r'` mode. This behavior is now deprecated and will be
removed in 3.8. (Contributed by Serhiy Storchaka in [issue
21708](https://bugs.python.org/issue21708).)

#### distutils¶

The undocumented `extra_path` argument to the `Distribution` constructor is
now considered deprecated and will raise a warning if set. Support for this
parameter will be removed in a future Python release. See [issue
27919](https://bugs.python.org/issue27919) for details.

#### grp¶

The support of non-integer arguments in
[`getgrgid()`](https://docs.python.org/3.6/library/grp.html#grp.getgrgid
"grp.getgrgid" ) has been deprecated. (Contributed by Serhiy Storchaka in
[issue 26129][234].)

#### importlib¶

The [`importlib.machinery.SourceFileLoader.load_module()`](https://docs.python
.org/3.6/library/importlib.html#importlib.machinery.SourceFileLoader.load\_modu
le "importlib.machinery.SourceFileLoader.load\_module" ) and [\`importlib.machin
ery.SourcelessFileLoader.load\_module()\`](https://docs.python.org/3.6/library/i
mportlib.html#importlib.machinery.SourcelessFileLoader.load\_module
"importlib.machinery.SourcelessFileLoader.load\_module" ) methods are now
deprecated. They were the only remaining implementations of [\`importlib.abc.Lo
ader.load\_module()\`](https://docs.python.org/3.6/library/importlib.html#import
lib.abc.Loader.load\_module "importlib.abc.Loader.load\_module" ) in
[`importlib`](https://docs.python.org/3.6/library/importlib.html#module-
importlib "importlib: The implementation of the import machinery." ) that had
not been deprecated in previous versions of Python in favour of [\`importlib.ab
c.Loader.exec\_module()\`](https://docs.python.org/3.6/library/importlib.html#im
portlib.abc.Loader.exec\_module "importlib.abc.Loader.exec\_module" ).

The [`importlib.machinery.WindowsRegistryFinder`](https://docs.python.org/3.6/
library/importlib.html#importlib.machinery.WindowsRegistryFinder
"importlib.machinery.WindowsRegistryFinder" ) class is now deprecated. As of
3.6.0, it is still added to
[`sys.meta_path`](https://docs.python.org/3.6/library/sys.html#sys.meta\_path
"sys.meta\_path" ) by default (on Windows), but this may change in future
releases.

#### os¶

Undocumented support of general [bytes-like
objects](https://docs.python.org/3.6/glossary.html#term-bytes-like-object) as
paths in [`os`][235] functions,
[`compile()`](https://docs.python.org/3.6/library/functions.html#compile
"compile" ) and similar functions is now deprecated. (Contributed by Serhiy
Storchaka in [issue 25791][236] and [issue
26754](https://bugs.python.org/issue26754).)

#### re¶

Support for inline flags `(?letters)` in the middle of the regular expression
has been deprecated and will be removed in a future Python version. Flags at
the start of a regular expression are still allowed. (Contributed by Serhiy
Storchaka in [issue 22493][237].)

#### ssl¶

OpenSSL 0.9.8, 1.0.0 and 1.0.1 are deprecated and no longer supported. In the
future the [`ssl`](https://docs.python.org/3.6/library/ssl.html#module-ssl
"ssl: TLS/SSL wrapper for socket objects" ) module will require at least
OpenSSL 1.0.2 or 1.1.0.

SSL-related arguments like `certfile`, `keyfile` and `check_hostname` in
[`ftplib`](https://docs.python.org/3.6/library/ftplib.html#module-ftplib
"ftplib: FTP protocol client \(requires sockets\)." ),
[`http.client`](https://docs.python.org/3.6/library/http.client.html#module-
http.client "http.client: HTTP and HTTPS protocol client \(requires
sockets\)." ), [`imaplib`](https://docs.python.org/3.6/library/imaplib.html
# module-imaplib "imaplib: IMAP4 protocol client \(requires sockets\)." ),
[`poplib`](https://docs.python.org/3.6/library/poplib.html#module-poplib
"poplib: POP3 protocol client \(requires sockets\)." ), and
[`smtplib`](https://docs.python.org/3.6/library/smtplib.html#module-smtplib
"smtplib: SMTP protocol client \(requires sockets\)." ) have been deprecated
in favor of `context`. (Contributed by Christian Heimes in [issue
28022](https://bugs.python.org/issue28022).)

A couple of protocols and functions of the
[`ssl`][238] module are now deprecated. Some features will no
longer be available in future versions of OpenSSL. Other features are
deprecated in favor of a different API. (Contributed by Christian Heimes in
[issue 28022][239] and [issue
26470](https://bugs.python.org/issue26470).)

#### tkinter¶

The [`tkinter.tix`](https://docs.python.org/3.6/library/tkinter.tix.html
# module-tkinter.tix "tkinter.tix: Tk Extension Widgets for Tkinter" ) module
is now deprecated.
[`tkinter`](https://docs.python.org/3.6/library/tkinter.html#module-tkinter
"tkinter: Interface to Tcl/Tk for graphical user interfaces" ) users should
use [`tkinter.ttk`](https://docs.python.org/3.6/library/tkinter.ttk.html
# module-tkinter.ttk "tkinter.ttk: Tk themed widget set" ) instead.

#### venv¶

The `pyvenv` script has been deprecated in favour of `python3 -m venv`. This
prevents confusion as to what Python interpreter `pyvenv` is connected to and
thus what Python interpreter will be used by the virtual environment.
(Contributed by Brett Cannon in [issue
25154](https://bugs.python.org/issue25154).)

### Deprecated functions and types of the C API¶

Undocumented functions `PyUnicode_AsEncodedObject()`,
`PyUnicode_AsDecodedObject()`, `PyUnicode_AsEncodedUnicode()` and
`PyUnicode_AsDecodedUnicode()` are deprecated now. Use the [generic codec
based API](https://docs.python.org/3.6/c-api/codec.html#codec-registry)
instead.

### Deprecated Build Options¶

The `--with-system-ffi` configure flag is now on by default on non-macOS UNIX
platforms. It may be disabled by using `--without-system-ffi`, but using the
flag is deprecated and will not be accepted in Python 3.7. macOS is unaffected
by this change. Note that many OS distributors already use the \`--with-system-
ffi\` flag when building their system Python.

## Removed¶

### API and Feature Removals¶

  * Unknown escapes consisting of `'\'` and an ASCII letter in regular expressions will now cause an error. In replacement templates for [`re.sub()`][240] they are still allowed, but deprecated. The [`re.LOCALE`][241] flag can now only be used with binary patterns.
  * `inspect.getmoduleinfo()` was removed (was deprecated since CPython 3.3). [`inspect.getmodulename()`][242] should be used for obtaining the module name for a given path. (Contributed by Yury Selivanov in [issue 13248][243].)
  * `traceback.Ignore` class and `traceback.usage`, `traceback.modname`, `traceback.fullmodname`, `traceback.find_lines_from_code`, `traceback.find_lines`, `traceback.find_strings`, `traceback.find_executable_lines` methods were removed from the [`traceback`][244] module. They were undocumented methods deprecated since Python 3.2 and equivalent functionality is available from private methods.
  * The `tk_menuBar()` and `tk_bindForTraversal()` dummy methods in [`tkinter`][245] widget classes were removed (corresponding Tk commands were obsolete since Tk 4.0).
  * The [`open()`][246] method of the [`zipfile.ZipFile`][247] class no longer supports the `'U'` mode (was deprecated since Python 3.4). Use [`io.TextIOWrapper`][248] for reading compressed text files in [universal newlines][249] mode.
  * The undocumented `IN`, `CDROM`, `DLFCN`, `TYPES`, `CDIO`, and `STROPTS` modules have been removed. They had been available in the platform specific `Lib/plat-*/` directories, but were chronically out of date, inconsistently available across platforms, and unmaintained. The script that created these modules is still available in the source distribution at [Tools/scripts/h2py.py][250].
  * The deprecated `asynchat.fifo` class has been removed.

## Porting to Python 3.6¶

This section lists previously described changes and other bugfixes that may
require changes to your code.

### Changes in 'python' Command Behavior¶

  * The output of a special Python build with defined `COUNT_ALLOCS`, `SHOW_ALLOC_COUNT` or `SHOW_TRACK_COUNT` macros is now off by default. It can be re-enabled using the `-X showalloccount` option. It now outputs to `stderr` instead of `stdout`. (Contributed by Serhiy Storchaka in [issue 23034][251].)

### Changes in the Python API¶

  * [`open()`][252] will no longer allow combining the `'U'` mode flag with `'+'`. (Contributed by Jeff Balogh and John O'Connor in [issue 2091][253].)

  * [`sqlite3`][254] no longer implicitly commits an open transaction before DDL statements.

  * On Linux, [`os.urandom()`][255] now blocks until the system urandom entropy pool is initialized to increase the security.

  * When [`importlib.abc.Loader.exec_module()`][256] is defined, [`importlib.abc.Loader.create_module()`][257] must also be defined.

  * [`PyErr_SetImportError()`][258] now sets [`TypeError`][259] when its **msg** argument is not set. Previously only `NULL` was returned.

  * The format of the `co_lnotab` attribute of code objects changed to support a negative line number delta. By default, Python does not emit bytecode with a negative line number delta. Functions using `frame.f_lineno`, `PyFrame_GetLineNumber()` or `PyCode_Addr2Line()` are not affected. Functions directly decoding `co_lnotab` should be updated to use a signed 8-bit integer type for the line number delta, but this is only required to support applications using a negative line number delta. See `Objects/lnotab_notes.txt` for the `co_lnotab` format and how to decode it, and see the [**PEP 511**][260] for the rationale.

  * The functions in the [`compileall`][261] module now return booleans instead of `1` or `0` to represent success or failure, respectively. Thanks to booleans being a subclass of integers, this should only be an issue if you were doing identity checks for `1` or `0`. See [issue 25768][262].

  * Reading the `port` attribute of [`urllib.parse.urlsplit()`][263] and [`urlparse()`][264] results now raises [`ValueError`][265] for out-of-range values, rather than returning [`None`][266]. See [issue 20059][267].

  * The [`imp`][268] module now raises a [`DeprecationWarning`][269] instead of [`PendingDeprecationWarning`][270].

  * The following modules have had missing APIs added to their `__all__` attributes to match the documented APIs: [`calendar`][271], [`cgi`][272], [`csv`][273], [`ElementTree`][274], [`enum`][275], [`fileinput`][276], [`ftplib`][277], [`logging`][278], [`mailbox`][279], [`mimetypes`][280], [`optparse`][281], [`plistlib`][282], [`smtpd`][283], [`subprocess`][284], [`tarfile`][285], [`threading`][286] and [`wave`][287]. This means they will export new symbols when `import *` is used. (Contributed by Joel Taddei and Jacek Kołodziej in [issue 23883][288].)

  * When performing a relative import, if `__package__` does not compare equal to `__spec__.parent` then [`ImportWarning`][289] is raised. (Contributed by Brett Cannon in [issue 25791][290].)

  * When a relative import is performed and no parent package is known, then [`ImportError`][291] will be raised. Previously, [`SystemError`][292] could be raised. (Contributed by Brett Cannon in [issue 18018][293].)

  * Servers based on the [`socketserver`][294] module, including those defined in [`http.server`][295], [`xmlrpc.server`][296] and [`wsgiref.simple_server`][297], now only catch exceptions derived from [`Exception`][298]. Therefore if a request handler raises an exception like [`SystemExit`][299] or [`KeyboardInterrupt`][300], [`handle_error()`][301] is no longer called, and the exception will stop a single-threaded server. (Contributed by Martin Panter in [issue 23430][302].)

  * [`spwd.getspnam()`][303] now raises a [`PermissionError`][304] instead of [`KeyError`][305] if the user doesn't have privileges.

  * The [`socket.socket.close()`][306] method now raises an exception if an error (e.g. `EBADF`) was reported by the underlying system call. (Contributed by Martin Panter in [issue 26685][307].)

  * The _decode\_data_ argument for the [`smtpd.SMTPChannel`][308] and [`smtpd.SMTPServer`][309] constructors is now `False` by default. This means that the argument passed to [`process_message()`][310] is now a bytes object by default, and `process_message()` will be passed keyword arguments. Code that has already been updated in accordance with the deprecation warning generated by 3.5 will not be affected.

  * All optional arguments of the [`dump()`][311], [`dumps()`][312], [`load()`][313] and [`loads()`][314] functions and [`JSONEncoder`][315] and [`JSONDecoder`][316] class constructors in the [`json`][317] module are now [keyword-only][318]. (Contributed by Serhiy Storchaka in [issue 18726][319].)

  * Subclasses of [`type`][320] which don't override `type.__new__` may no longer use the one-argument form to get the type of an object.

  * As part of [**PEP 487**][321], the handling of keyword arguments passed to [`type`][322] (other than the metaclass hint, `metaclass`) is now consistently delegated to [`object.__init_subclass__()`][323]. This means that `type.__new__()` and `type.__init__()` both now accept arbitrary keyword arguments, but [`object.__init_subclass__()`][324] (which is called from `type.__new__()`) will reject them by default. Custom metaclasses accepting additional keyword arguments will need to adjust their calls to `type.__new__()` (whether direct or via [`super`][325]) accordingly.

  * In `distutils.command.sdist.sdist`, the `default_format` attribute has been removed and is no longer honored. Instead, the gzipped tarfile format is the default on all platforms and no platform-specific selection is made. In environments where distributions are built on Windows and zip distributions are required, configure the project with a `setup.cfg` file containing the following:
```     [sdist]

    formats=zip

```

This behavior has also been backported to earlier Python versions by
Setuptools 26.0.0.

  * In the [`urllib.request`][326] module and the [`http.client.HTTPConnection.request()`][327] method, if no Content-Length header field has been specified and the request body is a file object, it is now sent with HTTP 1.1 chunked encoding. If a file object has to be sent to a HTTP 1.0 server, the Content-Length value now has to be specified by the caller. (Contributed by Demian Brecht and Rolf Krahl with tweaks from Martin Panter in [issue 12319][328].)

  * The [`DictReader`][329] now returns rows of type [`OrderedDict`][330]. (Contributed by Steve Holden in [issue 27842][331].)

  * The [`crypt.METHOD_CRYPT`][332] will no longer be added to `crypt.methods` if unsupported by the platform. (Contributed by Victor Stinner in [issue 25287][333].)

  * The _verbose_ and _rename_ arguments for [`namedtuple()`][334] are now keyword-only. (Contributed by Raymond Hettinger in [issue 25628][335].)

  * On Linux, [`ctypes.util.find_library()`][336] now looks in `LD_LIBRARY_PATH` for shared libraries. (Contributed by Vinay Sajip in [issue 9998][337].)

  * The [`imaplib.IMAP4`][338] class now handles flags containing the `']'` character in messages sent from the server to improve real-world compatibility. (Contributed by Lita Cho in [issue 21815][339].)

  * The `mmap.write()` function now returns the number of bytes written like other write methods. (Contributed by Jakub Stasiak in [issue 26335][340].)

  * The [`pkgutil.iter_modules()`][341] and [`pkgutil.walk_packages()`][342] functions now return [`ModuleInfo`][343] named tuples. (Contributed by Ramchandra Apte in [issue 17211][344].)

  * [`re.sub()`][345] now raises an error for invalid numerical group references in replacement templates even if the pattern is not found in the string. The error message for invalid group references now includes the group index and the position of the reference. (Contributed by SilentGhost, Serhiy Storchaka in [issue 25953][346].)

  * [`zipfile.ZipFile`][347] will now raise [`NotImplementedError`][348] for unrecognized compression values. Previously a plain [`RuntimeError`][349] was raised. Additionally, calling [`ZipFile`][350] methods on a closed ZipFile or calling the [`write()`][351] method on a ZipFile created with mode `'r'` will raise a [`ValueError`][352]. Previously, a [`RuntimeError`][353] was raised in those scenarios.

  * when custom metaclasses are combined with zero-argument [`super()`][354] or direct references from methods to the implicit `__class__` closure variable, the implicit `__classcell__` namespace entry must now be passed up to `type.__new__` for initialisation. Failing to do so will result in a [`DeprecationWarning`][355] in 3.6 and a [`RuntimeWarning`][356] in the future.

### Changes in the C API¶

  * The [`PyMem_Malloc()`][357] allocator family now uses the [pymalloc allocator][358] rather than the system `malloc()`. Applications calling [`PyMem_Malloc()`][359] without holding the GIL can now crash. Set the [`PYTHONMALLOC`][360] environment variable to `debug` to validate the usage of memory allocators in your application. See [issue 26249][361].
  * [`Py_Exit()`][362] (and the main interpreter) now override the exit status with 120 if flushing buffered data failed. See [issue 5319][363].

### CPython bytecode changes¶

There have been several major changes to the
[bytecode][364] in Python
3.6.

  * The Python interpreter now uses a 16-bit wordcode instead of bytecode. (Contributed by Demur Rumed with input and reviews from Serhiy Storchaka and Victor Stinner in [issue 26647][365] and [issue 28050][366].)
  * The new [`FORMAT_VALUE`][367] and [`BUILD_STRING`][368] opcodes as part of the formatted string literal implementation. (Contributed by Eric Smith in [issue 25483][369] and Serhiy Storchaka in [issue 27078][370].)
  * The new [`BUILD_CONST_KEY_MAP`][371] opcode to optimize the creation of dictionaries with constant keys. (Contributed by Serhiy Storchaka in [issue 27140][372].)
  * The function call opcodes have been heavily reworked for better performance and simpler implementation. The [`MAKE_FUNCTION`][373], [`CALL_FUNCTION`][374], [`CALL_FUNCTION_KW`][375] and `BUILD_MAP_UNPACK_WITH_CALL` opcodes have been modified, the new `CALL_FUNCTION_EX` and `BUILD_TUPLE_UNPACK_WITH_CALL` have been added, and `CALL_FUNCTION_VAR`, `CALL_FUNCTION_VAR_KW` and `MAKE_CLOSURE` opcodes have been removed. (Contributed by Demur Rumed in [issue 27095][376], and Serhiy Storchaka in [issue 27213][377], [issue 28257][378].)
  * The new [`SETUP_ANNOTATIONS`][379] and [`STORE_ANNOTATION`][380] opcodes have been added to support the new [variable annotation][381] syntax. (Contributed by Ivan Levkivskyi in [issue 27985][382].)

----
(C) [Copyright][383] 2001-2016, Python
Software Foundation.  
The Python Software Foundation is a non-profit corporation. [Please
donate.](https://www.python.org/psf/donations/)  
Last updated on Dec 15, 2016. [Found a
bug](https://docs.python.org/3.6/bugs.html)?  
Created using [Sphinx][384] 1.3.3.

[1]:	https://docs.python.org/3.6/whatsnew/3.6.html
[2]:	mailto:elvis@magic.io
[3]:	mailto:yury@magic.io
[4]:	https://www.python.org/dev/peps/pep-0494
[5]:	https://docs.python.org/3.6/library/secrets.html#module-secrets "secrets: Generate secure random numbers for managing secrets."
[6]:	https://docs.python.org/3.6/library/stdtypes.html#typesmapping
[7]:	https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-more.html
[8]:	https://docs.python.org/3.6/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O, event loop, coroutines and tasks."
[9]:	https://docs.python.org/3.6/glossary.html#term-path-like-object
[10]:	https://docs.python.org/3.6/library/datetime.html#module-datetime "datetime: Basic date and time types."
[11]:	https://docs.python.org/3.6/library/typing.html#module-typing "typing: Support for type hints \(see PEP 484\)."
[12]:	https://docs.python.org/3.6/library/tracemalloc.html#module-tracemalloc "tracemalloc: Trace memory allocations."
[13]:	https://docs.python.org/3.6/library/secrets.html#module-secrets "secrets: Generate secure random numbers for managing secrets."
[14]:	https://docs.python.org/3.6/library/os.html#os.urandom "os.urandom"
[15]:	https://www.python.org/dev/peps/pep-0524
[16]:	https://docs.python.org/3.6/library/hashlib-blake2.html#module-hashlib "hashlib: BLAKE2 hash function for Python"
[17]:	https://docs.python.org/3.6/library/ssl.html#module-ssl "ssl: TLS/SSL wrapper for socket objects"
[18]:	https://docs.python.org/3.6/library/ssl.html#module-ssl "ssl: TLS/SSL wrapper for socket objects"
[19]:	https://docs.python.org/3.6/library/hashlib-blake2.html#module-hashlib "hashlib: BLAKE2 hash function for Python"
[20]:	https://docs.python.org/3.6/library/hashlib.html#hashlib.scrypt "hashlib.scrypt"
[21]:	https://docs.python.org/3.6/using/windows.html#max-path
[22]:	https://docs.python.org/3.6/using/windows.html#finding-modules
[23]:	https://docs.python.org/3.6/using/cmdline.html#envvar-PYTHONHOME
[24]:	https://docs.python.org/3.6/using/windows.html#finding-modules
[25]:	https://www.python.org/dev/peps/pep-0498
[26]:	https://docs.python.org/3.6/reference/lexical_analysis.html#f-strings
[27]:	https://docs.python.org/3.6/library/stdtypes.html#str.format "str.format"
[28]:	https://docs.python.org/3.6/library/functions.html#format "format"
[29]:	https://www.python.org/dev/peps/pep-0498
[30]:	https://docs.python.org/3.6/reference/lexical_analysis.html#f-strings
[31]:	https://www.python.org/dev/peps/pep-0484
[32]:	https://www.python.org/dev/peps/pep-0526
[33]:	http://github.com/python/mypy
[34]:	http://github.com/google/pytype
[35]:	https://www.python.org/dev/peps/pep-0515
[36]:	https://docs.python.org/3.6/library/string.html#formatspec
[37]:	https://www.python.org/dev/peps/pep-0515
[38]:	https://www.python.org/dev/peps/pep-0492
[39]:	https://www.python.org/dev/peps/pep-0525
[40]:	https://www.python.org/dev/peps/pep-0530
[41]:	https://www.python.org/dev/peps/pep-0530
[42]:	https://docs.python.org/3.6/library/functions.html#super "super"
[43]:	https://www.python.org/dev/peps/pep-0487
[44]:	https://www.python.org/dev/peps/pep-0487
[45]:	https://www.python.org/dev/peps/pep-0487
[46]:	https://docs.python.org/3.6/library/stdtypes.html#str "str"
[47]:	https://docs.python.org/3.6/library/functions.html#bytes "bytes"
[48]:	https://docs.python.org/3.6/library/functions.html#int "int"
[49]:	https://docs.python.org/3.6/library/stdtypes.html#str "str"
[50]:	https://docs.python.org/3.6/library/functions.html#bytes "bytes"
[51]:	https://docs.python.org/3.6/library/stdtypes.html#str "str"
[52]:	https://docs.python.org/3.6/library/functions.html#bytes "bytes"
[53]:	https://docs.python.org/3.6/library/functions.html#open "open"
[54]:	https://docs.python.org/3.6/library/os.html#module-os "os:

Miscellaneous operating system interfaces."
[55]:	https://www.python.org/dev/peps/pep-0519
[56]:	https://www.python.org/dev/peps/pep-0495
[57]:	https://www.python.org/dev/peps/pep-0495
[58]:	https://www.python.org/dev/peps/pep-0529
[59]:	https://www.python.org/dev/peps/pep-0528
[60]:	https://www.python.org/dev/peps/pep-0520
[61]:	https://www.python.org/dev/peps/pep-0468
[62]:	https://docs.python.org/3.6/library/stdtypes.html#typesmapping
[63]:	https://docs.python.org/3.6/library/stdtypes.html#typesmapping
[64]:	https://docs.python.org/3.6/library/stdtypes.html#dict "dict"
[65]:	https://www.python.org/dev/peps/pep-0523
[66]:	https://www.python.org/dev/peps/pep-0523
[67]:	https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc"
[68]:	https://docs.python.org/3.6/glossary.html#term-global-interpreter-lock
[69]:	https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_OBJ "PYMEM_DOMAIN_OBJ"
[70]:	https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_MEM "PYMEM_DOMAIN_MEM"
[71]:	https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc"
[72]:	https://docs.python.org/3.6/reference/datamodel.html#specialnames
[73]:	https://docs.python.org/3.6/reference/datamodel.html#object.__iter__ "object.__iter__"
[74]:	https://bugs.python.org/issue25958
[75]:	https://bugs.python.org/issue26823
[76]:	https://docs.python.org/3.6/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError"
[77]:	https://docs.python.org/3.6/library/exceptions.html#ImportError "ImportError"
[78]:	https://bugs.python.org/issue15767
[79]:	https://bugs.python.org/issue23722
[80]:	https://www.python.org/dev/peps/pep-0506
[81]:	https://docs.python.org/3.6/library/asyncio-eventloops.html#asyncio.get_event_loop "asyncio.get_event_loop"
[82]:	https://bugs.python.org/issue28613
[83]:	https://docs.python.org/3.6/library/asyncio-task.html#asyncio.ensure_future "asyncio.ensure_future"
[84]:	https://docs.python.org/3.6/glossary.html#term-awaitable
[85]:	https://docs.python.org/3.6/library/asyncio-task.html#asyncio.run_coroutine_threadsafe "asyncio.run_coroutine_threadsafe"
[86]:	https://docs.python.org/3.6/library/asyncio-protocol.html#asyncio.BaseTransport.is_closing "asyncio.BaseTransport.is_closing"
[87]:	https://github.com/MagicStack/uvloop
[88]:	https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Future "asyncio.Future"
[89]:	https://bugs.python.org/issue27041
[90]:	https://bugs.python.org/issue27040
[91]:	https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.StreamReader.readuntil "asyncio.StreamReader.readuntil"
[92]:	https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.StreamReader.readexactly "asyncio.StreamReader.readexactly"
[93]:	https://bugs.python.org/issue28370
[94]:	https://bugs.python.org/issue25593
[95]:	https://docs.python.org/3.6/library/exceptions.html#TypeError "TypeError"
[96]:	https://docs.python.org/3.6/library/exceptions.html#StopIteration "StopIteration"
[97]:	https://bugs.python.org/issue26221
[98]:	https://docs.python.org/3.6/library/asyncio-eventloop.html#asyncio.BaseEventLoop.connect_accepted_socket "asyncio.BaseEventLoop.connect_accepted_socket"
[99]:	https://bugs.python.org/issue27392
[100]:	https://bugs.python.org/issue27456
[101]:	https://docs.python.org/3.6/library/asyncio-eventloop.html#asyncio.AbstractEventLoop.shutdown_asyncgens "asyncio.AbstractEventLoop.shutdown_asyncgens"
[102]:	https://bugs.python.org/issue28003
[103]:	https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Future "asyncio.Future"
[104]:	https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Task "asyncio.Task"
[105]:	https://bugs.python.org/issue26081
[106]:	https://bugs.python.org/issue28544
[107]:	https://bugs.python.org/issue12345
[108]:	https://bugs.python.org/issue23229
[109]:	https://bugs.python.org/issue28720
[110]:	https://bugs.python.org/issue17941
[111]:	https://bugs.python.org/issue25628
[112]:	https://bugs.python.org/issue27664
[113]:	https://bugs.python.org/issue24277
[114]:	https://docs.python.org/3.6/library/enum.html#module-enum "enum:

Implementation of an enumeration class."
[115]:	https://docs.python.org/3.6/library/enum.html#enum.Flag "enum.Flag"
[116]:	https://bugs.python.org/issue26470
[117]:	https://bugs.python.org/issue26798
[118]:	https://bugs.python.org/issue27928
[119]:	https://bugs.python.org/issue12319
[120]:	https://bugs.python.org/issue24225
[121]:	https://bugs.python.org/issue15767
[122]:	https://bugs.python.org/issue17909
[123]:	https://bugs.python.org/issue24884
[124]:	https://docs.python.org/3.6/library/math.html#module-math "math:

Mathematical functions \(sin\(\) etc.\)."
[125]:	https://docs.python.org/3.6/library/cmath.html#module-cmath "cmath:

Mathematical functions for complex numbers."
[126]:	https://bugs.python.org/issue12345
[127]:	https://docs.python.org/3.6/library/os.html#module-os "os:

Miscellaneous operating system interfaces."
[128]:	https://docs.python.org/3.6/library/functions.html#bytes "bytes"
[129]:	https://docs.python.org/3.6/library/pdb.html#pdb.Pdb "pdb.Pdb"
[130]:	https://bugs.python.org/issue8637
[131]:	https://docs.python.org/3.6/library/functions.html#help "help"
[132]:	https://bugs.python.org/issue18844
[133]:	https://bugs.python.org/issue433028
[134]:	https://bugs.python.org/issue26870
[135]:	https://bugs.python.org/issue26536
[136]:	https://bugs.python.org/issue26907
[137]:	https://bugs.python.org/issue27744
[138]:	https://bugs.python.org/issue19500
[139]:	https://bugs.python.org/issue11734
[140]:	https://bugs.python.org/issue27932
[141]:	https://bugs.python.org/issue25485
[142]:	https://bugs.python.org/issue6422
[143]:	https://bugs.python.org/issue23552
[144]:	https://bugs.python.org/issue22115
[145]:	https://docs.python.org/3.6/glossary.html#term-provisional-api
[146]:	https://bugs.python.org/issue27598
[147]:	https://github.com/python/typing/issues/230
[148]:	http://unicode.org/versions/Unicode9.0.0/
[149]:	https://docs.python.org/3.6/library/unittest.mock.html#unittest.mock.Mock.assert_called "unittest.mock.Mock.assert_called"
[150]:	https://docs.python.org/3.6/library/unittest.mock.html#unittest.mock.Mock.assert_called_once "unittest.mock.Mock.assert_called_once"
[151]:	https://bugs.python.org/issue26323
[152]:	https://docs.python.org/3.6/library/unittest.mock.html#unittest.mock.Mock.reset_mock "unittest.mock.Mock.reset_mock"
[153]:	https://bugs.python.org/issue21271
[154]:	https://bugs.python.org/issue16099
[155]:	https://docs.python.org/3.6/library/venv.html#module-venv "venv:

Creation of virtual environments."
[156]:	https://bugs.python.org/issue27982
[157]:	https://bugs.python.org/issue26885
[158]:	https://bugs.python.org/issue26039
[159]:	https://bugs.python.org/issue26039
[160]:	https://bugs.python.org/issue26243
[161]:	https://bugs.python.org/issue16764
[162]:	https://bugs.python.org/issue26647
[163]:	https://bugs.python.org/issue28050
[164]:	https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Future "asyncio.Future"
[165]:	https://bugs.python.org/issue26081
[166]:	https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Task "asyncio.Task"
[167]:	https://bugs.python.org/issue28544
[168]:	https://docs.python.org/3.6/library/typing.html#module-typing "typing: Support for type hints \(see PEP 484\)."
[169]:	https://bugs.python.org/issue24870
[170]:	https://bugs.python.org/issue25227
[171]:	https://bugs.python.org/issue25267
[172]:	https://bugs.python.org/issue25301
[173]:	https://bugs.python.org/issue25349
[174]:	https://bugs.python.org/issue25399
[175]:	https://docs.python.org/3.6/library/stdtypes.html#bytes.fromhex "bytes.fromhex"
[176]:	https://docs.python.org/3.6/library/stdtypes.html#bytearray.fromhex "bytearray.fromhex"
[177]:	https://bugs.python.org/issue25401
[178]:	https://bugs.python.org/issue26574
[179]:	https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc"
[180]:	https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_MEM "PYMEM_DOMAIN_MEM"
[181]:	https://docs.python.org/3.6/c-api/memory.html#pymalloc
[182]:	https://bugs.python.org/issue26249
[183]:	https://docs.python.org/3.6/library/pickle.html#pickle.load "pickle.load"
[184]:	https://docs.python.org/3.6/library/pickle.html#pickle.loads "pickle.loads"
[185]:	https://bugs.python.org/issue27056
[186]:	https://docs.python.org/3.6/glossary.html#term-keyword-argument
[187]:	https://docs.python.org/3.6/glossary.html#term-positional-argument
[188]:	https://bugs.python.org/issue27574
[189]:	https://docs.python.org/3.6/library/glob.html#glob.glob "glob.glob"
[190]:	https://docs.python.org/3.6/library/glob.html#glob.iglob "glob.iglob"
[191]:	https://docs.python.org/3.6/library/glob.html#module-glob "glob: Unix shell style pathname pattern expansion."
[192]:	https://bugs.python.org/issue25596
[193]:	https://docs.python.org/3.6/library/pathlib.html#module-pathlib "pathlib: Object-oriented filesystem paths"
[194]:	https://docs.python.org/3.6/library/os.html#os.scandir "os.scandir"
[195]:	https://bugs.python.org/issue26032
[196]:	https://docs.python.org/3.6/library/xml.etree.elementtree.html#module-xml.etree.ElementTree "xml.etree.ElementTree: Implementation of the ElementTree API."
[197]:	https://bugs.python.org/issue25638
[198]:	https://bugs.python.org/issue25873
[199]:	https://bugs.python.org/issue25869
[200]:	https://docs.python.org/3.6/library/fractions.html#fractions.Fraction "fractions.Fraction"
[201]:	https://bugs.python.org/issue25971
[202]:	https://www.python.org/dev/peps/pep-0007
[203]:	https://bugs.python.org/issue17884
[204]:	https://bugs.python.org/issue26865
[205]:	https://bugs.python.org/issue26539
[206]:	https://docs.python.org/3.6/glossary.html#term-global-interpreter-lock
[207]:	https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_OBJ "PYMEM_DOMAIN_OBJ"
[208]:	https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_MEM "PYMEM_DOMAIN_MEM"
[209]:	https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc"
[210]:	https://docs.python.org/3.6/c-api/init.html#c.Py_FinalizeEx "Py_FinalizeEx"
[211]:	https://bugs.python.org/issue5319
[212]:	https://docs.python.org/3.6/c-api/arg.html#c.PyArg_ParseTupleAndKeywords "PyArg_ParseTupleAndKeywords"
[213]:	https://docs.python.org/3.6/glossary.html#positional-only-parameter
[214]:	https://bugs.python.org/issue26282
[215]:	https://bugs.python.org/issue26823
[216]:	https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_SetImportErrorSubclass "PyErr_SetImportErrorSubclass"
[217]:	https://docs.python.org/3.6/library/exceptions.html#ImportError "ImportError"
[218]:	https://bugs.python.org/issue15767
[219]:	https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_ResourceWarning "PyErr_ResourceWarning"
[220]:	https://docs.python.org/3.6/library/exceptions.html#ResourceWarning "ResourceWarning"
[221]:	https://bugs.python.org/issue26567
[222]:	https://docs.python.org/3.6/c-api/sys.html#c.PyOS_FSPath "PyOS_FSPath"
[223]:	https://docs.python.org/3.6/glossary.html#term-path-like-object
[224]:	https://bugs.python.org/issue27186
[225]:	https://docs.python.org/3.6/c-api/unicode.html#c.PyUnicode_FSConverter "PyUnicode_FSConverter"
[226]:	https://docs.python.org/3.6/c-api/unicode.html#c.PyUnicode_FSDecoder "PyUnicode_FSDecoder"
[227]:	https://docs.python.org/3.6/glossary.html#term-path-like-object
[228]:	https://docs.python.org/3.6/using/cmdline.html#cmdoption--version
[229]:	https://docs.python.org/3.6/using/cmdline.html#cmdoption-V
[230]:	https://docs.python.org/3.6/library/sys.html#sys.version "sys.version"
[231]:	https://bugs.python.org/issue27364
[232]:	https://bugs.python.org/issue25002
[233]:	https://bugs.python.org/issue25002
[234]:	https://bugs.python.org/issue26129
[235]:	https://docs.python.org/3.6/library/os.html#module-os "os:

Miscellaneous operating system interfaces."
[236]:	https://bugs.python.org/issue25791
[237]:	https://bugs.python.org/issue22493
[238]:	https://docs.python.org/3.6/library/ssl.html#module-ssl "ssl: TLS/SSL

wrapper for socket objects"
[239]:	https://bugs.python.org/issue28022
[240]:	https://docs.python.org/3.6/library/re.html#re.sub "re.sub"
[241]:	https://docs.python.org/3.6/library/re.html#re.LOCALE "re.LOCALE"
[242]:	https://docs.python.org/3.6/library/inspect.html#inspect.getmodulename "inspect.getmodulename"
[243]:	https://bugs.python.org/issue13248
[244]:	https://docs.python.org/3.6/library/traceback.html#module-traceback "traceback: Print or retrieve a stack traceback."
[245]:	https://docs.python.org/3.6/library/tkinter.html#module-tkinter "tkinter: Interface to Tcl/Tk for graphical user interfaces"
[246]:	https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile.open "zipfile.ZipFile.open"
[247]:	https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile "zipfile.ZipFile"
[248]:	https://docs.python.org/3.6/library/io.html#io.TextIOWrapper "io.TextIOWrapper"
[249]:	https://docs.python.org/3.6/glossary.html#term-universal-newlines
[250]:	https://hg.python.org/cpython/file/3.6/Tools/scripts/h2py.py
[251]:	https://bugs.python.org/issue23034
[252]:	https://docs.python.org/3.6/library/functions.html#open "open"
[253]:	https://bugs.python.org/issue2091
[254]:	https://docs.python.org/3.6/library/sqlite3.html#module-sqlite3 "sqlite3: A DB-API 2.0 implementation using SQLite 3.x."
[255]:	https://docs.python.org/3.6/library/os.html#os.urandom "os.urandom"
[256]:	https://docs.python.org/3.6/library/importlib.html#importlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module"
[257]:	https://docs.python.org/3.6/library/importlib.html#importlib.abc.Loader.create_module "importlib.abc.Loader.create_module"
[258]:	https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_SetImportError "PyErr_SetImportError"
[259]:	https://docs.python.org/3.6/library/exceptions.html#TypeError "TypeError"
[260]:	https://www.python.org/dev/peps/pep-0511
[261]:	https://docs.python.org/3.6/library/compileall.html#module-compileall "compileall: Tools for byte-compiling all Python source files in a directory tree."
[262]:	https://bugs.python.org/issue25768
[263]:	https://docs.python.org/3.6/library/urllib.parse.html#urllib.parse.urlsplit "urllib.parse.urlsplit"
[264]:	https://docs.python.org/3.6/library/urllib.parse.html#urllib.parse.urlparse "urllib.parse.urlparse"
[265]:	https://docs.python.org/3.6/library/exceptions.html#ValueError "ValueError"
[266]:	https://docs.python.org/3.6/library/constants.html#None "None"
[267]:	https://bugs.python.org/issue20059
[268]:	https://docs.python.org/3.6/library/imp.html#module-imp "imp: Access the implementation of the import statement. \(deprecated\)"
[269]:	https://docs.python.org/3.6/library/exceptions.html#DeprecationWarning "DeprecationWarning"
[270]:	https://docs.python.org/3.6/library/exceptions.html#PendingDeprecationWarning "PendingDeprecationWarning"
[271]:	https://docs.python.org/3.6/library/calendar.html#module-calendar "calendar: Functions for working with calendars, including some emulation of the Unix cal program."
[272]:	https://docs.python.org/3.6/library/cgi.html#module-cgi "cgi: Helpers for running Python scripts via the Common Gateway Interface."
[273]:	https://docs.python.org/3.6/library/csv.html#module-csv "csv: Write and read tabular data to and from delimited files."
[274]:	https://docs.python.org/3.6/library/xml.etree.elementtree.html#module-xml.etree.ElementTree "xml.etree.ElementTree: Implementation of the ElementTree API."
[275]:	https://docs.python.org/3.6/library/enum.html#module-enum "enum: Implementation of an enumeration class."
[276]:	https://docs.python.org/3.6/library/fileinput.html#module-fileinput "fileinput: Loop over standard input or a list of files."
[277]:	https://docs.python.org/3.6/library/ftplib.html#module-ftplib "ftplib: FTP protocol client \(requires sockets\)."
[278]:	https://docs.python.org/3.6/library/logging.html#module-logging "logging: Flexible event logging system for applications."
[279]:	https://docs.python.org/3.6/library/mailbox.html#module-mailbox "mailbox: Manipulate mailboxes in various formats"
[280]:	https://docs.python.org/3.6/library/mimetypes.html#module-mimetypes "mimetypes: Mapping of filename extensions to MIME types."
[281]:	https://docs.python.org/3.6/library/optparse.html#module-optparse "optparse: Command-line option parsing library. \(deprecated\)"
[282]:	https://docs.python.org/3.6/library/plistlib.html#module-plistlib "plistlib: Generate and parse Mac OS X plist files."
[283]:	https://docs.python.org/3.6/library/smtpd.html#module-smtpd "smtpd: A SMTP server implementation in Python."
[284]:	https://docs.python.org/3.6/library/subprocess.html#module-subprocess "subprocess: Subprocess management."
[285]:	https://docs.python.org/3.6/library/tarfile.html#module-tarfile "tarfile: Read and write tar-format archive files."
[286]:	https://docs.python.org/3.6/library/threading.html#module-threading "threading: Thread-based parallelism."
[287]:	https://docs.python.org/3.6/library/wave.html#module-wave "wave: Provide an interface to the WAV sound format."
[288]:	https://bugs.python.org/issue23883
[289]:	https://docs.python.org/3.6/library/exceptions.html#ImportWarning "ImportWarning"
[290]:	https://bugs.python.org/issue25791
[291]:	https://docs.python.org/3.6/library/exceptions.html#ImportError "ImportError"
[292]:	https://docs.python.org/3.6/library/exceptions.html#SystemError "SystemError"
[293]:	https://bugs.python.org/issue18018
[294]:	https://docs.python.org/3.6/library/socketserver.html#module-socketserver "socketserver: A framework for network servers."
[295]:	https://docs.python.org/3.6/library/http.server.html#module-http.server "http.server: HTTP server and request handlers."
[296]:	https://docs.python.org/3.6/library/xmlrpc.server.html#module-xmlrpc.server "xmlrpc.server: Basic XML-RPC server implementations."
[297]:	https://docs.python.org/3.6/library/wsgiref.html#module-wsgiref.simple_server "wsgiref.simple_server: A simple WSGI HTTP server."
[298]:	https://docs.python.org/3.6/library/exceptions.html#Exception "Exception"
[299]:	https://docs.python.org/3.6/library/exceptions.html#SystemExit "SystemExit"
[300]:	https://docs.python.org/3.6/library/exceptions.html#KeyboardInterrupt "KeyboardInterrupt"
[301]:	https://docs.python.org/3.6/library/socketserver.html#socketserver.BaseServer.handle_error "socketserver.BaseServer.handle_error"
[302]:	https://bugs.python.org/issue23430
[303]:	https://docs.python.org/3.6/library/spwd.html#spwd.getspnam "spwd.getspnam"
[304]:	https://docs.python.org/3.6/library/exceptions.html#PermissionError "PermissionError"
[305]:	https://docs.python.org/3.6/library/exceptions.html#KeyError "KeyError"
[306]:	https://docs.python.org/3.6/library/socket.html#socket.socket.close "socket.socket.close"
[307]:	https://bugs.python.org/issue26685
[308]:	https://docs.python.org/3.6/library/smtpd.html#smtpd.SMTPChannel "smtpd.SMTPChannel"
[309]:	https://docs.python.org/3.6/library/smtpd.html#smtpd.SMTPServer "smtpd.SMTPServer"
[310]:	https://docs.python.org/3.6/library/smtpd.html#smtpd.SMTPServer.process_message "smtpd.SMTPServer.process_message"
[311]:	https://docs.python.org/3.6/library/json.html#json.dump "json.dump"
[312]:	https://docs.python.org/3.6/library/json.html#json.dumps "json.dumps"
[313]:	https://docs.python.org/3.6/library/json.html#json.load "json.load"
[314]:	https://docs.python.org/3.6/library/json.html#json.loads "json.loads"
[315]:	https://docs.python.org/3.6/library/json.html#json.JSONEncoder "json.JSONEncoder"
[316]:	https://docs.python.org/3.6/library/json.html#json.JSONDecoder "json.JSONDecoder"
[317]:	https://docs.python.org/3.6/library/json.html#module-json "json: Encode and decode the JSON format."
[318]:	https://docs.python.org/3.6/glossary.html#keyword-only-parameter
[319]:	https://bugs.python.org/issue18726
[320]:	https://docs.python.org/3.6/library/functions.html#type "type"
[321]:	https://www.python.org/dev/peps/pep-0487
[322]:	https://docs.python.org/3.6/library/functions.html#type "type"
[323]:	https://docs.python.org/3.6/reference/datamodel.html#object.__init_subclass__ "object.__init_subclass__"
[324]:	https://docs.python.org/3.6/reference/datamodel.html#object.__init_subclass__ "object.__init_subclass__"
[325]:	https://docs.python.org/3.6/library/functions.html#super "super"
[326]:	https://docs.python.org/3.6/library/urllib.request.html#module-urllib.request "urllib.request: Extensible library for opening URLs."
[327]:	https://docs.python.org/3.6/library/http.client.html#http.client.HTTPConnection.request "http.client.HTTPConnection.request"
[328]:	https://bugs.python.org/issue12319
[329]:	https://docs.python.org/3.6/library/csv.html#csv.DictReader "csv.DictReader"
[330]:	https://docs.python.org/3.6/library/collections.html#collections.OrderedDict "collections.OrderedDict"
[331]:	https://bugs.python.org/issue27842
[332]:	https://docs.python.org/3.6/library/crypt.html#crypt.METHOD_CRYPT "crypt.METHOD_CRYPT"
[333]:	https://bugs.python.org/issue25287
[334]:	https://docs.python.org/3.6/library/collections.html#collections.namedtuple "collections.namedtuple"
[335]:	https://bugs.python.org/issue25628
[336]:	https://docs.python.org/3.6/library/ctypes.html#ctypes.util.find_library "ctypes.util.find_library"
[337]:	https://bugs.python.org/issue9998
[338]:	https://docs.python.org/3.6/library/imaplib.html#imaplib.IMAP4 "imaplib.IMAP4"
[339]:	https://bugs.python.org/issue21815
[340]:	https://bugs.python.org/issue26335
[341]:	https://docs.python.org/3.6/library/pkgutil.html#pkgutil.iter_modules "pkgutil.iter_modules"
[342]:	https://docs.python.org/3.6/library/pkgutil.html#pkgutil.walk_packages "pkgutil.walk_packages"
[343]:	https://docs.python.org/3.6/library/pkgutil.html#pkgutil.ModuleInfo "pkgutil.ModuleInfo"
[344]:	https://bugs.python.org/issue17211
[345]:	https://docs.python.org/3.6/library/re.html#re.sub "re.sub"
[346]:	https://bugs.python.org/issue25953
[347]:	https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile "zipfile.ZipFile"
[348]:	https://docs.python.org/3.6/library/exceptions.html#NotImplementedError "NotImplementedError"
[349]:	https://docs.python.org/3.6/library/exceptions.html#RuntimeError "RuntimeError"
[350]:	https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile "zipfile.ZipFile"
[351]:	https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile.write "zipfile.ZipFile.write"
[352]:	https://docs.python.org/3.6/library/exceptions.html#ValueError "ValueError"
[353]:	https://docs.python.org/3.6/library/exceptions.html#RuntimeError "RuntimeError"
[354]:	https://docs.python.org/3.6/library/functions.html#super "super"
[355]:	https://docs.python.org/3.6/library/exceptions.html#DeprecationWarning "DeprecationWarning"
[356]:	https://docs.python.org/3.6/library/exceptions.html#RuntimeWarning "RuntimeWarning"
[357]:	https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc"
[358]:	https://docs.python.org/3.6/c-api/memory.html#pymalloc
[359]:	https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc"
[360]:	https://docs.python.org/3.6/using/cmdline.html#envvar-PYTHONMALLOC
[361]:	https://bugs.python.org/issue26249
[362]:	https://docs.python.org/3.6/c-api/sys.html#c.Py_Exit "Py_Exit"
[363]:	https://bugs.python.org/issue5319
[364]:	https://docs.python.org/3.6/glossary.html#term-bytecode
[365]:	https://bugs.python.org/issue26647
[366]:	https://bugs.python.org/issue28050
[367]:	https://docs.python.org/3.6/library/dis.html#opcode-FORMAT_VALUE
[368]:	https://docs.python.org/3.6/library/dis.html#opcode-BUILD_STRING
[369]:	https://bugs.python.org/issue25483
[370]:	https://bugs.python.org/issue27078
[371]:	https://docs.python.org/3.6/library/dis.html#opcode-BUILD_CONST_KEY_MAP
[372]:	https://bugs.python.org/issue27140
[373]:	https://docs.python.org/3.6/library/dis.html#opcode-MAKE_FUNCTION
[374]:	https://docs.python.org/3.6/library/dis.html#opcode-CALL_FUNCTION
[375]:	https://docs.python.org/3.6/library/dis.html#opcode-CALL_FUNCTION_KW
[376]:	https://bugs.python.org/issue27095
[377]:	https://bugs.python.org/issue27213
[378]:	https://bugs.python.org/issue28257
[379]:	https://docs.python.org/3.6/library/dis.html#opcode-SETUP_ANNOTATIONS
[380]:	https://docs.python.org/3.6/library/dis.html#opcode-STORE_ANNOTATION
[381]:	https://docs.python.org/3.6/glossary.html#term-variable-annotation
[382]:	https://bugs.python.org/issue27985
[383]:	https://docs.python.org/3.6/copyright.html
[384]:	http://sphinx.pocoo.org/
