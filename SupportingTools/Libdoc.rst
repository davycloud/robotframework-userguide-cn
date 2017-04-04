.. role:: name(emphasis)
.. role:: setting(emphasis)

.. _libdoc:

库文档工具(Libdoc)
===================

`原文 <http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#library-documentation-tool-libdoc>`_

.. contents::
   :depth: 1
   :local:

Libdoc是Robot Framework内置的工具, 用来为测试库和资源文件生成关键字的文档. 文档分为HTML和XML格式, 前者供人阅读, 后者供 RIDE_ 和其它工具使用. Libdoc还提供了几个特别的命令来在控制台显示库和资源的信息.

可以创建的文档包括:

- 使用 :ref:`Python <Python libraries>` 或 :ref:`Java <Java libraries>` 开发的静态测试库,
- :ref:`动态API <dynamic libraries>` 测试库, 包括远程库,
- :ref:`resource files`

此外, 还可以使用以前Libdoc创建的XML spec文件作为输入.


通常用法
--------

.. Synopsis

总览
~~~~

::

    python -m robot.libdoc [options] library_or_resource output_file
    python -m robot.libdoc [options] library_or_resource list|show|version [names]

.. Options

选项
~~~~

  -f, --format <html|xml>  Specifies whether to generate HTML or XML output.
                           If this options is not used, the format is got
                           from the extension of the output file.
  -F, --docformat <robot|html|text|rest>
                           Specifies the source documentation format. Possible
                           values are Robot Framework's documentation format,
                           HTML, plain text, and reStructuredText. Default value
                           can be specified in test library source code and
                           the initial default value is `robot`.
                           New in Robot Framework 2.7.5.
  -N, --name <newname>     Sets the name of the documented library or resource.
  -V, --version <newversion>  Sets the version of the documented library or
                           resource. The default value for test libraries is
                           :ref:`got from the source code <specifying library version>`.
  -P, --pythonpath <path>  Additional locations where to search for libraries
                           and resources similarly as when :ref:`running tests <Using --pythonpath option>`.
  -E, --escape <what:with>  Escapes characters which are problematic in console.
                           `what` is the name of the character to escape
                           and `with` is the string to escape it with.
                           Available escapes are listed in the :option:`--help`
                           output.
  -h, --help               Prints this help.


.. Alternative execution

其它执行方式
~~~~~~~~~~~~

虽然上面的总览中只用Python来执行Libdoc, 但是还可以使用Jython和IronPython. 实际上, 要为Java库生成文档时, Jython是必需的.

上面Libdoc是作为一个安装模块(``python -m robot.libdoc``)来执行, 此外还可以作为脚本来执行::

    python path/robot/libdoc.py [options] arguments

如果你是 :ref:`manual installation`, 或者只是把 :file:`robot` 目录拷贝到系统某个目录时,  按脚本执行就会非常有用.


.. Specifying library or resource file

指定测试库或资源文件
~~~~~~~~~~~~~~~~~~~~

.. Python libraries and dynamic libraries with name or path

Python库和动态库的名字或路径
''''''''''''''''''''''''''''''''''''

当测试库是用Python实现的, 或者使用了 :ref:`dynamic library API`, 那么可以通过库的名字或者库源码所在路径来指定. 当使用名字时, 将在 :ref:`module search path` 内搜索测试库, 并且格式必须和在Robot Framework的测试数据中一样.

如果测试库的导入需要参数, 这些参数必须跟在名字或路径的后面, 使用双冒号连接, 像 ``MyLibrary::arg1::arg2`` 这样. 如果参数的变化导致库中关键字变化, 或者是文档变化, 则最好使用 :option:`--name` 相应修改库的名字. (也就是说为不同参数生成不同名字的文档.)

.. Java libraries with path

Java库的路径
'''''''''''''

使用 :ref:`static library API` 实现的Java测试库可以指定源文件的路径, 此外, 当Libdoc运行时, 必须要能在 ``CLASSPATH``内找到 :file:`tools.jar` (Java JDK的一部分). 

注意, 为Java测试库生成文档只能使用Jython.

.. Resource files with path

资源文件的路径
''''''''''''''

资源文件必须总是通过路径指定. 如果路径不存在, 资源文件还是会像执行测试用例时一样, 尝试在 :ref:`module search path` 内进行搜索.

.. Generating documentation

