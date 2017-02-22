.. _libdoc:

.. Library documentation tool (Libdoc)

库文档工具(Libdoc)
===================================

.. contents::
   :depth: 1
   :local:

Libdoc是Robot Framework内置的工具, 用来为测试库和资源文件生成关键字的文档. 文档分为HTML和XML格式, 前者供人阅读, 后者供 RIDE_ 和其它工具使用. Libdoc还提供了几个特别的命令来在控制台显示库和资源的信息.

可以创建的文档包括:

- 使用 Python__ 或 Java__ 开发的静态测试库,
- `动态API`__ 测试库, 包括远程库,
- `资源文件`_

- test libraries implemented with Python__ or Java__ using the normal
  static library API,
- test libraries using the `dynamic API`__, including remote libraries, and
- `resource files`_.

此外, 还可以使用以前Libdoc创建的XML spec文件作为输入.

__ `Python libraries`_
__ `Java libraries`_
__ `Dynamic libraries`_

.. General usage

通常用法
-------------

.. Synopsis

总览
~~~~~~~~

::

    python -m robot.libdoc [options] library_or_resource output_file
    python -m robot.libdoc [options] library_or_resource list|show|version [names]

.. Options

选项
~~~~~~~

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
                           `got from the source code`__.
  -P, --pythonpath <path>  Additional locations where to search for libraries
                           and resources similarly as when `running tests`__.
  -E, --escape <what:with>  Escapes characters which are problematic in console.
                           `what` is the name of the character to escape
                           and `with` is the string to escape it with.
                           Available escapes are listed in the :option:`--help`
                           output.
  -h, --help               Prints this help.

__ `Specifying library version`_
__ `Using --pythonpath option`_

.. Alternative execution

其它执行方式
~~~~~~~~~~~~~~~~~~~~~

虽然上面的总览中只用Python来执行Libdoc, 但是还可以使用Jython和IronPython. 实际上, 要为Java库生成文档时, Jython是必需的.

上面Libdoc是作为一个安装模块(`python -m robot.libdoc`)来执行, 此外还可以作为脚本来执行::

    python path/robot/libdoc.py [options] arguments

如果你是 `手动安装`_, 或者只是把 :file:`robot` 目录拷贝到系统某个目录时,  按脚本执行就会非常有用.


.. Specifying library or resource file

指定测试库或资源文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Python libraries and dynamic libraries with name or path

Python库和动态库的名字或路径
''''''''''''''''''''''''''''''''''''

当测试库是用Python实现的, 或者使用了 `动态库API`_, 那么可以通过库的名字或者库源码所在路径来指定. 当使用名字时, 将在 `模块搜索路径`_ 内搜索测试库, 并且格式必须和在Robot Framework的测试数据中一样.

如果测试库的导入需要参数, 这些参数必须跟在名字或路径的后面, 使用双冒号连接, 像 `MyLibrary::arg1::arg2` 这样. 如果参数的变化导致库中关键字变化, 或者是文档变化, 则最好使用 :option:`--name` 相应修改库的名字. (也就是说为不同参数生成不同名字的文档.)

.. Java libraries with path

Java库的路径
''''''''''''''''''''''''

使用 `静态库API`_ 实现的Java测试库可以指定源文件的路径, 此外, 当Libdoc运行时, 必须要能在 ``CLASSPATH``内找到 :file:`tools.jar` (Java JDK的一部分). 

注意, 为Java测试库生成文档只能使用Jython.

.. Resource files with path

资源文件的路径
''''''''''''''''''''''''

资源文件必须总是通过路径指定. 如果路径不存在, 资源文件还是会像执行测试用例时一样, 尝试在 `模块搜索路径`_ 内进行搜索.

.. Generating documentation

生成文档
~~~~~~~~~~~~~~~~~~~~~~~~

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

`list`
    列出测试库或资源文件所包含的关键字列表. 可以传递表示模式的参数来过滤展示结果, 只有那些名字包含了给定模式的关键字才会列出来.
`show`
    显示测试库或资源文件的文档. 也可以传递表示名字的参数(可以是多个). 名字匹配上的关键字才会展示. 特殊参数值 `intro` 可用来只显示测试库的介绍(introduction)和导入章节.
