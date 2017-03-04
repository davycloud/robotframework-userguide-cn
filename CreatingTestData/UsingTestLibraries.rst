.. _Using test libraries:

使用测试库
====================

测试库中包含底层的关键字, 通常称之为 *库关键字*, 这些关键字与被测系统直接交互.

所有的测试用例都会使用到来自某个库中的关键字, 一般是通过更高层次的 `user keywords`_.

本章介绍如何使用测试库和其提供的关键字. `Creating test libraries`_ 在其它章节讨论.

.. contents::
   :depth: 2
   :local:

.. Importing libraries

导入库
-------------------

测试库一般是通过设置 :setting:`Library` 设置项来导入, 不过也可以通过使用调用关键字 :name:`Import Library` 来导入.

.. Using `Library` setting

使用 `Library` 设置
~~~~~~~~~~~~~~~~~~~~~~~

测试库通常在 Setting 表格中设置 :setting:`Library` 来导入, 库名称跟在 `Library` 后面. 不同于大部分的其它数据, 库名称既是大小写敏感的, 也是空白敏感的. 如果一个测试库是在某个包里的, 则必须指明完整的包名称路径.

某些情况下, 测试库可以接受参数. 这些参数跟在库名称后面. 和 `arguments to keywords`__ 差不多, 测试库的参数也可以使用默认值, 不定参数, 以及命名参数. 库名称和参数都可以使用变量. 例如:

__ `Using arguments`_

.. sourcecode:: robotframework

   *** Settings ***
   Library    OperatingSystem 
   Library    my.package.TestLibrary
   Library    MyLibrary    arg1    arg2
   Library    ${LIBRARY}

可以导入测试库的文件包括 `test case files`_, `resource files`_ 和 `test suite initialization files`_. 所有这些场景中, 在这些文件中导入的测试库中的所有关键字, 在文件内都是可见的. 对于资源文件, 这些关键字在引用这些资源文件的地方也是可见的.

.. Using `Import Library` keyword

使用 `Import Library` 关键字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

导入测试库的另一种方式是使用 BuiltIn_ 库提供的关键字 :name:`Import Library`. 该关键字接受库名称以及可能的参数作为它的参数. 被导入的库中的关键字在调用 :name:`Import Library` 关键字的测试套件中可用.

当测试库在测试执行前不能导入, 只能在执行过程中通过某些关键字来启用时, 这种方法就可以发挥作用了.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Do Something 
       Import Library    MyLibrary    arg1    arg2
       KW From MyLibrary

.. Specifying library to import

指定要导入的库
----------------------------

要导入的库既可以通过库的名称, 也可以通过库的路径来指定. 这两种方式, 不管是使用设置  :setting:`Library` 还是调用 :name:`Import Library` 关键字, 都是一样的.

Libraries to import can be specified either by using the library name
or the path to the library. These approaches work the same way regardless
is the library imported using the :setting:`Library` setting or the
:name:`Import Library` keyword.

.. Using library name

使用库名
~~~~~~~~~~~~~~~~~~

最常见的是使用库名称来指定要导入的测试库, 就像本章前面所有的例子所示那样. 这种情况下, Robot Framework 试图从 `module search path`_ 中查找实现该库的类或者模块. 一般来讲, 通过安装的方式创建的库应该自动在模块搜寻路径内, 其他库的搜索路径可能需要单独的配置.

这种方法最大的好处是当模块搜索路径配置完成后(通常使用 `start-up script`_ 完成), 普通用户无需关注测试库实际安装位置. 缺点则是当要导入自己写的, 有可能很简单的一个库, 需要进行额外的配置.

.. Using physical path to library

使用库的路径
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

另一种导入库的方法是使用文件系统的路径来指定库. 类似于指定 `resource and variable files`_ 的路径, 库的路径也被认为是相对于当前测试数据文件所在目录的(当然, 指定绝对路径也是可以的). 

这种方式最大的好处是无需配置模块搜索路径.

Another mechanism for specifying the library to import is using a
path to it in the file system. This path is considered relative to the
directory where current test data file is situated similarly as paths
to `resource and variable files`_. The main benefit of this approach
is that there is no need to configure the module search path.

如果库是一个文件, 则路径必须包含扩展名. 对Python库, 扩展名是 :file:`.py`, 对Java库则是 :file:`.class` 或 :file:`.java`, 不过.class文件必须存在. 如果Python库是一个文件夹, 该路径最后必须有一个斜杠结尾(`/`).

下面的例子展示了几种不同的情况.