生成文档
~~~~~~~~

当要生成HTML或XML格式的文档时, 输出文件必须作为第2参数指定, 跟在测试库/资源文件的名字或路径后面. 输出的格式将自动通过文件扩展名来判断, 也可以通过选项 :option:`--format` 设置.

一些例子::

   python -m robot.libdoc OperatingSystem OperatingSystem.html
   python -m robot.libdoc --name MyLibrary Remote::http://10.0.0.42:8270 MyLibrary.xml
   python -m robot.libdoc test/resource.html doc/resource_doc.html
   jython -m robot.libdoc --version 1.0 MyJavaLibrary.java MyJavaLibrary.html
   jython -m robot.libdoc my.organization.DynamicJavaLibrary my.organization.DynamicJavaLibrary.xml

.. Viewing information on console

在控制台查看信息
~~~~~~~~~~~~~~~

Libdoc有3个特别的命令在控制台显示信息. 这些命令被用来取代输出文件的名字, 并且它们也可以带上额外的参数.

``list``
    列出测试库或资源文件所包含的关键字列表. 可以传递表示模式的参数来过滤展示结果, 只有那些名字包含了给定模式的关键字才会列出来.
``show``
    显示测试库或资源文件的文档. 也可以传递表示名字的参数(可以是多个). 名字匹配上的关键字才会展示. 特殊参数值 `intro` 可用来只显示测试库的介绍(introduction)和导入章节.
``version``
    显示测试的版本.

``list`` 和 ``show`` 命令支持的可选名字模式都是大小写和空格无关的, 并且都支持使用 ``*`` 和 ``?`` 作为通配符.

例如::

  python -m robot.libdoc Dialogs list
  python -m robot.libdoc Selenium2Library list browser
  python -m robot.libdoc Remote::10.0.0.42:8270 show
  python -m robot.libdoc Dialogs show PauseExecution execute*
  python -m robot.libdoc Selenium2Library show intro
  python -m robot.libdoc Selenium2Library version

.. Writing documentation

编写文档
--------

本章讨论如何为基于 :ref:`Python <Python libraries>` 和 :ref:`Java <Java libraries>` 的测试库(静态和动态)和 :ref:`资源文件 <resource file documentation>` 编写文档.

:ref:`Creating test libraries` 和 :ref:`resource files` 的更多细节在本手册其它章节讨论.


.. _Python libraries:

Python测试库
~~~~~~~~~~~~~

使用 :ref:`static library API` 的Python测试库的文档就是实现关键字的类或模块或方法的文档字符串. 文档的第一行被视作关键字的简短介绍(例如, 可以是某个工具提示的链接), 所以要求尽可能有概括性, 不要太长.  

下面这个简单的例子展示了通常文档是怎么写的, 本章最后还有一个 :ref:`略长的例子 <Libdoc example>`.

.. sourcecode:: python

    class ExampleLib:
        """Library for demo purposes.

        This library is only used in an example and it doesn't do anything useful.
        """

        def my_keyword(self):
            """Does nothing."""
            pass

        def your_keyword(self, arg):
            """Takes one argument and *does nothing* with it.

            Examples:
            | Your Keyword | xxx |
            | Your Keyword | yyy |
            """
            pass

.. tip:: 如果你要Python库的文档里使用non-ASCII字符(比如中文), 
         必须设置 `源码的encoding <http://www.python.org/dev/peps/pep-0263>`_ 为UTF-8, 或者以Unicode创建文档字符串.

         关于Python文档字符串更多内容, 请参阅 `PEP-257`_.

.. _PEP-257: http://www.python.org/dev/peps/pep-0257

.. _Java libraries:

Java测试库
~~~~~~~~~~~

使用 :ref:`static library API` 的Java测试库的文档就是实现关键字的类或方法的 `文档注释 <http://en.wikipedia.org/wiki/Javadoc>`_. 

此时Libdoc内部实际使用的是Javadoc工具, 因此才要求 :file:`tools.jar` 必须包含在 ``CLASSPATH``. 该jar文件是Java SDK的一部分, 应该可以在SDK安装路径的 :file:`bin` 目录下.

下面这个简单例子中的文档和上面的Python例子完全一样(功能也是一样的).

