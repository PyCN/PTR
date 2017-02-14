原文：[5. The import system](https://docs.python.org/3/reference/import.html)

---
  
### [目录](https://docs.python.org/3/contents.html)

  * 5. 导入系统
    * 5.1. `importlib`
    * 5.2. 包
      * 5.2.1. 常规包
      * 5.2.2. 名字空间包
    * 5.3. 搜索
      * 5.3.1. 模块缓存
      * 5.3.2. 查找器和加载器
      * 5.3.3. 导入钩子
      * 5.3.4. 元路径
    * 5.4. 加载
      * 5.4.1. 加载器
      * 5.4.2. 子模块
      * 5.4.3. Module spec
      * 5.4.4. 导入相关的模块属性
      * 5.4.5. module.__path__
      * 5.4.6. Module reprs
    * 5.5. 基于路径的查找器
      * 5.5.1. Path entry finders
      * 5.5.2. Path entry finder protocol
    * 5.6. Replacing the standard import system
    * 5.7. Special considerations for __main__
      * 5.7.1. __main__.__spec__
    * 5.8. Open issues
    * 5.9. References

通过[导入](https://docs.python.org/3/glossary.html#term-importing)过程，一个[模块](https://docs.python.org/3/glossary.html#term-module)中的Python代码可以访问另一个模块中的代码。[`import`](https://docs.python.org/3/reference/simple_stmts.html#import)语句是调用导入机制最常见的方式，但是它并非唯一的方式。诸如[`importlib.import_module()`](https://docs.python.org/3/library/importlib.html#importlib.import_module "importlib.import_module" )和内置的[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )这样的函数也可以被用来调用导入机制。

[`import`](https://docs.python.org/3/reference/simple_stmts.html#import)语句组合了两种操作；它搜索命名模块，然后将搜索的结果绑定到本地域中的一个名字上。[`import`](https://docs.python.org/3/reference/simple_stmts.html#import)语句的搜索操作被定义成对[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )函数的调用，使用合适的参数。[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )的返回值被用来执行[`import`](https://docs.python.org/3/reference/simple_stmts.html#import)语句的名字绑定操作。见[`import`](https://docs.python.org/3/reference/simple_stmts.html#import)语句，了解该名字绑定操作的确切细节。

对[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )的直接调用只进行模块搜索，以及在找到的情况下的模块创建操作。虽然某些副作用可能会发生，例如父包的导入，不同缓存 (包括[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" ))的更新，但是只有[`import`](https://docs.python.org/3/reference/simple_stmts.html#import)语句会执行名字绑定操作。

当把[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )作为一个import语句的一部分时，会调用标准的内置[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )。调用导入系统的其他机制 (例如[`importlib.import_module()`](https://docs.python.org/3/library/importlib.html#importlib.import_module "importlib.import_module" )) 或许会选择推翻[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )，而使用它自己的方法来实现import语义。

当一个模块第一次被导入的时候，Python搜索这个模块，如果找到，则创建一个module对象 [1]，并初始化它。如果不能找到该命名模块，那么会抛出一个[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。Python实现了在调用导入机制时搜索命名模块的各种策略。这些策略可以通过使用以下部分描述的各种钩子来进行修改和扩展。

版本3.3中的改动：已更新导入系统，完整实现了[**PEP 302**](https://www.python.org/dev/peps/pep-0302)的第二阶段。不再有任何隐式导入机制 —— 完整的导入系统通过[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )公开出来。此外，已实现原生的namespace包支持 (见[**PEP 420**](https://www.python.org/dev/peps/pep-0420))。

## 5.1. [`importlib`](https://docs.python.org/3/library/importlib.html#module-importlib "importlib: The implementation of the import machinery." )

[`importlib`](https://docs.python.org/3/library/importlib.html#module-importlib "importlib: The implementation of the import machinery." )模块提供了用以与导入系统交互的丰富的API。例如[`importlib.import_module()`](https://docs.python.org/3/library/importlib.html#importlib.import_module "importlib.import_module" )提供了一个推荐的比内置的[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )更简单的API，用来调用导入机制。参考[`importlib`](https://docs.python.org/3/library/importlib.html#module-importlib "importlib: The implementation of the import machinery." )库文档以获取额外的细节。

## 5.2. Packages

Python只有一种类型的module对象，而所有的模块都是这个类型，无论这个模块是用Python，C，还是其他什么语言实现的。为了帮助组织模块并提供一个命名层级，Python有[包](https://docs.python.org/3/glossary.html#term-package)的概念。

你可以把包想成文件系统上的目录，而模块则是目录中的文件，但是不要太按字面意思来看待这种类比，因为包和模块无需来源于文件系统。出于这个文档的目的，我们会使用目录和文件这种方便的类比。就像文件系统目录，包是按层级组织的，而包自身可能会包含子包，以及常规模块。

记住，所有的包都是模块，但并非所有的模块都是包，这很重要。或者换句话说，包只是一种特殊类型的模块。特别是，任何包含一个`__path__`属性的模块，都会被当成一个包。

所有的模块都有一个名字。子包名用点(.)与它们的父包名分隔开来，类似于Python标准的属性访问语法。因此，你可能有一个名为[`sys`](https://docs.python.org/3/library/sys.html#module-sys "sys: Accesssystem-specific parameters and functions." )的模块和一个名为[`email`](https://docs.python.org/3/library/email.html#module-email "email:Package supporting the parsing, manipulating, and generating email messages.")的包，反过来，有一个名为[`email.mime`](https://docs.python.org/3/library/email.mime.html#module-email.mime "email.mime: Build MIME messages." )的子包以及在那个包中的一个名为`email.mime.text`的模块。

### 5.2.1. 常规包

Python定义了两种类型的包，[常规包](https://docs.python.org/3/glossary.html#term-regular-package)和[名字空间包](https://docs.python.org/3/glossary.html#term-namespace-package)。常规包是传统的包，它们在Python 3.2以及更早的版本中就存在了。一个常规包通常会作为一个包含一个`__init__.py`文件的目录来实现。当导入了一个常规包，就会隐式执行这个`__init__.py`文件，而它定义的对象会绑定到这个包的名字空间中的名字上。`__init__.py`可以包含任何其他模块可以包含的相同的Python代码，而在导入这个模块的时候，Python会添加一些额外的属性到这个模块上。

例如，以下文件系统布局定义了一个带有三个子包的顶级`parent`包：

[code]

    parent/
        __init__.py
        one/
            __init__.py
        two/
            __init__.py
        three/
            __init__.py
    
[/code]

导入`parent.one`将会隐式执行`parent/__init__.py`和`parent/one/__init__.py`。随后对`parent.two`或者`parent.three`的导入将会分别执行`parent/two/__init__.py`和`parent/three/__init__.py`。

### 5.2.2. 名字空间包

一个名字空间包是各种[部分(portion)](https://docs.python.org/3/glossary.html#term-portion)的组成，其中，每个部分都共享了一个子包给父包。这些部分可能位于文件系统中的不同位置。这些部分也有可能位于zip文件中、网络上、或者任何其他Python在导入时搜索的位置。名字空间包可能或者可能不直接对应于文件系统上的对象；它们或许是没有实体的虚拟模块。

名字空间包的`__path__`属性并不适用一个普通的列表。相反，它们使用一个自定义的可迭代类型，which will automatically perform a new
search for package portions on the next import attempt within that package if
the path of their parent package (or
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )
for a top level package) changes.

With namespace packages, there is no `parent/__init__.py` file. In fact, there
may be multiple `parent` directories found during import search, where each
one is provided by a different portion. Thus `parent/one` may not be
physically located next to `parent/two`. In this case, Python will create a
namespace package for the top-level `parent` package whenever it or one of its
subpackages is imported.

See also [**PEP 420**](https://www.python.org/dev/peps/pep-0420) for the
namespace package specification.

## 5.3. 搜索

要开始搜索，Python需要导入模块（或者包，但是出于这个讨论的目的，它们的区别是不重要的）[完整的限定](https://docs.python.org/3/glossary.html#term-qualified-name)名。这个名字可能来自于[`import`](https://docs.python.org/3/reference/simple_stmts.html#import)语句的各种参数，或者来自于[`importlib.import_module()`](https://docs.python.org/3/library/importlib.html#importlib.import_module "importlib.import_module" )或者[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )函数的参数。

这个名字将会被用于导入搜索的各种阶段，并且它可能是到一个子模块的点分路径，例如，`foo.bar.baz`。在这种情况下，Python首先试着导入`foo`，然后是`foo.bar`，最后是`foo.bar.baz`。如果任何中间导入失败，那么就会抛出[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。

### 5.3.1. 模块缓存

导入搜索期间检查的第一个地方是[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )。这个映射提供了所有之前已经被导入的模块的缓存，包括中间路径。因此，如果之前导入了`foo.bar.baz`，那么[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )将会包含`foo`, `foo.bar`, 和`foo.bar.baz`项。每个键对应的值是对应的module对象。

在导入期间，会在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中查找模块名，如果存在的话，对应的值就是满足导入的模块，然后此过程完成。然而，如果值是`None`，那么会抛出一个[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。如果找不到模块名，那么Python将会继续搜索该模块。

[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )是可写的。删除一个键也许不会销毁对应的模块 (因为其他模块可能有到这个模块的引用)，但是它将会时该命名模块的缓存项无效，引发Python在这个模块的下一次导入的时候重新搜索这个命名模块。也可以分配键值为`None`，强制该模块的下一次导入引发[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。

请小心提防，即使你保留了一个到该module对象的引用，使其在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中的缓存项无效，然后重新导入该命名模块，但是这两个module对象将_不_是同一个对象。相反，[`importlib.reload()`](https://docs.python.org/3/library/importlib.html#importlib.reload "importlib.reload" )将会重用_相同的_模块对象，并且通过重新运行该模块代码来简单重新初始化模块内容。

### 5.3.2. 查找器和加载器

如果在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules  "sys.modules" )中并未找到该命名模块，那么会调用Python的导入协议来查找和加载该模块。这个协议由两个概念上的对象组成，[查找器](https://docs.python.org/3/glossary.html#term-finder)和[加载器](https://docs.python.org/3/glossary.html#term-loader)。一个查找器的工作是决定它是否可以使用任何它所知道的策略来查找该命名模块。同时实现这两个接口的对象被称之为[导入器](https://docs.python.org/3/glossary.html#term-importer) —— 当它们发现他们可以加载所请求的模块时，返回自身。

Python包含了大量的默认查找器和导入器。前者知道如何定位到内置模块，而后者知道如何定位到冻结模块。一个third default finder searches an [import path](https://docs.python.org/3/glossary.html#term-import-path) for modules.
The [import path](https://docs.python.org/3/glossary.html#term-import-path) is
a list of locations that may name file system paths or zip files. It can also
be extended to search for any locatable resource, such as those identified by
URLs.

The import machinery is extensible, so new finders can be added to extend the
range and scope of module searching.

Finders do not actually load modules. If they can find the named module, they
return a _module spec_, an encapsulation of the module's import-related
information, which the import machinery then uses when loading the module.

The following sections describe the protocol for finders and loaders in more
detail, including how you can create and register new ones to extend the
import machinery.

Changed in version 3.4: In previous versions of Python, finders returned
[loaders](https://docs.python.org/3/glossary.html#term-loader) directly,
whereas now they return module specs which _contain_ loaders. Loaders are
still used during import but have fewer responsibilities.

### 5.3.3. Import hooks¶

The import machinery is designed to be extensible; the primary mechanism for
this are the _import hooks_. There are two types of import hooks: _meta hooks_
and _import path hooks_.

Meta hooks are called at the start of import processing, before any other
import processing has occurred, other than
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ) cache look up. This allows meta hooks to override
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )
processing, frozen modules, or even built-in modules. Meta hooks are
registered by adding new finder objects to
[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path
"sys.meta_path" ), as described below.

Import path hooks are called as part of
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )
(or `package.__path__`) processing, at the point where their associated path
item is encountered. Import path hooks are registered by adding new callables
to
[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks
"sys.path_hooks" ) as described below.

### 5.3.4. 元路径

当在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中找不到命名模块时，Python接下来会搜索[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )，它包含了一个元路径查找器对象列表。查询这些查找器来看看它们是否知道如何处理该命名模块。元路径查找器必须实现一个名为[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )的方法，这个方法接受三个参数：一个名字，一个导入路径，和（可选的）一个目标模块。元路径查找器可以使用任何它想要的策略来决定它是否可以处理该命名模块。

如果该元路径查找器知道如何处理这个命名模块，那么它返回一个spec对象。如果它不能处理这个命名模块，那么它返回`None`。如果[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )处理到达了它的列表的尾部都没有返回一个spec，那么会抛出一个[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。抛出的任何其他异常将会简单地向外层传播，中止导入过程。

带两个或三个参数调用元路径查找器的[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )方法。第一个是导入模块的完整限定名，例如，`foo.bar.baz`。第二个是用来进行模块搜索的路径项。对于顶层模块，第二个参数是`None`，但是对于子模块或者子包，第二个参数是父包的`__path__`属性。如果不能访问适当的`__path__`属性，那么会抛出一个[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。第三个参数是一个存在的module对象，它将会是稍后的加载目标。导入系统只有在重载期间才会传递进一个目标模块。

The meta path may be traversed multiple times for a single import request. For
example, assuming none of the modules involved has already been cached,
importing `foo.bar.baz` will first perform a top level import, calling
`mpf.find_spec("foo", None, None)` on each meta path finder (`mpf`). After
`foo` has been imported, `foo.bar` will be imported by traversing the meta
path a second time, calling `mpf.find_spec("foo.bar", foo.__path__, None)`.
Once `foo.bar` has been imported, the final traversal will call
`mpf.find_spec("foo.bar.baz", foo.bar.__path__, None)`.

Some meta path finders only support top level imports. These importers will
always return `None` when anything other than `None` is passed as the second
argument.

Python's default
[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path
"sys.meta_path" ) has three meta path finders, one that knows how to import
built-in modules, one that knows how to import frozen modules, and one that
knows how to import modules from an [import
path](https://docs.python.org/3/glossary.html#term-import-path) (i.e. the
[path based finder](https://docs.python.org/3/glossary.html#term-path-based-
finder)).

Changed in version 3.4: The [`find_spec()`](https://docs.python.org/3/library/
importlib.html#importlib.abc.MetaPathFinder.find_spec
"importlib.abc.MetaPathFinder.find_spec" ) method of meta path finders
replaced [`find_module()`](https://docs.python.org/3/library/importlib.html#im
portlib.abc.MetaPathFinder.find_module
"importlib.abc.MetaPathFinder.find_module" ), which is now deprecated. While
it will continue to work without change, the import machinery will try it only
if the finder does not implement `find_spec()`.

## 5.4. Loading¶

If and when a module spec is found, the import machinery will use it (and the
loader it contains) when loading the module. Here is an approximation of what
happens during the loading portion of import:

[code]

    module = None
    if spec.loader is not None and hasattr(spec.loader, 'create_module'):
        # It is assumed 'exec_module' will also be defined on the loader.
        module = spec.loader.create_module(spec)
    if module is None:
        module = ModuleType(spec.name)
    # The import-related module attributes get set here:
    _init_module_attrs(spec, module)
    
    if spec.loader is None:
        if spec.submodule_search_locations is not None:
            # namespace package
            sys.modules[spec.name] = module
        else:
            # unsupported
            raise ImportError
    elif not hasattr(spec.loader, 'exec_module'):
        module = spec.loader.load_module(spec.name)
        # Set __loader__ and __package__ if missing.
    else:
        sys.modules[spec.name] = module
        try:
            spec.loader.exec_module(module)
        except BaseException:
            try:
                del sys.modules[spec.name]
            except KeyError:
                pass
            raise
    return sys.modules[spec.name]
    
[/code]

Note the following details:

>   * If there is an existing module object with the given name in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ), import will have already returned it.

>   * The module will exist in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ) before the loader executes the module code. This is crucial
because the module code may (directly or indirectly) import itself; adding it
to [`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ) beforehand prevents unbounded recursion in the worst case and
multiple loading in the best.

>   * If loading fails, the failing module - and only the failing module -
gets removed from
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ). Any module already in the
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ) cache, and any module that was successfully loaded as a side-
effect, must remain in the cache. This contrasts with reloading where even the
failing module is left in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ).

>   * After the module is created but before execution, the import machinery
sets the import-related module attributes ("_init_module_attrs" in the pseudo-
code example above), as summarized in a later section.

>   * Module execution is the key moment of loading in which the module's
namespace gets populated. Execution is entirely delegated to the loader, which
gets to decide what gets populated and how.

>   * The module created during loading and passed to exec_module() may not be
the one returned at the end of import [2].

>

Changed in version 3.4: The import system has taken over the boilerplate
responsibilities of loaders. These were previously performed by the [`importli
b.abc.Loader.load_module()`](https://docs.python.org/3/library/importlib.html#
importlib.abc.Loader.load_module "importlib.abc.Loader.load_module" ) method.

### 5.4.1. Loaders¶

Module loaders provide the critical function of loading: module execution. The
import machinery calls the [`importlib.abc.Loader.exec_module()`](https://docs
.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module
"importlib.abc.Loader.exec_module" ) method with a single argument, the module
object to execute. Any value returned from [`exec_module()`](https://docs.pyth
on.org/3/library/importlib.html#importlib.abc.Loader.exec_module
"importlib.abc.Loader.exec_module" ) is ignored.

Loaders must satisfy the following requirements:

>   * If the module is a Python module (as opposed to a built-in module or a
dynamically loaded extension), the loader should execute the module's code in
the module's global name space (`module.__dict__`).

>   * If the loader cannot execute the module, it should raise an
[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError
"ImportError" ), although any other exception raised during [`exec_module()`](
https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_mod
ule "importlib.abc.Loader.exec_module" ) will be propagated.

>

In many cases, the finder and loader can be the same object; in such cases the
[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc
.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" ) method
would just return a spec with the loader set to `self`.

Module loaders may opt in to creating the module object during loading by
implementing a [`create_module()`](https://docs.python.org/3/library/importlib
.html#importlib.abc.Loader.create_module "importlib.abc.Loader.create_module"
) method. It takes one argument, the module spec, and returns the new module
object to use during loading. `create_module()` does not need to set any
attributes on the module object. If the method returns `None`, the import
machinery will create the new module itself.

New in version 3.4: The create_module() method of loaders.

Changed in version 3.4: The [`load_module()`](https://docs.python.org/3/librar
y/importlib.html#importlib.abc.Loader.load_module
"importlib.abc.Loader.load_module" ) method was replaced by [`exec_module()`](
https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_mod
ule "importlib.abc.Loader.exec_module" ) and the import machinery assumed all
the boilerplate responsibilities of loading.

For compatibility with existing loaders, the import machinery will use the
`load_module()` method of loaders if it exists and the loader does not also
implement `exec_module()`. However, `load_module()` has been deprecated and
loaders should implement `exec_module()` instead.

The `load_module()` method must implement all the boilerplate loading
functionality described above in addition to executing the module. All the
same constraints apply, with some additional clarification:

>   * If there is an existing module object with the given name in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ), the loader must use that existing module. (Otherwise, [`impor
tlib.reload()`](https://docs.python.org/3/library/importlib.html#importlib.rel
oad "importlib.reload" ) will not work correctly.) If the named module does
not exist in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ), the loader must create a new module object and add it to
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ).

>   * The module _must_ exist in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ) before the loader executes the module code, to prevent
unbounded recursion or multiple loading.

>   * If loading fails, the loader must remove any modules it has inserted
into [`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ), but it must remove **only** the failing module(s), and only
if the loader itself has loaded the module(s) explicitly.

>

Changed in version 3.5: A [`DeprecationWarning`](https://docs.python.org/3/lib
rary/exceptions.html#DeprecationWarning "DeprecationWarning" ) is raised when
`exec_module()` is defined but `create_module()` is not. Starting in Python
3.6 it will be an error to not define `create_module()` on a loader attached
to a ModuleSpec.

### 5.4.2. Submodules¶

When a submodule is loaded using any mechanism (e.g. `importlib` APIs, the
`import` or `import-from` statements, or built-in `__import__()`) a binding is
placed in the parent module's namespace to the submodule object. For example,
if package `spam` has a submodule `foo`, after importing `spam.foo`, `spam`
will have an attribute `foo` which is bound to the submodule. Let's say you
have the following directory structure:

[code]

    spam/
        __init__.py
        foo.py
        bar.py
    
[/code]

and `spam/__init__.py` has the following lines in it:

[code]

    from .foo import Foo
    from .bar import Bar
    
[/code]

then executing the following puts a name binding to `foo` and `bar` in the
`spam` module:

[code]

    >>> import spam
    >>> spam.foo
    <module 'spam.foo' from '/tmp/imports/spam/foo.py'>
    >>> spam.bar
    <module 'spam.bar' from '/tmp/imports/spam/bar.py'>
    
[/code]

Given Python's familiar name binding rules this might seem surprising, but
it's actually a fundamental feature of the import system. The invariant
holding is that if you have `sys.modules['spam']` and
`sys.modules['spam.foo']` (as you would after the above import), the latter
must appear as the `foo` attribute of the former.

### 5.4.3. Module spec¶

The import machinery uses a variety of information about each module during
import, especially before loading. Most of the information is common to all
modules. The purpose of a module's spec is to encapsulate this import-related
information on a per-module basis.

Using a spec during import allows state to be transferred between import
system components, e.g. between the finder that creates the module spec and
the loader that executes it. Most importantly, it allows the import machinery
to perform the boilerplate operations of loading, whereas without a module
spec the loader had that responsibility.

See [`ModuleSpec`](https://docs.python.org/3/library/importlib.html#importlib.
machinery.ModuleSpec "importlib.machinery.ModuleSpec" ) for more specifics on
what information a module's spec may hold.

New in version 3.4.

### 5.4.4. Import-related module attributes¶

The import machinery fills in these attributes on each module object during
loading, based on the module's spec, before the loader executes the module.

`__name__`¶

    

The `__name__` attribute must be set to the fully-qualified name of the
module. This name is used to uniquely identify the module in the import
system.

`__loader__`¶

    

The `__loader__` attribute must be set to the loader object that the import
machinery used when loading the module. This is mostly for introspection, but
can be used for additional loader-specific functionality, for example getting
data associated with a loader.

`__package__`¶

    

The module's `__package__` attribute must be set. Its value must be a string,
but it can be the same value as its `__name__`. When the module is a package,
its `__package__` value should be set to its `__name__`. When the module is
not a package, `__package__` should be set to the empty string for top-level
modules, or for submodules, to the parent package's name. See [**PEP
366**](https://www.python.org/dev/peps/pep-0366) for further details.

This attribute is used instead of `__name__` to calculate explicit relative
imports for main modules, as defined in [**PEP
366**](https://www.python.org/dev/peps/pep-0366). It is expected to have the
same value as `__spec__.parent`.

Changed in version 3.6: The value of `__package__` is expected to be the same
as `__spec__.parent`.

`__spec__`¶

    

The `__spec__` attribute must be set to the module spec that was used when
importing the module. Setting `__spec__` appropriately applies equally to
[modules initialized during interpreter startup](https://docs.python.org/3/ref
erence/toplevel_components.html#programs). The one exception is `__main__`,
where `__spec__` is set to None in some cases.

When `__package__` is not defined, `__spec__.parent` is used as a fallback.

New in version 3.4.

Changed in version 3.6: `__spec__.parent` is used as a fallback when
`__package__` is not defined.

`__path__`¶

    

If the module is a package (either regular or namespace), the module object's
`__path__` attribute must be set. The value must be iterable, but may be empty
if `__path__` has no further significance. If `__path__` is not empty, it must
produce strings when iterated over. More details on the semantics of
`__path__` are given below.

Non-package modules should not have a `__path__` attribute.

`__file__`¶

    

`__cached__`¶

    

`__file__` is optional. If set, this attribute's value must be a string. The
import system may opt to leave `__file__` unset if it has no semantic meaning
(e.g. a module loaded from a database).

If `__file__` is set, it may also be appropriate to set the `__cached__`
attribute which is the path to any compiled version of the code (e.g. byte-
compiled file). The file does not need to exist to set this attribute; the
path can simply point to where the compiled file would exist (see [**PEP
3147**](https://www.python.org/dev/peps/pep-3147)).

It is also appropriate to set `__cached__` when `__file__` is not set.
However, that scenario is quite atypical. Ultimately, the loader is what makes
use of `__file__` and/or `__cached__`. So if a loader can load from a cached
module but otherwise does not load from a file, that atypical scenario may be
appropriate.

### 5.4.5. module.__path__¶

By definition, if a module has an `__path__` attribute, it is a package,
regardless of its value.

A package's `__path__` attribute is used during imports of its subpackages.
Within the import machinery, it functions much the same as
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" ),
i.e. providing a list of locations to search for modules during import.
However, `__path__` is typically much more constrained than
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" ).

`__path__` must be an iterable of strings, but it may be empty. The same rules
used for [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path
"sys.path" ) also apply to a package's `__path__`, and
[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks
"sys.path_hooks" ) (described below) are consulted when traversing a package's
`__path__`.

A package's `__init__.py` file may set or alter the package's `__path__`
attribute, and this was typically the way namespace packages were implemented
prior to [**PEP 420**](https://www.python.org/dev/peps/pep-0420). With the
adoption of [**PEP 420**](https://www.python.org/dev/peps/pep-0420), namespace
packages no longer need to supply `__init__.py` files containing only
`__path__` manipulation code; the import machinery automatically sets
`__path__` correctly for the namespace package.

### 5.4.6. Module reprs¶

By default, all modules have a usable repr, however depending on the
attributes set above, and in the module's spec, you can more explicitly
control the repr of module objects.

If the module has a spec (`__spec__`), the import machinery will try to
generate a repr from it. If that fails or there is no spec, the import system
will craft a default repr using whatever information is available on the
module. It will try to use the `module.__name__`, `module.__file__`, and
`module.__loader__` as input into the repr, with defaults for whatever
information is missing.

Here are the exact rules used:

>   * If the module has a `__spec__` attribute, the information in the spec is
used to generate the repr. The "name", "loader", "origin", and "has_location"
attributes are consulted.

>   * If the module has a `__file__` attribute, this is used as part of the
module's repr.

>   * If the module has no `__file__` but does have a `__loader__` that is not
`None`, then the loader's repr is used as part of the module's repr.

>   * Otherwise, just use the module's `__name__` in the repr.

>

Changed in version 3.4: Use of [`loader.module_repr()`](https://docs.python.or
g/3/library/importlib.html#importlib.abc.Loader.module_repr
"importlib.abc.Loader.module_repr" ) has been deprecated and the module spec
is now used by the import machinery to generate a module repr.

For backward compatibility with Python 3.3, the module repr will be generated
by calling the loader's [`module_repr()`](https://docs.python.org/3/library/im
portlib.html#importlib.abc.Loader.module_repr
"importlib.abc.Loader.module_repr" ) method, if defined, before trying either
approach described above. However, the method is deprecated.

## 5.5. The Path Based Finder¶

As mentioned previously, Python comes with several default meta path finders.
One of these, called the [path based
finder](https://docs.python.org/3/glossary.html#term-path-based-finder) ([`Pat
hFinder`](https://docs.python.org/3/library/importlib.html#importlib.machinery
.PathFinder "importlib.machinery.PathFinder" )), searches an [import
path](https://docs.python.org/3/glossary.html#term-import-path), which
contains a list of [path entries](https://docs.python.org/3/glossary.html
#term-path-entry). Each path entry names a location to search for modules.

The path based finder itself doesn't know how to import anything. Instead, it
traverses the individual path entries, associating each of them with a path
entry finder that knows how to handle that particular kind of path.

The default set of path entry finders implement all the semantics for finding
modules on the file system, handling special file types such as Python source
code (`.py` files), Python byte code (`.pyc` files) and shared libraries (e.g.
`.so` files). When supported by the
[`zipimport`](https://docs.python.org/3/library/zipimport.html#module-
zipimport "zipimport: support for importing Python modules from ZIP archives."
) module in the standard library, the default path entry finders also handle
loading all of these file types (other than shared libraries) from zipfiles.

Path entries need not be limited to file system locations. They can refer to
URLs, database queries, or any other location that can be specified as a
string.

The path based finder provides additional hooks and protocols so that you can
extend and customize the types of searchable path entries. For example, if you
wanted to support path entries as network URLs, you could write a hook that
implements HTTP semantics to find modules on the web. This hook (a callable)
would return a [path entry finder](https://docs.python.org/3/glossary.html
#term-path-entry-finder) supporting the protocol described below, which was
then used to get a loader for the module from the web.

A word of warning: this section and the previous both use the term _finder_,
distinguishing between them by using the terms [meta path
finder](https://docs.python.org/3/glossary.html#term-meta-path-finder) and
[path entry finder](https://docs.python.org/3/glossary.html#term-path-entry-
finder). These two types of finders are very similar, support similar
protocols, and function in similar ways during the import process, but it's
important to keep in mind that they are subtly different. In particular, meta
path finders operate at the beginning of the import process, as keyed off the
[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path
"sys.meta_path" ) traversal.

By contrast, path entry finders are in a sense an implementation detail of the
path based finder, and in fact, if the path based finder were to be removed
from
[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path
"sys.meta_path" ), none of the path entry finder semantics would be invoked.

### 5.5.1. Path entry finders¶

The [path based finder](https://docs.python.org/3/glossary.html#term-path-
based-finder) is responsible for finding and loading Python modules and
packages whose location is specified with a string [path
entry](https://docs.python.org/3/glossary.html#term-path-entry). Most path
entries name locations in the file system, but they need not be limited to
this.

As a meta path finder, the [path based
finder](https://docs.python.org/3/glossary.html#term-path-based-finder)
implements the [`find_spec()`](https://docs.python.org/3/library/importlib.htm
l#importlib.abc.MetaPathFinder.find_spec
"importlib.abc.MetaPathFinder.find_spec" ) protocol previously described,
however it exposes additional hooks that can be used to customize how modules
are found and loaded from the [import
path](https://docs.python.org/3/glossary.html#term-import-path).

Three variables are used by the [path based
finder](https://docs.python.org/3/glossary.html#term-path-based-finder),
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" ),
[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks
"sys.path_hooks" ) and [`sys.path_importer_cache`](https://docs.python.org/3/l
ibrary/sys.html#sys.path_importer_cache "sys.path_importer_cache" ). The
`__path__` attributes on package objects are also used. These provide
additional ways that the import machinery can be customized.

[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )
contains a list of strings providing search locations for modules and
packages. It is initialized from the `PYTHONPATH` environment variable and
various other installation- and implementation-specific defaults. Entries in
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )
can name directories on the file system, zip files, and potentially other
"locations" (see the [`site`](https://docs.python.org/3/library/site.html
#module-site "site: Module responsible for site-specific configuration." )
module) that should be searched for modules, such as URLs, or database
queries. Only strings and bytes should be present on
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" );
all other data types are ignored. The encoding of bytes entries is determined
by the individual [path entry finders](https://docs.python.org/3/glossary.html
#term-path-entry-finder).

The [path based finder](https://docs.python.org/3/glossary.html#term-path-
based-finder) is a [meta path finder](https://docs.python.org/3/glossary.html
#term-meta-path-finder), so the import machinery begins the [import
path](https://docs.python.org/3/glossary.html#term-import-path) search by
calling the path based finder's [`find_spec()`](https://docs.python.org/3/libr
ary/importlib.html#importlib.machinery.PathFinder.find_spec
"importlib.machinery.PathFinder.find_spec" ) method as described previously.
When the `path` argument to [`find_spec()`](https://docs.python.org/3/library/
importlib.html#importlib.machinery.PathFinder.find_spec
"importlib.machinery.PathFinder.find_spec" ) is given, it will be a list of
string paths to traverse - typically a package's `__path__` attribute for an
import within that package. If the `path` argument is `None`, this indicates a
top level import and
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )
is used.

The path based finder iterates over every entry in the search path, and for
each of these, looks for an appropriate [path entry
finder](https://docs.python.org/3/glossary.html#term-path-entry-finder) ([`Pat
hEntryFinder`](https://docs.python.org/3/library/importlib.html#importlib.abc.
PathEntryFinder "importlib.abc.PathEntryFinder" )) for the path entry. Because
this can be an expensive operation (e.g. there may be stat() call overheads
for this search), the path based finder maintains a cache mapping path entries
to path entry finders. This cache is maintained in [`sys.path_importer_cache`]
(https://docs.python.org/3/library/sys.html#sys.path_importer_cache
"sys.path_importer_cache" ) (despite the name, this cache actually stores
finder objects rather than being limited to
[importer](https://docs.python.org/3/glossary.html#term-importer) objects). In
this way, the expensive search for a particular [path
entry](https://docs.python.org/3/glossary.html#term-path-entry) location's
[path entry finder](https://docs.python.org/3/glossary.html#term-path-entry-
finder) need only be done once. User code is free to remove cache entries from
[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.pat
h_importer_cache "sys.path_importer_cache" ) forcing the path based finder to
perform the path entry search again [3].

If the path entry is not present in the cache, the path based finder iterates
over every callable in
[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks
"sys.path_hooks" ). Each of the [path entry
hooks](https://docs.python.org/3/glossary.html#term-path-entry-hook) in this
list is called with a single argument, the path entry to be searched. This
callable may either return a [path entry
finder](https://docs.python.org/3/glossary.html#term-path-entry-finder) that
can handle the path entry, or it may raise
[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError
"ImportError" ). An
[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError
"ImportError" ) is used by the path based finder to signal that the hook
cannot find a [path entry finder](https://docs.python.org/3/glossary.html
#term-path-entry-finder) for that [path
entry](https://docs.python.org/3/glossary.html#term-path-entry). The exception
is ignored and [import path](https://docs.python.org/3/glossary.html#term-
import-path) iteration continues. The hook should expect either a string or
bytes object; the encoding of bytes objects is up to the hook (e.g. it may be
a file system encoding, UTF-8, or something else), and if the hook cannot
decode the argument, it should raise
[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError
"ImportError" ).

If
[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks
"sys.path_hooks" ) iteration ends with no [path entry
finder](https://docs.python.org/3/glossary.html#term-path-entry-finder) being
returned, then the path based finder's [`find_spec()`](https://docs.python.org
/3/library/importlib.html#importlib.machinery.PathFinder.find_spec
"importlib.machinery.PathFinder.find_spec" ) method will store `None` in [`sys
.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_imp
orter_cache "sys.path_importer_cache" ) (to indicate that there is no finder
for this path entry) and return `None`, indicating that this [meta path
finder](https://docs.python.org/3/glossary.html#term-meta-path-finder) could
not find the module.

If a [path entry finder](https://docs.python.org/3/glossary.html#term-path-
entry-finder) _is_ returned by one of the [path entry
hook](https://docs.python.org/3/glossary.html#term-path-entry-hook) callables
on
[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks
"sys.path_hooks" ), then the following protocol is used to ask the finder for
a module spec, which is then used when loading the module.

The current working directory - denoted by an empty string - is handled
slightly differently from other entries on
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" ).
First, if the current working directory is found to not exist, no value is
stored in [`sys.path_importer_cache`](https://docs.python.org/3/library/sys.ht
ml#sys.path_importer_cache "sys.path_importer_cache" ). Second, the value for
the current working directory is looked up fresh for each module lookup.
Third, the path used for [`sys.path_importer_cache`](https://docs.python.org/3
/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" ) and
returned by [`importlib.machinery.PathFinder.find_spec()`](https://docs.python
.org/3/library/importlib.html#importlib.machinery.PathFinder.find_spec
"importlib.machinery.PathFinder.find_spec" ) will be the actual current
working directory and not the empty string.

### 5.5.2. Path entry finder protocol¶

In order to support imports of modules and initialized packages and also to
contribute portions to namespace packages, path entry finders must implement
the [`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib
.abc.PathEntryFinder.find_spec "importlib.abc.PathEntryFinder.find_spec" )
method.

[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc
.PathEntryFinder.find_spec "importlib.abc.PathEntryFinder.find_spec" ) takes
two argument, the fully qualified name of the module being imported, and the
(optional) target module. `find_spec()` returns a fully populated spec for the
module. This spec will always have "loader" set (with one exception).

To indicate to the import machinery that the spec represents a namespace
[portion](https://docs.python.org/3/glossary.html#term-portion). the path
entry finder sets "loader" on the spec to `None` and
"submodule_search_locations" to a list containing the portion.

Changed in version 3.4: [`find_spec()`](https://docs.python.org/3/library/impo
rtlib.html#importlib.abc.PathEntryFinder.find_spec
"importlib.abc.PathEntryFinder.find_spec" ) replaced [`find_loader()`](https:/
/docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_l
oader "importlib.abc.PathEntryFinder.find_loader" ) and [`find_module()`](http
s://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.fin
d_module "importlib.abc.PathEntryFinder.find_module" ), both of which are now
deprecated, but will be used if `find_spec()` is not defined.

Older path entry finders may implement one of these two deprecated methods
instead of `find_spec()`. The methods are still respected for the sake of
backward compatibility. However, if `find_spec()` is implemented on the path
entry finder, the legacy methods are ignored.

[`find_loader()`](https://docs.python.org/3/library/importlib.html#importlib.a
bc.PathEntryFinder.find_loader "importlib.abc.PathEntryFinder.find_loader" )
takes one argument, the fully qualified name of the module being imported.
`find_loader()` returns a 2-tuple where the first item is the loader and the
second item is a namespace [portion](https://docs.python.org/3/glossary.html
#term-portion). When the first item (i.e. the loader) is `None`, this means
that while the path entry finder does not have a loader for the named module,
it knows that the path entry contributes to a namespace portion for the named
module. This will almost always be the case where Python is asked to import a
namespace package that has no physical presence on the file system. When a
path entry finder returns `None` for the loader, the second item of the
2-tuple return value must be a sequence, although it can be empty.

If `find_loader()` returns a non-`None` loader value, the portion is ignored
and the loader is returned from the path based finder, terminating the search
through the path entries.

For backwards compatibility with other implementations of the import protocol,
many path entry finders also support the same, traditional `find_module()`
method that meta path finders support. However path entry finder
`find_module()` methods are never called with a `path` argument (they are
expected to record the appropriate path information from the initial call to
the path hook).

The `find_module()` method on path entry finders is deprecated, as it does not
allow the path entry finder to contribute portions to namespace packages. If
both `find_loader()` and `find_module()` exist on a path entry finder, the
import system will always call `find_loader()` in preference to
`find_module()`.

## 5.6. Replacing the standard import system¶

The most reliable mechanism for replacing the entire import system is to
delete the default contents of
[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path
"sys.meta_path" ), replacing them entirely with a custom meta path hook.

If it is acceptable to only alter the behaviour of import statements without
affecting other APIs that access the import system, then replacing the builtin
[`__import__()`](https://docs.python.org/3/library/functions.html#__import__
"__import__" ) function may be sufficient. This technique may also be employed
at the module level to only alter the behaviour of import statements within
that module.

To selectively prevent import of some modules from a hook early on the meta
path (rather than disabling the standard import system entirely), it is
sufficient to raise `ModuleNoFoundError` directly from [`find_spec()`](https:/
/docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_sp
ec "importlib.abc.MetaPathFinder.find_spec" ) instead of returning `None`. The
latter indicates that the meta path search should continue, while raising an
exception terminates it immediately.

## 5.7. Special considerations for __main__¶

The [`__main__`](https://docs.python.org/3/library/__main__.html#module-
__main__ "__main__: The environment where the top-level script is run." )
module is a special case relative to Python's import system. As noted [elsewhe
re](https://docs.python.org/3/reference/toplevel_components.html#programs),
the `__main__` module is directly initialized at interpreter startup, much
like [`sys`](https://docs.python.org/3/library/sys.html#module-sys "sys:
Access system-specific parameters and functions." ) and
[`builtins`](https://docs.python.org/3/library/builtins.html#module-builtins
"builtins: The module that provides the built-in namespace." ). However,
unlike those two, it doesn't strictly qualify as a built-in module. This is
because the manner in which `__main__` is initialized depends on the flags and
other options with which the interpreter is invoked.

### 5.7.1. __main__.__spec__¶

Depending on how [`__main__`](https://docs.python.org/3/library/__main__.html
#module-__main__ "__main__: The environment where the top-level script is
run." ) is initialized, `__main__.__spec__` gets set appropriately or to
`None`.

When Python is started with the
[`-m`](https://docs.python.org/3/using/cmdline.html#cmdoption-m) option,
`__spec__` is set to the module spec of the corresponding module or package.
`__spec__` is also populated when the `__main__` module is loaded as part of
executing a directory, zipfile or other
[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )
entry.

In [the remaining cases](https://docs.python.org/3/using/cmdline.html#using-
on-interface-options) `__main__.__spec__` is set to `None`, as the code used
to populate the [`__main__`](https://docs.python.org/3/library/__main__.html
#module-__main__ "__main__: The environment where the top-level script is
run." ) does not correspond directly with an importable module:

  * interactive prompt
  * -c switch
  * running from stdin
  * running directly from a source or bytecode file

Note that `__main__.__spec__` is always `None` in the last case, _even if_ the
file could technically be imported directly as a module instead. Use the
[`-m`](https://docs.python.org/3/using/cmdline.html#cmdoption-m) switch if
valid module metadata is desired in
[`__main__`](https://docs.python.org/3/library/__main__.html#module-__main__
"__main__: The environment where the top-level script is run." ).

Note also that even when `__main__` corresponds with an importable module and
`__main__.__spec__` is set accordingly, they're still considered _distinct_
modules. This is due to the fact that blocks guarded by `if __name__ ==
"__main__":` checks only execute when the module is used to populate the
`__main__` namespace, and not during normal import.

## 5.8. Open issues¶

XXX It would be really nice to have a diagram.

XXX * (import_machinery.rst) how about a section devoted just to the
attributes of modules and packages, perhaps expanding upon or supplanting the
related entries in the data model reference page?

XXX runpy, pkgutil, et al in the library manual should all get "See Also"
links at the top pointing to the new import system section.

XXX Add more explanation regarding the different ways in which `__main__` is
initialized?

XXX Add more info on `__main__` quirks/pitfalls (i.e. copy from [**PEP
395**](https://www.python.org/dev/peps/pep-0395)).

## 5.9. References¶

The import machinery has evolved considerably since Python's early days. The
original [specification for
packages](http://legacy.python.org/doc/essays/packages.html) is still
available to read, although some details have changed since the writing of
that document.

The original specification for
[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path
"sys.meta_path" ) was [**PEP 302**](https://www.python.org/dev/peps/pep-0302),
with subsequent extension in [**PEP
420**](https://www.python.org/dev/peps/pep-0420).

[**PEP 420**](https://www.python.org/dev/peps/pep-0420) introduced [namespace
packages](https://docs.python.org/3/glossary.html#term-namespace-package) for
Python 3.3. [**PEP 420**](https://www.python.org/dev/peps/pep-0420) also
introduced the `find_loader()` protocol as an alternative to `find_module()`.

[**PEP 366**](https://www.python.org/dev/peps/pep-0366) describes the addition
of the `__package__` attribute for explicit relative imports in main modules.

[**PEP 328**](https://www.python.org/dev/peps/pep-0328) introduced absolute
and explicit relative imports and initially proposed `__name__` for semantics
[**PEP 366**](https://www.python.org/dev/peps/pep-0366) would eventually
specify for `__package__`.

[**PEP 338**](https://www.python.org/dev/peps/pep-0338) defines executing
modules as scripts.

[**PEP 451**](https://www.python.org/dev/peps/pep-0451) adds the encapsulation
of per-module import state in spec objects. It also off-loads most of the
boilerplate responsibilities of loaders back onto the import machinery. These
changes allow the deprecation of several APIs in the import system and also
addition of new methods to finders and loaders.

Footnotes

[1]| See [`types.ModuleType`](https://docs.python.org/3/library/types.html#typ
es.ModuleType "types.ModuleType" ).  
---|---  
[2]| The importlib implementation avoids using the return value directly.
Instead, it gets the module object by looking the module name up in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ). The indirect effect of this is that an imported module may
replace itself in
[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules
"sys.modules" ). This is implementation-specific behavior that is not
guaranteed to work in other Python implementations.  
---|---  
[3]| In legacy code, it is possible to find instances of [`imp.NullImporter`](
https://docs.python.org/3/library/imp.html#imp.NullImporter "imp.NullImporter"
) in the [`sys.path_importer_cache`](https://docs.python.org/3/library/sys.htm
l#sys.path_importer_cache "sys.path_importer_cache" ). It is recommended that
code be changed to use `None` instead. See [Porting Python
code](https://docs.python.org/3/whatsnew/3.3.html#portingpythoncode) for more
details.  