`version`
    显示测试的版本.

`show`
    Show library/resource documentation. Can be limited to show only
    certain keywords by passing names as arguments. Keyword is shown if
    its name matches any given name. Special argument `intro` will show
    only the library introduction and importing sections.
`version`
    Show library version

`list` 和 `show` 命令支持的可选名字模式都是大小写和空格无关的, 并且都支持使用 `*` 和 `?` 作为通配符.

例如::

  python -m robot.libdoc Dialogs list
  python -m robot.libdoc Selenium2Library list browser
  python -m robot.libdoc Remote::10.0.0.42:8270 show
  python -m robot.libdoc Dialogs show PauseExecution execute*
  python -m robot.libdoc Selenium2Library show intro
  python -m robot.libdoc Selenium2Library version

.. Writing documentation

编写文档
---------------------

本章讨论如何为基于 Python_ 和 Java_ 的测试库(静态和动态)和 `资源文件`__ 编写文档.
`创建测试库`_ 和 `资源文件`_ 的更多细节在本手册其它章节讨论.

This section discusses writing documentation for Python__ and Java__ based test
libraries that use the static library API as well as for `dynamic libraries`_
and `resource files`__. `Creating test libraries`_ and `resource files`_ is
described in more details elsewhere in the User Guide.

__ `Python libraries`_
__ `Java libraries`_
__ `Resource file documentation`_

.. Python libraries

Python测试库
~~~~~~~~~~~~~~~~

使用 `静态库API`_ 的Python测试库的文档就是实现关键字的类或模块或方法的文档字符串. 文档的第一行被视作关键字的简短介绍(例如, 可以是某个工具提示的链接), 所以要求尽可能有概括性, 不要太长.  

下面这个简单的例子展示了通常文档是怎么写的, 本章最后还有一个 `略长的例子`__.

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
         必须设置 `源码的encoding`__ 为UTF-8, 或者以Unicode创建文档字符串.

         关于Python文档字符串更多内容, 请参阅 `PEP-257`__.

__ `Libdoc example`_
__ http://www.python.org/dev/peps/pep-0263
__ http://www.python.org/dev/peps/pep-0257

.. Java libraries

Java测试库
~~~~~~~~~~~~~~

使用 `静态库API`_ 的Java测试库的文档就是实现关键字的类或方法的 `文档注释`__. 此时Libdoc内部实际使用的是Javadoc工具, 因此才要求 :file:`tools.jar` 必须包含在 ``CLASSPATH``. 该jar文件是Java SDK的一部分, 应该可以在SDK安装路径的 :file:`bin` 目录下.

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

__ http://en.wikipedia.org/wiki/Javadoc

.. Dynamic libraries

动态测试库
~~~~~~~~~~~~~~~~~

为了给动态测试库生成有意义的文档, 测试库必须使用 `get_keyword_arguments` 和 `get_keyword_documentation` 这两个方法返回关键字的参数名字和文档(方法名还可是camelCase命名 `getKeywordArguments` 和 `getKeywordDocumentation`). 

还可以为 `get_keyword_documentation` 设置两个特殊的属性 `__intro__` 和 `__init__` 来实现测试库的通用文档.

更多信息和示例请参见 `Dynamic library API`_ 章节.

To be able to generate meaningful documentation for dynamic libraries,
the libraries must return keyword argument names and documentation using
`get_keyword_arguments` and `get_keyword_documentation`
methods (or using their camelCase variants `getKeywordArguments`
and `getKeywordDocumentation`). Libraries can also support
general library documentation via special `__intro__` and
`__init__` values to the `get_keyword_documentation` method.

See the `Dynamic library API`_ section for more information about how to
create these methods.

.. Importing section

导入部分
~~~~~~~~~~~~~~~~~

文档中有一个单独的章节用来说明测试库是如何导入的, 这部分是基于库的初始化方法. 

对Python库而言, 如果 `__init__` 方法除了 `self` 还有其它参数, 则方法的文档和参数都会展示. 对Java库来说, 如果有接受public的构造函数, 所有public构造函数都会展示.