.. sourcecode:: java

    /**
     * Library for demo purposes.
     *
     * This library is only used in an example and it doesn't do anything useful.
     */
    public class ExampleLib {

        /**
         * Does nothing.
         */
        public void myKeyword() {
        }

        /**
         * Takes one argument and *does nothing* with it.
         *
         * Examples:
         * | Your Keyword | xxx |
         * | Your Keyword | yyy |
         */
        public void yourKeyword(String arg) {
        }
    }


.. _dynamic libraries:

动态测试库
~~~~~~~~~~

为了给动态测试库生成有意义的文档, 测试库必须使用 ``get_keyword_arguments`` 和 ``get_keyword_documentation`` 这两个方法返回关键字的参数名字和文档(方法名还可是camelCase命名 ``getKeywordArguments`` 和 ``getKeywordDocumentation``). 

还可以为 ``get_keyword_documentation`` 设置两个特殊的属性 ``__intro__`` 和 ``__init__`` 来实现测试库的通用文档.

更多信息和示例请参见 :ref:`Dynamic library API` 章节.

.. _importing section:

导入段落
~~~~~~~~

文档中有一个单独的章节用来说明测试库是如何导入的, 这部分是基于库的初始化方法. 

对Python库而言, 如果 ``__init__`` 方法除了 ``self`` 还有其它参数, 则方法的文档和参数都会展示. 对Java库来说, 如果有接受public的构造函数, 所有public构造函数都会展示.

.. sourcecode:: python

   class TestLibrary:

       def __init__(self, mode='default')
           """Creates new TestLibrary. `mode` argument is used to determine mode."""
           self.mode = mode

       def some_keyword(self, arg):
           """Does something based on given `arg`.

           What is done depends on the `mode` specified when `importing` the library.
           """
           if self.mode == 'secret':
                # ...

.. _Resource file documentation:

资源文件的文档
~~~~~~~~~~~~~~

资源文件中的关键字可以使用 :setting:`[Documentation]` 来设置文档.

文档的第一行(直到 :ref:`implicit newline <Newlines in test data>` 或显式地 ``\n``)被视作关键字的简短介绍, 和测试库类似.

资源文件还可以在Setting表格中通过 :setting:`Documentation` 为整个资源文件设置文档.

资源文件还可能包含变量, 这些变量不会记入文档.

.. sourcecode:: robotframework

   *** Settings ***
   Documentation    Resource file for demo purposes.
   ...              This resource is only used in an example and it doesn't do anything useful.

   *** Keywords ***
   My Keyword
       [Documentation]   Does nothing
       No Operation

   Your Keyword
       [Arguments]  ${arg}
       [Documentation]   Takes one argument and *does nothing* with it.
       ...
       ...    Examples:
       ...    | Your Keyword | xxx |
       ...    | Your Keyword | yyy |
       No Operation


.. Documentation syntax

文档的语法
----------

Libdoc支持的文档语法包括Robot Framework自身的 :ref:`documentation syntax`, HTML, 纯文本和 reStructuredText_. 测试库文档的格式可以通过在 :ref:`测试库源码 <Specifying documentation format>` 中设置属性 ``ROBOT_LIBRARY_DOC_FORMAT`` 来指定, 也可以通过命令行选项 :option:`--docformat (-F)` 来指定. 这两种情况下, 对应上面4种类型的值分别是: ``ROBOT`` (默认值), ``HTML``, ``TEXT`` and ``reST``. 注意这些值是大小写无关的.

Robot Framework自己的文档格式是默认的, 也是推荐使用的格式. 如果测试库代码是已存在并且已经包含了文档, 则其它的格式就会很有用.

其它格式是在 Robot Framework 2.7.5开始支持的.

.. Robot Framework documentation syntax

Robot Framework文档语法
~~~~~~~~~~~~~~~~~~~~~~~~

Robot Framework自己的 :ref:`documentation syntax` 中最最重要的特性就是使用 ``*bold*`` and ``_italic_``, 自定义链接, 自动转换URL, 创建表格, 以及使用竖线的格式化文本块(常用来展示例子). 如果文档篇幅略长, 则Robot Framework 2.7.5版本后开始支持的章节标题功能也很方便.

下面的例子展示了一些最重要的格式化功能. 注意, 由于这是默认的格式, 所以没有必要设置 `ROBOT_LIBRARY_DOC_FORMAT` 属性, 也无需在命令行指定格式.

