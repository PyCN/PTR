原文：[5. The import system](https://docs.python.org/3/reference/import.html)

---

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


    parent/
        __init__.py
        one/
            __init__.py
        two/
            __init__.py
        three/
            __init__.py
    

导入`parent.one`将会隐式执行`parent/__init__.py`和`parent/one/__init__.py`。随后对`parent.two`或者`parent.three`的导入将会分别执行`parent/two/__init__.py`和`parent/three/__init__.py`。

### 5.2.2. 名字空间包

一个名字空间包是各种[部分(portion)](https://docs.python.org/3/glossary.html#term-portion)的组成，其中，每个部分都共享了一个子包给父包。这些部分可能位于文件系统中的不同位置。这些部分也有可能位于zip文件中、网络上、或者任何其他Python在导入时搜索的位置。名字空间包可能或者可能不直接对应于文件系统上的对象；它们或许是没有实体的虚拟模块。

名字空间包的`__path__`属性并不使用一个普通的列表。相反，它们使用一个自定义的可迭代类型，如果它们父包的路径(或者对于一个顶级包，[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" ))发生了改变，会在那个包中的下一次导入尝试时，自动进行对包部分的新的搜索。

对于名字空间包，没有`parent/__init__.py`文件。事实上，在导入搜索过程中，可能会找到多个`parent`目录，其中，每一个都是由一个不同的部分提供的。因此，`parent/one`在物理上，可能不会挨着`parent/two`。在这种情况下，每当导入顶级`parent`包或者它其中一个子包时，Python将会为这个顶级`parent`包创建一个名字空间包。

另见[**PEP 420**](https://www.python.org/dev/peps/pep-0420)，了解名字空间包的详细说明。

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

Python包含了大量的默认查找器和导入器。第一个知道如何定位到内置模块，而第二个知道如何定位到冻结模块。第三哥默认的查找器搜索[导入路径](https://docs.python.org/3/glossary.html#term-import-path)来查找模块。[导入路径](https://docs.python.org/3/glossary.html#term-import-path)是一个位置列表，这些位置可能是命名文件系统路径，或者zip文件。也可以扩展以搜索任何可定位资源，例如那些由URL标识的资源。

导入机制是可扩展的，因此，可以添加新的查找器来扩展模块搜索的范围。

查找器实际上并不加载模块。如果它们可以找到命名模块，那么它们返回一个_模块spec(module spec)_，它是模块导入相关信息的封装，导入机制稍后会在加载模块的时候使用它们。

以下部分更详细地描述了查找器和加载器的协议，包括你可以如何创建和注册新的来扩展导入机制。

版本3.4改动：在Python前面的版本中，查找器直接返回[加载器](https://docs.python.org/3/glossary.html#term-loader)，而现在，它们返回模块spec，其中_包含_加载器。仍会在导入期间使用加载器，但它承担较少的责任。

### 5.3.3. 导入钩子

导入机制被设计为可扩展的；而其主要机制是_导入钩子_。有两种类型的导入钩子：_元钩子_和_导入路径钩子_。

元钩子是在导入过程的开始，任何其他导入过程发生之前调用的，不用于[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )缓存查找。这样，元钩子就可以覆盖[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )处理，冻结模块，或者甚至是内置模块。元钩子是通过添加新的查找器对象到[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )来注册的，如下所述。

导入路径钩子作为[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )(或者`package.__path__`)处理的一部分来调用，在碰到它们相关联路径入口的那个时候。导入路径钩子是通过添加新的可回调对象到[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks" )来注册的，如下所述。

### 5.3.4. 元路径

当在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中找不到命名模块时，Python接下来会搜索[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )，它包含了一个元路径查找器对象列表。查询这些查找器来看看它们是否知道如何处理该命名模块。元路径查找器必须实现一个名为[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )的方法，这个方法接受三个参数：一个名字，一个导入路径，和（可选的）一个目标模块。元路径查找器可以使用任何它想要的策略来决定它是否可以处理该命名模块。

如果该元路径查找器知道如何处理这个命名模块，那么它返回一个spec对象。如果它不能处理这个命名模块，那么它返回`None`。如果[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )处理到达了它的列表的尾部都没有返回一个spec，那么会抛出一个[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。抛出的任何其他异常将会简单地向外层传播，中止导入过程。

带两个或三个参数调用元路径查找器的[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )方法。第一个是导入模块的完整限定名，例如，`foo.bar.baz`。第二个是用来进行模块搜索的路径入口。对于顶级模块，第二个参数是`None`，但是对于子模块或者子包，第二个参数是父包的`__path__`属性。如果不能访问适当的`__path__`属性，那么会抛出一个[`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError" )。第三个参数是一个存在的module对象，它将会是稍后的加载目标。导入系统只有在重载期间才会传递进一个目标模块。

对于单个导入请求，元路径也许会被多次遍历。例如，假设涉及的模块都没有被缓存，那么导入`foo.bar.baz`会首先执行一次顶级导入，在么个元路径查找器(`mpf`)上调用`mpf.find_spec("foo", None, None)`。在导入`foo`之后，将通过第二次遍历元路径，调用`mpf.find_spec("foo.bar", foo.__path__, None)`来导入`foo.bar`。一旦导入了`foo.bar`，最后一次遍历将会调用`mpf.find_spec("foo.bar.baz", foo.bar.__path__, None)`。

一些元路径查找器只支持顶级导入。当把非`None`作为第二个参数传递时，这些导入器将总是返回`None`。

Python默认的[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )有三个元路径查找器，一个直到如何导入内置模块，一个知道如何导入冻结模块，而另一个知道如何导入来自[导入路径](https://docs.python.org/3/glossary.html#term-import-path) (例如，[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder))的模块。

版本3.4改动：元路径查找器的[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )方法替代了[`find_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_module "importlib.abc.MetaPathFinder.find_module" )，后者已经弃用了。虽然它将在无需改动的情况下继续工作，但是导入机制只有在查找器没有实现`find_spec()`的情况下才会试着使用它。

## 5.4. 加载

如果（当）找到了一个模块spec，在加载该模块的时候，导入机制将会使用它 (以及它包含的加载器)。这里是在导入的加载部分过程中所发生的事情的近似：


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
    

注意以下细节：

>   * 如果在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中，对于给定的名字，已经有一个module对象了，那么导入将会返回它。

>   * 在加载器执行模块代码之前，模块将会存在于[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中。这至关重要，因为模块代码也许（直接或者间接）导入自身；预先将其添加到[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )防止最糟糕情况下的无限递归以及最佳情况下的多次加载。

>   * 如果加载失败，那么失败的模块，并且只有失败的模块，会从[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中移除。任何已经在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )缓存中模块，以及任何作为副作用成功加载的模块，必须保留在缓存中。这与重载相反，对于后者而言，即使是失败模块，也会被留在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中。

>   * 在创建模块之后，执行模块之前，导入机制设置导入相关的模块属性 (上面的伪代码样例中的"_init_module_attrs")，如在[后面部分](https://docs.python.org/3/reference/import.html#import-mod-attrs)中所提到的。

>   * 模块执行时加载的关键之处，其中，模块的命名空间被填充。执行被完全委托给加载器，它决定要填充什么，以及如何填充。

>   * 在加载期间创建以及被传递给exec_module()的模块也许并不是在导入结束时返回的那个 [2]。

>

版本3.4改动：导入系统已经接管了加载器的样板功能。这些之前是由[`importlib.abc.Loader.load_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.load_module "importlib.abc.Loader.load_module" )方法所执行的。

### 5.4.1. 加载器

模块加载器提供加载的关键功能：模块执行。导入机制带单个参数（要执行的module对象）调用[`importlib.abc.Loader.exec_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module" )方法。忽略任何从[`exec_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module" )返回的值。

加载器必须满足以下要求：

>   * 如果模块是一个Python模块 (而不是一个内置模块或者一个动态加载的扩展)，那么加载器应该在模块的全局名字空间(`module.__dict__`)中执行该模块的代码。

>   * 如果加载器不能够执行该模块，那么它应该抛出一个[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError
"ImportError" )，虽然任何其他在[`exec_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_mod ule "importlib.abc.Loader.exec_module" )期间引发的异常都将被扩散。

>

在许多情况下，查找器和加载器可以是同一个对象；在这样的场景下，[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )方法会值返回一个加载器被设置成`self`的spec。

模块加载器可以通过实现[`create_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.create_module "importlib.abc.Loader.create_module")方法，在加载期间，选择创建module对象。这个方法接受一个参数，模块spec(module spec)，并返回加载期间要使用的新的module对象。`create_module()`并不需要在module对象上设置任何属性。如果该方法返回`None`，那么，导入机制将会创建新模块自身。

版本3.4新特性：加载器的create_module()方法。

版本3.4改动：[`load_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.load_module "importlib.abc.Loader.load_module" )方法被[`exec_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.exec_module "importlib.abc.Loader.exec_module" )替代，而导入机制承担所有加载的样板责任。

要与现有的加载器兼容，如果加载器的`load_module()`存在，并且加载器也没有实现`exec_module()`，那么导入机制会使用加载器的`load_module()`方法。但是，`load_module()`已被启用，而加载器应该实现`exec_module()`。

除了执行模块，`load_module()`方法必须实现上述的所有样板加载功能。所有相同的约束都适用，有一些而外的澄清：

>   * 如果在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中，存在与现有名字相同的module对象，那么加载器必须使用哪个现有模块。(否则，[`importlib.reload()`](https://docs.python.org/3/library/importlib.html#importlib.reload "importlib.reload" )不会正确工作。) 如果命名模块在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中并不存在，那么加载器必须创建一个新的module对象，并把它添加到[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中。

>   * 在加载器执行模块代码之前，模块_必须_存在于[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )之中，以防止无限递归或者多重加载。

>   * 如果加载失败，那么加载器必须移除任何它已经插入到[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中的模块，但是它必须**只**移除失败的模块，并且只有加载器自身已明确加载的那些模块。

>

版本3.5改动： 当定义了`exec_module()`，而未定义`create_module()`时，会抛出[`DeprecationWarning`](https://docs.python.org/3/library/exceptions.html#DeprecationWarning "DeprecationWarning" )。从Python 3.6开始，不在附加到一个ModuleSpec上的加载器上定义`create_module()`，这种情况将会是一种错误。

### 5.4.2. 子模块

当使用任意机制(例如，`importlib` API，`import`或者`import-from`语句，或者内置的`__import__()`)加载一个子模块时，会在父模块的名字空间中对子模块对象进行绑定。例如，如果在导入`spam.foo`之后，包`spam`有一个子模块`foo`，那么`spam`将会有一个属性`foo`，它被绑定到该子模块。假设你有以下目录结构：


    spam/
        __init__.py
        foo.py
        bar.py
    

而在`spam/__init__.py`中，有以下行：


    from .foo import Foo
    from .bar import Bar
    

那么，执行以下代码，会在`spam`模块中放置一个到`foo`和`foo`的名字绑定：


    >>> import spam
    >>> spam.foo
    <module 'spam.foo' from '/tmp/imports/spam/foo.py'>
    >>> spam.bar
    <module 'spam.bar' from '/tmp/imports/spam/bar.py'>
    

给定Python熟悉的名字绑定规则，这也许看起来令人惊讶，但是它实际上是导入系统的一个基本特征。不变的是，如果你有`sys.modules['spam']`和`sys.modules['spam.foo']` (在上面的导入之后，就会有)，那么后者必须作为前者的`foo`属性出现。

### 5.4.3. 模块spec（Module spec）

在导入过程中，特别是在加载之前，导入机制使用关于每个模块的各种信息。大部分的信息对所有模块而言都是通用的。模块spec的目的在于，在每个模块基础上，封装导入相关的信息。

在导入期间使用一个spec允许在导入系统组件之间传输状态，例如，在创建模块spec的查找器和执行spec的加载器之间。最重要的是，它允许导入机制执行加载的样板操作，反之，没有模块spec，加载器则承担那个责任。

见[`ModuleSpec`](https://docs.python.org/3/library/importlib.html#importlib.machinery.ModuleSpec "importlib.machinery.ModuleSpec" )，了解关于模块可能包含的信息的更多详细信息。

版本3.4新特性。

### 5.4.4. 导入相关的模块属性

在加载器执行模块之前，加载期间，基于模块spec，导入机制在每个module对象上填充这些属性。

`__name__`

    

`__name__`属性必须设置为模块的完全限定名。这个名字被用来在导入系统中唯一标识该模块。

`__loader__`

    

`__loader__`属性必须设置为导入机制在加载该模块的时候使用的加载器对象。这大部分是用于自省，但可以被用于额外的加载特定的功能，例如，获取与加载器关联的数据。

`__package__`

    

必须设置模块的`__package__`属性。它的值必须是一个字符串，但是，它可以是与`__name__`一样的值。当该模块是一个包的时候，必须设置它的`__package__`值为它的`__name__`。当该模块不是一个包的时候，对于顶级模块，应该设置`__package__`为空字符串，或者对于子模块，应该设置为其父包名。见[**PEP 366**](https://www.python.org/dev/peps/pep-0366)，以了解更多细节。

这个属性被用来取代`__name__`计算主模块的显式相对导入，如[**PEP 366**](https://www.python.org/dev/peps/pep-0366)中定义。期望它拥有与`__spec__.parent`相同的值。

版本3.6改动：期望`__package__`的值与`__spec__.parent`相同。

`__spec__`

    

必须设置`__spec__`属性为在导入该模块时使用的模块spec。适当地设置`__spec__`同样适用于[解释器启动期间初始化的模块](https://docs.python.org/3/reference/toplevel_components.html#programs)。唯一的例外是`__main__`，这里，在某些情况下，`__spec__`被设置为None。

当`__package__`未定义时，`__spec__.parent`被当成回退使用。

版本3.4新特性.

版本3.6改动：当`__package__`未定义时，`__spec__.parent`被当成回退使用。

`__path__`

    

如果模块是一个包 (无论是常规的还是名字空间)，都必须设置module对象的`__path__`属性。值必须是可迭代的，但如果`__path__`没有进一步的意义，则可以为空。如果`__path__`非空，那么在迭代的时候，必须生成字符串。下面给出了关于`__path__`语义的更多细节。

非包模块不应该有`__path__`属性。

`__file__`

    

`__cached__`

    

`__file__`是可选的。如果设置了这个属性，那么它的值必须是字符串。如果导入系统没有语义（例如，一个从数据库加载的模块），则可以保留`__file__`未设置。

如果设置了`__file__`，那么也可以适当地设置`__cached__`属性，这是到代码的任何编译版本 (例如，字节编译文件) 的路径。要设置这个属性，文件无需存在；路径可以简单指向编译文件将存在的地方 (见[**PEP 3147**](https://www.python.org/dev/peps/pep-3147)).

当未设置`__file__`的时候，也可以适当地设置`__cached__`。然而，这种情况是相当不正常的。最终，加载器是利用`__file__`和/或者`__cached__`的那个东东。因此，如果一个加载器可以从一个已缓存模块加载，但不能从文件加载，那么，那个非典型场景也许是适当的。

### 5.4.5. module.__path__

根据定义，如果一个模块拥有一个`__path__`属性，那么它是一个包，而不管它的值是什么。

在一个包的子包的导入期间，会使用这个包的`__path__`属性。在导入机制中，它的功能几乎与[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )一致，例如，提供导入期间用以搜索模块的位置列表。然而，`__path__`通常比[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )更多约束。

`__path__`必须是字符串的可迭代对象，但它也可以是空的。用于[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )的规则同样也适用于包的`__path__`，而在遍历一个包的`__path__`的时候，会参考[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks" ) (下述)。

一个包的`__init__.py`文件也许会设置或者改变该包的`__path__`属性，这通常是名字空间包在[**PEP 420**](https://www.python.org/dev/peps/pep-0420)之前实现的方式。随着[**PEP 420**](https://www.python.org/dev/peps/pep-0420)的采纳，名字空间包不再需要提供只包含`__path__`操作代码的`__init__.py`文件；导入机制自动为名字空间包正确设置`__path__`。

### 5.4.6. 模块repr

默认情况下，所有的模块都有一个可用的repr，然而，根据上面设置的属性，在模块spec中，可以更明确地控制module对象的repr。

如果模块有一个spec (`__spec__`)，那么导入机制将会尝试从中生成一个repr。如果失败了，或者没有spec，那么导入系统会利用该模块上的任何可用信息来创建默认的repr。它会试着使用`module.__name__`，`module.__file__`，和`module.__loader__`作为repr的输入，对于任何缺失的信息，使用默认值。

以下是使用的确切规则：

>   * 如果模块拥有一个`__spec__`属性，那么spec重的信息会被用来生成repr。会参考"name", "loader", "origin", 和"has_location"属性。

>   * 如果模块拥有一个`__file__`属性，那么这会作为模块的repr的部分使用。

>   * 如果模块没有`__file__`，但有一个非`None`的`__loader__`，那么该加载器的repr会作为模块的repr的部分使用。

>   * 否则，只需在repr中使用模块的`__name__`。

>

版本3.4改动：[`loader.module_repr()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.module_repr "importlib.abc.Loader.module_repr" )的使用已被启用，而现在，导入机制使用模块spec来生成一个模块repr。

要向后兼容Python 3.3，在尝试任何上述方法之前，如果定义了加载器的[`module_repr()`](https://docs.python.org/3/library/importlib.html#importlib.abc.Loader.module_repr "importlib.abc.Loader.module_repr" )方法，那么将会调用它来生成模块repr。但是，这个方法已弃用。

## 5.5. 基于路径的查找器

如前所述，Python附带了几个默认的元路径查找器。其中之一称为[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder) ([`PathFinder`](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder "importlib.machinery.PathFinder" ))，它搜索一个[导入路径](https://docs.python.org/3/glossary.html#term-import-path)，其中包含一个[路径入口](https://docs.python.org/3/glossary.html #term-path-entry)列表。每个路径入口命名一个搜索模块的位置。

基于路径的查找器自身并不知道如何导入。相反，它遍历各个路径入口，将它们每个与知道如何处理该特定类型的路径的路径入口查找器相关联。

路径入口查找器的默认集实现了在文件系统上查找模块的所有语义，处理特殊文件类型，例如Python源代码 (`.py`文件)，Python字节码(`.pyc`文件)，以及共享库 (例如，`.so`文件)。当标准库中的[`zipimport`](https://docs.python.org/3/library/zipimport.html#module-zipimport "zipimport: support for importing Python modules from ZIP archives.")模块支持的时候，默认路径入口查找器还处理来自zip文件的所有这些文件类型 (除共享库外) 的加载。

路径入口不必限于文件系统位置。它们可以适用于URL，数据库查询，或者任何其他可以指定为字符串的位置。

基于路径的查找器提供额外的钩子和协议，以便你能扩展和自定义可搜索路径入口的类型。例如，如果你想要支持网络URL作为路径入口，那么你可以编写一个实现了HTTP语义来在网上查找模块的钩子。这个钩子 (一个可调用对象) 将返回一个[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)，支持下述协议，然后用来从网上获取模块的加载器。

提醒一句：本节和前面部分都使用了术语_查找器_，通过使用术语[元路径查找器](https://docs.python.org/3/glossary.html#term-meta-path-finder)和[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)来区分它们。这两个类型的查找器是非常相似的，支持类似协议，并且在导入过程中功能类似，但是请务必记住，它们有微妙的区别。特别是，元路径查找器在导入过程的开始时执行，切断[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )遍历。

相比之下，某种意义上，路径入口查找器是基于路径的查找器一种实现细节，并且，实际上，如果将基于路径的查找器从[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )移除，那么，不会调用任何路径入口查找器语义。

### 5.5.1. 路径入口查找器

[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-based-finder)负责查找和加载位置由一个字符串[路径入口](https://docs.python.org/3/glossary.html#term-path-entry)指定的Python模块和包。大部分的路径入口在文件系统中命名位置，但是它们无需受限于此。

作为一个元路径查找器，[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder)实现了前面提到的[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )协议，但是，它公开了额外的钩子，可以用来自定义怎样查找并从[导入路径](https://docs.python.org/3/glossary.html#term-import-path)中加载模块。

[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder)使用三个变量：[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )，[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks" )和[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" )。也使用包对象上的`__path__`属性。这些提供了可以自定义导入机制的额外的方法。

[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )包含了模块和包的搜索位置字符串组成的列表。它是根据`PYTHONPATH`环境变量和各种其他安装以及实现相关的默认值进行初始化的。[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )中的项可以在文件系统、zip文件和潜在的其他应该可以用来搜索模块的“位置” (见[`site`](https://docs.python.org/3/library/site.html#module-site "site: Module responsible for site-specific configuration." )，例如URL，或者数据库查询，上命名目录。[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )应该只存在字符串和字节；忽略所有其他数据类型。字节项的编码由各个[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)决定。

[基于路径的查找器](https://docs.python.org/3/glossary.html#term-path-based-finder)是一种[元路径查找器](https://docs.python.org/3/glossary.html#term-meta-path-finder)，因此，导入机制通过调用前面描述的基于路径的查找器的[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.machinery.athFinder.find_spec "importlib.machinery.PathFinder.find_spec" )方法来开始[导入路径](https://docs.python.org/3/glossary.html#term-import-path)搜索。当为[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder.find_spec "importlib.machinery.PathFinder.find_spec" )提供了`path`参数值时，这个值将会是要遍历的字符串路径列表 —— 通常是一个包中用于导入的包的`__path__`属性。如果`path`参数值是`None`，表示这是一个顶级导入，会使用[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )。

基于路径的查找器遍历搜索路径中的每一项，对于它们中的每一个，为其寻找一个适当的[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder) ([`PathEntryFinder`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder "importlib.abc.PathEntryFinder" ))。由于这可能会是一种昂贵的操作 (例如，对于这种搜索，可能存在stat()调用开销)，因此，该基于路径的查找器维护一个缓存，将路径入口映射到路径入口查找器。这个缓存是在[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" )中维护的 (尽管名字如此，但是这个缓存实际上存储了查找器对象，而不是受限于[导入器](https://docs.python.org/3/glossary.html#term-importer) objects)。通过这种方式，对于特定[路径入口](https://docs.python.org/3/glossary.html#term-path-entry)位置的[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)的昂贵搜索，只需要进行一次。用户代码可以自由地将缓存项从[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" )中删除，强制基于路径的查找器再次进行路径入口搜索 [3]。

如果路径入口并不存在于缓存中，那么基于路径的查找器会迭代[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks" )中的每个可调用对象。这个列表中的每个[路径入口钩子](https://docs.python.org/3/glossary.html#term-path-entry-hook)在调用时带单个参数，即要搜索的路径入口。这个可调用对象要么返回一个可以处理该路径入口的[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)，要么引发[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError "ImportError" )。基于路径的查找器使用[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError "ImportError" )来表示钩子无法为那个[路径入口](https://docs.python.org/3/glossary.html#term-path-entry)找到一个[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)。忽略该异常，并继续[导入路径](https://docs.python.org/3/glossary.html#term-import-path)迭代。该钩子应期望一个字符串或者字节对象参数；字节对象的编码取决于钩子 (例如，它可能是一个文件系统编码，UTF-8，或其他什么的)，如果钩子不能解码参数，那么它应该引发[`ImportError`](https://docs.python.org/3/library/exceptions.html#ImportError "ImportError" )。

如果[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks" )迭代的结果是不返回任何一个[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)，那么，基于路径的查找器的[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder.find_spec "importlib.machinery.PathFinder.find_spec" )方法将会把`None`存在[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" )中 (表示对于这个路径入口，没有查找器)，并返回`None`，表示这个[元路径查找器](https://docs.python.org/3/glossary.html#term-meta-path-finder)无法找到此模块。

如果[`sys.path_hooks`](https://docs.python.org/3/library/sys.html#sys.path_hooks "sys.path_hooks" )上的任意一个[路径入口钩子](https://docs.python.org/3/glossary.html#term-path-entry-hook)可调用对象返回了一个[路径入口查找器](https://docs.python.org/3/glossary.html#term-path-entry-finder)，那么会使用以下协议来向查找器获取一个模块spec，这在稍后加载该模块时会被使用。

当前工作目录（由空字符串表示）与[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )上的其他入口的处理方式稍有不同。首先，如果发现当前工作目录不存在，那么，不在[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" )中存储任何值。第二，对于每次模块查找，会搜索刷新的当前工作目录的值。第三，用于[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" )和[`importlib.machinery.PathFinder.find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.machinery.PathFinder.find_spec "importlib.machinery.PathFinder.find_spec" )返回的路径将会是真正的当前工作目录，而不是空字符串。

### 5.5.2. 路径入口查找器协议

为了支持模块和已初始包的导入，以及贡献部分到名字空间包中，路径入口查找器必须实现[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_spec "importlib.abc.PathEntryFinder.find_spec" )方法。

[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_spec "importlib.abc.PathEntryFinder.find_spec" )接受两个参数，导入模块的完全限定名，以及（可选的）目标模块。`find_spec()`返回模块的一个完全填充的spec。这个spec将总是拥有“加载器”集合 (有一个例外)。

要向导入机制表明该spec表示一个名字空间[部分](https://docs.python.org/3/glossary.html#term-portion)，路径入口查找器设置spec上的“加载器”为`None`，设置"submodule_search_locations"为一个包含该部分的列表。

版本3.4改动： [`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_spec "importlib.abc.PathEntryFinder.find_spec" )替换了[`find_loader()`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_loader "importlib.abc.PathEntryFinder.find_loader" )和[`find_module()`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_module "importlib.abc.PathEntryFinder.find_module" )，这两个现在都已弃用了，但如果未定义`find_spec()`，则仍会使用它们。

较老的路径入口查找器可能实现这两个已弃用方法其中的一个，而不是`find_spec()`。出于向后兼容，这些方法仍然会被尊重。然而，如果在路径入口查找器上实现了`find_spec()`，那么就会忽略这些遗留方法。

[`find_loader()`](https://docs.python.org/3/library/importlib.html#importlib.abc.PathEntryFinder.find_loader "importlib.abc.PathEntryFinder.find_loader" )接受一个参数，导入模块的一个完全受限名。`find_loader()`返回一个二元元组，其中，第一个元素是加载器，第二个元素则是一个名字空间[部分](https://docs.python.org/3/glossary.html#term-portion)。当第一个元素 (即，加载器) 为`None`时，意味着，对于这个命名模块，虽然路径入口查找器没有一个加载器，但是它知道，路径入口贡献给了一个名字空间部分。这将几乎总是这种情况：让Python导入一个在文件系统上没有物理实体的名字空间包。当一个路径入口查找器返回的加载器为`None`的时候，二元元组的第二个元素的返回值必须是一个序列，虽然它可以是空的。

如果`find_loader()`返回一个非`None`加载器值，那么，忽略部分(portion)，并从基于路径的查找器返回加载器，终止对路径入口的搜索。

为了与导入协议的其他实现向后兼容，许多路径入口查找器也支持相同，传统的，为元路径查找器所支持的`find_module()`方法。然而，从不用一个`path`参数调用路径入口查找器的`find_module()`方法 (期望它们记录来自于对路径钩子的初始调用的适当的路径信息)。

路径入口查找器上的`find_module()`方法已被启用，因为它不允许路径入口查找器共享部分到名字空间包中。如果在一个路径入口查找器上同时存在`find_loader()`和`find_module()`，那么那么，导入系统将总是优先于`find_module()`调用`find_loader()`。

## 5.6. 替换标准输入系统

替换整个导入系统的最可靠的机制是删除[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )中的默认内容，用一个自定义的元路径钩子来完全替换它们。

如果可以接受在不影响其他访问导入系统的API的情况下，只改变导入语句的行为，那么，替换内置的[`__import__()`](https://docs.python.org/3/library/functions.html#__import__ "__import__" )函数可能就足够了。这种技术也可以在模块级别使用，用来仅改变那个模块中的导入语句行为。

在元路径早期，要选择性地阻止钩子上一些模块的导入 (而不是完全禁用标准导入系统)，则直接从[`find_spec()`](https://docs.python.org/3/library/importlib.html#importlib.abc.MetaPathFinder.find_spec "importlib.abc.MetaPathFinder.find_spec" )抛出`ModuleNoFoundError`，而不是返回`None`。后者暗示元路径搜索应该继续下去，而引发一个异常会立即终止它。

## 5.7. 对于__main__的特殊考量

相对于Python的导入系统，[`__main__`](https://docs.python.org/3/library/__main__.html#module-__main__ "__main__: The environment where the top-level script is run." )模块是一种特殊情况。如[在其他地方](https://docs.python.org/3/reference/toplevel_components.html#programs)所述的，`__main__`模块是在解释器启动时被直接初始化的，就像[`sys`](https://docs.python.org/3/library/sys.html#module-sys "sys:Access system-specific parameters and functions." )和[`builtins`](https://docs.python.org/3/library/builtins.html#module-builtins "builtins: The module that provides the built-in namespace." )一样。但是，和那两个不一样的是，它并不是严格意义上的内置模块。这是因为，初始化`__main__`的方式取决于调用解释器的标志和其他选项。

### 5.7.1. __main__.__spec__

根据[`__main__`](https://docs.python.org/3/library/__main__.html #module-__main__ "__main__: The environment where the top-level script is run." )的初始化方式，会正确地设置`__main__.__spec__`，或者设为`None`。

当带[`-m`](https://docs.python.org/3/using/cmdline.html#cmdoption-m)选项启动Python时，`__spec__`会被设置为相应模块或包的模块spec。当`__main__`模块作为执行一个目录、zip文件或者其他[`sys.path`](https://docs.python.org/3/library/sys.html#sys.path "sys.path" )入口的一部分加载的时候，也会填充`__spec__`。

在[其他情况](https://docs.python.org/3/using/cmdline.html#using-on-interface-options)下，设置`__main__.__spec__`为`None`，因为用来填充[`__main__`](https://docs.python.org/3/library/__main__.html#module-__main__ "__main__: The environment where the top-level script is run." )的代码不直接与一个可导入模块相对应：

  * 交互提示
  * -c交换
  * 从标准输入运行
  * 直接从一个源代码或者字节码文件运行

注意，在最后一种情况下，`__main__.__spec__`总是`None`，_即使_从技术上来讲，可以把文件当成模块直接导入。如果在[`__main__`](https://docs.python.org/3/library/__main__.html#module-__main__ "__main__: The environment where the top-level script is run." )中，需要有效的模块元数据，那么使用[`-m`](https://docs.python.org/3/using/cmdline.html#cmdoption-m)交换。

还要注意，即使当`__main__`对应一个可导入模块，并且对应设置了`__main__.__spec__`，但它们仍然会被当成_有区别的_模块。这是因为，由`if __name__ == "__main__":`所监管的块检查只有当该模块被用来填充`__main__`名字空间，并且不是处于正常的导入过程中，才会执行。

## 5.8. 已知问题

XXX 要是有个图将会非常棒。

XXX * (import_machinery.rst) 有个专用于模块和包属性的章节怎样？或许扩展或取代数据模型参考页面上的相关条目？

XXX 库手册中的runpy，pkgutil等都应该有个“另见”链接，放在顶部，指向新的导入系统部分。

XXX 添加关于初始化`__main__`的不同方式的更多解释？

XXX 增加关于`__main__`异常/陷阱的更多信息 (例如，从[**PEP 395**](https://www.python.org/dev/peps/pep-0395)拷贝过来)。

## 5.9. 参考

自Python早期起，导入机制已经发生了很大的变化。原始的[包规范](http://legacy.python.org/doc/essays/packages.html)仍然可读，虽然自编写那篇文档起，一些细节发生了改变。

[`sys.meta_path`](https://docs.python.org/3/library/sys.html#sys.meta_path "sys.meta_path" )的原始规格是[**PEP 302**](https://www.python.org/dev/peps/pep-0302)，随后在[**PEP 420**](https://www.python.org/dev/peps/pep-0420)中进行了扩展。

[**PEP 420**](https://www.python.org/dev/peps/pep-0420)为Python 3.3引入了[名字空间包](https://docs.python.org/3/glossary.html#term-namespace-package)。[**PEP 420**](https://www.python.org/dev/peps/pep-0420)还引入了`find_loader()`协议，作为`find_module()`的一种替换。

[**PEP 366**](https://www.python.org/dev/peps/pep-0366)描述了`__package__`属性的附加信息，用于在主模块中的显式相对导入。

[**PEP 328**](https://www.python.org/dev/peps/pep-0328)介绍了绝对和显式相对导入，并且为语义[**PEP 366**](https://www.python.org/dev/peps/pep-0366)最初提议的`__name__`，将最终指定`__package__`。

[**PEP 338**](https://www.python.org/dev/peps/pep-0338)定义了将模块作为脚本执行。

[**PEP 451**](https://www.python.org/dev/peps/pep-0451)添加了每个模块的导入状态的封装到spec对象中。它还卸载加载器的大部分样本责任到导入机制上。这些改动允许弃用导入系统中的一些API，以及添加新方法到查找器和加载器中。

脚注

[1] 见[`types.ModuleType`](https://docs.python.org/3/library/types.html#types.ModuleType "types.ModuleType" )。

[2] importlib实现避免了直接使用返回值。相反，它通过在[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中查找模块名字来获取module对象。这种行为的非直接影响是，一个已导入的模块或许会替换掉[`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules" )中的自己。这是实现特有的行为，不保证在其他Python实现中也能工作。

[3] 在遗留代码中，是有可能在[`sys.path_importer_cache`](https://docs.python.org/3/library/sys.html#sys.path_importer_cache "sys.path_importer_cache" )中找到[`imp.NullImporter`](https://docs.python.org/3/library/imp.html#imp.NullImporter "imp.NullImporter" )实例的。推荐修改代码，使用`None`来代替。见[Porting Python code](https://docs.python.org/3/whatsnew/3.3.html#portingpythoncode)，以了解更多细节。
