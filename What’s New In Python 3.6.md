原文：[What’s New In Python 3.6](https://docs.python.org/3.6/whatsnew/3.6.html)

# What's New In Python 3.6¶

Release:| 3.6.0  
---|---  
Date:| December 15, 2016  
Editors:| Elvis Pranskevichus
&lt;[elvis@magic.io](mailto:elvis%40magic.io)&gt;, Yury Selivanov
&lt;[yury@magic.io](mailto:yury%40magic.io)&gt;  
  
This article explains the new features in Python 3.6, compared to 3.5.

See also

[**PEP 494**](https://www.python.org/dev/peps/pep-0494) \- Python 3.6 Release
Schedule

## 摘要 - 发布亮点¶

新的语法特性：

  * PEP 498, 格式化字符串字面量
  * PEP 515, 数字字面量中的下划线
  * PEP 526, 变量注解中的语法
  * PEP 525, 异步生成器
  * PEP 520: 异步解析式

新的库模块

  * [`secrets`](https://docs.python.org/3.6/library/secrets.html#module-secrets "secrets: Generate secure random numbers for managing secrets." ): PEP 506 - 在标准库中添加了Secrets模块

CPython实现的改进：

  * 重新实现了[字典(dict)](https://docs.python.org/3.6/library/stdtypes.html#typesmapping)类型，以便能像[PyPy的字典类型](https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-more.html)一样使用更紧凑的表达方式。与Python 3.5相比，这使字典的内存用量减少了20%到25%。
  * 用新协定优化了类的自定义建立。
  * 类属性定义顺序(class attribute definition order)现在被保留了
  * **kwargs内的元素顺序现在对应于将关键字(保留字）参数传递给函数的顺序
  * 新增了对DTrace和SystemTap probing的支持。
  * 新PYTHONMALLOC环境变量现在可用于调试解释器内存分配与访问错误。

标准库的重大改进：

  * 为[asyncio](https://docs.python.org/3.6/library/asyncio.html#module-asyncio "asyncio: Asynchronous I/O, event loop, coroutines and tasks." )模块开发了新功能、显著的可用性、性能优化，以及大量的错误修复。 从Python 3.6开始，asyncio模块不再是临时的了，其API也进入了稳定状态。
  * 实现了用于支持[类路径对象(path-like objects)](https://docs.python.org/3.6/glossary.html#term-path-like-object)的新文件系统路径协议。 所有在路径(path)上使用的标准库函数都已更新，以便适应于新协议。
  * [datetime](https://docs.python.org/3.6/library/datetime.html#module-datetime "datetime: Basic date and time types." )模块已获得对本地时间消歧(Local Time Disambiguation)的支持。
  * 针对[typing](https://docs.python.org/3.6/library/typing.html#module-typing "typing: Support for type hints \(see PEP 484\)." )模块的一些改进，使其不再是临时模块。
  * The [`tracemalloc`](https://docs.python.org/3.6/library/tracemalloc.html#module-tracemalloc "tracemalloc: Trace memory allocations." ) module has been significantly reworked and is now used to provide better output for [`ResourceWarning`](https://docs.python.org/3.6/library/exceptions.html#ResourceWarning "ResourceWarning" ) as well as provide better diagnostics for memory allocation errors. See the PYTHONMALLOC section for more information.

Security improvements:

  * The new [`secrets`](https://docs.python.org/3.6/library/secrets.html#module-secrets "secrets: Generate secure random numbers for managing secrets." ) module has been added to simplify the generation of cryptographically strong pseudo-random numbers suitable for managing secrets such as account authentication, tokens, and similar.
  * On Linux, [`os.urandom()`](https://docs.python.org/3.6/library/os.html#os.urandom "os.urandom" ) now blocks until the system urandom entropy pool is initialized to increase the security. See the [**PEP 524**](https://www.python.org/dev/peps/pep-0524) for the rationale.
  * The [`hashlib`](https://docs.python.org/3.6/library/hashlib-blake2.html#module-hashlib "hashlib: BLAKE2 hash function for Python" ) and [`ssl`](https://docs.python.org/3.6/library/ssl.html#module-ssl "ssl: TLS/SSL wrapper for socket objects" ) modules now support OpenSSL 1.1.0.
  * The default settings and feature set of the [`ssl`](https://docs.python.org/3.6/library/ssl.html#module-ssl "ssl: TLS/SSL wrapper for socket objects" ) module have been improved.
  * The [`hashlib`](https://docs.python.org/3.6/library/hashlib-blake2.html#module-hashlib "hashlib: BLAKE2 hash function for Python" ) module received support for the BLAKE2, SHA-3 and SHAKE hash algorithms and the [`scrypt()`](https://docs.python.org/3.6/library/hashlib.html#hashlib.scrypt "hashlib.scrypt" ) key derivation function.

Windows improvements:

  * PEP 528 and PEP 529, Windows filesystem and console encoding changed to UTF-8.
  * The `py.exe` launcher, when used interactively, no longer prefers Python 2 over Python 3 when the user doesn't specify a version (via command line arguments or a config file). Handling of shebang lines remains unchanged - "python" refers to Python 2 in that case.
  * `python.exe` and `pythonw.exe` have been marked as long-path aware, which means that the 260 character path limit may no longer apply. See [removing the MAX_PATH limitation](https://docs.python.org/3.6/using/windows.html#max-path) for details.
  * A `._pth` file can be added to force isolated mode and fully specify all search paths to avoid registry and environment lookup. See [the documentation](https://docs.python.org/3.6/using/windows.html#finding-modules) for more information.
  * A `python36.zip` file now works as a landmark to infer [`PYTHONHOME`](https://docs.python.org/3.6/using/cmdline.html#envvar-PYTHONHOME). See [the documentation](https://docs.python.org/3.6/using/windows.html#finding-modules) for more information.

## 新特性

### PEP 498: 格式化字符串

[**PEP 498**](https://www.python.org/dev/peps/pep-0498)引入了一种新的字符串：_f-strings_, 或者[格式化字符串](https://docs.python.org/3.6/reference/lexical_analysis.html#f-strings)。

格式化字符串带`'f'`前缀，类似于[`str.format()`](https://docs.python.org/3.6/library/stdtypes.html#str.format "str.format" )接受的格式字符串。它们包含了由花括号括起来的替换字段。替换字段是表达式，它们会在运行时计算，然后使用[`format()`](https://docs.python.org/3.6/library/functions.html#format "format" )协议进行格式化：

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

[**PEP 498**](https://www.python.org/dev/peps/pep-0498) - 字符串插值。

    PEP由Eric V. Smith编写和实现。

[特性文档](https://docs.python.org/3.6/reference/lexical_analysis.html#f-strings)。

### PEP 526: 变量注释语法

[**PEP 484**](https://www.python.org/dev/peps/pep-0484)引入了函数参数的类型注释的标准，又名类型提示。这个PEP添加了用来注释变量（包括类变量和实例变量）类型的语法：

```

    primes: List[int] = []
    
    captain: str  # Note: no initial value!
    
    class Starship:
        stats: Dict[str, int] = {}
    
```

正如函数注释，Python解释器不附加任何特殊意义到变量注释上，只是将它们存储在一个类或者模块的`__annotations__`属性中。

与静态类型语言中的变量声明相比，注释语法的目的在于提供一种简单的方式，通过抽象语法树和`__annotations__`属性，来为第三方工具和库指定结构化类型元数据。

又见

[**PEP 526**](https://www.python.org/dev/peps/pep-0526) - 变量注释语法。

    PEP由Ryan Gonzalez, Philip House, Ivan Levkivskyi, Lisa Roach, 和Guido van Rossum编写。由Ivan Levkivskyi实现。

使用或将要使用这个新语法的工具：[mypy](http://github.com/python/mypy), [pytype](http://github.com/google/pytype), PyCharm等等。

### PEP 515: 数值文字中的下划线

[**PEP 515**](https://www.python.org/dev/peps/pep-0515)添加了在数值文字中使用下划线的能力，以提高可读性。例如：

```

    >>> 1_000_000_000_000_000
    1000000000000000
    >>> 0x_FF_FF_FF_FF
    4294967295
    
```

数字之间和任何基本符号之后允许单个下划线。不允许前置、后置或者多个连续的下划线。

[字符串格式化](https://docs.python.org/3.6/library/string.html#formatspec)语言现在还支持`'_'`选项，该选项用来通知对浮点表示类型和整型表示类型`'d'`，会把下划线当成千位分隔符使用。对于整型表示类型`'b'`, `'o'`, `'x'`, 和`'X'`, 下划线将会被插入到每4个数字之间：

```

    >>> '{:_}'.format(1000000)
    '1_000_000'
    >>> '{:_x}'.format(0xFFFFFFFF)
    'ffff_ffff'
    
```

又见

[**PEP 515**](https://www.python.org/dev/peps/pep-0515) - 数值文字中的下划线

    PEP由Georg Brandl和Serhiy Storchaka编写。

### PEP 525: Asynchronous Generators¶

[**PEP 492**](https://www.python.org/dev/peps/pep-0492) introduced support for
native coroutines and `async` / `await` syntax to Python 3.5. A notable
limitation of the Python 3.5 implementation is that it was not possible to use
`await` and `yield` in the same function body. In Python 3.6 this restriction
has been lifted, making it possible to define _asynchronous generators_:

```

    async def ticker(delay, to):
        """Yield numbers from 0 to *to* every *delay* seconds."""
        for i in range(to):
            yield i
            await asyncio.sleep(delay)
    
```

The new syntax allows for faster and more concise code.

See also

[**PEP 525**](https://www.python.org/dev/peps/pep-0525) - Asynchronous
Generators

    PEP written and implemented by Yury Selivanov.

### PEP 530: Asynchronous Comprehensions¶

[**PEP 530**](https://www.python.org/dev/peps/pep-0530) adds support for using
`async for` in list, set, dict comprehensions and generator expressions:

```

    result = [i async for i in aiter() if i % 2]
    
```

Additionally, `await` expressions are supported in all kinds of
comprehensions:

```

    result = [await fun() for fun in funcs if await condition()]
    
```

See also

[**PEP 530**](https://www.python.org/dev/peps/pep-0530) - Asynchronous
Comprehensions

    PEP written and implemented by Yury Selivanov.

### PEP 487: Simpler customization of class creation¶

It is now possible to customize subclass creation without using a metaclass.
The new `__init_subclass__` classmethod will be called on the base class
whenever a new subclass is created:

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

In order to allow zero-argument
[`super()`](https://docs.python.org/3.6/library/functions.html#super "super" )
calls to work correctly from [`__init_subclass__()`](https://docs.python.org/3
.6/reference/datamodel.html#object.__init_subclass__
"object.__init_subclass__" ) implementations, custom metaclasses must ensure
that the new `__classcell__` namespace entry is propagated to `type.__new__`
(as described in [Creating the class
object](https://docs.python.org/3.6/reference/datamodel.html#class-object-
creation)).

See also

[**PEP 487**](https://www.python.org/dev/peps/pep-0487) - Simpler
customization of class creation

    PEP written and implemented by Martin Teichmann.

[Feature documentation](https://docs.python.org/3.6/reference/datamodel.html
#class-customization)

### PEP 487: Descriptor Protocol Enhancements¶

[**PEP 487**](https://www.python.org/dev/peps/pep-0487) extends the descriptor
protocol has to include the new optional [`__set_name__()`](https://docs.pytho
n.org/3.6/reference/datamodel.html#object.__set_name__ "object.__set_name__" )
method. Whenever a new class is defined, the new method will be called on all
descriptors included in the definition, providing them with a reference to the
class being defined and the name given to the descriptor within the class
namespace. In other words, instances of descriptors can now know the attribute
name of the descriptor in the owner class:

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

See also

[**PEP 487**](https://www.python.org/dev/peps/pep-0487) - Simpler
customization of class creation

    PEP written and implemented by Martin Teichmann.

[Feature documentation](https://docs.python.org/3.6/reference/datamodel.html#d
escriptors)

### PEP 519: Adding a file system path protocol¶

File system paths have historically been represented as
[`str`](https://docs.python.org/3.6/library/stdtypes.html#str "str" ) or
[`bytes`](https://docs.python.org/3.6/library/functions.html#bytes "bytes" )
objects. This has led to people who write code which operate on file system
paths to assume that such objects are only one of those two types (an
[`int`](https://docs.python.org/3.6/library/functions.html#int "int" )
representing a file descriptor does not count as that is not a file path).
Unfortunately that assumption prevents alternative object representations of
file system paths like
[`pathlib`](https://docs.python.org/3.6/library/pathlib.html#module-pathlib
"pathlib: Object-oriented filesystem paths" ) from working with pre-existing
code, including Python's standard library.

To fix this situation, a new interface represented by
[`os.PathLike`](https://docs.python.org/3.6/library/os.html#os.PathLike
"os.PathLike" ) has been defined. By implementing the [`__fspath__()`](https:/
/docs.python.org/3.6/library/os.html#os.PathLike.__fspath__
"os.PathLike.__fspath__" ) method, an object signals that it represents a
path. An object can then provide a low-level representation of a file system
path as a [`str`](https://docs.python.org/3.6/library/stdtypes.html#str "str"
) or [`bytes`](https://docs.python.org/3.6/library/functions.html#bytes
"bytes" ) object. This means an object is considered [path-
like](https://docs.python.org/3.6/glossary.html#term-path-like-object) if it
implements
[`os.PathLike`](https://docs.python.org/3.6/library/os.html#os.PathLike
"os.PathLike" ) or is a
[`str`](https://docs.python.org/3.6/library/stdtypes.html#str "str" ) or
[`bytes`](https://docs.python.org/3.6/library/functions.html#bytes "bytes" )
object which represents a file system path. Code can use
[`os.fspath()`](https://docs.python.org/3.6/library/os.html#os.fspath
"os.fspath" ),
[`os.fsdecode()`](https://docs.python.org/3.6/library/os.html#os.fsdecode
"os.fsdecode" ), or
[`os.fsencode()`](https://docs.python.org/3.6/library/os.html#os.fsencode
"os.fsencode" ) to explicitly get a
[`str`](https://docs.python.org/3.6/library/stdtypes.html#str "str" ) and/or
[`bytes`](https://docs.python.org/3.6/library/functions.html#bytes "bytes" )
representation of a path-like object.

The built-in
[`open()`](https://docs.python.org/3.6/library/functions.html#open "open" )
function has been updated to accept
[`os.PathLike`](https://docs.python.org/3.6/library/os.html#os.PathLike
"os.PathLike" ) objects, as have all relevant functions in the
[`os`](https://docs.python.org/3.6/library/os.html#module-os "os:
Miscellaneous operating system interfaces." ) and
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

[**PEP 519**](https://www.python.org/dev/peps/pep-0519) - Adding a file system
path protocol

    PEP written by Brett Cannon and Koos Zevenhoven.

### PEP 495: Local Time Disambiguation¶

In most world locations, there have been and will be times when local clocks
are moved back. In those times, intervals are introduced in which local clocks
show the same time twice in the same day. In these situations, the information
displayed on a local clock (or stored in a Python datetime instance) is
insufficient to identify a particular moment in time.

[**PEP 495**](https://www.python.org/dev/peps/pep-0495) adds the new _fold_
attribute to instances of [`datetime.datetime`](https://docs.python.org/3.6/li
brary/datetime.html#datetime.datetime "datetime.datetime" ) and [`datetime.tim
e`](https://docs.python.org/3.6/library/datetime.html#datetime.time
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

[**PEP 495**](https://www.python.org/dev/peps/pep-0495) - Local Time
Disambiguation

    PEP written by Alexander Belopolsky and Tim Peters, implementation by Alexander Belopolsky.

### PEP 529: Change Windows filesystem encoding to UTF-8¶

Representing filesystem paths is best performed with str (Unicode) rather than
bytes. However, there are some situations where using bytes is sufficient and
correct.

Prior to Python 3.6, data loss could result when using bytes paths on Windows.
With this change, using bytes to represent paths is now supported on Windows,
provided those bytes are encoded with the encoding returned by [`sys.getfilesy
stemencoding()`](https://docs.python.org/3.6/library/sys.html#sys.getfilesyste
mencoding "sys.getfilesystemencoding" ), which now defaults to `'utf-8'`.

Applications that do not use str to represent paths should use
[`os.fsencode()`](https://docs.python.org/3.6/library/os.html#os.fsencode
"os.fsencode" ) and
[`os.fsdecode()`](https://docs.python.org/3.6/library/os.html#os.fsdecode
"os.fsdecode" ) to ensure their bytes are correctly encoded. To revert to the
previous behaviour, set [`PYTHONLEGACYWINDOWSFSENCODING`](https://docs.python.
org/3.6/using/cmdline.html#envvar-PYTHONLEGACYWINDOWSFSENCODING) or call [`sys
._enablelegacywindowsfsencoding()`](https://docs.python.org/3.6/library/sys.ht
ml#sys._enablelegacywindowsfsencoding "sys._enablelegacywindowsfsencoding" ).

See [**PEP 529**](https://www.python.org/dev/peps/pep-0529) for more
information and discussion of code modifications that may be required.

### PEP 528: Change Windows console encoding to UTF-8¶

The default console on Windows will now accept all Unicode characters and
provide correctly read str objects to Python code. `sys.stdin`, `sys.stdout`
and `sys.stderr` now default to utf-8 encoding.

This change only applies when using an interactive console, and not when
redirecting files or pipes. To revert to the previous behaviour for
interactive console use, set [`PYTHONLEGACYWINDOWSIOENCODING`](https://docs.py
thon.org/3.6/using/cmdline.html#envvar-PYTHONLEGACYWINDOWSIOENCODING).

See also

[**PEP 528**](https://www.python.org/dev/peps/pep-0528) - Change Windows
console encoding to UTF-8

    PEP written and implemented by Steve Dower.

### PEP 520: Preserving Class Attribute Definition Order¶

Attributes in a class definition body have a natural ordering: the same order
in which the names appear in the source. This order is now preserved in the
new class's
[`__dict__`](https://docs.python.org/3.6/library/stdtypes.html#object.__dict__
"object.__dict__" ) attribute.

Also, the effective default class _execution_ namespace (returned from [type._
_prepare__()](https://docs.python.org/3.6/reference/datamodel.html#prepare))
is now an insertion-order-preserving mapping.

See also

[**PEP 520**](https://www.python.org/dev/peps/pep-0520) - Preserving Class
Attribute Definition Order

    PEP written and implemented by Eric Snow.

### PEP 468: Preserving Keyword Argument Order¶

`**kwargs` in a function signature is now guaranteed to be an insertion-order-
preserving mapping.

See also

[**PEP 468**](https://www.python.org/dev/peps/pep-0468) - Preserving Keyword
Argument Order

    PEP written and implemented by Eric Snow.

### New [dict](https://docs.python.org/3.6/library/stdtypes.html#typesmapping)
implementation¶

The [dict](https://docs.python.org/3.6/library/stdtypes.html#typesmapping)
type now uses a "compact" representation [pioneered by
PyPy](https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-
more.html). The memory usage of the new
[`dict()`](https://docs.python.org/3.6/library/stdtypes.html#dict "dict" ) is
between 20% and 25% smaller compared to Python 3.5.

The order-preserving aspect of this new implementation is considered an
implementation detail and should not be relied upon (this may change in the
future, but it is desired to have this new dict implementation in the language
for a few releases before changing the language spec to mandate order-
preserving semantics for all current and future Python implementations; this
also helps preserve backwards-compatibility with older versions of the
language where random iteration order is still in effect, e.g. Python 3.5).

(Contributed by INADA Naoki in [issue
27350](https://bugs.python.org/issue27350). Idea [originally suggested by
Raymond Hettinger](https://mail.python.org/pipermail/python-
dev/2012-December/123028.html).)

### PEP 523: Adding a frame evaluation API to CPython¶

While Python provides extensive support to customize how code executes, one
place it has not done so is in the evaluation of frame objects. If you wanted
some way to intercept frame evaluation in Python there really wasn't any way
without directly manipulating function pointers for defined functions.

[**PEP 523**](https://www.python.org/dev/peps/pep-0523) changes this by
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

[**PEP 523**](https://www.python.org/dev/peps/pep-0523) - Adding a frame
evaluation API to CPython

    PEP written by Brett Cannon and Dino Viehland.

### PYTHONMALLOC environment variable¶

The new [`PYTHONMALLOC`](https://docs.python.org/3.6/using/cmdline.html
#envvar-PYTHONMALLOC) environment variable allows setting the Python memory
allocators and installing debug hooks.

It is now possible to install debug hooks on Python memory allocators on
Python compiled in release mode using `PYTHONMALLOC=debug`. Effects of debug
hooks:

  * Newly allocated memory is filled with the byte `0xCB`
  * Freed memory is filled with the byte `0xDB`
  * Detect violations of the Python memory allocator API. For example, `PyObject_Free()` called on a memory block allocated by [`PyMem_Malloc()`](https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc" ).
  * Detect writes before the start of a buffer (buffer underflows)
  * Detect writes after the end of a buffer (buffer overflows)
  * Check that the [GIL](https://docs.python.org/3.6/glossary.html#term-global-interpreter-lock) is held when allocator functions of [`PYMEM_DOMAIN_OBJ`](https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_OBJ "PYMEM_DOMAIN_OBJ" ) (ex: `PyObject_Malloc()`) and [`PYMEM_DOMAIN_MEM`](https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_MEM "PYMEM_DOMAIN_MEM" ) (ex: [`PyMem_Malloc()`](https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc" )) domains are called.

Checking if the GIL is held is also a new feature of Python 3.6.

See the [`PyMem_SetupDebugHooks()`](https://docs.python.org/3.6/c-api/memory.h
tml#c.PyMem_SetupDebugHooks "PyMem_SetupDebugHooks" ) function for debug hooks
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
  * It is now possible to set a [special method](https://docs.python.org/3.6/reference/datamodel.html#specialnames) to `None` to indicate that the corresponding operation is not available. For example, if a class sets [`__iter__()`](https://docs.python.org/3.6/reference/datamodel.html#object.__iter__ "object.__iter__" ) to `None`, the class is not iterable. (Contributed by Andrew Barnert and Ivan Levkivskyi in [issue 25958](https://bugs.python.org/issue25958).)
  * Long sequences of repeated traceback lines are now abbreviated as `"[Previous line repeated {count} more times]"` (see traceback for an example). (Contributed by Emanuel Barry in [issue 26823](https://bugs.python.org/issue26823).)
  * Import now raises the new exception [`ModuleNotFoundError`](https://docs.python.org/3.6/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" ) (subclass of [`ImportError`](https://docs.python.org/3.6/library/exceptions.html#ImportError "ImportError" )) when it cannot find a module. Code that currently checks for ImportError (in try-except) will still work. (Contributed by Eric Snow in [issue 15767](https://bugs.python.org/issue15767).)
  * Class methods relying on zero-argument `super()` will now work correctly when called from metaclass methods during class creation. (Contributed by Martin Teichmann in [issue 23722](https://bugs.python.org/issue23722).)

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

[**PEP 506**](https://www.python.org/dev/peps/pep-0506) - Adding A Secrets
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

  * The [`get_event_loop()`](https://docs.python.org/3.6/library/asyncio-eventloops.html#asyncio.get_event_loop "asyncio.get_event_loop" ) function has been changed to always return the currently running loop when called from couroutines and callbacks. (Contributed by Yury Selivanov in [issue 28613](https://bugs.python.org/issue28613).)
  * The [`ensure_future()`](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.ensure_future "asyncio.ensure_future" ) function and all functions that use it, such as `loop.run_until_complete()`, now accept all kinds of [awaitable objects](https://docs.python.org/3.6/glossary.html#term-awaitable). (Contributed by Yury Selivanov.)
  * New [`run_coroutine_threadsafe()`](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.run_coroutine_threadsafe "asyncio.run_coroutine_threadsafe" ) function to submit coroutines to event loops from other threads. (Contributed by Vincent Michel.)
  * New [`Transport.is_closing()`](https://docs.python.org/3.6/library/asyncio-protocol.html#asyncio.BaseTransport.is_closing "asyncio.BaseTransport.is_closing" ) method to check if the transport is closing or closed. (Contributed by Yury Selivanov.)
  * The `loop.create_server()` method can now accept a list of hosts. (Contributed by Yann Sionneau.)
  * New `loop.create_future()` method to create Future objects. This allows alternative event loop implementations, such as [uvloop](https://github.com/MagicStack/uvloop), to provide a faster [`asyncio.Future`](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Future "asyncio.Future" ) implementation. (Contributed by Yury Selivanov in [issue 27041](https://bugs.python.org/issue27041).)
  * New `loop.get_exception_handler()` method to get the current exception handler. (Contributed by Yury Selivanov in [issue 27040](https://bugs.python.org/issue27040).)
  * New [`StreamReader.readuntil()`](https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.StreamReader.readuntil "asyncio.StreamReader.readuntil" ) method to read data from the stream until a separator bytes sequence appears. (Contributed by Mark Korenberg.)
  * The performance of [`StreamReader.readexactly()`](https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.StreamReader.readexactly "asyncio.StreamReader.readexactly" ) has been improved. (Contributed by Mark Korenberg in [issue 28370](https://bugs.python.org/issue28370).)
  * The `loop.getaddrinfo()` method is optimized to avoid calling the system `getaddrinfo` function if the address is already resolved. (Contributed by A. Jesse Jiryu Davis.)
  * The `loop.stop()` method has been changed to stop the loop immediately after the current iteration. Any new callbacks scheduled as a result of the last iteration will be discarded. (Contributed by Guido van Rossum in [issue 25593](https://bugs.python.org/issue25593).)
  * `Future.set_exception` will now raise [`TypeError`](https://docs.python.org/3.6/library/exceptions.html#TypeError "TypeError" ) when passed an instance of the [`StopIteration`](https://docs.python.org/3.6/library/exceptions.html#StopIteration "StopIteration" ) exception. (Contributed by Chris Angelico in [issue 26221](https://bugs.python.org/issue26221).)
  * New [`loop.connect_accepted_socket()`](https://docs.python.org/3.6/library/asyncio-eventloop.html#asyncio.BaseEventLoop.connect_accepted_socket "asyncio.BaseEventLoop.connect_accepted_socket" ) method to be used by servers that accept connections outside of asyncio, but that use asyncio to handle them. (Contributed by Jim Fulton in [issue 27392](https://bugs.python.org/issue27392).)
  * `TCP_NODELAY` flag is now set for all TCP transports by default. (Contributed by Yury Selivanov in [issue 27456](https://bugs.python.org/issue27456).)
  * New [`loop.shutdown_asyncgens()`](https://docs.python.org/3.6/library/asyncio-eventloop.html#asyncio.AbstractEventLoop.shutdown_asyncgens "asyncio.AbstractEventLoop.shutdown_asyncgens" ) to properly close pending asynchronous generators before closing the loop. (Contributed by Yury Selivanov in [issue 28003](https://bugs.python.org/issue28003).)
  * [`Future`](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Future "asyncio.Future" ) and [`Task`](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Task "asyncio.Task" ) classes now have an optimized C implementation which makes asyncio code up to 30% faster. (Contributed by Yury Selivanov and INADA Naoki in [issue 26081](https://bugs.python.org/issue26081) and [issue 28544](https://bugs.python.org/issue28544).)

### binascii¶

The [`b2a_base64()`](https://docs.python.org/3.6/library/binascii.html#binasci
i.b2a_base64 "binascii.b2a_base64" ) function now accepts an optional
_newline_ keyword argument to control whether the newline character is
appended to the return value. (Contributed by Victor Stinner in [issue
25357](https://bugs.python.org/issue25357).)

### cmath¶

The new [`cmath.tau`](https://docs.python.org/3.6/library/cmath.html#cmath.tau
"cmath.tau" ) (τ) constant has been added. (Contributed by Lisa Roach in
[issue 12345](https://bugs.python.org/issue12345), see [**PEP
628**](https://www.python.org/dev/peps/pep-0628) for details.)

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
Dickinson in [issue 23229](https://bugs.python.org/issue23229).)

### collections¶

The new [`Collection`](https://docs.python.org/3.6/library/collections.abc.htm
l#collections.abc.Collection "collections.abc.Collection" ) abstract base
class has been added to represent sized iterable container classes.
(Contributed by Ivan Levkivskyi, docs by Neil Girdhar in [issue
27598](https://bugs.python.org/issue27598).)

The new [`Reversible`](https://docs.python.org/3.6/library/collections.abc.htm
l#collections.abc.Reversible "collections.abc.Reversible" ) abstract base
class represents iterable classes that also provide the [`__reversed__()`](htt
ps://docs.python.org/3.6/reference/datamodel.html#object.__reversed__
"object.__reversed__" ) method. (Contributed by Ivan Levkivskyi in [issue
25987](https://bugs.python.org/issue25987).)

The new [`AsyncGenerator`](https://docs.python.org/3.6/library/collections.abc
.html#collections.abc.AsyncGenerator "collections.abc.AsyncGenerator" )
abstract base class represents asynchronous generators. (Contributed by Yury
Selivanov in [issue 28720](https://bugs.python.org/issue28720).)

The [`namedtuple()`](https://docs.python.org/3.6/library/collections.html#coll
ections.namedtuple "collections.namedtuple" ) function now accepts an optional
keyword argument _module_, which, when specified, is used for the `__module__`
attribute of the returned named tuple class. (Contributed by Raymond Hettinger
in [issue 17941](https://bugs.python.org/issue17941).)

The _verbose_ and _rename_ arguments for [`namedtuple()`](https://docs.python.
org/3.6/library/collections.html#collections.namedtuple
"collections.namedtuple" ) are now keyword-only. (Contributed by Raymond
Hettinger in [issue 25628](https://bugs.python.org/issue25628).)

Recursive [`collections.deque`](https://docs.python.org/3.6/library/collection
s.html#collections.deque "collections.deque" ) instances can now be pickled.
(Contributed by Serhiy Storchaka in [issue
26482](https://bugs.python.org/issue26482).)

### concurrent.futures¶

The [`ThreadPoolExecutor`](https://docs.python.org/3.6/library/concurrent.futu
res.html#concurrent.futures.ThreadPoolExecutor
"concurrent.futures.ThreadPoolExecutor" ) class constructor now accepts an
optional _thread_name_prefix_ argument to make it possible to customize the
names of the threads created by the pool. (Contributed by Gregory P. Smith in
[issue 27664](https://bugs.python.org/issue27664).)

### contextlib¶

The [`contextlib.AbstractContextManager`](https://docs.python.org/3.6/library/
contextlib.html#contextlib.AbstractContextManager
"contextlib.AbstractContextManager" ) class has been added to provide an
abstract base class for context managers. It provides a sensible default
implementation for __enter__() which returns `self` and leaves __exit__() an
abstract method. A matching class has been added to the
[`typing`](https://docs.python.org/3.6/library/typing.html#module-typing
"typing: Support for type hints \(see PEP 484\)." ) module as [`typing.Context
Manager`](https://docs.python.org/3.6/library/typing.html#typing.ContextManage
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

The [`datetime.strftime()`](https://docs.python.org/3.6/library/datetime.html#
datetime.datetime.strftime "datetime.datetime.strftime" ) and [`date.strftime(
)`](https://docs.python.org/3.6/library/datetime.html#datetime.date.strftime
"datetime.date.strftime" ) methods now support ISO 8601 date directives `%G`,
`%u` and `%V`. (Contributed by Ashley Anderson in [issue
12006](https://bugs.python.org/issue12006).)

The [`datetime.isoformat()`](https://docs.python.org/3.6/library/datetime.html
#datetime.datetime.isoformat "datetime.datetime.isoformat" ) function now
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
.html#decimal.Decimal.as_integer_ratio "decimal.Decimal.as_integer_ratio" )
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
David Murray in [issue 24277](https://bugs.python.org/issue24277).)

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
/email.policy.html#email.policy.Policy.message_factory
"email.policy.Policy.message_factory" ), that controls what class is used by
default when the parser creates new message objects. For the [`email.policy.co
mpat32`](https://docs.python.org/3.6/library/email.policy.html#email.policy.co
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
[`enum`](https://docs.python.org/3.6/library/enum.html#module-enum "enum:
Implementation of an enumeration class." ) module:
[`Flag`](https://docs.python.org/3.6/library/enum.html#enum.Flag "enum.Flag" )
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
t.hook_encoded "fileinput.hook_encoded" ) now supports the _errors_ argument.
(Contributed by Joseph Hackman in [issue
25788](https://bugs.python.org/issue25788).)

### hashlib¶

[`hashlib`](https://docs.python.org/3.6/library/hashlib-blake2.html#module-
hashlib "hashlib: BLAKE2 hash function for Python" ) supports OpenSSL 1.1.0.
The minimum recommend version is 1.0.2. (Contributed by Christian Heimes in
[issue 26470](https://bugs.python.org/issue26470).)

BLAKE2 hash functions were added to the module.
[`blake2b()`](https://docs.python.org/3.6/library/hashlib-
blake2.html#hashlib.blake2b "hashlib.blake2b" ) and
[`blake2s()`](https://docs.python.org/3.6/library/hashlib-
blake2.html#hashlib.blake2s "hashlib.blake2s" ) are always available and
support the full feature set of BLAKE2. (Contributed by Christian Heimes in
[issue 26798](https://bugs.python.org/issue26798) based on code by Dmitry
Chestnykh and Samuel Neves. Documentation written by Dmitry Chestnykh.)

The SHA-3 hash functions `sha3_224()`, `sha3_256()`, `sha3_384()`,
`sha3_512()`, and SHAKE hash functions `shake_128()` and `shake_256()` were
added. (Contributed by Christian Heimes in [issue
16113](https://bugs.python.org/issue16113). Keccak Code Package by Guido
Bertoni, Joan Daemen, Michaël Peeters, Gilles Van Assche, and Ronny Van Keer.)

The password-based key derivation function
[`scrypt()`](https://docs.python.org/3.6/library/hashlib.html#hashlib.scrypt
"hashlib.scrypt" ) is now available with OpenSSL 1.1.0 and newer. (Contributed
by Christian Heimes in [issue 27928](https://bugs.python.org/issue27928).)

### http.client¶

[`HTTPConnection.request()`](https://docs.python.org/3.6/library/http.client.h
tml#http.client.HTTPConnection.request "http.client.HTTPConnection.request" )
and [`endheaders()`](https://docs.python.org/3.6/library/http.client.html#http
.client.HTTPConnection.endheaders "http.client.HTTPConnection.endheaders" )
both now support chunked encoding request bodies. (Contributed by Demian
Brecht and Rolf Krahl in [issue 12319](https://bugs.python.org/issue12319).)

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
[issue 24225](https://bugs.python.org/issue24225). Most idlelib patches since
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
by Eric Snow in [issue 15767](https://bugs.python.org/issue15767).)

[`importlib.util.LazyLoader`](https://docs.python.org/3.6/library/importlib.ht
ml#importlib.util.LazyLoader "importlib.util.LazyLoader" ) now calls [`create_
module()`](https://docs.python.org/3.6/library/importlib.html#importlib.abc.Lo
ader.create_module "importlib.abc.Loader.create_module" ) on the wrapped
loader, removing the restriction that [`importlib.machinery.BuiltinImporter`](
https://docs.python.org/3.6/library/importlib.html#importlib.machinery.Builtin
Importer "importlib.machinery.BuiltinImporter" ) and [`importlib.machinery.Ext
ensionFileLoader`](https://docs.python.org/3.6/library/importlib.html#importli
b.machinery.ExtensionFileLoader "importlib.machinery.ExtensionFileLoader" )
couldn't be used with [`importlib.util.LazyLoader`](https://docs.python.org/3.
6/library/importlib.html#importlib.util.LazyLoader "importlib.util.LazyLoader"
).

[`importlib.util.cache_from_source()`](https://docs.python.org/3.6/library/imp
ortlib.html#importlib.util.cache_from_source
"importlib.util.cache_from_source" ), [`importlib.util.source_from_cache()`](h
ttps://docs.python.org/3.6/library/importlib.html#importlib.util.source_from_c
ache "importlib.util.source_from_cache" ), and [`importlib.util.spec_from_file
_location()`](https://docs.python.org/3.6/library/importlib.html#importlib.uti
l.spec_from_file_location "importlib.util.spec_from_file_location" ) now
accept a [path-like object](https://docs.python.org/3.6/glossary.html#term-
path-like-object).

### inspect¶

The [`inspect.signature()`](https://docs.python.org/3.6/library/inspect.html#i
nspect.signature "inspect.signature" ) function now reports the implicit `.0`
parameters generated by the compiler for comprehension and generator
expression scopes as if they were positional-only parameters called
`implicit0`. (Contributed by Jelle Zijlstra in [issue
19611](https://bugs.python.org/issue19611).)

To reduce code churn when upgrading from Python 2.7 and the legacy [`inspect.g
etargspec()`](https://docs.python.org/3.6/library/inspect.html#inspect.getargs
pec "inspect.getargspec" ) API, the previously documented deprecation of [`ins
pect.getfullargspec()`](https://docs.python.org/3.6/library/inspect.html#inspe
ct.getfullargspec "inspect.getfullargspec" ) has been reversed. While this
function is convenient for single/source Python 2/3 code bases, the richer [`i
nspect.signature()`](https://docs.python.org/3.6/library/inspect.html#inspect.
signature "inspect.signature" ) interface remains the recommended approach for
new code. (Contributed by Nick Coghlan in [issue
27172](https://bugs.python.org/issue27172))

### json¶

[`json.load()`](https://docs.python.org/3.6/library/json.html#json.load
"json.load" ) and
[`json.loads()`](https://docs.python.org/3.6/library/json.html#json.loads
"json.loads" ) now support binary input. Encoded JSON should be represented
using either UTF-8, UTF-16, or UTF-32. (Contributed by Serhiy Storchaka in
[issue 17909](https://bugs.python.org/issue17909).)

### logging¶

The new [`WatchedFileHandler.reopenIfNeeded()`](https://docs.python.org/3.6/li
brary/logging.handlers.html#logging.handlers.WatchedFileHandler.reopenIfNeeded
"logging.handlers.WatchedFileHandler.reopenIfNeeded" ) method has been added
to add the ability to check if the log file needs to be reopened. (Contributed
by Marian Horban in [issue 24884](https://bugs.python.org/issue24884).)

### math¶

The tau (τ) constant has been added to the
[`math`](https://docs.python.org/3.6/library/math.html#module-math "math:
Mathematical functions \(sin\(\) etc.\)." ) and
[`cmath`](https://docs.python.org/3.6/library/cmath.html#module-cmath "cmath:
Mathematical functions for complex numbers." ) modules. (Contributed by Lisa
Roach in [issue 12345](https://bugs.python.org/issue12345), see [**PEP
628**](https://www.python.org/dev/peps/pep-0628) for details.)

### multiprocessing¶

[Proxy Objects](https://docs.python.org/3.6/library/multiprocessing.html
#multiprocessing-proxy-objects) returned by `multiprocessing.Manager()` can
now be nested. (Contributed by Davin Potts in [issue
6766](https://bugs.python.org/issue6766).)

### os¶

See the summary of PEP 519 for details on how the
[`os`](https://docs.python.org/3.6/library/os.html#module-os "os:
Miscellaneous operating system interfaces." ) and
[`os.path`](https://docs.python.org/3.6/library/os.path.html#module-os.path
"os.path: Operations on pathnames." ) modules now support [path-like
objects](https://docs.python.org/3.6/glossary.html#term-path-like-object).

[`scandir()`](https://docs.python.org/3.6/library/os.html#os.scandir
"os.scandir" ) now supports
[`bytes`](https://docs.python.org/3.6/library/functions.html#bytes "bytes" )
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
to increase the security. See the [**PEP
524**](https://www.python.org/dev/peps/pep-0524) for the rationale.

The Linux `getrandom()` syscall (get random bytes) is now exposed as the new
[`os.getrandom()`](https://docs.python.org/3.6/library/os.html#os.getrandom
"os.getrandom" ) function. (Contributed by Victor Stinner, part of the [**PEP
524**](https://www.python.org/dev/peps/pep-0524))

### pathlib¶

[`pathlib`](https://docs.python.org/3.6/library/pathlib.html#module-pathlib
"pathlib: Object-oriented filesystem paths" ) now supports [path-like
objects](https://docs.python.org/3.6/glossary.html#term-path-like-object).
(Contributed by Brett Cannon in [issue
27186](https://bugs.python.org/issue27186).)

See the summary of PEP 519 for details.

### pdb¶

The [`Pdb`](https://docs.python.org/3.6/library/pdb.html#pdb.Pdb "pdb.Pdb" )
class constructor has a new optional _readrc_ argument to control whether
`.pdbrc` files should be read.

### pickle¶

Objects that need `__new__` called with keyword arguments can now be pickled
using [pickle protocols](https://docs.python.org/3.6/library/pickle.html
#pickle-protocols) older than protocol version 4. Protocol version 4 already
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
in [issue 8637](https://bugs.python.org/issue8637).)

[`help()`](https://docs.python.org/3.6/library/functions.html#help "help" )
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
in [issue 18844](https://bugs.python.org/issue18844).)

### re¶

Added support of modifier spans in regular expressions. Examples:
`'(?i:p)ython'` matches `'python'` and `'Python'`, but not `'PYTHON'`;
`'(?i)g(?-i:v)r'` matches `'GvR'` and `'gvr'`, but not `'GVR'`. (Contributed
by Serhiy Storchaka in [issue 433028](https://bugs.python.org/issue433028).)

Match object groups can be accessed by `__getitem__`, which is equivalent to
`group()`. So `mo['name']` is now equivalent to `mo.group('name')`.
(Contributed by Eric Smith in [issue
24454](https://bugs.python.org/issue24454).)

`Match` objects now support [`index-like objects`](https://docs.python.org/3.6
/reference/datamodel.html#object.__index__ "object.__index__" ) as group
indices. (Contributed by Jeroen Demeyer and Xiang Zhang in [issue
27177](https://bugs.python.org/issue27177).)

### readline¶

Added [`set_auto_history()`](https://docs.python.org/3.6/library/readline.html
#readline.set_auto_history "readline.set_auto_history" ) to enable or disable
automatic addition of input to the history list. (Contributed by Tyler
Crompton in [issue 26870](https://bugs.python.org/issue26870).)

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
compatibility) through the new _punctuation_chars_ argument to control which
characters are treated as punctuation. (Contributed by Vinay Sajip in [issue
1521950](https://bugs.python.org/issue1521950).)

### site¶

When specifying paths to add to
[`sys.path`](https://docs.python.org/3.6/library/sys.html#sys.path "sys.path"
) in a .pth file, you may now specify file paths on top of directories (e.g.
zip files). (Contributed by Wolfgang Langner in [issue
26587](https://bugs.python.org/issue26587)).

### sqlite3¶

[`sqlite3.Cursor.lastrowid`](https://docs.python.org/3.6/library/sqlite3.html#
sqlite3.Cursor.lastrowid "sqlite3.Cursor.lastrowid" ) now supports the
`REPLACE` statement. (Contributed by Alex LordThorsen in [issue
16864](https://bugs.python.org/issue16864).)

### socket¶

The [`ioctl()`](https://docs.python.org/3.6/library/socket.html#socket.socket.
ioctl "socket.socket.ioctl" ) function now supports the [`SIO_LOOPBACK_FAST_PA
TH`](https://docs.python.org/3.6/library/socket.html#socket.SIO_LOOPBACK_FAST_
PATH "socket.SIO_LOOPBACK_FAST_PATH" ) control code. (Contributed by Daniel
Stokes in [issue 26536](https://bugs.python.org/issue26536).)

The [`getsockopt()`](https://docs.python.org/3.6/library/socket.html#socket.so
cket.getsockopt "socket.socket.getsockopt" ) constants `SO_DOMAIN`,
`SO_PROTOCOL`, `SO_PEERSEC`, and `SO_PASSSEC` are now supported. (Contributed
by Christian Heimes in [issue 26907](https://bugs.python.org/issue26907).)

The [`setsockopt()`](https://docs.python.org/3.6/library/socket.html#socket.so
cket.setsockopt "socket.socket.setsockopt" ) now supports the
`setsockopt(level, optname, None, optlen: int)` form. (Contributed by
Christian Heimes in [issue 27744](https://bugs.python.org/issue27744).)

The socket module now supports the address family
[`AF_ALG`](https://docs.python.org/3.6/library/socket.html#socket.AF_ALG
"socket.AF_ALG" ) to interface with Linux Kernel crypto API. `ALG_*`,
`SOL_ALG` and [`sendmsg_afalg()`](https://docs.python.org/3.6/library/socket.h
tml#socket.socket.sendmsg_afalg "socket.socket.sendmsg_afalg" ) were added.
(Contributed by Christian Heimes in [issue
27744](https://bugs.python.org/issue27744) with support from Victor Stinner.)

### socketserver¶

Servers based on the
[`socketserver`](https://docs.python.org/3.6/library/socketserver.html#module-
socketserver "socketserver: A framework for network servers." ) module,
including those defined in
[`http.server`](https://docs.python.org/3.6/library/http.server.html#module-
http.server "http.server: HTTP server and request handlers." ),
[`xmlrpc.server`](https://docs.python.org/3.6/library/xmlrpc.server.html
#module-xmlrpc.server "xmlrpc.server: Basic XML-RPC server implementations." )
and [`wsgiref.simple_server`](https://docs.python.org/3.6/library/wsgiref.html
#module-wsgiref.simple_server "wsgiref.simple_server: A simple WSGI HTTP
server." ), now support the [context
manager](https://docs.python.org/3.6/glossary.html#term-context-manager)
protocol. (Contributed by Aviv Palivoda in [issue
26404](https://bugs.python.org/issue26404).)

The `wfile` attribute of [`StreamRequestHandler`](https://docs.python.org/3.6/
library/socketserver.html#socketserver.StreamRequestHandler
"socketserver.StreamRequestHandler" ) classes now implements the [`io.Buffered
IOBase`](https://docs.python.org/3.6/library/io.html#io.BufferedIOBase
"io.BufferedIOBase" ) writable interface. In particular, calling [`write()`](h
ttps://docs.python.org/3.6/library/io.html#io.BufferedIOBase.write
"io.BufferedIOBase.write" ) is now guaranteed to send the data in full.
(Contributed by Martin Panter in [issue
26721](https://bugs.python.org/issue26721).)

### ssl¶

[`ssl`](https://docs.python.org/3.6/library/ssl.html#module-ssl "ssl: TLS/SSL
wrapper for socket objects" ) supports OpenSSL 1.1.0. The minimum recommend
version is 1.0.2. (Contributed by Christian Heimes in [issue
26470](https://bugs.python.org/issue26470).)

3DES has been removed from the default cipher suites and ChaCha20 Poly1305
cipher suites have been added. (Contributed by Christian Heimes in [issue
27850](https://bugs.python.org/issue27850) and [issue
27766](https://bugs.python.org/issue27766).)

[`SSLContext`](https://docs.python.org/3.6/library/ssl.html#ssl.SSLContext
"ssl.SSLContext" ) has better default configuration for options and ciphers.
(Contributed by Christian Heimes in [issue
28043](https://bugs.python.org/issue28043).)

SSL session can be copied from one client-side connection to another with the
new [`SSLSession`](https://docs.python.org/3.6/library/ssl.html#ssl.SSLSession
"ssl.SSLSession" ) class. TLS session resumption can speed up the initial
handshake, reduce latency and improve performance (Contributed by Christian
Heimes in [issue 19500](https://bugs.python.org/issue19500) based on a draft
by Alex Warhawk.)

The new [`get_ciphers()`](https://docs.python.org/3.6/library/ssl.html#ssl.SSL
Context.get_ciphers "ssl.SSLContext.get_ciphers" ) method can be used to get a
list of enabled ciphers in order of cipher priority.

All constants and flags have been converted to
[`IntEnum`](https://docs.python.org/3.6/library/enum.html#enum.IntEnum
"enum.IntEnum" ) and `IntFlags`. (Contributed by Christian Heimes in [issue
28025](https://bugs.python.org/issue28025).)

Server and client-side specific TLS protocols for
[`SSLContext`](https://docs.python.org/3.6/library/ssl.html#ssl.SSLContext
"ssl.SSLContext" ) were added. (Contributed by Christian Heimes in [issue
28085](https://bugs.python.org/issue28085).)

### statistics¶

A new [`harmonic_mean()`](https://docs.python.org/3.6/library/statistics.html#
statistics.harmonic_mean "statistics.harmonic_mean" ) function has been added.
(Contributed by Steven D'Aprano in [issue
27181](https://bugs.python.org/issue27181).)

### struct¶

[`struct`](https://docs.python.org/3.6/library/struct.html#module-struct
"struct: Interpret bytes as packed binary data." ) now supports IEEE 754 half-
precision floats via the `'e'` format specifier. (Contributed by Eli Stevens,
Mark Dickinson in [issue 11734](https://bugs.python.org/issue11734).)

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
function now includes the _platform_version_ field which contains the accurate
major version, minor version and build number of the current operating system,
rather than the version that is being emulated for the process (Contributed by
Steve Dower in [issue 27932](https://bugs.python.org/issue27932).)

### telnetlib¶

[`Telnet`](https://docs.python.org/3.6/library/telnetlib.html#telnetlib.Telnet
"telnetlib.Telnet" ) is now a context manager (contributed by Stéphane Wirtel
in [issue 25485](https://bugs.python.org/issue25485)).

### time¶

The
[`struct_time`](https://docs.python.org/3.6/library/time.html#time.struct_time
"time.struct_time" ) attributes `tm_gmtoff` and `tm_zone` are now available on
all platforms.

### timeit¶

The new [`Timer.autorange()`](https://docs.python.org/3.6/library/timeit.html#
timeit.Timer.autorange "timeit.Timer.autorange" ) convenience method has been
added to call [`Timer.timeit()`](https://docs.python.org/3.6/library/timeit.ht
ml#timeit.Timer.timeit "timeit.Timer.timeit" ) repeatedly so that the total
run time is greater or equal to 200 milliseconds. (Contributed by Steven
D'Aprano in [issue 6422](https://bugs.python.org/issue6422).)

[`timeit`](https://docs.python.org/3.6/library/timeit.html#module-timeit
"timeit: Measure the execution time of small code snippets." ) now warns when
there is substantial (4x) variance between best and worst times. (Contributed
by Serhiy Storchaka in [issue 23552](https://bugs.python.org/issue23552).)

### tkinter¶

Added methods `trace_add()`, `trace_remove()` and `trace_info()` in the
`tkinter.Variable` class. They replace old methods `trace_variable()`,
`trace()`, `trace_vdelete()` and `trace_vinfo()` that use obsolete Tcl
commands and might not work in future versions of Tcl. (Contributed by Serhiy
Storchaka in [issue 22115](https://bugs.python.org/issue22115)).

### traceback¶

Both the traceback module and the interpreter's builtin exception display now
abbreviate long sequences of repeated lines in tracebacks as shown in the
following example:

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

(Contributed by Emanuel Barry in [issue
26823](https://bugs.python.org/issue26823).)

### tracemalloc¶

The [`tracemalloc`](https://docs.python.org/3.6/library/tracemalloc.html
#module-tracemalloc "tracemalloc: Trace memory allocations." ) module now
supports tracing memory allocations in multiple different address spaces.

The new [`DomainFilter`](https://docs.python.org/3.6/library/tracemalloc.html#
tracemalloc.DomainFilter "tracemalloc.DomainFilter" ) filter class has been
added to filter block traces by their address space (domain).

(Contributed by Victor Stinner in [issue
26588](https://bugs.python.org/issue26588).)

### typing¶

Starting with Python 3.6 the
[`typing`](https://docs.python.org/3.6/library/typing.html#module-typing
"typing: Support for type hints \(see PEP 484\)." ) module is no longer
provisional and its API is considered stable.

Since the [`typing`](https://docs.python.org/3.6/library/typing.html#module-
typing "typing: Support for type hints \(see PEP 484\)." ) module was
[provisional](https://docs.python.org/3.6/glossary.html#term-provisional-api)
in Python 3.5, all changes introduced in Python 3.6 have also been backported
to Python 3.5.x.

The [`typing`](https://docs.python.org/3.6/library/typing.html#module-typing
"typing: Support for type hints \(see PEP 484\)." ) module has a much improved
support for generic type aliases. For example `Dict[str, Tuple[S, T]]` is now
a valid type annotation. (Contributed by Guido van Rossum in [Github
#195](https://github.com/python/typing/pull/195).)

The [`typing.ContextManager`](https://docs.python.org/3.6/library/typing.html#
typing.ContextManager "typing.ContextManager" ) class has been added for
representing [`contextlib.AbstractContextManager`](https://docs.python.org/3.6
/library/contextlib.html#contextlib.AbstractContextManager
"contextlib.AbstractContextManager" ). (Contributed by Brett Cannon in [issue
25609](https://bugs.python.org/issue25609).)

The [`typing.Collection`](https://docs.python.org/3.6/library/typing.html#typi
ng.Collection "typing.Collection" ) class has been added for representing [`co
llections.abc.Collection`](https://docs.python.org/3.6/library/collections.abc
.html#collections.abc.Collection "collections.abc.Collection" ). (Contributed
by Ivan Levkivskyi in [issue 27598](https://bugs.python.org/issue27598).)

The [`typing.ClassVar`](https://docs.python.org/3.6/library/typing.html#typing
.ClassVar "typing.ClassVar" ) type construct has been added to mark class
variables. As introduced in [**PEP
526**](https://www.python.org/dev/peps/pep-0526), a variable annotation
wrapped in ClassVar indicates that a given attribute is intended to be used as
a class variable and should not be set on instances of that class.
(Contributed by Ivan Levkivskyi in [Github
#280](https://github.com/python/typing/issues/280).)

A new [`TYPE_CHECKING`](https://docs.python.org/3.6/library/typing.html#typing
.TYPE_CHECKING "typing.TYPE_CHECKING" ) constant that is assumed to be `True`
by the static type chekers, but is `False` at runtime. (Contributed by Guido
van Rossum in [Github #230](https://github.com/python/typing/issues/230).)

A new
[`NewType()`](https://docs.python.org/3.6/library/typing.html#typing.NewType
"typing.NewType" ) helper function has been added to create lightweight
distinct types for annotations:

```

    from typing import NewType
    
    UserId = NewType('UserId', int)
    some_id = UserId(524313)
    
```

The static type checker will treat the new type as if it were a subclass of
the original type. (Contributed by Ivan Levkivskyi in [Github
#189](https://github.com/python/typing/issues/189).)

### unicodedata¶

The [`unicodedata`](https://docs.python.org/3.6/library/unicodedata.html
#module-unicodedata "unicodedata: Access the Unicode Database." ) module now
uses data from [Unicode 9.0.0](http://unicode.org/versions/Unicode9.0.0/).
(Contributed by Benjamin Peterson.)

### unittest.mock¶

The [`Mock`](https://docs.python.org/3.6/library/unittest.mock.html#unittest.m
ock.Mock "unittest.mock.Mock" ) class has the following improvements:

  * Two new methods, [`Mock.assert_called()`](https://docs.python.org/3.6/library/unittest.mock.html#unittest.mock.Mock.assert_called "unittest.mock.Mock.assert_called" ) and [`Mock.assert_called_once()`](https://docs.python.org/3.6/library/unittest.mock.html#unittest.mock.Mock.assert_called_once "unittest.mock.Mock.assert_called_once" ) to check if the mock object was called. (Contributed by Amit Saha in [issue 26323](https://bugs.python.org/issue26323).)
  * The [`Mock.reset_mock()`](https://docs.python.org/3.6/library/unittest.mock.html#unittest.mock.Mock.reset_mock "unittest.mock.Mock.reset_mock" ) method now has two optional keyword only arguments: _return_value_ and _side_effect_. (Contributed by Kushal Das in [issue 21271](https://bugs.python.org/issue21271).)

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
Nikolay Bogoychev in [issue 16099](https://bugs.python.org/issue16099).)

### venv¶

[`venv`](https://docs.python.org/3.6/library/venv.html#module-venv "venv:
Creation of virtual environments." ) accepts a new parameter `--prompt`. This
parameter provides an alternative prefix for the virtual environment.
(Proposed by Łukasz Balcerzak and ported to 3.6 by Stéphane Wirtel in [issue
22829](https://bugs.python.org/issue22829).)

### warnings¶

A new optional _source_ parameter has been added to the [`warnings.warn_explic
it()`](https://docs.python.org/3.6/library/warnings.html#warnings.warn_explici
t "warnings.warn_explicit" ) function: the destroyed object which emitted a [`
ResourceWarning`](https://docs.python.org/3.6/library/exceptions.html#Resource
Warning "ResourceWarning" ). A _source_ attribute has also been added to
`warnings.WarningMessage` (contributed by Victor Stinner in [issue
26568](https://bugs.python.org/issue26568) and [issue
26567](https://bugs.python.org/issue26567)).

When a [`ResourceWarning`](https://docs.python.org/3.6/library/exceptions.html
#ResourceWarning "ResourceWarning" ) warning is logged, the
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
[`REG_QWORD`](https://docs.python.org/3.6/library/winreg.html#winreg.REG_QWORD
"winreg.REG_QWORD" ). (Contributed by Clement Rouault in [issue
23026](https://bugs.python.org/issue23026).)

### winsound¶

Allowed keyword arguments to be passed to
[`Beep`](https://docs.python.org/3.6/library/winsound.html#winsound.Beep
"winsound.Beep" ), [`MessageBeep`](https://docs.python.org/3.6/library/winsoun
d.html#winsound.MessageBeep "winsound.MessageBeep" ), and [`PlaySound`](https:
//docs.python.org/3.6/library/winsound.html#winsound.PlaySound
"winsound.PlaySound" ) ([issue 27982](https://bugs.python.org/issue27982)).

### xmlrpc.client¶

The [`xmlrpc.client`](https://docs.python.org/3.6/library/xmlrpc.client.html
#module-xmlrpc.client "xmlrpc.client: XML-RPC client access." ) module now
supports unmarshalling additional data types used by the Apache XML-RPC
implementation for numerics and `None`. (Contributed by Serhiy Storchaka in
[issue 26885](https://bugs.python.org/issue26885).)

### zipfile¶

A new [`ZipInfo.from_file()`](https://docs.python.org/3.6/library/zipfile.html
#zipfile.ZipInfo.from_file "zipfile.ZipInfo.from_file" ) class method allows
making a
[`ZipInfo`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipInfo
"zipfile.ZipInfo" ) instance from a filesystem file. A new [`ZipInfo.is_dir()`
](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipInfo.is_dir
"zipfile.ZipInfo.is_dir" ) method can be used to check if the
[`ZipInfo`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipInfo
"zipfile.ZipInfo" ) instance represents a directory. (Contributed by Thomas
Kluyver in [issue 26039](https://bugs.python.org/issue26039).)

The [`ZipFile.open()`](https://docs.python.org/3.6/library/zipfile.html#zipfil
e.ZipFile.open "zipfile.ZipFile.open" ) method can now be used to write data
into a ZIP file, as well as for extracting data. (Contributed by Thomas
Kluyver in [issue 26039](https://bugs.python.org/issue26039).)

### zlib¶

The [`compress()`](https://docs.python.org/3.6/library/zlib.html#zlib.compress
"zlib.compress" ) and
[`decompress()`](https://docs.python.org/3.6/library/zlib.html#zlib.decompress
"zlib.decompress" ) functions now accept keyword arguments. (Contributed by
Aviv Palivoda in [issue 26243](https://bugs.python.org/issue26243) and Xiang
Zhang in [issue 16764](https://bugs.python.org/issue16764) respectively.)

## Optimizations¶

  * The Python interpreter now uses a 16-bit wordcode instead of bytecode which made a number of opcode optimizations possible. (Contributed by Demur Rumed with input and reviews from Serhiy Storchaka and Victor Stinner in [issue 26647](https://bugs.python.org/issue26647) and [issue 28050](https://bugs.python.org/issue28050).)
  * The [`asyncio.Future`](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Future "asyncio.Future" ) class now has an optimized C implementation. (Contributed by Yury Selivanov and INADA Naoki in [issue 26081](https://bugs.python.org/issue26081).)
  * The [`asyncio.Task`](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.Task "asyncio.Task" ) class now has an optimized C implementation. (Contributed by Yury Selivanov in [issue 28544](https://bugs.python.org/issue28544).)
  * Various implementation improvements in the [`typing`](https://docs.python.org/3.6/library/typing.html#module-typing "typing: Support for type hints \(see PEP 484\)." ) module (such as caching of generic types) allow up to 30 times performance improvements and reduced memory footprint.
  * The ASCII decoder is now up to 60 times as fast for error handlers `surrogateescape`, `ignore` and `replace` (Contributed by Victor Stinner in [issue 24870](https://bugs.python.org/issue24870)).
  * The ASCII and the Latin1 encoders are now up to 3 times as fast for the error handler `surrogateescape` (Contributed by Victor Stinner in [issue 25227](https://bugs.python.org/issue25227)).
  * The UTF-8 encoder is now up to 75 times as fast for error handlers `ignore`, `replace`, `surrogateescape`, `surrogatepass` (Contributed by Victor Stinner in [issue 25267](https://bugs.python.org/issue25267)).
  * The UTF-8 decoder is now up to 15 times as fast for error handlers `ignore`, `replace` and `surrogateescape` (Contributed by Victor Stinner in [issue 25301](https://bugs.python.org/issue25301)).
  * `bytes % args` is now up to 2 times faster. (Contributed by Victor Stinner in [issue 25349](https://bugs.python.org/issue25349)).
  * `bytearray % args` is now between 2.5 and 5 times faster. (Contributed by Victor Stinner in [issue 25399](https://bugs.python.org/issue25399)).
  * Optimize [`bytes.fromhex()`](https://docs.python.org/3.6/library/stdtypes.html#bytes.fromhex "bytes.fromhex" ) and [`bytearray.fromhex()`](https://docs.python.org/3.6/library/stdtypes.html#bytearray.fromhex "bytearray.fromhex" ): they are now between 2x and 3.5x faster. (Contributed by Victor Stinner in [issue 25401](https://bugs.python.org/issue25401)).
  * Optimize `bytes.replace(b'', b'.')` and `bytearray.replace(b'', b'.')`: up to 80% faster. (Contributed by Josh Snider in [issue 26574](https://bugs.python.org/issue26574)).
  * Allocator functions of the [`PyMem_Malloc()`](https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc" ) domain ([`PYMEM_DOMAIN_MEM`](https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_MEM "PYMEM_DOMAIN_MEM" )) now use the [pymalloc memory allocator](https://docs.python.org/3.6/c-api/memory.html#pymalloc) instead of `malloc()` function of the C library. The pymalloc allocator is optimized for objects smaller or equal to 512 bytes with a short lifetime, and use `malloc()` for larger memory blocks. (Contributed by Victor Stinner in [issue 26249](https://bugs.python.org/issue26249)).
  * [`pickle.load()`](https://docs.python.org/3.6/library/pickle.html#pickle.load "pickle.load" ) and [`pickle.loads()`](https://docs.python.org/3.6/library/pickle.html#pickle.loads "pickle.loads" ) are now up to 10% faster when deserializing many small objects (Contributed by Victor Stinner in [issue 27056](https://bugs.python.org/issue27056)).
  * Passing [keyword arguments](https://docs.python.org/3.6/glossary.html#term-keyword-argument) to a function has an overhead in comparison with passing [positional arguments](https://docs.python.org/3.6/glossary.html#term-positional-argument). Now in extension functions implemented with using Argument Clinic this overhead is significantly decreased. (Contributed by Serhiy Storchaka in [issue 27574](https://bugs.python.org/issue27574)).
  * Optimized [`glob()`](https://docs.python.org/3.6/library/glob.html#glob.glob "glob.glob" ) and [`iglob()`](https://docs.python.org/3.6/library/glob.html#glob.iglob "glob.iglob" ) functions in the [`glob`](https://docs.python.org/3.6/library/glob.html#module-glob "glob: Unix shell style pathname pattern expansion." ) module; they are now about 3-6 times faster. (Contributed by Serhiy Storchaka in [issue 25596](https://bugs.python.org/issue25596)).
  * Optimized globbing in [`pathlib`](https://docs.python.org/3.6/library/pathlib.html#module-pathlib "pathlib: Object-oriented filesystem paths" ) by using [`os.scandir()`](https://docs.python.org/3.6/library/os.html#os.scandir "os.scandir" ); it is now about 1.5-4 times faster. (Contributed by Serhiy Storchaka in [issue 26032](https://bugs.python.org/issue26032)).
  * [`xml.etree.ElementTree`](https://docs.python.org/3.6/library/xml.etree.elementtree.html#module-xml.etree.ElementTree "xml.etree.ElementTree: Implementation of the ElementTree API." ) parsing, iteration and deepcopy performance has been significantly improved. (Contributed by Serhiy Storchaka in [issue 25638](https://bugs.python.org/issue25638), [issue 25873](https://bugs.python.org/issue25873), and [issue 25869](https://bugs.python.org/issue25869).)
  * Creation of [`fractions.Fraction`](https://docs.python.org/3.6/library/fractions.html#fractions.Fraction "fractions.Fraction" ) instances from floats and decimals is now 2 to 3 times faster. (Contributed by Serhiy Storchaka in [issue 25971](https://bugs.python.org/issue25971).)

## Build and C API Changes¶

  * Python now requires some C99 support in the toolchain to build. Most notably, Python now uses standard integer types and macros in place of custom macros like `PY_LONG_LONG`. For more information, see [**PEP 7**](https://www.python.org/dev/peps/pep-0007) and [issue 17884](https://bugs.python.org/issue17884).
  * Cross-compiling CPython with the Android NDK and the Android API level set to 21 (Android 5.0 Lollilop) or greater runs successfully. While Android is not yet a supported platform, the Python test suite runs on the Android emulator with only about 16 tests failures. See the Android meta-issue [issue 26865](https://bugs.python.org/issue26865).
  * The `--enable-optimizations` configure flag has been added. Turning it on will activate expensive optimizations like PGO. (Original patch by Alecsandru Patrascu of Intel in [issue 26539](https://bugs.python.org/issue26539).)
  * The [GIL](https://docs.python.org/3.6/glossary.html#term-global-interpreter-lock) must now be held when allocator functions of [`PYMEM_DOMAIN_OBJ`](https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_OBJ "PYMEM_DOMAIN_OBJ" ) (ex: `PyObject_Malloc()`) and [`PYMEM_DOMAIN_MEM`](https://docs.python.org/3.6/c-api/memory.html#c.PYMEM_DOMAIN_MEM "PYMEM_DOMAIN_MEM" ) (ex: [`PyMem_Malloc()`](https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc" )) domains are called.
  * New [`Py_FinalizeEx()`](https://docs.python.org/3.6/c-api/init.html#c.Py_FinalizeEx "Py_FinalizeEx" ) API which indicates if flushing buffered data failed. (Contributed by Martin Panter in [issue 5319](https://bugs.python.org/issue5319).)
  * [`PyArg_ParseTupleAndKeywords()`](https://docs.python.org/3.6/c-api/arg.html#c.PyArg_ParseTupleAndKeywords "PyArg_ParseTupleAndKeywords" ) now supports [positional-only parameters](https://docs.python.org/3.6/glossary.html#positional-only-parameter). Positional-only parameters are defined by empty names. (Contributed by Serhiy Storchaka in [issue 26282](https://bugs.python.org/issue26282)).
  * `PyTraceback_Print` method now abbreviates long sequences of repeated lines as `"[Previous line repeated {count} more times]"`. (Contributed by Emanuel Barry in [issue 26823](https://bugs.python.org/issue26823).)
  * The new [`PyErr_SetImportErrorSubclass()`](https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_SetImportErrorSubclass "PyErr_SetImportErrorSubclass" ) function allows for specifying a subclass of [`ImportError`](https://docs.python.org/3.6/library/exceptions.html#ImportError "ImportError" ) to raise. (Contributed by Eric Snow in [issue 15767](https://bugs.python.org/issue15767).)
  * The new [`PyErr_ResourceWarning()`](https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_ResourceWarning "PyErr_ResourceWarning" ) function can be used to generate a [`ResourceWarning`](https://docs.python.org/3.6/library/exceptions.html#ResourceWarning "ResourceWarning" ) providing the source of the resource allocation. (Contributed by Victor Stinner in [issue 26567](https://bugs.python.org/issue26567).)
  * The new [`PyOS_FSPath()`](https://docs.python.org/3.6/c-api/sys.html#c.PyOS_FSPath "PyOS_FSPath" ) function returns the file system representation of a [path-like object](https://docs.python.org/3.6/glossary.html#term-path-like-object). (Contributed by Brett Cannon in [issue 27186](https://bugs.python.org/issue27186).)
  * The [`PyUnicode_FSConverter()`](https://docs.python.org/3.6/c-api/unicode.html#c.PyUnicode_FSConverter "PyUnicode_FSConverter" ) and [`PyUnicode_FSDecoder()`](https://docs.python.org/3.6/c-api/unicode.html#c.PyUnicode_FSDecoder "PyUnicode_FSDecoder" ) functions will now accept [path-like objects](https://docs.python.org/3.6/glossary.html#term-path-like-object).

## Other Improvements¶

  * When [`--version`](https://docs.python.org/3.6/using/cmdline.html#cmdoption--version) (short form: [`-V`](https://docs.python.org/3.6/using/cmdline.html#cmdoption-V)) is supplied twice, Python prints [`sys.version`](https://docs.python.org/3.6/library/sys.html#sys.version "sys.version" ) for detailed information.
```     
$ ./python -VV

    Python 3.6.0b4+ (3.6:223967b49e49+, Nov 21 2016, 20:55:04)
    [GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.42.1)]
    
```

## Deprecated¶

### New Keywords¶

`async` and `await` are not recommended to be used as variable, class,
function or module names. Introduced by [**PEP
492**](https://www.python.org/dev/peps/pep-0492) in Python 3.5, they will
become proper keywords in Python 3.7. Starting in Python 3.6, the use of
`async` or `await` as names will generate a [`DeprecationWarning`](https://doc
s.python.org/3.6/library/exceptions.html#DeprecationWarning
"DeprecationWarning" ).

### Deprecated Python behavior¶

Raising the [`StopIteration`](https://docs.python.org/3.6/library/exceptions.h
tml#StopIteration "StopIteration" ) exception inside a generator will now
generate a [`DeprecationWarning`](https://docs.python.org/3.6/library/exceptio
ns.html#DeprecationWarning "DeprecationWarning" ), and will trigger a [`Runtim
eError`](https://docs.python.org/3.6/library/exceptions.html#RuntimeError
"RuntimeError" ) in Python 3.7. See [PEP 479: Change StopIteration handling
inside generators](https://docs.python.org/3.6/whatsnew/3.5.html#whatsnew-
pep-479) for details.

The [`__aiter__()`](https://docs.python.org/3.6/reference/datamodel.html#objec
t.__aiter__ "object.__aiter__" ) method is now expected to return an
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
by Emanuel Barry in [issue 27364](https://bugs.python.org/issue27364).)

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
by Mariatta in [issue 25002](https://bugs.python.org/issue25002).)

#### asyncore¶

The [`asyncore`](https://docs.python.org/3.6/library/asyncore.html#module-
asyncore "asyncore: A base class for developing asynchronous socket handling
services." ) has been deprecated in favor of
[`asyncio`](https://docs.python.org/3.6/library/asyncio.html#module-asyncio
"asyncio: Asynchronous I/O, event loop, coroutines and tasks." ). (Contributed
by Mariatta in [issue 25002](https://bugs.python.org/issue25002).)

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
[issue 26129](https://bugs.python.org/issue26129).)

#### importlib¶

The [`importlib.machinery.SourceFileLoader.load_module()`](https://docs.python
.org/3.6/library/importlib.html#importlib.machinery.SourceFileLoader.load_modu
le "importlib.machinery.SourceFileLoader.load_module" ) and [`importlib.machin
ery.SourcelessFileLoader.load_module()`](https://docs.python.org/3.6/library/i
mportlib.html#importlib.machinery.SourcelessFileLoader.load_module
"importlib.machinery.SourcelessFileLoader.load_module" ) methods are now
deprecated. They were the only remaining implementations of [`importlib.abc.Lo
ader.load_module()`](https://docs.python.org/3.6/library/importlib.html#import
lib.abc.Loader.load_module "importlib.abc.Loader.load_module" ) in
[`importlib`](https://docs.python.org/3.6/library/importlib.html#module-
importlib "importlib: The implementation of the import machinery." ) that had
not been deprecated in previous versions of Python in favour of [`importlib.ab
c.Loader.exec_module()`](https://docs.python.org/3.6/library/importlib.html#im
portlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module" ).

The [`importlib.machinery.WindowsRegistryFinder`](https://docs.python.org/3.6/
library/importlib.html#importlib.machinery.WindowsRegistryFinder
"importlib.machinery.WindowsRegistryFinder" ) class is now deprecated. As of
3.6.0, it is still added to
[`sys.meta_path`](https://docs.python.org/3.6/library/sys.html#sys.meta_path
"sys.meta_path" ) by default (on Windows), but this may change in future
releases.

#### os¶

Undocumented support of general [bytes-like
objects](https://docs.python.org/3.6/glossary.html#term-bytes-like-object) as
paths in [`os`](https://docs.python.org/3.6/library/os.html#module-os "os:
Miscellaneous operating system interfaces." ) functions,
[`compile()`](https://docs.python.org/3.6/library/functions.html#compile
"compile" ) and similar functions is now deprecated. (Contributed by Serhiy
Storchaka in [issue 25791](https://bugs.python.org/issue25791) and [issue
26754](https://bugs.python.org/issue26754).)

#### re¶

Support for inline flags `(?letters)` in the middle of the regular expression
has been deprecated and will be removed in a future Python version. Flags at
the start of a regular expression are still allowed. (Contributed by Serhiy
Storchaka in [issue 22493](https://bugs.python.org/issue22493).)

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
#module-imaplib "imaplib: IMAP4 protocol client \(requires sockets\)." ),
[`poplib`](https://docs.python.org/3.6/library/poplib.html#module-poplib
"poplib: POP3 protocol client \(requires sockets\)." ), and
[`smtplib`](https://docs.python.org/3.6/library/smtplib.html#module-smtplib
"smtplib: SMTP protocol client \(requires sockets\)." ) have been deprecated
in favor of `context`. (Contributed by Christian Heimes in [issue
28022](https://bugs.python.org/issue28022).)

A couple of protocols and functions of the
[`ssl`](https://docs.python.org/3.6/library/ssl.html#module-ssl "ssl: TLS/SSL
wrapper for socket objects" ) module are now deprecated. Some features will no
longer be available in future versions of OpenSSL. Other features are
deprecated in favor of a different API. (Contributed by Christian Heimes in
[issue 28022](https://bugs.python.org/issue28022) and [issue
26470](https://bugs.python.org/issue26470).)

#### tkinter¶

The [`tkinter.tix`](https://docs.python.org/3.6/library/tkinter.tix.html
#module-tkinter.tix "tkinter.tix: Tk Extension Widgets for Tkinter" ) module
is now deprecated.
[`tkinter`](https://docs.python.org/3.6/library/tkinter.html#module-tkinter
"tkinter: Interface to Tcl/Tk for graphical user interfaces" ) users should
use [`tkinter.ttk`](https://docs.python.org/3.6/library/tkinter.ttk.html
#module-tkinter.ttk "tkinter.ttk: Tk themed widget set" ) instead.

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
by this change. Note that many OS distributors already use the `--with-system-
ffi` flag when building their system Python.

## Removed¶

### API and Feature Removals¶

  * Unknown escapes consisting of `'\'` and an ASCII letter in regular expressions will now cause an error. In replacement templates for [`re.sub()`](https://docs.python.org/3.6/library/re.html#re.sub "re.sub" ) they are still allowed, but deprecated. The [`re.LOCALE`](https://docs.python.org/3.6/library/re.html#re.LOCALE "re.LOCALE" ) flag can now only be used with binary patterns.
  * `inspect.getmoduleinfo()` was removed (was deprecated since CPython 3.3). [`inspect.getmodulename()`](https://docs.python.org/3.6/library/inspect.html#inspect.getmodulename "inspect.getmodulename" ) should be used for obtaining the module name for a given path. (Contributed by Yury Selivanov in [issue 13248](https://bugs.python.org/issue13248).)
  * `traceback.Ignore` class and `traceback.usage`, `traceback.modname`, `traceback.fullmodname`, `traceback.find_lines_from_code`, `traceback.find_lines`, `traceback.find_strings`, `traceback.find_executable_lines` methods were removed from the [`traceback`](https://docs.python.org/3.6/library/traceback.html#module-traceback "traceback: Print or retrieve a stack traceback." ) module. They were undocumented methods deprecated since Python 3.2 and equivalent functionality is available from private methods.
  * The `tk_menuBar()` and `tk_bindForTraversal()` dummy methods in [`tkinter`](https://docs.python.org/3.6/library/tkinter.html#module-tkinter "tkinter: Interface to Tcl/Tk for graphical user interfaces" ) widget classes were removed (corresponding Tk commands were obsolete since Tk 4.0).
  * The [`open()`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile.open "zipfile.ZipFile.open" ) method of the [`zipfile.ZipFile`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile "zipfile.ZipFile" ) class no longer supports the `'U'` mode (was deprecated since Python 3.4). Use [`io.TextIOWrapper`](https://docs.python.org/3.6/library/io.html#io.TextIOWrapper "io.TextIOWrapper" ) for reading compressed text files in [universal newlines](https://docs.python.org/3.6/glossary.html#term-universal-newlines) mode.
  * The undocumented `IN`, `CDROM`, `DLFCN`, `TYPES`, `CDIO`, and `STROPTS` modules have been removed. They had been available in the platform specific `Lib/plat-*/` directories, but were chronically out of date, inconsistently available across platforms, and unmaintained. The script that created these modules is still available in the source distribution at [Tools/scripts/h2py.py](https://hg.python.org/cpython/file/3.6/Tools/scripts/h2py.py).
  * The deprecated `asynchat.fifo` class has been removed.

## Porting to Python 3.6¶

This section lists previously described changes and other bugfixes that may
require changes to your code.

### Changes in 'python' Command Behavior¶

  * The output of a special Python build with defined `COUNT_ALLOCS`, `SHOW_ALLOC_COUNT` or `SHOW_TRACK_COUNT` macros is now off by default. It can be re-enabled using the `-X showalloccount` option. It now outputs to `stderr` instead of `stdout`. (Contributed by Serhiy Storchaka in [issue 23034](https://bugs.python.org/issue23034).)

### Changes in the Python API¶

  * [`open()`](https://docs.python.org/3.6/library/functions.html#open "open" ) will no longer allow combining the `'U'` mode flag with `'+'`. (Contributed by Jeff Balogh and John O'Connor in [issue 2091](https://bugs.python.org/issue2091).)

  * [`sqlite3`](https://docs.python.org/3.6/library/sqlite3.html#module-sqlite3 "sqlite3: A DB-API 2.0 implementation using SQLite 3.x." ) no longer implicitly commits an open transaction before DDL statements.

  * On Linux, [`os.urandom()`](https://docs.python.org/3.6/library/os.html#os.urandom "os.urandom" ) now blocks until the system urandom entropy pool is initialized to increase the security.

  * When [`importlib.abc.Loader.exec_module()`](https://docs.python.org/3.6/library/importlib.html#importlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module" ) is defined, [`importlib.abc.Loader.create_module()`](https://docs.python.org/3.6/library/importlib.html#importlib.abc.Loader.create_module "importlib.abc.Loader.create_module" ) must also be defined.

  * [`PyErr_SetImportError()`](https://docs.python.org/3.6/c-api/exceptions.html#c.PyErr_SetImportError "PyErr_SetImportError" ) now sets [`TypeError`](https://docs.python.org/3.6/library/exceptions.html#TypeError "TypeError" ) when its **msg** argument is not set. Previously only `NULL` was returned.

  * The format of the `co_lnotab` attribute of code objects changed to support a negative line number delta. By default, Python does not emit bytecode with a negative line number delta. Functions using `frame.f_lineno`, `PyFrame_GetLineNumber()` or `PyCode_Addr2Line()` are not affected. Functions directly decoding `co_lnotab` should be updated to use a signed 8-bit integer type for the line number delta, but this is only required to support applications using a negative line number delta. See `Objects/lnotab_notes.txt` for the `co_lnotab` format and how to decode it, and see the [**PEP 511**](https://www.python.org/dev/peps/pep-0511) for the rationale.

  * The functions in the [`compileall`](https://docs.python.org/3.6/library/compileall.html#module-compileall "compileall: Tools for byte-compiling all Python source files in a directory tree." ) module now return booleans instead of `1` or `0` to represent success or failure, respectively. Thanks to booleans being a subclass of integers, this should only be an issue if you were doing identity checks for `1` or `0`. See [issue 25768](https://bugs.python.org/issue25768).

  * Reading the `port` attribute of [`urllib.parse.urlsplit()`](https://docs.python.org/3.6/library/urllib.parse.html#urllib.parse.urlsplit "urllib.parse.urlsplit" ) and [`urlparse()`](https://docs.python.org/3.6/library/urllib.parse.html#urllib.parse.urlparse "urllib.parse.urlparse" ) results now raises [`ValueError`](https://docs.python.org/3.6/library/exceptions.html#ValueError "ValueError" ) for out-of-range values, rather than returning [`None`](https://docs.python.org/3.6/library/constants.html#None "None" ). See [issue 20059](https://bugs.python.org/issue20059).

  * The [`imp`](https://docs.python.org/3.6/library/imp.html#module-imp "imp: Access the implementation of the import statement. \(deprecated\)" ) module now raises a [`DeprecationWarning`](https://docs.python.org/3.6/library/exceptions.html#DeprecationWarning "DeprecationWarning" ) instead of [`PendingDeprecationWarning`](https://docs.python.org/3.6/library/exceptions.html#PendingDeprecationWarning "PendingDeprecationWarning" ).

  * The following modules have had missing APIs added to their `__all__` attributes to match the documented APIs: [`calendar`](https://docs.python.org/3.6/library/calendar.html#module-calendar "calendar: Functions for working with calendars, including some emulation of the Unix cal program." ), [`cgi`](https://docs.python.org/3.6/library/cgi.html#module-cgi "cgi: Helpers for running Python scripts via the Common Gateway Interface." ), [`csv`](https://docs.python.org/3.6/library/csv.html#module-csv "csv: Write and read tabular data to and from delimited files." ), [`ElementTree`](https://docs.python.org/3.6/library/xml.etree.elementtree.html#module-xml.etree.ElementTree "xml.etree.ElementTree: Implementation of the ElementTree API." ), [`enum`](https://docs.python.org/3.6/library/enum.html#module-enum "enum: Implementation of an enumeration class." ), [`fileinput`](https://docs.python.org/3.6/library/fileinput.html#module-fileinput "fileinput: Loop over standard input or a list of files." ), [`ftplib`](https://docs.python.org/3.6/library/ftplib.html#module-ftplib "ftplib: FTP protocol client \(requires sockets\)." ), [`logging`](https://docs.python.org/3.6/library/logging.html#module-logging "logging: Flexible event logging system for applications." ), [`mailbox`](https://docs.python.org/3.6/library/mailbox.html#module-mailbox "mailbox: Manipulate mailboxes in various formats" ), [`mimetypes`](https://docs.python.org/3.6/library/mimetypes.html#module-mimetypes "mimetypes: Mapping of filename extensions to MIME types." ), [`optparse`](https://docs.python.org/3.6/library/optparse.html#module-optparse "optparse: Command-line option parsing library. \(deprecated\)" ), [`plistlib`](https://docs.python.org/3.6/library/plistlib.html#module-plistlib "plistlib: Generate and parse Mac OS X plist files." ), [`smtpd`](https://docs.python.org/3.6/library/smtpd.html#module-smtpd "smtpd: A SMTP server implementation in Python." ), [`subprocess`](https://docs.python.org/3.6/library/subprocess.html#module-subprocess "subprocess: Subprocess management." ), [`tarfile`](https://docs.python.org/3.6/library/tarfile.html#module-tarfile "tarfile: Read and write tar-format archive files." ), [`threading`](https://docs.python.org/3.6/library/threading.html#module-threading "threading: Thread-based parallelism." ) and [`wave`](https://docs.python.org/3.6/library/wave.html#module-wave "wave: Provide an interface to the WAV sound format." ). This means they will export new symbols when `import *` is used. (Contributed by Joel Taddei and Jacek Kołodziej in [issue 23883](https://bugs.python.org/issue23883).)

  * When performing a relative import, if `__package__` does not compare equal to `__spec__.parent` then [`ImportWarning`](https://docs.python.org/3.6/library/exceptions.html#ImportWarning "ImportWarning" ) is raised. (Contributed by Brett Cannon in [issue 25791](https://bugs.python.org/issue25791).)

  * When a relative import is performed and no parent package is known, then [`ImportError`](https://docs.python.org/3.6/library/exceptions.html#ImportError "ImportError" ) will be raised. Previously, [`SystemError`](https://docs.python.org/3.6/library/exceptions.html#SystemError "SystemError" ) could be raised. (Contributed by Brett Cannon in [issue 18018](https://bugs.python.org/issue18018).)

  * Servers based on the [`socketserver`](https://docs.python.org/3.6/library/socketserver.html#module-socketserver "socketserver: A framework for network servers." ) module, including those defined in [`http.server`](https://docs.python.org/3.6/library/http.server.html#module-http.server "http.server: HTTP server and request handlers." ), [`xmlrpc.server`](https://docs.python.org/3.6/library/xmlrpc.server.html#module-xmlrpc.server "xmlrpc.server: Basic XML-RPC server implementations." ) and [`wsgiref.simple_server`](https://docs.python.org/3.6/library/wsgiref.html#module-wsgiref.simple_server "wsgiref.simple_server: A simple WSGI HTTP server." ), now only catch exceptions derived from [`Exception`](https://docs.python.org/3.6/library/exceptions.html#Exception "Exception" ). Therefore if a request handler raises an exception like [`SystemExit`](https://docs.python.org/3.6/library/exceptions.html#SystemExit "SystemExit" ) or [`KeyboardInterrupt`](https://docs.python.org/3.6/library/exceptions.html#KeyboardInterrupt "KeyboardInterrupt" ), [`handle_error()`](https://docs.python.org/3.6/library/socketserver.html#socketserver.BaseServer.handle_error "socketserver.BaseServer.handle_error" ) is no longer called, and the exception will stop a single-threaded server. (Contributed by Martin Panter in [issue 23430](https://bugs.python.org/issue23430).)

  * [`spwd.getspnam()`](https://docs.python.org/3.6/library/spwd.html#spwd.getspnam "spwd.getspnam" ) now raises a [`PermissionError`](https://docs.python.org/3.6/library/exceptions.html#PermissionError "PermissionError" ) instead of [`KeyError`](https://docs.python.org/3.6/library/exceptions.html#KeyError "KeyError" ) if the user doesn't have privileges.

  * The [`socket.socket.close()`](https://docs.python.org/3.6/library/socket.html#socket.socket.close "socket.socket.close" ) method now raises an exception if an error (e.g. `EBADF`) was reported by the underlying system call. (Contributed by Martin Panter in [issue 26685](https://bugs.python.org/issue26685).)

  * The _decode_data_ argument for the [`smtpd.SMTPChannel`](https://docs.python.org/3.6/library/smtpd.html#smtpd.SMTPChannel "smtpd.SMTPChannel" ) and [`smtpd.SMTPServer`](https://docs.python.org/3.6/library/smtpd.html#smtpd.SMTPServer "smtpd.SMTPServer" ) constructors is now `False` by default. This means that the argument passed to [`process_message()`](https://docs.python.org/3.6/library/smtpd.html#smtpd.SMTPServer.process_message "smtpd.SMTPServer.process_message" ) is now a bytes object by default, and `process_message()` will be passed keyword arguments. Code that has already been updated in accordance with the deprecation warning generated by 3.5 will not be affected.

  * All optional arguments of the [`dump()`](https://docs.python.org/3.6/library/json.html#json.dump "json.dump" ), [`dumps()`](https://docs.python.org/3.6/library/json.html#json.dumps "json.dumps" ), [`load()`](https://docs.python.org/3.6/library/json.html#json.load "json.load" ) and [`loads()`](https://docs.python.org/3.6/library/json.html#json.loads "json.loads" ) functions and [`JSONEncoder`](https://docs.python.org/3.6/library/json.html#json.JSONEncoder "json.JSONEncoder" ) and [`JSONDecoder`](https://docs.python.org/3.6/library/json.html#json.JSONDecoder "json.JSONDecoder" ) class constructors in the [`json`](https://docs.python.org/3.6/library/json.html#module-json "json: Encode and decode the JSON format." ) module are now [keyword-only](https://docs.python.org/3.6/glossary.html#keyword-only-parameter). (Contributed by Serhiy Storchaka in [issue 18726](https://bugs.python.org/issue18726).)

  * Subclasses of [`type`](https://docs.python.org/3.6/library/functions.html#type "type" ) which don't override `type.__new__` may no longer use the one-argument form to get the type of an object.

  * As part of [**PEP 487**](https://www.python.org/dev/peps/pep-0487), the handling of keyword arguments passed to [`type`](https://docs.python.org/3.6/library/functions.html#type "type" ) (other than the metaclass hint, `metaclass`) is now consistently delegated to [`object.__init_subclass__()`](https://docs.python.org/3.6/reference/datamodel.html#object.__init_subclass__ "object.__init_subclass__" ). This means that `type.__new__()` and `type.__init__()` both now accept arbitrary keyword arguments, but [`object.__init_subclass__()`](https://docs.python.org/3.6/reference/datamodel.html#object.__init_subclass__ "object.__init_subclass__" ) (which is called from `type.__new__()`) will reject them by default. Custom metaclasses accepting additional keyword arguments will need to adjust their calls to `type.__new__()` (whether direct or via [`super`](https://docs.python.org/3.6/library/functions.html#super "super" )) accordingly.

  * In `distutils.command.sdist.sdist`, the `default_format` attribute has been removed and is no longer honored. Instead, the gzipped tarfile format is the default on all platforms and no platform-specific selection is made. In environments where distributions are built on Windows and zip distributions are required, configure the project with a `setup.cfg` file containing the following:
```     [sdist]

    formats=zip
    
```

This behavior has also been backported to earlier Python versions by
Setuptools 26.0.0.

  * In the [`urllib.request`](https://docs.python.org/3.6/library/urllib.request.html#module-urllib.request "urllib.request: Extensible library for opening URLs." ) module and the [`http.client.HTTPConnection.request()`](https://docs.python.org/3.6/library/http.client.html#http.client.HTTPConnection.request "http.client.HTTPConnection.request" ) method, if no Content-Length header field has been specified and the request body is a file object, it is now sent with HTTP 1.1 chunked encoding. If a file object has to be sent to a HTTP 1.0 server, the Content-Length value now has to be specified by the caller. (Contributed by Demian Brecht and Rolf Krahl with tweaks from Martin Panter in [issue 12319](https://bugs.python.org/issue12319).)

  * The [`DictReader`](https://docs.python.org/3.6/library/csv.html#csv.DictReader "csv.DictReader" ) now returns rows of type [`OrderedDict`](https://docs.python.org/3.6/library/collections.html#collections.OrderedDict "collections.OrderedDict" ). (Contributed by Steve Holden in [issue 27842](https://bugs.python.org/issue27842).)

  * The [`crypt.METHOD_CRYPT`](https://docs.python.org/3.6/library/crypt.html#crypt.METHOD_CRYPT "crypt.METHOD_CRYPT" ) will no longer be added to `crypt.methods` if unsupported by the platform. (Contributed by Victor Stinner in [issue 25287](https://bugs.python.org/issue25287).)

  * The _verbose_ and _rename_ arguments for [`namedtuple()`](https://docs.python.org/3.6/library/collections.html#collections.namedtuple "collections.namedtuple" ) are now keyword-only. (Contributed by Raymond Hettinger in [issue 25628](https://bugs.python.org/issue25628).)

  * On Linux, [`ctypes.util.find_library()`](https://docs.python.org/3.6/library/ctypes.html#ctypes.util.find_library "ctypes.util.find_library" ) now looks in `LD_LIBRARY_PATH` for shared libraries. (Contributed by Vinay Sajip in [issue 9998](https://bugs.python.org/issue9998).)

  * The [`imaplib.IMAP4`](https://docs.python.org/3.6/library/imaplib.html#imaplib.IMAP4 "imaplib.IMAP4" ) class now handles flags containing the `']'` character in messages sent from the server to improve real-world compatibility. (Contributed by Lita Cho in [issue 21815](https://bugs.python.org/issue21815).)

  * The `mmap.write()` function now returns the number of bytes written like other write methods. (Contributed by Jakub Stasiak in [issue 26335](https://bugs.python.org/issue26335).)

  * The [`pkgutil.iter_modules()`](https://docs.python.org/3.6/library/pkgutil.html#pkgutil.iter_modules "pkgutil.iter_modules" ) and [`pkgutil.walk_packages()`](https://docs.python.org/3.6/library/pkgutil.html#pkgutil.walk_packages "pkgutil.walk_packages" ) functions now return [`ModuleInfo`](https://docs.python.org/3.6/library/pkgutil.html#pkgutil.ModuleInfo "pkgutil.ModuleInfo" ) named tuples. (Contributed by Ramchandra Apte in [issue 17211](https://bugs.python.org/issue17211).)

  * [`re.sub()`](https://docs.python.org/3.6/library/re.html#re.sub "re.sub" ) now raises an error for invalid numerical group references in replacement templates even if the pattern is not found in the string. The error message for invalid group references now includes the group index and the position of the reference. (Contributed by SilentGhost, Serhiy Storchaka in [issue 25953](https://bugs.python.org/issue25953).)

  * [`zipfile.ZipFile`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile "zipfile.ZipFile" ) will now raise [`NotImplementedError`](https://docs.python.org/3.6/library/exceptions.html#NotImplementedError "NotImplementedError" ) for unrecognized compression values. Previously a plain [`RuntimeError`](https://docs.python.org/3.6/library/exceptions.html#RuntimeError "RuntimeError" ) was raised. Additionally, calling [`ZipFile`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile "zipfile.ZipFile" ) methods on a closed ZipFile or calling the [`write()`](https://docs.python.org/3.6/library/zipfile.html#zipfile.ZipFile.write "zipfile.ZipFile.write" ) method on a ZipFile created with mode `'r'` will raise a [`ValueError`](https://docs.python.org/3.6/library/exceptions.html#ValueError "ValueError" ). Previously, a [`RuntimeError`](https://docs.python.org/3.6/library/exceptions.html#RuntimeError "RuntimeError" ) was raised in those scenarios.

  * when custom metaclasses are combined with zero-argument [`super()`](https://docs.python.org/3.6/library/functions.html#super "super" ) or direct references from methods to the implicit `__class__` closure variable, the implicit `__classcell__` namespace entry must now be passed up to `type.__new__` for initialisation. Failing to do so will result in a [`DeprecationWarning`](https://docs.python.org/3.6/library/exceptions.html#DeprecationWarning "DeprecationWarning" ) in 3.6 and a [`RuntimeWarning`](https://docs.python.org/3.6/library/exceptions.html#RuntimeWarning "RuntimeWarning" ) in the future.

### Changes in the C API¶

  * The [`PyMem_Malloc()`](https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc" ) allocator family now uses the [pymalloc allocator](https://docs.python.org/3.6/c-api/memory.html#pymalloc) rather than the system `malloc()`. Applications calling [`PyMem_Malloc()`](https://docs.python.org/3.6/c-api/memory.html#c.PyMem_Malloc "PyMem_Malloc" ) without holding the GIL can now crash. Set the [`PYTHONMALLOC`](https://docs.python.org/3.6/using/cmdline.html#envvar-PYTHONMALLOC) environment variable to `debug` to validate the usage of memory allocators in your application. See [issue 26249](https://bugs.python.org/issue26249).
  * [`Py_Exit()`](https://docs.python.org/3.6/c-api/sys.html#c.Py_Exit "Py_Exit" ) (and the main interpreter) now override the exit status with 120 if flushing buffered data failed. See [issue 5319](https://bugs.python.org/issue5319).

### CPython bytecode changes¶

There have been several major changes to the
[bytecode](https://docs.python.org/3.6/glossary.html#term-bytecode) in Python
3.6.

  * The Python interpreter now uses a 16-bit wordcode instead of bytecode. (Contributed by Demur Rumed with input and reviews from Serhiy Storchaka and Victor Stinner in [issue 26647](https://bugs.python.org/issue26647) and [issue 28050](https://bugs.python.org/issue28050).)
  * The new [`FORMAT_VALUE`](https://docs.python.org/3.6/library/dis.html#opcode-FORMAT_VALUE) and [`BUILD_STRING`](https://docs.python.org/3.6/library/dis.html#opcode-BUILD_STRING) opcodes as part of the formatted string literal implementation. (Contributed by Eric Smith in [issue 25483](https://bugs.python.org/issue25483) and Serhiy Storchaka in [issue 27078](https://bugs.python.org/issue27078).)
  * The new [`BUILD_CONST_KEY_MAP`](https://docs.python.org/3.6/library/dis.html#opcode-BUILD_CONST_KEY_MAP) opcode to optimize the creation of dictionaries with constant keys. (Contributed by Serhiy Storchaka in [issue 27140](https://bugs.python.org/issue27140).)
  * The function call opcodes have been heavily reworked for better performance and simpler implementation. The [`MAKE_FUNCTION`](https://docs.python.org/3.6/library/dis.html#opcode-MAKE_FUNCTION), [`CALL_FUNCTION`](https://docs.python.org/3.6/library/dis.html#opcode-CALL_FUNCTION), [`CALL_FUNCTION_KW`](https://docs.python.org/3.6/library/dis.html#opcode-CALL_FUNCTION_KW) and `BUILD_MAP_UNPACK_WITH_CALL` opcodes have been modified, the new `CALL_FUNCTION_EX` and `BUILD_TUPLE_UNPACK_WITH_CALL` have been added, and `CALL_FUNCTION_VAR`, `CALL_FUNCTION_VAR_KW` and `MAKE_CLOSURE` opcodes have been removed. (Contributed by Demur Rumed in [issue 27095](https://bugs.python.org/issue27095), and Serhiy Storchaka in [issue 27213](https://bugs.python.org/issue27213), [issue 28257](https://bugs.python.org/issue28257).)
  * The new [`SETUP_ANNOTATIONS`](https://docs.python.org/3.6/library/dis.html#opcode-SETUP_ANNOTATIONS) and [`STORE_ANNOTATION`](https://docs.python.org/3.6/library/dis.html#opcode-STORE_ANNOTATION) opcodes have been added to support the new [variable annotation](https://docs.python.org/3.6/glossary.html#term-variable-annotation) syntax. (Contributed by Ivan Levkivskyi in [issue 27985](https://bugs.python.org/issue27985).)

----
(C) [Copyright](https://docs.python.org/3.6/copyright.html) 2001-2016, Python
Software Foundation.  
The Python Software Foundation is a non-profit corporation. [Please
donate.](https://www.python.org/psf/donations/)  
Last updated on Dec 15, 2016. [Found a
bug](https://docs.python.org/3.6/bugs.html)?  
Created using [Sphinx](http://sphinx.pocoo.org/) 1.3.3.