.. sourcecode:: python

    """Example library in Robot Framework format.

    - Formatting with *bold* and _italic_.
    - URLs like http://example.com are turned to links.
    - Custom links like [http://robotframework.org|Robot Framework] are supported.
    - Linking to `My Keyword` works.
    """

    def my_keyword():
        """Nothing more to see here."""

.. HTML documentation syntax

HTML文档语法
~~~~~~~~~~~~~

当使用HTML格式时, 基本上可以自由使用HTML的语法来创建文档. 主要的缺点是HTML的标记符号可读性不高, 所以源代码中的文档维护和阅读会比较困难.

以HTML写成的文档Libdoc不做转换和转义处理, 直接使用. 不过还支持一个特殊的语法用来 :ref:`链接到关键字 <linking to keywords>`, 格式如: ```My Keyword```.

下面的例子包含了前面例子中相同的格式. 这里必须要设定 ``ROBOT_LIBRARY_DOC_FORMAT``  属性, 或者要在命令行中给定 ``--docformat HTML``.

.. sourcecode:: python

    """Example library in HTML format.

    <ul>
      <li>Formatting with <b>bold</b> and <i>italic</i>.
      <li>URLs are not turned to links automatically.
      <li>Custom links like <a href="http://www.w3.org/html">HTML</a> are supported.
      <li>Linking to `My Keyword` works.
    </ul>
    """
    ROBOT_LIBRARY_DOC_FORMAT = 'HTML'

    def my_keyword():
        """Nothing more to see here."""

.. Plain text documentation syntax

纯文本文档语法
~~~~~~~~~~~~~~

使用纯文本格式时, Libdoc基本上是照原来的样子使用. 换行和其它空格也保留, 除了缩进. HTML特殊字符 (``<>&``) 将转义处理. 唯一做的格式化操作是将URL转换为可点击的链接, 并且支持 :ref:`internal linking` 如  ```My Keyword```.


.. sourcecode:: python

    """Example library in plain text format.

    - Formatting is not supported.
    - URLs like http://example.com are turned to links.
    - Custom links are not supported.
    - Linking to `My Keyword` works.
    """
    ROBOT_LIBRARY_DOC_FORMAT = 'text'

    def my_keyword():
        """Nothing more to see here"""

.. reStructuredText documentation syntax

reStructuredText文档语法
~~~~~~~~~~~~~~~~~~~~~~~~~

reStructuredText_ 是一个简单但是又很强大的标记语言, 被广泛用在Python项目的文档中(包括本用户手册). 使用该格式的最大限制是必须要安装 docutils_ 模块才能生成文档. 

