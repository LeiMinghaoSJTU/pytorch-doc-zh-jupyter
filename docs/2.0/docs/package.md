# torch.package [¶](#torch-package "此标题的永久链接")

> 译者：[片刻小哥哥](https://github.com/jiangzhonglian)
>
> 项目地址：<https://pytorch.apachecn.org/2.0/docs/package>
>
> 原始地址：<https://pytorch.org/docs/stable/package.html>


`torch.package` 添加了对创建包含工件和任意 PyTorch 代码的包的支持。这些包可以保存、共享、用于在以后或在不同的机器上加载和执行模型，甚至可以使用“torch::deploy”部署到生产环境。


 本文档包含教程、操作指南、解释和 API 参考，可帮助您了解有关“torch.package”及其使用方法的更多信息。


!!! warning "警告"

     该模块依赖于不安全的“pickle”模块。仅解包您信任的数据。


 有可能构建恶意的 pickle 数据，该数据将**在解包过程中执行任意代码。永远不要解包可能来自不受信任的来源或可能已被篡改的数据。


 有关更多信息，请查看“pickle”模块的[文档](https://docs.python.org/3/library/pickle.html)。



* [教程](#tutorials)



+ [打包你的第一个模型](#packaging-your-first-model)
* [我该如何...](#how-do-i)



+ [查看包内有什么？](#see-what-is-inside-a-package) 
+ [查看为什么给定模块作为依赖项包含在内？](#see-why-a-given-module
- was-included-as-a-dependency) 
+ [在我的包中包含任意资源并稍后访问它们？](#include-throne-resources-with-my-package-and-access-them-later) 
+ [自定义如何一个类被打包了？](#customize-how-a-class-is-packaged) 
+ [在我的源代码中测试它是否在包内执行？](#test-in-my-source-code-是否在包内执行) 
+ [将代码修补到包中？](#patch-code-into-a-package) 
+ [从打包代码访问包内容？]( #access-package-contents-from-packaged-code) 
+ [区分打包代码和非打包代码？](#distinguish
- Between-packaged-code-and-non-packaged-code) 
+ [重新导出导入的对象？](#re-export-an-imported-object) 
+ [打包 TorchScript 模块?](#package-a-torchscript-module)
* [说明](#explanation)



+ [`torch.package` 格式概述](#torch-package-format-overview) 
+ [`torch.package` 如何查找代码的依赖项](#how-torch-package-finds-your-code-s-dependencies ) 
+ [依赖管理](#dependency-management) 
+ [`torch.package` 锋利边缘](#torch-package-sharp-edges) 
+ [`torch.package` 如何保持包彼此隔离](#how -torch-package-keeps-packages-isolated-from-each-other)
* [API 参考](#api-reference)


## [教程](#id1) [¶](#tutorials "此标题的永久链接")


### [打包你的第一个模型](#id2) [¶](#packaging-your-first-model "永久链接到此标题")


 [Colab](https://colab.research.google.com/drive/1lFZkLyViGfXxB-m3jqlyTQuYToo3XLo-) 上提供了指导您打包和解包简单模型的教程。完成本练习后，您将熟悉用于创建和使用Torch 包的基本API。


## [我该如何…](#id3) [¶](#how-do-i "此标题的永久链接")


### [查看包内有什么？](#id4) [¶](#see-what-is-inside-a-package "此标题的永久链接")


#### 将包视为 ZIP 存档 [¶](#treat-the-package-like-a-zip-archive "Permalink to this header")


 “torch.package”的容器格式是 ZIP，因此任何使用标准 ZIP 文件的工具都应该可以用于探索内容。与 ZIP 文件交互的一些常见方法：



* `unzip my_package.pt` 会将 `torch.package` 存档解压到磁盘，您可以在其中自由检查其内容。


```
$ unzip my_package.pt && tree my_package
my_package
├── .data
│   ├── 94304870911616.storage
│   ├── 94304900784016.storage
│   ├── extern_modules
│   └── version
├── models
│   └── model_1.pkl
└── torchvision
    └── models
        ├── resnet.py
        └── utils.py
~ cd my_package && cat torchvision/models/resnet.py
...

```



* Python `zipfile` 模块提供了读取和写入 ZIP 存档内容的标准方法。


```
from zipfile import ZipFile
with ZipFile("my_package.pt") as myzip:
    file_bytes = myzip.read("torchvision/models/resnet.py")
    # edit file_bytes in some way
    myzip.writestr("torchvision/models/resnet.py", new_file_bytes)

```



* vim 具有本地读取 ZIP 档案的能力。您甚至可以编辑文件并将它们“写入”回存档中！


```
# add this to your .vimrc to treat `*.pt` files as zip files
au BufReadCmd *.pt call zip#Browse(expand("<amatch>"))

~ vi my_package.pt

```


#### 使用 `file_struct()` API [¶](#use-the-file-struct-api "永久链接到此标题")


[`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 提供了一个 `file_struct()` 方法，它将返回一个可打印和可查询的 [`Directory`](#torch.package.Directory " torch.package.Directory") 对象。 [`Directory`](#torch.package.Directory "torch.package.Directory") 对象是一个简单的目录结构，您可以使用它来探索 `torch.package` 的当前内容。


 [`Directory`](#torch.package.Directory "torch.package.Directory") 对象本身是可直接打印的，并将打印出文件树表示。要过滤返回的内容，请使用 glob 样式的“include”和“exclude”过滤参数。


```
with PackageExporter('my_package.pt') as pe:
    pe.save_pickle('models', 'model_1.pkl', mod)

importer = PackageImporter('my_package.pt')
# can limit printed items with include/exclude args
print(importer.file_structure(include=["**/utils.py", "**/*.pkl"], exclude="**/*.storage"))
print(importer.file_structure()) # will print out all files

```


 输出：


```
# filtered with glob pattern:
#    include=["**/utils.py", "**/*.pkl"], exclude="**/*.storage"
─── my_package.pt
    ├── models
    │   └── model_1.pkl
    └── torchvision
        └── models
            └── utils.py

# all files
─── my_package.pt
    ├── .data
    │   ├── 94304870911616.storage
    │   ├── 94304900784016.storage
    │   ├── extern_modules
    │   └── version
    ├── models
    │   └── model_1.pkl
    └── torchvision
        └── models
            ├── resnet.py
            └── utils.py

```


 您还可以使用 `has_file()` 方法查询 [`Directory`](#torch.package.Directory "torch.package.Directory") 对象。


```
importer_file_structure = importer.file_structure()
found: bool = importer_file_structure.has_file("package_a/subpackage.py")

```


### [查看为什么给定模块被包含为依赖项？](#id5) [¶](#see-why-a-given-module-was-included-as-a-dependency "永久链接到此标题" )


 假设有一个给定的模块 `foo` ，并且您想知道为什么您的 [`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 将 `foo` 作为依赖项拉入。


[`PackageExporter.get_rdeps()`](#torch.package.PackageExporter.get_rdeps "torch.package.PackageExporter.get_rdeps") 将返回直接依赖于 `foo` 的所有模块。


 如果您想查看给定模块 `src` 如何依赖于 `foo` ，请使用 [`PackageExporter.all_paths()`](#torch.package.PackageExporter.all_paths "torch.package.PackageExporter.all_paths")方法将返回一个点格式的图表，显示 `src` 和 `foo` 之间的所有依赖路径。


 如果您只想查看 [`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 的整个依赖关系图，您可以使用 [`PackageExporter.dependency_graph_string()`] (#torch.package.PackageExporter.dependency_graph_string "torch.package.PackageExporter.dependency_graph_string") 。


### [在我的包中包含任意资源并稍后访问它们？](#id6) [¶](#include-任意资源-with-my-package-and-access-them-later "永久链接到此标题" )


[`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 公开了三个方法， `save_pickle` 、 `save_text` 和 `save_binary` ，允许您保存Python对象、文本、和二进制数据到一个包。


```
with torch.PackageExporter("package.pt") as exporter:
    # Pickles the object and saves to `my_resources/tensor.pkl` in the archive.
    exporter.save_pickle("my_resources", "tensor.pkl", torch.randn(4))
    exporter.save_text("config_stuff", "words.txt", "a sample string")
    exporter.save_binary("raw_data", "binary", my_bytes)

```


[`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 公开名为 `load_pickle` 、 `load_text` 和 `load_binary` 的补充方法，允许您加载 Python 对象、文本和包中的二进制数据。


```
importer = torch.PackageImporter("package.pt")
my_tensor = importer.load_pickle("my_resources", "tensor.pkl")
text = importer.load_text("config_stuff", "words.txt")
binary = importer.load_binary("raw_data", "binary")

```


### [自定义类的打包方式？](#id7) [¶](#customize-how-a-class-is-packaged "永久链接到此标题")


`torch.package` 允许自定义类的打包方式。通过在类上定义方法 `__reduce_package__` 并定义相应的解包函数来访问此行为。这类似于为Python的正常pickle过程定义`__reduce__`。


 脚步：


1. 在目标类上定义方法`__reduce_package__(self, exporter: PackageExporter)`。此方法应该完成将类实例保存在包内的工作，并且应该返回相应的解包函数的元组，其中包含调用解包函数所需的参数。当“PackageExporter”遇到目标类的实例时，会调用此方法。2。为类定义一个解包函数。这个解包函数应该完成重建并返回类实例的工作。函数签名的第一个参数应该是“PackageImporter”实例，其余参数由用户定义。


```
# foo.py [Example of customizing how class Foo is packaged]
from torch.package import PackageExporter, PackageImporter
import time


Foo 类: def __init__(self, my_string: str): super().__init__() self.my_string = my_string self.time_imported = 0 self.time_exported = 0 def __reduce_package__(self, exporter: PackageExporter): """ 由 ``torch.package.PackageExporter`` 的 Pickler 的 ``persistent_id 调用`` 当保存此对象的实例时。此方法应该完成将此对象保存在 ``torch.package`` 存档​​中的工作。返回带参数的函数以从 ``torch.package.PackageImporter 加载对象``'s Pickler 的 ``persist_load`` 函数。 """ # 使用此模式可确保命名不会与正常依赖项发生冲突， # 在此模块名称下保存的任何内容不应与生成的包中的其他 # 项冲突\ _module_name = f"foo
- generated._{exporter.get_unique_id()}" exporter.save_text( generated_module_name, "foo.txt", self.my_string 
+ ",带有导出器修改！", ) time_exported = time.clock_gettime(1) # 返回带有要调用的参数的解包函数 return (unpackage_foo, ( generated_module_name, time_exported,))


def unpackage_foo(
    importer: PackageImporter, generated_module_name: str, time_exported: float
) -> Foo:
 """
 Called by ``torch.package.PackageImporter``'s Pickler's ``persistent_load`` function
 when depickling a Foo object.
 Performs work of loading and returning a Foo instance from a ``torch.package`` archive.
 """
    time_imported = time.clock_gettime(1)
    foo = Foo(importer.load_text(generated_module_name, "foo.txt"))
    foo.time_imported = time_imported
    foo.time_exported = time_exported
    return foo

```



```
# example of saving instances of class Foo

import torch
from torch.package import PackageImporter, PackageExporter
import foo

foo_1 = foo.Foo("foo_1 initial string")
foo_2 = foo.Foo("foo_2 initial string")
with PackageExporter('foo_package.pt') as pe:
    # save as normal, no extra work necessary
    pe.save_pickle('foo_collection', 'foo1.pkl', foo_1)
    pe.save_pickle('foo_collection', 'foo2.pkl', foo_2)

pi = PackageImporter('foo_package.pt')
print(pi.file_structure())
imported_foo = pi.load_pickle('foo_collection', 'foo1.pkl')
print(f"foo_1 string: '{imported_foo.my_string}'")
print(f"foo_1 export time: {imported_foo.time_exported}")
print(f"foo_1 import time: {imported_foo.time_imported}")

```



```
# output of running above script
─── foo_package
    ├── foo-generated
    │   ├── _0
    │   │   └── foo.txt
    │   └── _1
    │       └── foo.txt
    ├── foo_collection
    │   ├── foo1.pkl
    │   └── foo2.pkl
    └── foo.py

foo_1 string: 'foo_1 initial string, with reduction modification!'
foo_1 export time: 9857706.650140837
foo_1 import time: 9857706.652698385

```


### [在我的源代码中测试它是否在包内执行？](#id8) [¶](#test-in-my-source-code-whether-or-not-it-is-executing -inside-a-package"此标题的永久链接")


 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 会将属性 `__torch_package__` 添加到它初始化的每个模块中。您的代码可以检查此属性是否存在，以确定它是否在打包上下文中执行。


```
# In foo/bar.py:

if "__torch_package__" in dir():  # true if the code is being loaded from a package
    def is_in_package():
        return True

    UserException = Exception
else:
    def is_in_package():
        return False

    UserException = UnpackageableException

```


 现在，代码的行为会有所不同，具体取决于代码是通过 Python 环境正常导入还是从 `torch.package` 导入。


```
from foo.bar import is_in_package

print(is_in_package())  # False

loaded_module = PackageImporter(my_package).import_module("foo.bar")
loaded_module.is_in_package()  # True

```


**警告**：一般来说，根据是否打包而使代码表现不同是不好的做法。这可能会导致难以调试的问题，这些问题对您导入代码的方式敏感。如果您的包打算被大量使用，请考虑重组您的代码，以便无论它如何加载，它的行为方式都相同。


### [将代码修补到包中？](#id9) [¶](#patch-code-into-a-package "永久链接到此标题")


[`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 提供了一种 `save_source_string()` 方法，允许将任意 Python 源代码保存到您选择的模块中。


```
with PackageExporter(f) as exporter:
    # Save the my_module.foo available in your current Python environment.
    exporter.save_module("my_module.foo")

    # This saves the provided string to my_module/foo.py in the package archive.
    # It will override the my_module.foo that was previously saved.
    exporter.save_source_string("my_module.foo", textwrap.dedent(
 """ def my_function():
 print('hello world')
 """
    ))

    # If you want to treat my_module.bar as a package
    # (e.g. save to `my_module/bar/__init__.py` instead of `my_module/bar.py)
    # pass is_package=True,
    exporter.save_source_string("my_module.bar",
                                "def foo(): print('hello')
",
                                is_package=True)

importer = PackageImporter(f)
importer.import_module("my_module.foo").my_function()  # prints 'hello world'

```


### [从打包代码访问包内容？](#id10) [¶](#access-package-contents-from-packaged-code "永久链接到此标题")


[`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 实现 [importlib.resources](https://docs.python.org/3/library/importlib.html#module-importlib. resources) 用于从包内部访问资源的 API。


```
with PackageExporter(f) as exporter:
    # saves text to my_resource/a.txt in the archive
    exporter.save_text("my_resource", "a.txt", "hello world!")
    # saves the tensor to my_pickle/obj.pkl
    exporter.save_pickle("my_pickle", "obj.pkl", torch.ones(2, 2))

    # see below for module contents
    exporter.save_module("foo")
    exporter.save_module("bar")

```


 “importlib.resources” API 允许从打包代码中访问资源。


```
# foo.py:
import importlib.resources
import my_resource

# returns "hello world!"
def get_my_resource():
    return importlib.resources.read_text(my_resource, "a.txt")

```


 建议使用“importlib.resources”从打包代码中访问包内容，因为它符合 Python 标准。但是，也可以从打包代码中访问父 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 实例本身。


```
# bar.py:
import torch_package_importer # this is the PackageImporter that imported this module.

# Prints "hello world!", equivalent to importlib.resources.read_text
def get_my_resource():
    return torch_package_importer.load_text("my_resource", "a.txt")

# You also do things that the importlib.resources API does not support, like loading
# a pickled object from the package.
def get_my_pickle():
    return torch_package_importer.load_pickle("my_pickle", "obj.pkl")

```


### [区分打包代码和非打包代码？](#id11) [¶](#distinguish
- Between-packaged-code-and-non-packaged-code "Permalink to this header")


 要判断对象的代码是否来自“torch.package”，请使用“torch.package.is_from_package()”函数。注意：如果对象来自包，但其定义来自标记为“的模块” extern` 或来自 `stdlib` ，此检查将返回 `False` 。


```
importer = PackageImporter(f)
mod = importer.import_module('foo')
obj = importer.load_pickle('model', 'model.pkl')
txt = importer.load_text('text', 'my_test.txt')

assert is_from_package(mod)
assert is_from_package(obj)
assert not is_from_package(txt) # str is from stdlib, so this will return False

```


### [重新导出导入的对象？](#id12) [¶](#re-export-an-imported-object "永久链接到此标题")


 要重新导出之前由 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 导入的对象，您必须创建新的 [`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 知道原始的 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter")，以便它可以找到对象依赖项的源代码。


```
importer = PackageImporter(f)
obj = importer.load_pickle("model", "model.pkl")

# re-export obj in a new package
with PackageExporter(f2, importer=(importer, sys_importer)) as exporter:
    exporter.save_pickle("model", "model.pkl", obj)

```


### [打包 TorchScript 模块？](#id13) [¶](#package-a-torchscript-module "永久链接到此标题")


 要打包 TorchScript 模型，请使用与任何其他对象相同的 `save_pickle` 和 `load_pickle` API。还支持保存作为属性或子模块的 TorchScript 对象，无需额外工作。


```
# save TorchScript just like any other object
with PackageExporter(file_name) as e:
    e.save_pickle("res", "script_model.pkl", scripted_model)
    e.save_pickle("res", "mixed_model.pkl", python_model_with_scripted_submodule)
# load as normal
importer = PackageImporter(file_name)
loaded_script = importer.load_pickle("res", "script_model.pkl")
loaded_mixed = importer.load_pickle("res", "mixed_model.pkl"

```


## [说明](#id14) [¶](#explanation "此标题的永久链接")


### [`torch.package` 格式概述](#id15) [¶](#torch-package-format-overview "此标题的永久链接")


 “torch.package”文件是一个 ZIP 存档，通常使用“.pt”扩展名。 ZIP 存档内有两种文件：



* 框架文件，放置在 `.data/` 中。 
* 用户文件，即其他所有内容。


 例如，“torchvision”中完全封装的 ResNet 模型如下所示：


```
resnet
├── .data  # All framework-specific data is stored here.
│   │      # It's named to avoid conflicts with user-serialized code.
│   ├── 94286146172688.storage  # tensor data
│   ├── 94286146172784.storage
│   ├── extern_modules  # text file with names of extern modules (e.g. 'torch')
│   ├── version         # version metadata
│   ├── ...
├── model  # the pickled model
│   └── model.pkl
└── torchvision  # all code dependencies are captured as source files
    └── models
        ├── resnet.py
        └── utils.py

```


#### 框架文件 [¶](#framework-files "此标题的永久链接")


 `.data/` 目录属于 torch.package，其内容被认为是私有实现细节。 `torch.package` 格式不保证 `.data/` 的内容，但所做的任何更改将向后兼容(也就是说，新版本的 PyTorch 将始终能够加载旧的 `torch.packages` )。


 目前，`.data/`目录包含以下项目：



* `version` ：序列化格式的版本号，以便 `torch.package` 导入基础结构知道如何加载此包。
* `extern_modules` ：被视为 `extern:class:` 的模块列表包导入器`。 ``extern` 模块将使用加载环境的系统导入器导入。
* `*.storage` ：序列化tensor数据。


```
.data
├── 94286146172688.storage
├── 94286146172784.storage
├── extern_modules
├── version
├── ...

```


#### 用户文件 [¶](#user-files "此标题的永久链接")


 存档中的所有其他文件均由用户放置在那里。布局与 Python [常规包](https://docs.python.org/3/reference/import.html#regular-packages) 相同。要更深入地了解 Python 打包的工作原理，请参阅[这篇文章](https://www.python.org/doc/essays/packages/)(它有点过时了，因此请使用[Python]仔细检查实现细节参考文档](https://docs.python.org/3/library/importlib.html))。


```
<package root>
├── model  # the pickled model
│   └── model.pkl
├── another_package
│   ├── __init__.py
│   ├── foo.txt         # a resource file , see importlib.resources
│   └── ...
└── torchvision
    └── models
        ├── resnet.py   # torchvision.models.resnet
        └── utils.py    # torchvision.models.utils

```


### [如何 `torch.package` 找到代码的依赖项](#id16) [¶](#how-torch-package-finds-your-code-s-dependencies "永久链接到此标题")


#### 分析对象的依赖关系 [¶](#analyzing-an-object-s-dependencies "永久链接到此标题")


 当您发出 `save_pickle(obj,...)` 调用时，[`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 将正常 pickle 对象。然后，它使用“pickletools”标准库模块来解析pickle字节码。


 在 pickle 中，对象与“GLOBAL”操作码一起保存，该操作码描述了在哪里可以找到对象类型的实现，例如：


```
GLOBAL 'torchvision.models.resnet Resnet`

```


 依赖解析器将收集所有“GLOBAL”操作并将它们标记为 pickle 对象的依赖项。有关 pickle 和 pickle 格式的更多信息，请参阅 [Python 文档](https://docs.python.org/3 /library/pickle.html) 。


#### 分析模块的依赖关系 [¶](#analyzing-a-module-s-dependencies"永久链接到此标题")


 当 Python 模块被识别为依赖项时，“torch.package”会遍历模块的 python AST 表示形式，并查找完全支持标准形式的导入语句：“from x import y”、“import z”、“from w import v”当遇到这些导入语句之一时，“torch.package”将导入的模块注册为依赖项，然后它们本身以相同的 AST 步行方式进行解析。


**注意**：AST 解析对 `__import__(...)` 语法的支持有限，并且不支持 `importlib.import_module` 调用。一般来说，您不应期望“torch.package”能够检测到动态导入。


### [依赖管理](#id17) [¶](#dependency-management "永久链接到此标题")


`torch.package` 自动查找您的代码和对象所依赖的 Python 模块。此过程称为依赖项解析。对于依赖项解析器找到的每个模块，您必须指定要执行的*操作*。


 允许的操作有：



* `intern` : 将此模块放入包中。
* `extern` : 将此模块声明为包的外部依赖项。
* `mock` : 存根此模块。
* `deny` : 依赖于此模块将引发包导出时出错。


 最后，还有一个更重要的操作，从技术上讲，它不是 `torch.package` 的一部分：



* 重构：删除或更改代码中的依赖项。


 请注意，操作仅在整个 Python 模块上定义。没有办法“只”打包模块中的函数或类，而忽略其余部分。这是设计使然。 Python 不提供模块中定义的对象之间清晰的边界。依赖组织的唯一定义单元是模块，因此这就是“torch.package”所使用的。


 使用模式将操作应用于模块。模式可以是模块名称( `"foo.bar"` )或 glob(如 `"foo.**"` )。您可以使用 [`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 上的方法将模式与操作关联起来，例如


```
my_exporter.intern("torchvision.**")
my_exporter.extern("numpy")

```


 如果模块与模式匹配，则会对其应用相应的操作。对于给定的模块，将按照定义的顺序检查模式，并采取第一个操作。


#### `intern`[¶](#intern "此标题的永久链接")


 如果一个模块是 `intern` 的，它将被放入包中。


 此操作是您的模型代码，或您想要打包的任何相关代码。例如，如果您尝试从“torchvision”打包 ResNet，则需要“实习”模块 torchvision.models.resnet。


 在包导入时，当您的打包代码尝试导入“intern”模块时，PackageImporter 将在您的包中查找该模块。如果找不到该模块，则会引发错误。这可以确保每个 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 与加载环境隔离 
- 即使您的包和加载环境中都有 `my_interned_module` 可用, [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 将仅使用您的包中的版本。


**注意**：只有 Python 源模块可以进行 `intern` 编辑。其他类型的模块，如 C 扩展模块和字节码模块，如果您尝试“实习”它们，将会引发错误。这些类型的模块需要进行 `mock` 或 `extern` 编辑。


#### `extern`[¶](#extern "此标题的永久链接")


 如果模块是“extern”，则不会将其打包。相反，它将被添加到该包的外部依赖项列表中。您可以在 `package_exporter.extern_modules` 上找到此列表。


 在包导入时，当打包的代码尝试导入 `extern` 模块时， [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 将使用默认的 Python 导入器来查找该模块，如下所示如果你做了 `importlib.import_module("my_externed_module")` 。如果找不到该模块，则会出现错误。


 通过这种方式，您可以依赖包中的第三方库，例如“numpy”和“scipy”，而无需打包它们。


**警告**：如果任何外部库以向后不兼容的方式发生更改，您的包可能无法加载。如果您需要包的长期可重复性，请尝试限制“extern”的使用。


#### `mock`[¶](#mock "此标题的永久链接")


 如果一个模块是 `mock` 的，它将不会被打包。相反，存根模块将被打包在其位置。存根模块将允许您从中检索对象(这样“from my_mocked_module import foo”就不会出错)，但对该对象的任何使用都会引发“NotImplementedError”。


`mock` 应该用于您“知道”在加载的包中不需要的代码，但您仍然希望可在非打包内容中使用。例如，初始化/配置代码，或仅用于调试/的代码训练。


**警告**：一般来说，“mock”应该作为最后的手段使用。它引入了打包代码和非打包代码之间的行为差​​异，这可能会导致以后的混乱。相反，您更愿意重构代码以删除不需要的依赖项。


#### 重构 [¶](#refactoring "此标题的永久链接")


 管理依赖关系的最好方法是根本没有依赖关系！通常，可以重构代码以删除不必要的依赖项。以下是一些编写具有干净依赖关系的代码的指南(这通常也是良好的实践！)：


**仅包括您使用的**。不要在代码中留下未使用的导入。依赖解析器不够聪明，无法判断它们确实未使用，并且会尝试处理它们。


**验证您的进口**。例如，不要编写 import foo 然后使用 `foo.bar.baz` ，而是更喜欢编写 `from foo.bar import baz` 。这更准确地指定了您真正的依赖项( `foo.bar` )，并让依赖项解析器知道您不需要所有的 `foo` 。


**将具有不相关功能的大文件拆分为较小的文件**。如果您的 utils 模块包含一堆不相关的功能，则任何依赖于 utils 的模块都需要引入大量不相关的依赖项，即使您只需要其中的一小部分。更喜欢定义可以相互独立打包的单一用途模块。


#### 模式 [¶](#patterns "此标题的永久链接")


 模式允许您使用方便的语法指定模块组。模式的语法和行为遵循 Bazel/Buck [glob()](https://docs.bazel.build/versions/master/be/functions.html#glob) 。


 我们尝试与模式匹配的模块称为候选模块。候选由由分隔符字符串分隔的段列表组成，例如`foo.bar.baz` 。


 一个模式包含一个或多个段。段可以是：



* 完全匹配的文字字符串(例如 `foo` )。
* 包含通配符的字符串(例如 `torch` 或 `foo*baz*` )。通配符匹配任何字符串，包括空字符串。
* 双通配符 ( `**` )。这与零个或多个完整片段匹配。


 例子：



* `torch.**` : 匹配 `torch` 及其所有子模块，例如`torch.nn` 和 `torch.nn.function`.
* `torch.*` ：匹配 `torch.nn` 或 `torch.function` ，但不匹配 `torch.nn.function` 或 `torch`
* ` torch*.**` ：匹配 `torch` 、 `torchvision` 及其所有子模块


 指定操作时，您可以传递多个模式，例如


```
exporter.intern(["torchvision.models.**", "torchvision.utils.**"])

```


 如果模块与任何模式匹配，则模块将与此操作匹配。


 您还可以指定要排除的模式，例如


```
exporter.mock("**", exclude=["torchvision.**"])

```


 如果模块与任何排除模式匹配，则该模块将不会与此操作匹配。在此示例中，我们模拟除“torchvision”及其子模块之外的所有模块。


 当一个模块可能与多个操作匹配时，将执行定义的第一个操作。


### [`torch.package` 锐边](#id18) [¶](#torch-package-sharp-edges "此标题的永久链接")


#### 避免在模块中使用全局状态 [¶](#avoid-global-state-in-your-modules "Permalink to this header")


 Python 使绑定对象和在模块级范围内运行代码变得非常容易。这通常没问题——毕竟，函数和类都是以这种方式绑定名称的。然而，当您在模块范围定义一个对象并打算对其进行变异、引入可变全局状态时，事情会变得更加复杂。


 可变全局状态非常有用——它可以减少样板文件，允许开放注册到表中，等等。但是除非非常小心地使用，否则与“torch.package”一起使用时可能会导致复杂化。


 每个 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 都会为其内容创建一个独立的环境。这很好，因为这意味着我们加载多个包并确保它们彼此隔离，但是当以假设共享可变全局状态的方式编写模块时，这种行为可能会产生难以调试的错误。


#### 包和加载环境之间不共享类型 [¶](#types-are-not-shared
- Between-packages-and-the-loading-environment "Permalink to this header")


 从 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 导入的任何类都将是特定于该导入器的类的版本。例如：


```
from foo import MyClass

my_class_instance = MyClass()

with PackageExporter(f) as exporter:
    exporter.save_module("foo")

importer = PackageImporter(f)
imported_MyClass = importer.import_module("foo").MyClass

assert isinstance(my_class_instance, MyClass)  # works
assert isinstance(my_class_instance, imported_MyClass)  # ERROR!

```


 在此示例中，“MyClass”和“imported_MyClass”*不是同一类型*。在这个特定的示例中，“MyClass”和“imported_MyClass”具有完全相同的实现，因此您可能认为可以将它们视为同一个类。但考虑一下这样的情况：“imported_MyClass”来自一个较旧的包，并且具有完全不同的“MyClass”实现——在这种情况下，将它们视为同一个类是不安全的。


 在底层，每个导入器都有一个前缀，允许它唯一地标识类：


```
print(MyClass.__name__)  # prints "foo.MyClass"
print(imported_MyClass.__name__)  # prints <torch_package_0>.foo.MyClass

```


 这意味着当参数之一来自包而另一个参数不是时，您不应期望“isinstance”检查起作用。如果您需要此功能，请考虑以下选项：



* 执行鸭子类型(仅使用类，而不是显式检查它是否属于给定类型)。 
* 使类型关系成为类契约的显式部分。例如，您可以添加属性标记 `self.handler = "handle_me_this_way"` 并让客户端代码检查 `handler` 的值，而不是直接检查类型。


### [`torch.package` 如何保持包彼此隔离](#id19) [¶](#how-torch-package-keeps-packages-isolated-from-each-other "永久链接到此标题")


 每个 PackageImporter(#torch.package.PackageImporter "torch.package.PackageImporter") 实例为其模块和对象创建一个独立、隔离的环境。包中的模块只能导入其他打包的模块，或标记为“extern”的模块。如果您使用多个 PackageImporter 实例来加载单个包，您将获得多个不交互的独立环境。


 这是通过使用自定义导入器扩展 Python 的导入基础设施来实现的。 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 提供与 `importlib` 导入器相同的核心 API；即，它实现了 `import_module` 和 `__import__` 方法。


 当您调用 [`PackageImporter.import_module()`](#torch.package.PackageImporter.import_module "torch.package.PackageImporter.import_module") 时， [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 将构造并返回一个新模块，就像系统导入程序一样。但是， [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 修补返回的模块以使用 `self` (即 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 实例)通过查看包而不是搜索用户的 Python 环境来满足未来的导入请求。


#### Mangling [¶](#mangling "此标题的永久链接")


 为了避免混淆(“这个 `foo.bar` 对象是我的包中的对象，还是我的 Python 环境中的对象？”)，[`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter")通过添加 *mangle 前缀
* 来破坏所有导入模块的 `__name__` 和 `__file__`。


 对于 `__name__` ，像 `torchvision.models.resnet18` 这样的名称会变成 `<torch_package_0>.torchvision.models.resnet18` 。


 对于 `__file__` ，像 `torchvision/models/resnet18.py` 这样的名称会变成 `<torch_package_0>.torchvision/modules/resnet18.py` 。


 名称修改有助于避免不同包之间无意中模块名称的双关，并通过使堆栈跟踪和打印语句更清楚地显示它们是否引用打包代码来帮助您进行调试。有关面向开发人员的重整的详细信息，请参阅“torch/package/”中的“mangling.md”。


## [API 参考](#id20) [¶](#api-reference "此标题的永久链接")


*班级*


 火炬.package。


 包装错误


 (*依赖_graph*，*调试



 =
 


 False
* ) [[source]](_modules/torch/package/package_exporter.html#PackagingError)[¶](#torch.package.PackagingError "此定义的永久链接")


 当导出包出现问题时会引发此异常。 `PackageExporter` 将尝试收集所有错误并立即将它们呈现给您。


*班级*


 火炬.package。


 EmptyMatchError [[source]](_modules/torch/package/package_exporter.html#EmptyMatchError)[¶](#torch.package.EmptyMatchError "此定义的永久链接")


 这是当mock或extern被标记为“allow_empty=False”时抛出的异常，并且在打包过程中与任何模块都不匹配。


*班级*


 火炬.package。


 包导出器


 ( *f
* , *importer=<torch.package.importer._SysImporter 对象>
* , *debug=False
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter)[¶](# torch.package.PackageExporter"此定义的永久链接")


 导出器允许您将代码包、腌制的 Python 数据以及任意二进制和文本资源写入独立的包中。


 导入可以以封闭的方式加载此代码，这样代码是从包而不是普通的 Python 导入系统加载的。这允许打包 PyTorch 模型代码和数据，以便它可以在服务器上运行或在将来用于迁移学习。


 包中包含的代码是在创建时从原始源中逐个文件复制的，文件格式是专门组织的 zip 文件。该包的未来用户可以解压该包并编辑代码，以便对其进行自定义修改。


 包的导入器确保模块中的代码只能从包内加载，除了使用 [`extern()`](#torch.package.PackageExporter.extern "torch.package.PackageExporter.extern 显式列为外部的模块) ").zip 存档中的文件 `extern_modules` 列出了包外部依赖的所有模块。这可以防止包在本地运行的“隐式”依赖关系，因为它正在导入本地安装的包，但当包被复制到另一台机器上。


 当源代码添加到包中时，导出器可以选择扫描它以获取更多代码依赖项(“dependency=True”)。它查找 import 语句，解析对限定模块名称的相对引用，并执行用户指定的操作(请参阅：[`extern()`](#torch.package.PackageExporter.extern "torch.package.PackageExporter.extern" ) 、 [`mock()`](#torch.package.PackageExporter.mock "torch.package.PackageExporter.mock") 和 [`intern()`](#torch.package.PackageExporter.intern "torch.package.PackageExporter.intern") )。


 __在里面__


 ( *f
* , *importer=<torch.package.importer._SysImporter 对象>
* , *debug=False
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.__init__)[¶] (#torch.package.PackageExporter.__init__"此定义的永久链接")


 创建一个出口商。


 参数 
* **f** ( [*Union*](https://docs.python.org/3/library/typing.html#typing.Union "(in Python v3.12)")*[
* [
* str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*,
* [*Path*](https://docs.python. org/3/library/pathlib.html#pathlib.Path "(Python v3.12)")*,
* [*BinaryIO*](https://docs.python.org/3/library/typing.html# Typing.BinaryIO "(Python v3.12)")*]
* ) – 导出到的位置。可以是包含文件名的 `string` /`Path` 对象或二进制 I/O 对象。
* **导入器** ( [*Union*](https://docs.python.org/3/library/typing. html#typing.Union "(在 Python v3.12 中)")*[
* *导入器
* *,
* [*序列*](https://docs.python.org/3/library/typing.html#typing.序列 "(in Python v3.12)")*[
* *Importer
* *]
* *]
* ) – 如果传递了单个导入器，则使用它来搜索模块。如果传递了一系列导入器，则使用 `OrderedImporter ` 将由它们构建。
* **debug** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 如果设置为 True，则将损坏模块的路径添加到 PackagingErrors。


 添加_依赖项


 (*模块_名称*，*依赖项



 =
 


 True
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.add_dependency)[¶](#torch.package.PackageExporter.add_dependency "此定义的永久链接")


 给定一个模块，根据用户指定的模式将其添加到依赖关系图中。


 所有_路径


 ( *src
* , *dst
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.all_paths)[¶](#torch.package.PackageExporter.all_paths "此定义的永久链接")


 返回子图的点表示


 其中包含从 src 到 dst 的所有路径。


 退货


 包含从 src 到 dst 的所有路径的点表示。( <https://graphviz.org/doc/info/lang.html
> )


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 close
 


 ( ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.close)[¶](#torch.package.PackageExporter.close "此定义的永久链接")


 将包写入文件系统。 [`close()`](#torch.package.PackageExporter.close "torch.package.PackageExporter.close") 之后的任何调用现在都无效。最好使用资源保护语法：


```
with PackageExporter("file.zip") as e:
    ...

```


 被拒绝_模块


 ( ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.denied_modules)[¶](#torch.package.PackageExporter.denied_modules "此定义的永久链接")


 返回当前被拒绝的所有模块。


 退货


 包含此包中将被拒绝的模块名称的列表。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(在 Python v3.12 中)") ]




 deny
 


 (*包括*，***，*排除



 =
 


 ()
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.deny)[¶](#torch.package.PackageExporter.deny "此定义的永久链接")


 阻止名称与包可以导入的模块列表中的给定 glob 模式匹配的模块。如果找到任何匹配包的依赖项，则会出现 [`PackagingError`](#torch.package.PackagingError "torch.package.PackagingError")被提出。


 参数 
* **include** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* ) – 一个字符串，例如`"my_package.my_subpackage"` ，或要外部的模块名称的字符串列表。这也可以是 glob 样式模式，如 [`mock()`](#torch.package.PackageExporter.mock "torch.package.PackageExporter.mock").
* **exclude** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* ) – 一个可选模式，排除一些与包括字符串。


 依赖_graph_string


 ( ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.dependency_graph_string)[¶](#torch.package.PackageExporter.dependency_graph_string "此定义的永久链接")


 返回包中依赖项的有向图字符串表示形式。


 退货


 包中依赖项的字符串表示形式。


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 外部的


 (*包括*，***，*排除



 =
 


 ()
* , *允许_空



 =
 


 True
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.extern)[¶](#torch.package.PackageExporter.extern "此定义的永久链接")


 在包可以导入的外部模块列表中包含“module”。这将阻止依赖项发现将其保存在包中。导入器将直接从标准导入系统加载外部模块。外部模块的代码也必须存在于加载包的过程中。


 参数 
* **include** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* ) – 一个字符串，例如`"my_package.my_subpackage"` ，或要外部的模块名称的字符串列表。这也可以是 glob 样式模式，如 [`mock()`](#torch.package.PackageExporter.mock "torch.package.PackageExporter.mock") 中所述。
* **排除** ( *Union
* 
* [
* *列表
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* *,
* [ *str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* ) – 一个可选模式，排除一些与 include 字符串匹配的模式.
* **allow_empty** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 可选标志指定在打包过程中调用“extern”方法指定的外部模块是否必须与某个模块匹配。如果使用 `allow_empty=False` 添加外部模块 globpattern ，并且调用 [`close()`](#torch.package.PackageExporter.close "torch.package.PackageExporter.close") (显式或通过`__exit__` )在任何模块匹配该模式之前，会抛出异常。如果`allow_empty=True`，则不会抛出此类异常。


 外部_模块


 ( ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.externed_modules)[¶](#torch.package.PackageExporter.externed_modules "此定义的永久链接")


 返回当前外部的所有模块。


 退货


 包含将在此包中外部使用的模块名称的列表。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(在 Python v3.12 中)") ]


 获取_rdeps


 ( *module_name
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.get_rdeps)[¶](#torch.package.PackageExporter.get_rdeps "此定义的永久链接")


 返回依赖于模块 `module_name` 的所有模块的列表。


 退货


 包含依赖于 `module_name` 的模块名称的列表。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(在 Python v3.12 中)") ]


 获取_唯一_id


 ( ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.get_unique_id)[¶](#torch.package.PackageExporter.get_unique_id "此定义的永久链接")


 获取一个 ID。该 ID 保证仅针对该包裹分发一次。


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 实习生


 (*包括*，***，*排除



 =
 


 ()
* , *允许_空



 =
 


 True
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.intern)[¶](#torch.package.PackageExporter.intern "此定义的永久链接")


 指定应打包的模块。模块必须匹配某些“intern”模式才能包含在包中并递归处理其依赖项。


 参数 
* **include** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* ) – 一个字符串，例如“my_package.my_subpackage”，或要外部的模块名称的字符串列表。这也可以是 glob 样式模式，如 [`mock()`](#torch.package.PackageExporter.mock "torch.package.PackageExporter.mock").
* **exclude** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* ) – 一个可选模式，排除一些与include string.
* **allow_empty** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 一个可选标志，指定此调用“intern”方法指定的实习生模块是否必须在打包期间与某个模块匹配。如果 `intern` 模块 globpattern 添加了 `allow_empty=False` ，并且 [`close()`](#torch.package.PackageExporter.close "torch.package.PackageExporter.close") 被调用(显式调用)或通过 `__exit__` )在任何模块匹配该模式之前，会抛出异常。如果 `allow_empty=True` ，则不会抛出此类异常。


 实习_模块


 ( ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.interned_modules)[¶](#torch.package.PackageExporter.interned_modules "此定义的永久链接")


 返回当前被保留的所有模块。


 退货


 包含将在此包中保留的模块名称的列表。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(在 Python v3.12 中)") ]




 mock
 


 (*包括*，***，*排除



 =
 


 ()
* , *允许_空



 =
 


 True
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.mock)[¶](#torch.package.PackageExporter.mock "此定义的永久链接")


 用模拟实现替换一些必需的模块。模拟模块将为从其访问的任何属性返回一个假对象。因为我们逐个文件复制，依赖解析有时会找到由模型文件导入但从未使用其功能的文件(例如自定义序列化代码或训练助手)。使用此函数可以模拟此功能，而无需修改原始代码。


 参数 
* **include** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* ) –


 一个字符串，例如`"my_package.my_subpackage"` ，或要模拟的模块名称的字符串列表。字符串也可以是可以匹配多个模块的全局样式模式字符串。任何与此模式字符串匹配的必需依赖项都将被自动模拟出来。


 例子 ：


`'torch.**'` – 匹配 `torch` 和 torch 的所有子模块，例如`'torch.nn'` 和 `'torch.nn.function'`


`'torch.*'` – 匹配 `'torch.nn'` 或 `'torch.function'` ，但不匹配 `'torch.nn.function'`
* **排除** ( *Union
* *[
* *列表
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* *,
* [*str *](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* ) – 一个可选模式，排除一些与包含字符串匹配的模式。例如`include='torch.**', except='torch.foo'` 将模拟除 `'torch.foo'` 之外的所有 torch 包，默认：是 `[]`.
* **allow_empty** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)") ) – 一个可选标志，指定模拟实现是否此调用指定的 [`mock()`](#torch.package.PackageExporter.mock "torch.package.PackageExporter.mock") 方法必须在打包期间与某个模块匹配。如果使用 `allow_empty=False` 添加模拟，并且调用 [`close()`](#torch.package.PackageExporter.close "torch.package.PackageExporter.close") (显式或通过 `\ __exit__` )并且模拟尚未与导出的包使用的模块匹配，则会抛出异常。如果 `allow_empty=True` ，则不会抛出此类异常。


 嘲笑_modules


 ( ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.mocked_modules)[¶](#torch.package.PackageExporter.mocked_modules "此定义的永久链接")


 返回当前被模拟的所有模块。


 退货


 包含将在此包中模拟的模块名称的列表。


 Return type


[*List*](https://docs.python.org/3/library/typing.html#typing.List "(Python v3.12)") [ [str](https://docs.python. org/3/library/stdtypes.html#str "(在 Python v3.12 中)") ]


 注册_extern_hook


 ( *hook
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.register_extern_hook)[¶](#torch.package.PackageExporter.register_extern_hook "此定义的永久链接")


 在导出器上注册外部钩子。


 每次模块与 [`extern()`](#torch.package.PackageExporter.extern "torch.package.PackageExporter.extern") 模式匹配时都会调用该钩子。它应该具有以下签名：


```
hook(exporter: PackageExporter, module_name: str) -> None

```


 挂钩将按照注册顺序被调用。


 退货


 一个句柄，可用于通过调用“handle.remove()”来删除添加的钩子。


 Return type


`torch.utils.hooks.RemovableHandle`


 注册_intern_hook


 ( *hook
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.register_intern_hook)[¶](#torch.package.PackageExporter.register_intern_hook "此定义的永久链接")


 在导出器上注册实习生挂钩。


 每次模块与 [`intern()`](#torch.package.PackageExporter.intern "torch.package.PackageExporter.intern") 模式匹配时都会调用该钩子。它应该具有以下签名：


```
hook(exporter: PackageExporter, module_name: str) -> None

```


 挂钩将按照注册顺序被调用。


 退货


 一个句柄，可用于通过调用“handle.remove()”来删除添加的钩子。


 Return type


`torch.utils.hooks.RemovableHandle`


 注册_mock_hook


 ( *hook
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.register_mock_hook)[¶](#torch.package.PackageExporter.register_mock_hook "此定义的永久链接")


 在导出器上注册模拟钩子。


 每次模块与 [`mock()`](#torch.package.PackageExporter.mock "torch.package.PackageExporter.mock") 模式匹配时都会调用该钩子。它应该具有以下签名：


```
hook(exporter: PackageExporter, module_name: str) -> None

```


 挂钩将按照注册顺序被调用。


 退货


 一个句柄，可用于通过调用“handle.remove()”来删除添加的钩子。


 Return type


`torch.utils.hooks.RemovableHandle`


 保存_binary


 ( *package
* , *resource
* , *binary
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.save_binary)[¶](#torch.package.PackageExporter.save_binary "此定义的永久链接")


 将原始字节保存到包中。


 参数 
* **package** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 模块包的名称这个资源应该去它(例如 `"my_package.my_subpackage"` )。
* **资源** ( [*str*](https://docs.python.org/3/library/stdtypes.html #str "(in Python v3.12)") ) – 资源的唯一名称，用于标识要加载的资源。
* **binary** ( [*str*](https://docs.python.org /3/library/stdtypes.html#str "(Python v3.12)") ) – 要保存的数据。


 保存_模块


 (*模块_名称*，*依赖项



 =
 


 True
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.save_module)[¶](#torch.package.PackageExporter.save_module "此定义的永久链接")


 将“module”的代码保存到包中。使用“importers”路径查找模块对象来解析模块的代码，然后使用其“__file__”属性查找源代码。


 参数 
* **module_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 例如`my_package.my_subpackage` ，将保存代码以为该包提供代码。
* **依赖项** ( [*bool*](https://docs.python.org/3/library/functions.html #bool "(在 Python v3.12 中)")*,
* *可选
* ) – 如果 `True` ，我们会扫描源代码以获取依赖项。


 保存_pickle


 (*包*，*资源*，*obj*，*依赖项



 =
 


 True
* , *pickle_protocol



 =
 


 3
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.save_pickle)[¶](#torch.package.PackageExporter.save_pickle"此定义的永久链接")


 使用 pickle 将 python 对象保存到存档中。相当于 [`torch.save()`](generated/torch.save.html#torch.save "torch.save") 但保存到存档而不是独立文件中。标准pickle不保存代码，只保存对象。如果“dependency”为true，此方法还将扫描需要模块的pickle对象来重建它们并保存相关代码。


 为了能够保存 `type(obj).__name__` 为 `my_module.MyObject` 的对象，`my_module.MyObject` 必须根据“进口商”订单。当保存之前打包的对象时，导入器的“import_module”方法需要出现在“importer”列表中才能工作。


 参数 
* **package** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 模块包的名称该资源应该进入(例如 `"my_package.my_subpackage"` )。
* **resource** ( [*str*](https://docs.python.org/3/library/stdtypes.html #str "(Python v3.12)") ) – 资源的唯一名称，用于标识要加载的资源。
* **obj** ( *Any
* ) – 要保存的对象，必须是可挑选的。
* **依赖项** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果`True` ，我们扫描源的依赖关系。


 保存_源_文件


 (*模块_名称*，*文件_或_目录*，*依赖项



 =
 


 True
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.save_source_file)[¶](#torch.package.PackageExporter.save_source_file "此定义的永久链接")


 将本地文件系统 `file_or_directory` 添加到源码包中，以提供 `module_name` 的代码。


 参数 
* **module_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 例如`"my_package.my_subpackage"` ，将保存代码以提供此包的代码。
* **file_or_directory** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 文件或代码目录的路径。如果是目录，则使用 [`save_source_file()`](#torch.package.PackageExporter.save_source_file "torch.package.PackageExporter.save_source_file") 递归复制目录中的所有 python 文件。如果文件名为“/__init__.py”，则代码将被视为包。
* **依赖项** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `True` ，我们会扫描源代码以获取依赖项。


 保存_source_string


 ( *module_name
* , *src
* , *is_package



 =
 


 错误*，*依赖项



 =
 


 True
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.save_source_string)[¶](#torch.package.PackageExporter.save_source_string "此定义的永久链接")


 添加 `src` 作为导出包中 `module_name` 的源代码。


 参数 
* **module_name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 例如`my_package.my_subpackage` ，将保存代码以为该包提供代码。
* **src** ( [*str*](https://docs.python.org/3/library/stdtypes. html#str "(in Python v3.12)") ) – 要为此包保存的 Python 源代码。
* **is_package** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*,
* *可选
* ) – 如果 `True` ，则此模块被视为包。包允许有子模块(例如 `my_package.my_subpackage.my_subsubpackage` )，并且可以在其中保存资源。默认为 `False`.
* **依赖关系** ( [*bool*](https://docs.python.org/3/library/functions.html#bool "(in Python v3.12)")*, 
* *可选
* ) – 如果为“True”，我们将扫描源以查找依赖项。


 保存_text


 ( *package
* , *resource
* , *text
* ) [[source]](_modules/torch/package/package_exporter.html#PackageExporter.save_text)[¶](#torch.package.PackageExporter.save_text "此定义的永久链接")


 将文本数据保存到包中。


 参数 
* **package** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 模块包的名称这个资源应该去它(例如 `"my_package.my_subpackage"` )。
* **资源** ( [*str*](https://docs.python.org/3/library/stdtypes.html #str "(Python v3.12)") ) – 资源的唯一名称，用于标识要加载的资源。
* **text** ( [*str*](https://docs.python.org /3/library/stdtypes.html#str "(Python v3.12)") ) – 要保存的内容。


*班级*


 火炬.package。


 包导入器


 ( *file_or_buffer
* , *module_allowed=<function PackageImporter.<lambda>>
* ) [[source]](_modules/torch/package/package_importer.html#PackageImporter)[¶](#torch.package.PackageImporter"此定义的永久链接")


 导入器允许您加载由 [`PackageExporter`](#torch.package.PackageExporter "torch.package.PackageExporter") 写入包的代码。代码以封闭的方式加载，使用包中的文件而不是普通的 python 导入系统。这允许打包 PyTorch 模型代码和数据，以便它可以在服务器上运行或在将来用于迁移学习。


 包的导入器确保模块中的代码只能从包内加载，导出期间明确列为外部的模块除外。zip 存档中的文件“extern_modules”列出了包外部依赖的所有模块。这可以防止包在本地运行的“隐式”依赖关系，因为它正在导入本地安装的包，但当包复制到另一台计算机时会失败。


 __在里面__


 ( *file_or_buffer
* , *module_allowed=<function PackageImporter.<lambda>>
* ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.__init__)[¶](#torch.package.PackageImporter.__init__"此定义的永久链接")


 打开 `file_or_buffer` 进行导入。这会检查导入的包是否只需要 `module_allowed` 允许的模块


 参数 
* **file_or_buffer** ( [*Union*](https://docs.python.org/3/library/typing.html#typing.Union "(in Python v3.12)")
* [
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*,
* *PyTorchFileReader
* *,
* [*路径
* ](https://docs.python.org/3/library/pathlib.html#pathlib.Path“(Python v3.12)”)*,
* [*BinaryIO*](https://docs.python. org/3/library/typing.html#typing.BinaryIO "(in Python v3.12)")*]
* ) – 类似文件的对象(必须实现 `read()` 、 `readline()` 、 ` tell()` 和 `seek()` )、字符串或包含文件名的 `os.PathLike` 对象。
* **module_allowed** ( *Callable
* *[
* *[
* [*str
* ](https://docs.python.org/3/library/stdtypes.html#str "(Python v3.12)")*]
* *,
* [*bool*](https://docs.python.org/3/library/functions.html#bool "(Python v3.12)")*]
* *,
* *可选
* ) – 确定是否允许外部提供的模块的方法。可用于确保加载的包不依赖于服务器不支持的模块。默认允许任何事情。


 提高


[**ImportError**](https://docs.python.org/3/library/exceptions.html#ImportError "(in Python v3.12)") – 如果包将使用不允许的模块。


 文件_结构


 ( *** ， *包括



 =
 


 '**'
* ， *排除



 =
 


 ()
* ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.file_struct)[¶](#torch.package.PackageImporter.file_struct "此定义的永久链接")


 返回包的 zip 文件的文件结构表示。


 参数 
* **include** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3 中).12)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* ) – 可选字符串，例如`"my_package.my_subpackage"` ，或可选的字符串列表，表示要包含在 zip 文件表示中的文件名称。这也可以是 glob 样式模式，如 [`PackageExporter.mock()`](#torch.package.PackageExporter.mock "torch.package.PackageExporter.mock")
* **排除** ( *Union
* *[
* *List
* *[
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中)")*]
* *,
* [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)")*]
* ) – 一个可选模式，排除名称与图案。


 退货


[`目录`](#torch.package.Directory "torch.package.Directory")


 Return type


[*目录*](#torch.package.Directory "torch.package.file_struct_representation.Directory")


 id
 


 ( ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.id)[¶](#torch.package.PackageImporter.id "此定义的永久链接")


 返回 torch.package 用于区分 [`PackageImporter`](#torch.package.PackageImporter "torch.package.PackageImporter") 实例的内部标识符。看起来像：


```
<torch_package_0>

```


 导入_模块


 ( *名称
* , *包



 =
 


 无
* ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.import_module)[¶](#torch.package.PackageImporter.import_module "此定义的永久链接")


 如果尚未加载模块，则从包中加载该模块，然后返回该模块。模块在本地加载到导入器，并将出现在 `self.modules` 而不是 `sys.modules` 中。


 参数 
* **name** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 的完全限定名称要加载的模块。
* **package** ( *[
* [*type*](https://docs.python.org/3/library/functions.html#type "(in Python v3.12)")
* ]
* *,
* *可选
* ) – 未使用，但存在以匹配 importlib.import_module 的签名。默认为“无”。


 退货


 (可能已经)加载的模块。


 Return type


[types.ModuleType](https://docs.python.org/3/library/types.html#types.ModuleType“(在Python v3.12中)”)


 加载_binary


 ( *package
* , *resource
* ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.load_binary)[¶](#torch.package.PackageImporter.load_binary "此定义的永久链接")


 加载原始字节。


 参数 
* **package** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 模块包的名称(例如 `"my_package.my_subpackage"` )。
* **资源** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 资源的唯一名称。


 退货


 加载的数据。


 Return type


[字节](https://docs.python.org/3/library/stdtypes.html#bytes“(在Python v3.12中)”)


 加载_pickle


 ( *包
* , *资源
* , *地图_位置



 =
 


 无
* ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.load_pickle)[¶](#torch.package.PackageImporter.load_pickle "此定义的永久链接")


 从包中解绑资源，使用 [`import_module()`](#torch.package.PackageImporter.import_module "torch.package.PackageImporter.import_module") 加载构造对象所需的任何模块。


 参数 
* **package** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 模块包的名称(例如 `"my_package.my_subpackage"` )。
* **资源** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 资源的唯一名称。
* **map_location** – 传递到 torch.load 以确定tensor如何映射到设备。默认为“无”。


 退货


 未腌制的物体。


 Return type


 Any
 


 加载_文本


 (*包*，*资源*，*编码



 =
 


 'utf-8'
* , *错误



 =
 


 'strict'
* ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.load_text)[¶](#torch.package.PackageImporter.load_text "此定义的永久链接")


 加载字符串。


 参数 
* **package** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 模块包的名称(例如 `"my_package.my_subpackage"` )。
* **资源** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 资源的唯一名称。
* **编码** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12)")*,
* *可选
* ) – 传递给 `decode` 。默认为 `'utf-8'`.
* **错误** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(在 Python v3.12 中) ")*,
* *可选
* ) – 传递给 `decode` 。默认为“严格”。


 退货


 加载的文本。


 Return type


[str](https://docs.python.org/3/library/stdtypes.html#str“(在Python v3.12中)”)


 python_version


 ( ) [[source]](_modules/torch/package/package_importer.html#PackageImporter.python_version)[¶](#torch.package.PackageImporter.python_version "此定义的永久链接")


 返回用于创建此包的 python 版本。


 注意：此功能是实验性的，不向前兼容。计划稍后将其移至锁定文件中。


 退货


`可选[str]` python 版本，例如3.8.9 或无(如果此包中未存储任何版本)


*班级*


 火炬.package。


 目录


 ( *name
* , *is_dir
* ) [[source]](_modules/torch/package/file_struction_representation.html#Directory)[¶](#torch.package.Directory "此定义的永久链接")


 文件结构表示。组织为具有目录子级列表的目录节点。包的目录是通过调用 [`PackageImporter.file_struct()`](#torch.package.PackageImporter.file_struct "torch.package.PackageImporter.file_struct") 创建的。


 有_file


 ( *filename
* ) [[source]](_modules/torch/package/file_struction_representation.html#Directory.has_file)[¶](#torch.package.Directory.has_file "此定义的永久链接")


 检查 [`Directory`](#torch.package.Directory "torch.package.Directory") 中是否存在文件。


 Parameters


**filename** ( [*str*](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.12)") ) – 要搜索的文件的路径。


 退货


 如果 [`Directory`](#torch.package.Directory "torch.package.Directory") 包含指定的文件。


 Return type


[bool](https://docs.python.org/3/library/functions.html#bool“(在Python v3.12中)”)