A separate section about how the library is imported is created based on its
initialization methods. For a Python library, if it has an  `__init__`
method that takes arguments in addition to `self`, its documentation and
arguments are shown. For a Java library, if it has a public constructor that
accepts arguments, all its public constructors are shown.

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

.. Resource file documentation

资源文件的文档
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Keywords in resource files can have documentation using
:setting:`[Documentation]` setting, and this documentation is also used by
Libdoc. First line of the documentation (until the first
`implicit newline`__ or explicit `\n`) is considered to be the short
documentation similarly as with test libraries.

Also the resource file itself can have :setting:`Documentation` in the
Setting table for documenting the whole resource file.

Possible variables in resource files can not be documented.

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

__ `Newlines in test data`_

Documentation syntax
--------------------

Libdoc supports documentation in Robot Framework's own `documentation
syntax`_, HTML, plain text, and reStructuredText_. The format to use can be
specified in `test library source code`__ using `ROBOT_LIBRARY_DOC_FORMAT`
attribute or given from the command line using :option:`--docformat (-F)` option.
In both cases the possible case-insensitive values are `ROBOT` (default),
`HTML`, `TEXT` and `reST`.

Robot Framework's own documentation format is the default and generally
recommended format. Other formats are especially useful when using existing
code with existing documentation in test libraries. Support for other formats
was added in Robot Framework 2.7.5.

__ `Specifying documentation format`_

Robot Framework documentation syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Most important features in Robot Framework's `documentation syntax`_ are
formatting using `*bold*` and `_italic_`, custom links and
automatic conversion of URLs to links, and the possibility to create tables and
pre-formatted text blocks (useful for examples) simply with pipe character.
If documentation gets longer, support for section titles (new in Robot
Framework 2.7.5) can also be handy.

Some of the most important formatting features are illustrated in the example
below. Notice that since this is the default format, there is no need to use
`ROBOT_LIBRARY_DOC_FORMAT` attribute nor give the format from the command
line.

.. sourcecode:: python

    """Example library in Robot Framework format.

    - Formatting with *bold* and _italic_.
    - URLs like http://example.com are turned to links.
    - Custom links like [http://robotframework.org|Robot Framework] are supported.
    - Linking to `My Keyword` works.
    """

    def my_keyword():
        """Nothing more to see here."""

HTML documentation syntax
~~~~~~~~~~~~~~~~~~~~~~~~~

When using HTML format, you can create documentation pretty much freely using
any syntax. The main drawback is that HTML markup is not that human friendly,
and that can make the documentation in the source code hard to maintain and read.
Documentation in HTML format is used by Libdoc directly without any
transformation or escaping. The special syntax for `linking to keywords`_ using
syntax like :codesc:`\`My Keyword\`` is supported, however.

Example below contains the same formatting examples as the previous example.
Now `ROBOT_LIBRARY_DOC_FORMAT` attribute must be used or format given
on the command line like `--docformat HTML`.

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

Plain text documentation syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When the plain text format is used, Libdoc uses the documentation as-is.
Newlines and other whitespace are preserved except for indentation, and
HTML special characters (`<>&`) escaped. The only formatting done is
turning URLs into clickable links and supporting `internal linking`_
like :codesc:`\`My Keyword\``.

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

reStructuredText documentation syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

reStructuredText_ is simple yet powerful markup syntax used widely in Python
projects (including this User Guide) and elsewhere. The main limitation
is that you need to have the docutils_ module installed to be able to generate
documentation using it. Because backtick characters have special meaning in
reStructuredText, `linking to keywords`_ requires them to be escaped like
:codesc:`\\\`My Keyword\\\``.

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

Internal linking
----------------

Libdoc supports internal linking to keywords and different
sections in the documentation. Linking is done by surrounding the
target name with backtick characters like :codesc:`\`target\``. Target
names are case-insensitive and possible targets are explained in the
subsequent sections.

There is no error or warning if a link target is not found, but instead Libdoc
just formats the text in italics. Earlier this formatting was recommended to
be used when referring to keyword arguments, but that was problematic because
it could accidentally create internal links. Nowadays it is recommended to
use `inline code style <inline styles_>`__ with double backticks like
:codesc:`\`\`argument\`\`` instead. The old formatting of single backticks
may even be removed in the future in favor of giving an error when a link
target is not found.