因为反引号(backtick)在reStructuredText中也有特殊意义, 所以要使用 `链接到关键字`_ 时, 必须要进行转义, 如: ``\`My Keyword\```.


.. sourcecode:: python

    """Example library in reStructuredText format.

    - Formatting with **bold** and *italic*.
    - URLs like http://example.com are turned to links.
    - Custom links like reStructuredText__ are supported.
    - Linking to \`My Keyword\` works but requires backtics to be escaped.

    __ http://docutils.sourceforge.net
    """
    ROBOT_LIBRARY_DOC_FORMAT = 'reST'

    def my_keyword():
        """Nothing more to see here"""

.. _internal linking:

内部链接
--------

Libdoc支持在文档的不同部分生成关键字的内部链接. 使用反引号将目标的名字括起来即可, 例如 ```target```. 目标名字是大小写无关的, 支持哪些目标将在下面的章节介绍. 

如果要链接的目标没有找到, Libdoc也不会报错或警告, 只是将文字设为斜体. 早期的时候, 曾经推荐使用这种方式来引用关键字的参数, 但是这有可能导致无意中创建了内部链接. 现在则推荐使用双反引号来标示参数, 例如 ````argument````. 使用单反引号而没有找到的链接目标的情况在未来的版本中有可能会以报错处理.

除了下面小节中的例子, 内部链接和参数格式化都将在本章结尾的 :ref:`长篇实例 <Libdoc example>` 中展示.


.. _linking to keywords:

链接到关键字
~~~~~~~~~~~~

测试库中所有的关键字都会自动创建链接目标, 可通过诸如 ```Keyword Name``` 的语法链接到. 下面的例子展示了两个关键字之间互相链接.


.. sourcecode:: python

   def keyword(log_level="INFO"):
       """Does something and logs the output using the given level.

       Valid values for log level` are "INFO" (default) "DEBUG" and "TRACE".

       See also `Another Keyword`.
       """
       # ...

   def another_keyword(argument, log_level="INFO"):
       """Does something with the given argument else and logs the output.

       See `Keyword` for information about valid log levels.
       """
       # ...

.. note:: 当使用 `reStructuredText文档语法`_ 时, 反引号必须要转义.

.. Linking to automatic sections

链接到段落
~~~~~~~~~~

Libdoc生成的文档总是自动包含了几个段落, 包括库的概述, 关键字的快捷方式, 和实际关键字. 如果测试库导入还需参数, 则还有单独的一个 :ref:`importing section`.

所有这些段落都是可作为链接目标的, 目标名字参见下表. 如何使用参见下一节的示例.


.. table:: Automatic section link targets
   :class: tabular

   ================  ===========================================================
        Section                               Target
   ================  ===========================================================
   Introduction      ```introduction``` and ```library introduction```
   Importing         ```importing``` and ```library importing```
   Shortcuts         ```shortcuts``` (New in Robot Framework 2.7.5.)
   Keywords          ```keywords``` (New in Robot Framework 2.7.5.)
   ================  ===========================================================

.. Linking to custom sections

链接到自定义段落
~~~~~~~~~~~~~~~~

从2.7.5版本开始, Robot Framework的 :ref:`documentation syntax` 开始支持自定义 :ref:`section titles`, 其中的标题自动创建为链接目标.

下面的例子展示了如何链接到自动段落和自定义段落.

.. sourcecode:: python

   """Library for Libdoc demonstration purposes.

   This library does not do anything useful.

   = My section  =

   We do have a custom section in the documentation, though.
   """

   def keyword():
       """Does nothing.

       See `introduction` for more information and `My section` to test how
       linking to custom sections works.
       """
       pass

.. note:: 链接到自定义段落只在使用 `Robot Framework文档语法`_ 时可用.

.. note:: Robot Framework 2.8版本之前, 段落标题只有第一级标题可以链接.

.. Representing arguments

表示参数
--------

Libdoc自动处理关键字的参数, 测试库中方法的参数和资源文件中用户关键字的参数, 将这些参数单独一列展示. 用户关键字的参数将去除 ``${}`` 或 ``@{}``, 以使得这些参数不管是来自哪里看起来都一个样. 

不管关键字是如何实现的, Libdoc都如同在Python中创建关键字那样的展示. 这个格式用下面的表格展示会更直接点.


.. table:: How Libdoc represents arguments
   :class: tabular

   +--------------------+----------------------------+--------------------------+
   |      Arguments     |      Now represented       |        Examples          |
   +====================+============================+==========================+
   | No arguments       | Empty column.              |                          |
   +--------------------+----------------------------+--------------------------+
   | One or more        | List of strings containing | | ``one_argument``       |
   | argument           | argument names.            | | ``a1, a2, a3``         |
   +--------------------+----------------------------+--------------------------+
   | Default values     | Default values separated   | | ``arg=default value``  |
   | for arguments      | from names with ``=``.     | | ``a, b=1, c=2``        |
   +--------------------+----------------------------+--------------------------+
   | Variable number    | Last (or second last with  | | ``*varargs``           |
   | of arguments       | kwargs) argument has ``*`` | | ``a, b=42, *rest``     |
   | (varargs)          | before its name.           |                          |
   +--------------------+----------------------------+--------------------------+
   | Free keyword       | Last arguments has         | | ``**kwargs``           |
   | arguments (kwargs) | ``**`` before its name.    | | ``a, b=42, **kws``     |
   |                    |                            | | ``*varargs, **kwargs`` |
   +--------------------+----------------------------+--------------------------+

当要在文档中引用关键字的参数时, 推荐使用 :ref:`行内代码样式 <inline styles_>`__ 来表示, 例如 ````argument````.


.. _Libdoc example:

Libdoc示例
-----------

下面的例子比较完整的展示了如何使用 `文档格式化`_ 中最有用的功能, :ref:`internal linking` 等等. 

点击 :doc:`LoggingLibrary.html` 查看生成的文档是什么样.

.. literalinclude:: LoggingLibrary.py



所有 `标准库`_ 都提供了由Libdoc生成的文档, 这些文档(以及源代码)都可作为更加真实的例子来参考学习.

__ ./LoggingLibrary.html