.. sourcecode:: robotframework

   *** Settings ***
   Library    PythonLibrary.py
   Library    /absolute/path/JavaLibrary.java
   Library    relative/path/PythonDirLib/    possible    arguments
   Library    ${RESOURCES}/Example.class


该方法的一个缺陷是, Python类实现的库名称必须和模块名一样. 此外, 使用 JAR包或者ZIP包发布的库不能使用这种方式.

`must be in a module with the same name as the class`__

__ `Test library names`_

.. Setting custom name to test library

测试库设置自定义名称
-----------------------------------

测试库的名称最终会在日志文件中的关键字名称的前面展示, 如果多个关键字重名, 则库名称必须作为 `keyword name is prefixed with the library name`__.

库的名称一般就是实现该库的模块或类名, 但在有些情况下有改变的需求:

- 需要不止一次的导入一个相同的库, 每次使用不同的参数. 如果每次是相同的名称则不可能做到.

- 库名非常长, 不方便. 比如, 有超长包名的Java库.

- 在不同的环境中使用不同的测试库, 但是希望以相同的名称引用它们.

- 库的名称起的不好, 有误导作用, 这时候改名当然是个更好的选择.

__ `Handling keywords with same names`_

指定新的名称的语法格式是使用 `WITH NAME` (此处区分大小写) 跟在原库名称的后面, 后面再跟上新的名称. 新指定的名称将展示在logs文件中, 并且当需要指定关键字的全名(:name:`LibraryName.Keyword Name`)时, 其中的库名也应该使用新的名字.

.. sourcecode:: robotframework

   *** Settings ***
   Library    com.company.TestLib    WITH NAME    TestLib
   Library    ${LIBRARY}             WITH NAME    MyName

如果库需要参数, 参数的位置在原库名和 `WITH NAME` 之间. 下面的例子展示了如何使用不同的参数多次导入同一个库.

.. sourcecode:: robotframework

   *** Settings ***
   Library    SomeLibrary    localhost        1234    WITH NAME    LocalLib
   Library    SomeLibrary    server.domain    8080    WITH NAME    RemoteLib

   *** Test Cases ***
   My Test
       LocalLib.Some Keyword     some arg       second arg
       RemoteLib.Some Keyword    another arg    whatever
       LocalLib.Another Keyword

使用 `WITH NAME` 指定库的别名的方法同样适用于 :name:`Import Library` 关键字.

.. Standard libraries

标准库
------------------

随 Robot Framework 版本一同发布的测试库称之为 *标准库*. 其中 BuiltIn_ 最特别, 因为它总是自动启用, 也就是说其中的关键字总是可用的. 其它的标准库如果要使用的话则需要导入.

.. Normal standard libraries

普通标准库
~~~~~~~~~~~~~~~~~~~~~~~~~

可用的标准库如下所列:

  - BuiltIn_
  - Collections_
  - DateTime_
  - Dialogs_
  - OperatingSystem_
  - Process_
  - Screenshot_
  - String_
  - Telnet_
  - XML_

.. _BuiltIn: ../libraries/BuiltIn.html
.. _Collections: ../libraries/Collections.html
.. _DateTime: ../libraries/DateTime.html
.. _Dialogs: ../libraries/Dialogs.html
.. _OperatingSystem: ../libraries/OperatingSystem.html
.. _Process: ../libraries/Process.html
.. _String: ../libraries/String.html
.. _Screenshot: ../libraries/Screenshot.html
.. _Telnet: ../libraries/Telnet.html
.. _XML: ../libraries/XML.html

.. Remote library

远程库
~~~~~~~~~~~~~~

除了上面列的普通标准库, 还有一个特殊的标准库, 远程库(:name:`Remote`). 远程库没有关键字, 它作为一个代理存在于Robot Framework和实际(远程的)测试库中间. 实际的测试库可以运行在其它机器上, 而且实现语言也不再限于Robot Framework原生支持的编程语言.

更多关于远程库的内容请参考 `Remote library interface`_ 章节.

.. External libraries

外部库
------------------

标准库之外的其它测试库都统称为 *外部库*. Robot Framework开源社区实现了若干通用的库, 比如 Selenium2Library_ 和 SwingLibrary_, 不过这些库并没有和框架打包发布. 要查看有哪些公开可用的外部库可以登录官网 http://robotframework.org.

通用的, 或定制的测试库, 显然都是由使用Robot Framework框架的小组实现的. 关于如何自己开发测试库, 请参考 `Creating test libraries`_ 章节.

不同的外部库有各种不同安装和使用方式. 有时可能还需要安装其它依赖. 所有测试库都应该有清晰的安装文档和使用说明, 最好实现自动安装.