In addition to the examples in the following sections, internal linking
and argument formatting is shown also in the `longer example`__ at the
end of this chapter.

__ `Libdoc example`_

Linking to keywords
~~~~~~~~~~~~~~~~~~~

All keywords the library have automatically create link targets and they can
be linked using syntax :codesc:`\`Keyword Name\``. This is illustrated with
the example below where both keywords have links to each others.

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

.. note:: When using `reStructuredText documentation syntax`_, backticks must
          be escaped like :codesc:`\\\`Keyword Name\\\``.

Linking to automatic sections
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The documentation generated by Libdoc always contains sections
for overall library introduction, shortcuts to keywords, and for
actual keywords.  If a library itself takes arguments, there is also
separate `importing section`_.

All these sections act as targets that can be linked, and the possible
target names are listed in the table below. Using these targets is
shown in the example of the next section.

.. table:: Automatic section link targets
   :class: tabular

   ================  ===========================================================
        Section                               Target
   ================  ===========================================================
   Introduction      :codesc:`\`introduction\`` and :codesc:`\`library introduction\``
   Importing         :codesc:`\`importing\`` and :codesc:`\`library importing\``
   Shortcuts         :codesc:`\`shortcuts\`` (New in Robot Framework 2.7.5.)
   Keywords          :codesc:`\`keywords\`` (New in Robot Framework 2.7.5.)
   ================  ===========================================================

Linking to custom sections
~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting from version 2.7.5, Robot Framework's `documentation syntax`_
supports custom `section titles`_, and the titles used in the
library or resource file introduction automatically create link
targets. The example below illustrates linking both to automatic and
custom sections:

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

.. note:: Linking to custom sections works only when using `Robot Framework
          documentation syntax`_.

.. note:: Prior to Robot Framework 2.8, only the first level section
          titles were linkable.

Representing arguments
----------------------

Libdoc handles keywords' arguments automatically so that
arguments specified for methods in libraries or user keywords in
resource files are listed in a separate column. User keyword arguments
are shown without `${}` or `@{}` to make arguments look
the same regardless where keywords originated from.

Regardless how keywords are actually implemented, Libdoc shows arguments
similarly as when creating keywords in Python. This formatting is explained
more thoroughly in the table below.

.. table:: How Libdoc represents arguments
   :class: tabular

   +--------------------+----------------------------+------------------------+
   |      Arguments     |      Now represented       |        Examples        |
   +====================+============================+========================+
   | No arguments       | Empty column.              |                        |
   +--------------------+----------------------------+------------------------+
   | One or more        | List of strings containing | | `one_argument`       |
   | argument           | argument names.            | | `a1, a2, a3`         |
   +--------------------+----------------------------+------------------------+
   | Default values     | Default values separated   | | `arg=default value`  |
   | for arguments      | from names with `=`.       | | `a, b=1, c=2`        |
   +--------------------+----------------------------+------------------------+
   | Variable number    | Last (or second last with  | | `*varargs`           |
   | of arguments       | kwargs) argument has `*`   | | `a, b=42, *rest`     |
   | (varargs)          | before its name.           |                        |
   +--------------------+----------------------------+------------------------+
   | Free keyword       | Last arguments has         | | `**kwargs`           |
   | arguments (kwargs) | `**` before its name.      | | `a, b=42, **kws`     |
   |                    |                            | | `*varargs, **kwargs` |
   +--------------------+----------------------------+------------------------+

When referring to arguments in keyword documentation, it is recommended to
use `inline code style <inline styles_>`__ like :codesc:`\`\`argument\`\``.

Libdoc example
--------------

The following example illustrates how to use the most important
`documentation formatting`_ possibilities, `internal linking`_, and so
on. `Click here`__ to see how the generated documentation looks like.

.. sourcecode:: python

   src/SupportingTools/LoggingLibrary.py

All `standard libraries`_ have documentation generated by
Libdoc and their documentation (and source code) act as a more
realistic examples.

__ src/SupportingTools/LoggingLibrary.html
