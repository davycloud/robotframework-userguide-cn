.. role:: name(emphasis)
.. role:: setting(emphasis)

.. _creating test libraries:

创建测试库
==========

Robot Framework 实际的测试能力都是由测试库提供. 目前已经有很多库存在, 其中甚至有些已经与框架绑定, 但是很多时候仍然需要创建新的测试库. 
得益于Robot Framework简单直接的API, 这一过程并不复杂.

.. contents::
   :depth: 2
   :local:



概述
----

支持的编程语言
^^^^^^^^^^^^^^

Robot Framework自身使用 `Python <http://python.org>`_ 编写, 很自然的, 扩展它的测试库也可以使用相同的语言. 
当使用 `Jython <http://jython.org>`_ 运行时, 也可以使用 Java_ 来实现测试库. 
纯的Python代码, 只要其没有使用Jython不支持的模块或语法, 在两种情况下都可以运行. 
当使用Python时, 可以利用 `Python C API <http://docs.python.org/c-api/index.html>`_ 使用C语言来实现测试库, 当然, 使用Python的 `ctypes <http://docs.python.org/library/ctypes.html>`_ 模块调用C代码则更简单.

使用这些原生支持的语言实现的测试库可以同时作为"包装器", 调用其他语言实现的
功能. 一个典型的例子就是 :ref:`remote library` 的实现. 当然还可以另起单独的进程来执行
外部脚本或者工具.

.. tip:: `Python Tutorial for Robot Framework Test Library Developers <http://code.google.com/p/robotframework/wiki/PythonTutorial>`_
         covers enough of Python language to get started writing test
         libraries using it. It also contains a simple example library
         and test cases that you can execute and otherwise investigate
         on your machine.

.. hint:: 译注: 上面提到的Python Tutorial链接已经失效, 从 `这里 <https://groups.google.com/d/msg/robotframework-users/Bb2oytBJQb4/7dXYaeHJGgAJ>`_ 看应该也不会再更新, 原内容已经迁移到 `GitHub上 <https://github.com/robotframework/robotframework/tree/master/doc/python>`_


不同的测试库API
^^^^^^^^^^^^^^^

Robot Framework 有三种不同的测试库API.

静态(Static) API

  最简单的办法是实现一个模块(用Python), 或者类(用Python或Java), 其中的
  方法(methods)直接映射为 :ref:`关键字名称 <keyword names>`. 关键字接受和方法相同的 :ref:`参数 <keyword arguments>`, 通过抛异常来 :ref:`报告错误 <reporting keyword status>`, 通过往标准输出里写入来写 :ref:`log <logging information>`, 同时可以
  通过 ``return`` 来 :ref:`返回结果 <returning values>`.

动态(Dynamic) API
  
  动态库类要提供一个用于获取实现的关键字名称的方法, 并提供另一个方法来执行具体给定的关键字包括参数. 这样, 要实现的关键字的名称, 可以在运行时动态决定. 其它功能, 如报告状态, 打印日志和返回结果都和静态API类似.

混合(Hybrid) API
  
  静态API和动态API的混合. 类有一个方法用来说明实现了哪些关键字, 这些关键字必须是
  直接可用的. 除了不是自动发现关键字, 其它功能都和静态API类似.

所有这些API都将在本章进行详述. 所有功能都是基于静态API的工作原理, 所以首先来讨论静态API的内容. :ref:`dynamic library API` 和 :ref:`hybrid library API` 与之的区别在随后的小节中分别讨论.

本章示例大部分都是Python实现, 但是对于Java开发者来说, 理解这些代码并不难.
在某些少数情况下, Python API和Java API有所差异, 此时会分别解释.


创建测试库类或模块
------------------

测试库的实现可以是Python模块, 也可以是Python或Java的类.

测试库名称
^^^^^^^^^^

测试库引入(import)时需要指定名称, 这个名称和实现测试库的模块名或类名一样.

举个例子, 如果有个Python模块 ``MyLibrary`` (文件是 :file:`MyLibrary.py`), 这就可作为一个测试库: :name:`MyLibrary`. 类似的, 一个不在任何包里面的Java类 ``YourLibrary``, 就是一个同名的测试库.

Python的类总是在模块里, 如果模块里实现的类的名称和模块名一致, Robot Framework
允许不带类名来引用这个库. 例如, 有一个类 ``MyLib`` 在文件 :file:`MyLib.py` 中,
引用库时只需使用名称 :name:`MyLib` 即可. 这个机制对于子模块同样有效, 如, 
``parent.MyLib`` 模块中有个类 ``MyLib``, 使用  :name:`parent.MyLib` 即可导入.
但是, 如果模块名和类名不一样, 则必须同时指明, 如 :name:`mymodule.MyLibrary` 
或者 :name:`parent.submodule.MyLib`.

非默认包里的Java类也必须指定全名, 例如, 包 ``com.mycompany.myproject`` 里的
类 ``MyLib`` 导入名称是: :name:`com.mycompany.myproject.MyLib`.

.. note:: 在同名模块中忽略类名只在 Robot Framework 2.8.4 及之后的版本有效.
          老的版本中仍然需要指定全名, 如 :name:`parent.MyLib.MyLib`.

.. tip:: 如果库名确实太长了, 比如Java的包名太长, 推荐使用 :ref:`WITH NAME syntax` 
         给库起个别名.

.. _providing arguments to test libraries:


给测试库提供参数
^^^^^^^^^^^^^^^^

实现为类的测试库都可以接受参数. 这些参数在Setting表中指定, 跟在库名后面,
当Robot Framework创建测试库的实例时, 把这些参数传给构造函数. 

实现为模块的测试库不可以接受参数, 试图给其传递参数的行为将导致报错.

库所需的参数的个数和库的构造函数的参数个数一样. 默认参数和不定数量参数的处理
类似于 :ref:`keyword arguments`. Java库不支持不定数量的参数. 

传递给库的参数, 包括库名本身, 都可以使用变量. 也就是说可以在某些时候, 例如命令行, 修改它们.

.. sourcecode:: robotframework

   *** Settings ***
   Library    MyLibrary     10.0.0.1    8080
   Library    AnotherLib    ${VAR}

下面是上面的例子中库的实现, 第一个是Python, 第二个是Java:

.. sourcecode:: python

  from example import Connection

  class MyLibrary:

      def __init__(self, host, port=80):
          self._conn = Connection(host, int(port))

      def send_message(self, message):
          self._conn.send(message)

.. sourcecode:: java

   public class AnotherLib {

       private String setting = null;

       public AnotherLib(String setting) {
           setting = setting;
       }

       public void doSomething() {
           if setting.equals("42") {
               // do something ...
           }
       }
   }

.. _test library scope:

测试库的作用域
^^^^^^^^^^^^^^

用类实现的库可以有内部状态, 这些状态可以被关键字或构造函数修改. 因为这些状态
会影响到关键字实际的行为, 所以, 保证一个测试用例不会意外地影响到另一个用例显
得非常重要. 这种依赖行为有可能造成非常难定位的bug. 例如, 添加了新的测试用例,
而这些用例使用库的方式并不一致.

Robot Framework 为了保证测试用例之间的独立性, 默认情况下, 它为每个测试用例
创建新的测试库实例. 然而, 这种方式不总是我们想要的, 比如有时测试用例需要共享
某个状态的时候. 此外, 那些无状态的库显然也不需要每次都创建新实例.

实例化测试库类的方式可以通过一个特别的属性  ``ROBOT_LIBRARY_SCOPE`` 来控制.
这个属性是一个字符串, 可以有以下三种取值:

``TEST CASE``
  为每个测试用例创建新的实例. 如果有suite setup和suite teardown的话, 同样
  也会新建. 这是默认行为.

``TEST SUITE``
  为每个测试集创建新的实例. 最底层的测试集, 也就是由测试用例文件组成的测试集,
  拥有属于自己的测试库实例, 高层的测试集, 都有属于自己的测试库实例.

``GLOBAL``
  整个测试执行过程中只有一个实例被创建. 所有的测试集和测试用例共享这个实例.
  通过模块创建的测试库都是全局的.


.. note:: 如果一个测试库被导入多次, 每次使用不同的
          :ref:`参数 <providing arguments to test libraries>`, 
          则不管有没有定义作用域, 每次都会新建一个实例.

当有状态的测试库定义了作用域为 ``TEST SUITE`` 或 ``GLOBAL`` , 建议测试库要
包含能清除这些状态的关键字. 这样, 在测试集 setup 或 teardown 时, 可以
调用这些关键字以保证下面的测试用例从一个明确的已知状态开始.

例如, :name:`SeleniumLibrary` 使用了 ``GLOBAL`` 作用域, 使得不同的测试
用例使用相同的浏览器, 而不是每次重新打开. 同时, 它还提供了关键字
:name:`Close All Browsers` 关闭所有浏览器.

下面是使用了 ``TEST SUITE`` 作用域的Python库的示例:

.. sourcecode:: python

    class ExampleLibrary:

        ROBOT_LIBRARY_SCOPE = 'TEST SUITE'

        def __init__(self):
            self._counter = 0

        def count(self):
            self._counter += 1
            print self._counter

        def clear_counter(self):
            self._counter = 0

使用了 ``GLOBAL`` 作用域的Java库的示例:

.. sourcecode:: java

    public class ExampleLibrary {

        public static final String ROBOT_LIBRARY_SCOPE = "GLOBAL";

        private int counter = 0;

        public void count() {
            counter += 1;
            System.out.println(counter);
        }

        public void clearCounter() {
            counter = 0;
        }
    }

.. _specifying library version:

指定测试库的版本
^^^^^^^^^^^^^^^^

当一个测试库投入使用, Robot Framework 会尝试获取它的版本号, 将该信息
写入到 日志_ 中以供调试. 库文档工具 Libdoc_ 在生成文档时也会写入该信息.

版本号信息在属性 `ROBOT_LIBRARY_VERSION` 中定义, 类似 :ref:`test library scope` 
中的 ``ROBOT_LIBRARY_SCOPE``. 如果 ``ROBOT_LIBRARY_VERSION`` 属性不存在,
则会尝试从 ``__version__`` 属性获取. 这些属性必须是类或者模块的属性.
对于Java库, 这个属性必须声明为 ``static final``.

使用 ``__version__`` 的Python模块示例:

.. sourcecode:: python

    __version__ = '0.1'

    def keyword():
        pass

使用 ``ROBOT_LIBRARY_VERSION`` 的Java类示例:

.. sourcecode:: java

    public class VersionExample {

        public static final String ROBOT_LIBRARY_VERSION = "1.0.2";

        public void keyword() {
        }
    }

.. _specifying documentation format:

指定文档格式
^^^^^^^^^^^^

从 Robot Framework 2.7.5版本开始, 库文档工具 Libdoc_ 开始支持多种格式.
如果不想使用 Robot Framework's 自己的 :ref:`documentation formatting`, 可以通过在源码中定义
属性 ``ROBOT_LIBRARY_DOC_FORMAT`` 来指定格式, 就跟指定 :ref:`作用域 <test library scope>` 和 :ref:`版本 <specifying library version>` 一样.

文档格式可指定的值包括: ``ROBOT`` (默认), ``HTML``, ``TEXT`` (纯文本),
和 ``reST`` (reStructuredText_). 这些值不区分大小写. 如果要使用 ``reST``
格式需要安装 docutils_ 模块.

下面使用Python和Java设置文档格式的例子分别使用了 reStructuredText 和 HTML
格式. 关于给测试库写文档的更多内容, 请参考 :ref:`Documenting libraries` 和 
Libdoc_ 章节.

.. sourcecode:: python

    """A library for *documentation format* demonstration purposes.

    This documentation is created using reStructuredText__. Here is a link
    to the only \`Keyword\`.

    __ http://docutils.sourceforge.net
    """

    ROBOT_LIBRARY_DOC_FORMAT = 'reST'

    def keyword():
        """**Nothing** to see here. Not even in the table below.

        =======  =====  =====
        Table    here   has
        nothing  to     see.
        =======  =====  =====
        """
        pass

.. sourcecode:: java

    /**
     * A library for <i>documentation format</i> demonstration purposes.
     *
     * This documentation is created using <a href="http://www.w3.org/html">HTML</a>.
     * Here is a link to the only `Keyword`.
     */
    public class DocFormatExample {

        public static final String ROBOT_LIBRARY_DOC_FORMAT = "HTML";

        /**<b>Nothing</b> to see here. Not even in the table below.
         *
         * <table>
         * <tr><td>Table</td><td>here</td><td>has</td></tr>
         * <tr><td>nothing</td><td>to</td><td>see.</td></tr>
         * </table>
         */
        public void keyword() {
        }
    }


.. _Library acting as listener:

作为监听器的测试库
^^^^^^^^^^^^^^^^^^

:ref:`listener interface` 可以让外部的监听器在测试执行过程中得到关于执行状态的通知.
例如, 当测试集, 测试用例和关键字开始执行或结束. 有时候这些通知对测试库
来说很有用, 可以使用 ``ROBOT_LIBRARY_LISTENER`` 注册一个自定义的监听器.
该属性的值应该是要使用的监听器的示例, 有可能是测试库本身. 更多内容和示例,
请参考 :ref:`test libraries as listeners` section.

创建静态关键字
--------------

.. _what methods are considered keywords:

充当关键字的方法
^^^^^^^^^^^^^^^^

当使用静态库API时, Robot Framework 利用反射机制获得类或模块实现的公有方法(public methods). 所有以下划线(_)开始的方法被排除, 在Java库里, 只在 ``java.lang.Object`` 里实现的方法也被忽略. 所有没被忽略的方法都被视为关键字. 

例如, 下面的Python和Java示例实现了关键字  :name:`My Keyword`.

.. sourcecode:: python

    class MyLibrary:

        def my_keyword(self, arg):
            return self._helper_method(arg)

        def _helper_method(self, arg):
            return arg.upper()

.. sourcecode:: java

    public class MyLibrary {

        public String myKeyword(String arg) {
            return helperMethod(arg);
        }

        private String helperMethod(String arg) {
            return arg.toUpperCase();
        }
    }

当库是Python模块实现的, 可以使用Python的 ``__all__`` 属性来限制到底
哪些方法是关键字. 如果使用了 ``__all__``, 只有列在其中的才会被当作
关键字, 例如下面的例子中, 实现了关键字 :name:`Example Keyword` 和 
:name:`Second Example`. 如果这个例子中没有 `__all__`, 那么其中的
:name:`Not Exposed As Keyword` 和 :name:`Current Thread` 也会
被视作关键字. `__all__` 最重要的作用就是确保那些import进来的帮助方法, 
如本例中的 `current_thread`, 不会被意外地暴露为关键字

.. sourcecode:: python

   from threading import current_thread

   __all__ = ['example_keyword', 'second_example']

   def example_keyword():
       if current_thread().name == 'MainThread':
           print 'Running in main thread'

   def second_example():
       pass

   def not_exposed_as_keyword():
       pass


关键字名称
^^^^^^^^^^

测试数据(test data)中使用的关键字名称, 与方法名称对比, 最终确定是哪个
方法实现了这个关键字. 名称的比较是忽略大小写的, 并且其中的空格和下划线
也都忽略掉. 例如, 方法名 ``hello`` 最终可以映射为的关键字名称可以是:
:name:`Hello`, :name:`hello` 甚至 :name:`h e l l o`. 反过来也类似,
例如, ``do_nothing`` 和 ``doNothing`` 这两个方法都可被当作 :name:`Do Nothing`
关键字的实现.

Python模块实现的测试库示例如下, :file:`MyLibrary.py`:

.. sourcecode:: python

  def hello(name):
      print "Hello, %s!" % name

  def do_nothing():
      pass

Java类实现的测试库示例如下, :file:`MyLibrary.java` file:

.. sourcecode:: java

  public class MyLibrary {

      public void hello(String name) {
          System.out.println("Hello, " + name + "!");
      }

      public void doNothing() {
      }

  }

下面的例子用来说明如何使用上面的测试库. 如果你想自己试一下, 首先要确保库的
位置在 :ref:`module search path`.

.. sourcecode:: robotframework

   *** Settings ***
   Library    MyLibrary

   *** Test Cases ***
   My Test
       Do Nothing
       Hello    world

.. _using a custom keyword name:

使用自定义的关键字名称
''''''''''''''''''''''

如果一个方法不想使用默认映射的关键字名称, 也可以明确指定为自定义的关键字名称.
这是通过设置方法的  ``robot_name`` 属性来实现的. 可以使用装饰器方法
 ``robot.api.deco.keyword`` 方便快捷的设置. 例如:

.. sourcecode:: python

  from robot.api.deco import keyword

  @keyword('Login Via User Panel')
  def login(username, password):
      # ...

.. sourcecode:: robotframework

   *** Test Cases ***
   My Test
       Login Via User Panel    ${username}    ${password}

如果使用装饰器时不带任何参数, 则这个装饰器不会改变关键字名称, 但是仍然
会创建  ``robot_name`` 属性. 这种情况对 :ref:`标记方法为关键字 <marking methods to expose as keywords>` , 同时又不改变关键字名称的时候很有用. 

设置自定义的关键字名称还使得库关键字可以接受使用 :ref:`嵌入参数 <embedding arguments into keyword names>` 语法的参数.


.. _keyword tags:

关键字标签
^^^^^^^^^^

从 Robot Framework 2.9 版本开始, 库关键字和  :ref:`用户关键字 <user keyword tags>` 都可以有标签.

库关键字通过设置方法的 ``robot_tags`` 属性实现, 该属性的值是要设置的标签的列表.
装饰器 ``robot.api.deco.keyword`` 可以按如下的方法来方便的指定这个属性:

.. sourcecode:: python

  from robot.api.deco import keyword

  @keyword(tags=['tag1', 'tag2'])
  def login(username, password):
      # ...

  @keyword('Custom name', ['tags', 'here'])
  def another_example():
      # ...

设置标签的另一个方法是在 :ref:`关键字文档 <documenting libraries>` 的最后一行给出, 以 ``Tags:`` 作为前缀开始, 后面跟着按逗号分开的标签. 例如: 

.. sourcecode:: python

  def login(username, password):
      """Log user in to SUT.

      Tags: tag1, tag2
      """
      # ...

.. _keyword arguments:

关键字参数
^^^^^^^^^^

对于静态和混合API, 关于一个关键字的参数表信息是直接从实现它的方法上获取的.
而使用了 :ref:`dynamic library API` 的库则使用其它的方式来传递这些信息, 所以本章不适用于动态库.

最常见也是最简单的情况是关键字需要确定数目的参数. 在这种情况下, Python和Java
方法简单地接受这些参数即可. 例如, 没有参数的方法对应的关键字也不需要参数, 只需
一个参数的方法对应的关键字也只需要一个参数, 以此类推.

下面Python关键字示例接受不同数量的参数:

.. sourcecode:: python

  def no_arguments():
      print "Keyword got no arguments."

  def one_argument(arg):
      print "Keyword got one argument '%s'." % arg

  def three_arguments(a1, a2, a3):
      print "Keyword got three arguments '%s', '%s' and '%s'." % (a1, a2, a3)

.. note:: 使用静态库API实现的Java库有一个很大的限制, 即不支持 :ref:`named argument syntax`. 如果
          你觉得这是一个障碍, 那要么改使用Python来实现, 要么切换到 :ref:`dynamic library API`


.. Default values to keywords

关键字的缺省值
^^^^^^^^^^^^^^

和函数类似, 关键字的有些参数有时需要有缺省值. Python 和 Java 对于处理方法的缺省值
使用不同的语法, 


.. Default values with Python

Python中的缺省值
''''''''''''''''

Python中, 方法总是只有一个实现, 在方法的签名中可能指定若干缺省值.
这种语法对Python程序员来说应该非常熟悉, 例如:

.. sourcecode:: python

   def one_default(arg='default'):
       print "Argument has value %s" % arg

   def multiple_defaults(arg1, arg2='default 1', arg3='default 2'):
       print "Got arguments %s, %s and %s" % (arg1, arg2, arg3)

上例中的第一个关键字, 可以接受 0 个或 1 个参数, 当 0 个参数时, 参数 ``arg``
使用缺省值 ``default``; 当有 1 个参数时, 参数 ``arg`` 就使用这个传入的值; 
而如果参数个数大于 1 , 则调用该关键字会报错失败.

第二个关键字中, 第1个参数总是需要指定的, 但是 第2和第3个都有缺省值, 所以,
使用该关键字可以传入 1 至 3 个参数.

.. sourcecode:: robotframework

   *** Test Cases ***
   Defaults
       One Default
       One Default    argument
       Multiple Defaults    required arg
       Multiple Defaults    required arg    optional
       Multiple Defaults    required arg    optional 1    optional 2


.. Default values with Java

Java中的缺省值
''''''''''''''

Java中, 一个方法可以有多个实现, 分别是不同的签名(重载). Robot Framework 将所有
这些实现都视作一个关键字, 这个关键字可以接受不同的参数, 以此实现了缺省值的
支持. 下面的例子在功能上和上面的Python例子完全一样:

.. sourcecode:: java

   public void oneDefault(String arg) {
       System.out.println("Argument has value " + arg);
   }

   public void oneDefault() {
       oneDefault("default");
   }

   public void multipleDefaults(String arg1, String arg2, String arg3) {
       System.out.println("Got arguments " + arg1 + ", " + arg2 + " and " + arg3);
   }

   public void multipleDefaults(String arg1, String arg2) {
       multipleDefaults(arg1, arg2, "default 2");
   }

   public void multipleDefaults(String arg1) {
       multipleDefaults(arg1, "default 1");
   }

.. Variable number of arguments (`*varargs`)

可变数量的参数
^^^^^^^^^^^^^^

Robot Framework 的关键字还支持接受任何数量的参数. 类似于缺省值,
实际的语法在Python和Java中有所差异.


Python中的可变数量的参数
''''''''''''''''''''''''

Python的语法本身就支持让方法可以接受任意数量的参数. 相同的语法同样作用于
测试库, 同时, 还可以与指定缺省值结合, 如下面的例子:

.. sourcecode:: python

  def any_arguments(*args):
      print "Got arguments:"
      for arg in args:
          print arg

  def one_required(required, *others):
      print "Required: %s\nOthers:" % required
      for arg in others:
          print arg

  def also_defaults(req, def1="default 1", def2="default 2", *rest):
      print req, def1, def2, rest

.. sourcecode:: robotframework

   *** Test Cases ***
   Varargs
       Any Arguments
       Any Arguments    argument
       Any Arguments    arg 1    arg 2    arg 3    arg 4    arg 5
       One Required     required arg
       One Required     required arg    another arg    yet another
       Also Defaults    required
       Also Defaults    required    these two    have defaults
       Also Defaults    1    2    3    4    5    6

.. _Variable number of arguments with Java:

Java中的可变数量的参数
''''''''''''''''''''''
Robot Framework 支持 :ref:`Java可变数量参数的语法 <http://docs.oracle.com/javase/1.5.0/docs/guide/language/varargs.html>`. 下面的例子和上面Python的例子
在功能上是一样的:

.. sourcecode:: java

  public void anyArguments(String... varargs) {
      System.out.println("Got arguments:");
      for (String arg: varargs) {
          System.out.println(arg);
      }
  }

  public void oneRequired(String required, String... others) {
      System.out.println("Required: " + required + "\nOthers:");
      for (String arg: others) {
          System.out.println(arg);
      }
  }

Robot Framework 从 2.8.3 版本开始, 还支持另一种方式来实现可变数量参数, 即
使用数组或者 ``java.util.List`` 作为最后一个参数, 或倒数第二个参数(如果最后一个参数是
 :ref:`任意关键字参数 <free keyword arguments>` **kwargs). 例如, 下面的示例和上面的功能是相同的:

.. sourcecode:: java

  public void anyArguments(String[] varargs) {
      System.out.println("Got arguments:");
      for (String arg: varargs) {
          System.out.println(arg);
      }
  }

  public void oneRequired(String required, List<String> others) {
      System.out.println("Required: " + required + "\nOthers:");
      for (String arg: others) {
          System.out.println(arg);
      }
  }

.. note:: 只有 `java.util.List` 支持作为 varargs, 它的任何子类型都不可以.

对于Java关键字来说, 支持可变数量的参数有一个限制: 只在方法只有一个签名时有效.
也就是说, Java实现的关键字不可能既使用缺省值又使用varargs. 而且, 只有 2.8 或
更新版本的 Robot Framework 支持在 :ref:`库的构造器 <providing arguments to test libraries>` 中使用varargs.


.. _free keyword arguments:
.. _**kwargs:

任意关键字参数
^^^^^^^^^^^^^^

Robot Framework 2.8版本增加了任意关键字参数, 即Python中的 ``**kwargs`` 语法.
如何在测试用例中使用这种语法的讨论在 :ref:`creating test cases` 章节下的 :ref:`free keyword arguments` 小节中. 

本章我们来看一下如何在测试库中使用它.

Robot Framework 2.8 added the support for free keyword arguments using Python's
`**kwargs` syntax. How to use the syntax in the test data is discussed
in `Free keyword arguments`_ section under `Creating test cases`_. In this
section we take a look at how to actually use it in custom test libraries.

Python中的任意关键字参数
''''''''''''''''''''''''

如果你对Python中的 kwargs 如何工作比较熟悉, 那么理解Robot Framework中的测试库是如何实现的就非常简单了. 下面的例子展示了最基础的功能:

.. sourcecode:: python

    def example_keyword(**stuff):
        for name, value in stuff.items():
            print name, value

.. sourcecode:: robotframework

   *** Test Cases ***
   Keyword Arguments
       Example Keyword    hello=world        # Logs 'hello world'.
       Example Keyword    foo=1    bar=42    # Logs 'foo 1' and 'bar 42'.

基本上, 所有以 :ref:`named argument syntax` ``name=value`` 形式跟在关键字调用最后面, 且不匹配其它任何参数的参数, 将以 kwargs 传入给关键字. 
如果想要避免一个字面字符串被当作任意关键字参数, 则其中的等号 ``=`` 必须被 :ref:`转义 <escaping>`, 例如 ``foo=quux`` 要写作 ``foo\=quux``.

下面的例子展示了综合使用普通参数, 可变数量参数(varargs), 和任意关键字参数(kwargs)的情况:

.. sourcecode:: python

  def various_args(arg, *varargs, **kwargs):
      print 'arg:', arg
      for value in varargs:
          print 'vararg:', value
      for name, value in sorted(kwargs.items()):
          print 'kwarg:', name, value

.. sourcecode:: robotframework

   *** Test Cases ***
   Positional
       Various Args    hello    world                # Logs 'arg: hello' and 'vararg: world'.

   Named
       Various Args    arg=value                     # Logs 'arg: value'.

   Kwargs
       Various Args    a=1    b=2    c=3             # Logs 'kwarg: a 1', 'kwarg: b 2' and 'kwarg: c 3'.
       Various Args    c=3    a=1    b=2             # Same as above. Order does not matter.

   Positional and kwargs
       Various Args    1    2    kw=3                # Logs 'arg: 1', 'vararg: 2' and 'kwarg: kw 3'.

   Named and kwargs
       Various Args    arg=value      hello=world    # Logs 'arg: value' and 'kwarg: hello world'.
       Various Args    hello=world    arg=value      # Same as above. Order does not matter.

要查看真实测试库中相同示例, 请参考 Process_ 库中的关键字 :name:`Run Process` 和 :name:`Start Keyword`.

For a real world example of using a signature exactly like in the above
example, see :name:`Run Process` and :name:`Start Keyword` keywords in the
Process_ library.


Java中的任意关键字参数
''''''''''''''''''''''

从Robot Framework 2.8.3版本开始, Java测试库也开始支持这种语法. Java语言本身是不支持kwargs语法的, 但是关键字可以利用 ``java.util.Map`` 类型作为最后一个参数, 来接受 kwargs.

如果一个Java关键字接受 kwargs, Robot Framework 会自动将关键字调用的末尾所有形如  ``name=value`` 的参数打包放入一个 ``Map`` , 然后传递给关键字方法. 例如, 下面的例子中的关键字使用起来和前面的Python示例完全一样:

.. sourcecode:: java

    public void exampleKeyword(Map<String, String> stuff):
        for (String key: stuff.keySet())
            System.out.println(key + " " + stuff.get(key));

    public void variousArgs(String arg, List<String> varargs, Map<String, Object> kwargs):
        System.out.println("arg: " + arg);
        for (String varg: varargs)
            System.out.println("vararg: " + varg);
        for (String key: kwargs.keySet())
            System.out.println("kwarg: " + key + " " + kwargs.get(key));

.. note:: kwargs 参数的类型必须是 `java.util.Map`, 而不是其子类.

.. note:: 和 :ref:`Java中的varargs <Variable number of arguments with Java>` 一样, kwargs的关键字也只能有一个方法签名. 


.. _argument types:

参数类型
^^^^^^^^

正常情况下, 关键字的参数以字符串的形式传递给 Robot Framework. 如果关键字需要其它的类型, 可以使用 :ref:`variables` 或者在关键字的内部将字符串转换为所需的类型. 
使用 :ref:`Java关键字 <Argument types with Java>`, 基础类型会自动的强制转换.


Python中的参数类型
''''''''''''''''''

因为Python的参数并没有任何的类型信息, 所以使用Python的库时不可能自动的将字符串转换为其它类型. 调用Python方法实现的关键字, 只要参数的数量正确, 调用就总是能够成功, 只不过如果参数不兼容, 后面的执行会失败. 幸运地是, 在Python中转换参数类型是很简单的事情:

.. sourcecode:: python

  def connect_to_host(address, port=25):
      port = int(port)
      # ...

.. _Argument types with Java:

Java中的参数类型
''''''''''''''''

Java方法的参数都有类型, 而且所有基础类型会自动处理. 这意味着, test data 中的字符串类型的参数, 在运行时刻强制转换为正确的类型. 可以转换的类型有:

- 整数型 (``byte``, ``short``, ``int``, ``long``)
- 浮点数 (``float`` 和 ``double``)
- 布尔型 (``boolean``)
- 上述类型的对象版本, 如. ``java.lang.Integer``

Java的关键字方法可能会有多个签名, 强制转换只有在有相同的或兼容的签名才会发生. 下面的例子中, 关键字  ``doubleArgument`` 和 ``compatibleTypes`` 可以强制转换, 但是 ``conflictingTypes`` 会发生冲突.

.. sourcecode:: java

   public void doubleArgument(double arg) {}

   public void compatibleTypes(String arg1, Integer arg2) {}
   public void compatibleTypes(String arg2, Integer arg2, Boolean arg3) {}

   public void conflictingTypes(String arg1, int arg2) {}
   public void conflictingTypes(int arg1, String arg2) {}

对于数值型的类型, 如果测试数据中的字符串包含数字, 就可以强制转换, 对于布尔型, 则必须包含字符串 ``true`` 或者 ``false``. 

强制转换只在测试数据的原始值是字符串的情况下才会发生, 当然还可以使用包含了正确数据类型的变量. 要应对冲突的方法签名, 使用变量是唯一的选择.

.. sourcecode:: robotframework

   *** Test Cases ***
   Coercion
       Double Argument     3.14
       Double Argument     2e16
       Compatible Types    Hello, world!    1234
       Compatible Types    Hi again!    -10    true

   No Coercion
       Double Argument    ${3.14}
       Conflicting Types    1       ${2}    # must use variables
       Conflicting Types    ${1}    2

从 Robot Framework 2.8 版本开始, 参数类型的强制转换在 :ref:`Java库的构造函数 <Providing arguments to test libraries>` 中也起作用.



使用装饰器
^^^^^^^^^^

当编写静态关键字时, 有时候使用Python的装饰器修改它们会很方便. 但是, 装饰器修改了函数的签名, 这会让 Robot Framework 在判断关键字能接受什么参数时产生混乱. 特别是用 Libdoc_ 创建库文档和使用 RIDE_ 时. 为了避免这种情况, 要么就不要用装饰器, 要么使用方便的 :ref:`装饰器模块 <http://micheles.googlecode.com/hg/decorator/documentation.html>` 创建保留签名的装饰器. 

.. hint:: 译注: 上面的链接貌似已经失效.


关键字中嵌入参数
^^^^^^^^^^^^^^^^

库关键字还能接受使用 :ref:`嵌入参数语法 <embedding arguments into keyword name>` 传递的参数. 可以使用装饰器 ``robot.api.deco.keyword`` 来创建 :ref:`自定义关键字名称 <using a custom keyword name>`, 其中包括所需语法.

.. sourcecode:: python

    from robot.api.deco import keyword

    @keyword('Add ${quantity:\d+} Copies Of ${item} To Cart')
    def add_copies_to_cart(quantity, item):
        # ...

.. sourcecode:: robotframework

   *** Test Cases ***
   My Test
       Add 7 Copies Of Coffee To Cart

.. Communicating with Robot Framework

与Robot Framework通讯
----------------------

当关键字方法被调用后, 它可以使用任何机制去和被测系统通讯. 同时, 它还可以发送消息给 Robot Framework的日志文件, 返回结果以保存到变量中, 最重要的, 报告该关键字是否通过了(passed).


报告关键字状态
^^^^^^^^^^^^^^

使用异常(exceptions)即可报告关键字状态. 如果一个方法的执行抛出了一个异常, 这个关键字的状态就是 ``FAIL``, 如果正常返回, 则状态是 ``PASS``.

错误消息会写入日志和报告文件. 控制台也会显示异常类型和异常消息. 一般的异常(如 ``AssertionError``, ``Exception``, 和 ``RuntimeError``), 只显示异常消息; 其它的异常, 消息的格式是 ``异常类型: 异常消息``.

The error message shown in logs, reports and the console is created
from the exception type and its message. With generic exceptions (for
example, `AssertionError`, `Exception`, and
`RuntimeError`), only the exception message is used, and with
others, the message is created in the format `ExceptionType:
Actual message`.

从 Robot Framework 2.8.2 版本开始, 也可以让自己的异常类型和一般异常一样, 失败消息中没有异常类型作为前缀. 要实现这个效果, 为自定义异常类添加一个特殊属性 ``ROBOT_SUPPRESS_NAME``, 并将值置为 ``True``.

Python:

.. sourcecode:: python

    class MyError(RuntimeError):
        ROBOT_SUPPRESS_NAME = True

Java:

.. sourcecode:: java

    public class MyError extends RuntimeException {
        public static final boolean ROBOT_SUPPRESS_NAME = true;
    }

无论什么情况下, 异常消息的内容都应该尽量明确, 提供足够的信息给用户.


.. HTML in error messages

错误消息中使用HTML
''''''''''''''''''

从 Robot Framework 2.8 版本开始, 在错误消息中以 `*HTML*` 开头, 就可以直接使用HTML格式的消息内容. 例如:

Starting from Robot Framework 2.8, it is also possible have HTML formatted
error messages by starting the message with text `*HTML*`:

.. sourcecode:: python

   raise AssertionError("*HTML* <a href='robotframework.org'>Robot Framework</a> rulez!!")

这种方式不但可以像上面例子一样, 在测试库中抛出一个异常时使用, 还可以在测试数据中, 由用户提供错误信息.

This method can be used both when raising an exception in a library, like
in the example above, and `when users provide an error message in the test data`__.

__ `Failures`_

自动截断长消息
Cutting long messages automatically
'''''''''''''''''''''''''''''''''''

如果一个错误消息超过了40行, 就会被自动截断以防止报告变得太长而难以阅读. 完整的错误信息总会在失败关键字的相关日志中显示.

If the error message is longer than 40 lines, it will be automatically
cut from the middle to prevent reports from getting too long and
difficult to read. The full error message is always shown in the log
message of the failed keyword.

错误回溯
Tracebacks
''''''''''

异常的回溯(traceback)信息在 `日志级别`_ 为 `DEBUG` 时也会被写入日志. 这些信息默认在日志文件中不可见, 普通用户对这些消息一般也不感兴趣. 在开发测试库时, 则一般会使用 `--loglevel DEBUG` 选项来运行测试以方便定位问题.

The traceback of the exception is also logged using `DEBUG` `log level`_.
These messages are not visible in log files by default because they are very
rarely interesting for normal users. When developing libraries, it is often a
good idea to run tests using `--loglevel DEBUG`.

停止测试执行
Stopping test execution
^^^^^^^^^^^^^^^^^^^^^^^

有时候一个异常的出现意味着 `整个测试的结束`__. 要实现这种效果, 为异常类设置一个特殊的  `ROBOT_EXIT_ON_FAILURE` 属性 , 并将其值设为 `True`. 例如:

It is possible to fail a test case so that `the whole test execution is
stopped`__. This is done simply by having a special `ROBOT_EXIT_ON_FAILURE`
attribute with `True` value set on the exception raised from the keyword.
This is illustrated in the examples below.

Python:

.. sourcecode:: python

    class MyFatalError(RuntimeError):
        ROBOT_EXIT_ON_FAILURE = True

Java:

.. sourcecode:: java

    public class MyFatalError extends RuntimeException {
        public static final boolean ROBOT_EXIT_ON_FAILURE = true;
    }

__ `优雅地停止整个测试`_

失败后继续测试执行
Continuing test execution despite of failures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

有时候, 即使出现了错误仍然需要测试继续往下执行. 为异常类设置特殊属性 `ROBOT_CONTINUE_ON_FAILURE`, 并将值设为 `True`. 例如:

It is possible to `continue test execution even when there are failures`__.
The way to signal this from test libraries is adding a special
`ROBOT_CONTINUE_ON_FAILURE` attribute with `True` value to the exception
used to communicate the failure. This is demonstrated by the examples below.

Python:

.. sourcecode:: python

    class MyContinuableError(RuntimeError):
        ROBOT_CONTINUE_ON_FAILURE = True

Java:

.. sourcecode:: java

    public class MyContinuableError extends RuntimeException {
        public static final boolean ROBOT_CONTINUE_ON_FAILURE = true;
    }

__ `Continue on failure`_

日志信息
^^^^^^^^^^^^^^^^^^^

异常消息不是为用户提供信息的唯一途径. 方法可以通过向标准输出流(stdout)或者标准错误流(stderr)写入的方式来写 `日志文件`_, 同时这种写入还可以使用不同的 `日志级别`_. 当然, 更好的写日志的方式是使用 `日志API`_.

Exception messages are not the only way to give information to the
users. In addition to them, methods can also send messages to `log
files`_ simply by writing to the standard output stream (stdout) or to
the standard error stream (stderr), and they can even use different
`log levels`_. Another, and often better, logging possibility is using
the `programmatic logging APIs`_.

默认情况下, 向标准输出中写入的所有内容都会以一条`INFO` 级别的日志被写入到日志文件. 向标准错误流中写入的消息处理也类似, 不过它们会在关键字结束时, 在初始的stderr中回显. 因此, 如果你需要在测试执行的时候在控制台显示消息, 可以使用stderr.

By default, everything written by a method into the standard output is
written to the log file as a single entry with the log level
`INFO`. Messages written into the standard error are handled
similarly otherwise, but they are echoed back to the original stderr
after the keyword execution has finished. It is thus possible to use
the stderr if you need some messages to be visible on the console where
tests are executed.

Using log levels
使用日志级别
''''''''''''''''

要使用其它的日志级别, 可以将明确的日志级别嵌入到日志消息中, 以 `*级别* 实际内容` 的格式提供. 其中 `*级别*` 必须在行首, 而且必须是下列日志级别的其中之一:  `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` 和 `HTML`.


To use other log levels than `INFO`, or to create several
messages, specify the log level explicitly by embedding the level into
the message in the format `*LEVEL* Actual log message`, where
`*LEVEL*` must be in the beginning of a line and `LEVEL` is
one of the available logging levels `TRACE`, `DEBUG`,
`INFO`, `WARN`, `ERROR` and `HTML`.

错误与警告
'''''''''''''''''''

`ERROR` 或 `WARN` 级别的消息会自动写入控制台, 并在日志文件中写入单独的 `测试执行错误章节`__. 这些都是为了让这些消息提示更加显著

Messages with `ERROR` or `WARN` level are automatically written to the
console and a separate `Test Execution Errors section`__ in the log
files. This makes these messages more visible than others and allows
using them for reporting important but non-critical problems to users.

.. note:: 在 Robot Framework 2.9 版本中, 把 ERRORs 自动写入测试执行错误章节
          作为新功能被加入.


.. note:: In Robot Framework 2.9, new functionality was added to automatically
          add ERRORs logged by keywords to the Test Execution Errors section.

__ `测试执行中的错误和警告`_

Logging HTML
''''''''''''

测试库写日志的所有内容, 默认情况下都会被转换为 可被安全表示为HTML 的格式. 例如, `<b>foo</b>` 在日志中会完全按原样展示, 而不是粗体的 **foo**. 
如果测试库希望显示格式化的内容, 或者链接, 图片等等, 就可以使用一种特殊的伪测试级别 `HTML`. Robot Framework 仍将这些消息按 `INFO` 级别写入日志, 但是可以使用任意的 HTML 语法. 
注意, 这个特性功能需要小心使用, 因为一个错误的 `</table>` 标签就有可能使整个日志文件变得非常糟糕.

当使用 `日志API`_ 时, 不同日志级别的方法都提供了一个可选选项 `html`, 如果想使用HTML格式的内容, 可以将其设置为 `True`


时间戳
''''''''''

默认情况下, 通过stdout或stderr记录的日志消息的时间戳是在关键字结束后获取到的. 这就意味着这个时间戳是不准确的, 特别是在一个长时间执行的关键字中, 想借此定位问题是有问题的.

如果有需要的话, 关键字可以为日志消息添加精确的时间戳. 这个时间戳必须以 `Unix时间戳`__ 的格式提供, 紧跟 `日志级别`_ 后面, 两者以冒号(:)隔开, 例如::

   *INFO:1308435758660* Message with timestamp
   *HTML:1308435758661* <b>HTML</b> message with timestamp

参考下面的示例, 添加这种时间戳对于Python和Java来说都是很容易的事情. 如果使用的是Python, 通过使用 `日志API`_ 会格外简单. 添加明确的时间戳的一个好处是其在 `远程库接口`_ 中仍然有效.


As illustrated by the examples below, adding the timestamp is easy
both using Python and Java. If you are using Python, it is, however,
even easier to get accurate timestamps using the `programmatic logging
APIs`_. A big benefit of adding timestamps explicitly is that this
approach works also with the `remote library interface`_.

Python:

.. sourcecode:: python

    import time

    def example_keyword():
        print '*INFO:%d* Message with timestamp' % (time.time()*1000)

Java:

.. sourcecode:: java

    public void exampleKeyword() {
        System.out.println("*INFO:" + System.currentTimeMillis() + "* Message with timestamp");
    }

__ http://en.wikipedia.org/wiki/Unix_epoch
__ `Using log levels`_

控制台日志
Logging to console
''''''''''''''''''

测试库如果想向控制台写入一些内容, 可以有几种选择. 前面已经讨论过的, 警告消息, 以及所有写入到stderr中内容会同时写入日志文件和控制台. 
这两种方式都有一个限制, 那就是消息只有等当前的关键字执行完毕后才会打印出来. 但是有个好处是, 这两种方法在Python和Java中都可用.

另一个方式只有Python支持, 那就是把消息写入  `sys.__stdout__` 或 `sys.__stderr__`. 这种方式, 消息会立即在控制台显示, 并且不会写入到日志文件. 例如:

.. sourcecode:: python

   import sys

   def my_keyword(arg):
      sys.__stdout__.write('Got arg %s\n' % arg)

最后一个选择就是使用 `Public日志API`_:

.. sourcecode:: python

   from robot.api import logger

   def log_to_console(arg):
      logger.console('Got arg %s' % arg)

   def log_to_console_and_log_file(arg)
      logger.info('Got arg %s' % arg, also_console=True)

.. Logging example
日志示例
'''''''''''''''

`INFO` 级别的日志可以胜任大多数情况. 比它更低的级别, `DEBUG` 和 `TRACE`, 用来打印调试信息. 这两种消息平常不怎么展示, 但在debugging测试库自身的问题时很有用. `WARN` 或 `ERROR` 级别可以使得消息提示更显著. 而 `HTML` 在需要多种格式的时候很有用.

下面的示例阐明了不同的日志级别是如何工作的. 对于Java程序员来说, 下面代码中的 `print 'message'` 可以认为是 `System.out.println("message");`.

.. sourcecode:: python

   print 'Hello from a library.'
   print '*WARN* Warning from a library.'
   print '*ERROR* Something unexpected happen that may indicate a problem in the test.'
   print '*INFO* Hello again!'
   print 'This will be part of the previous message.'
   print '*INFO* This is a new message.'
   print '*INFO* This is <b>normal text</b>.'
   print '*HTML* This is <b>bold</b>.'
   print '*HTML* <a href="http://robotframework.org">Robot Framework</a>'

.. raw:: html

   <table class="messages">
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="info level">INFO</td>
       <td class="msg">Hello from a library.</td>
     </tr>
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="warn level">WARN</td>
       <td class="msg">Warning from a library.</td>
     </tr>
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="error level">ERROR</td>
       <td class="msg">Something unexpected happen that may indicate a problem in the test.</td>
     </tr>
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="info level">INFO</td>
       <td class="msg">Hello again!<br>This will be part of the previous message.</td>
     </tr>
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="info level">INFO</td>
       <td class="msg">This is a new message.</td>
     </tr>
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="info level">INFO</td>
       <td class="msg">This is &lt;b&gt;normal text&lt;/b&gt;.</td>
     </tr>
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="info level">INFO</td>
       <td class="msg">This is <b>bold</b>.</td>
     </tr>
     <tr>
       <td class="time">16:18:42.123</td>
       <td class="info level">INFO</td>
       <td class="msg"><a href="http://robotframework.org">Robot Framework</a></td>
     </tr>
   </table>


.. Programmatic logging APIs
编程日志API
^^^^^^^^^^^^^^^^^^^^^^^^^

日志API, 相对于往stdout和stderr中写入内容, 提供了更清晰的写日志方式. 但是, 当前这些接口只对基于Python的库可用.

Public logging API
''''''''''''''''''
Robot Framework 提供了基于Python的日志API, 可以用来写日志文件和控制台. 测试库可以按照类似 `logger.info('My message')` 的方式来调用API, 以替代直接写stdout的方式 `print '*INFO* My message'`. 使用API接口不但看上去更清楚, 还有个好处是可以提供精确的 `时间戳`_.

The public logging API `is thoroughly documented`__ as part of the API
documentation at https://robot-framework.readthedocs.org. Below is
a simple usage example:

.. sourcecode:: python

   from robot.api import logger

   def my_keyword(arg):
       logger.debug('Got argument %s' % arg)
       do_something()
       logger.info('<i>This</i> is a boring example', html=True)
       logger.console('Hello, console!')

使用这个日志API的一个明显的限制是会使测试库依赖于 Robot Framework. 在 2.8.7 版本之前, Robot还必须是运行状态才可用. 从 2.8.7 版本开始, 如果Robot不在运行中, 消息会自动重定向到Python的标准 logging__ 模块.

An obvious limitation is that test libraries using this logging API have
a dependency to Robot Framework. Before version 2.8.7 Robot also had
to be running for the logging to work. Starting from Robot Framework 2.8.7
if Robot is not running the messages are redirected automatically to Python's
standard logging__ module.

__ https://robot-framework.readthedocs.org/en/latest/autodoc/robot.api.html#module-robot.api.logger
__ http://docs.python.org/library/logging.html

使用Python标准 `logging` 模块
Using Python's standard `logging` module
''''''''''''''''''''''''''''''''''''''''

除了 `public logging API`_, Robot Framework 提供对Python标准日志模块 logging__ 的支持. 使用这个模块后, 所有root logger收到的消息都会自动传递给 Robot Framework的日志文件. 同样, 该API提供了精确 时间戳_ 的支持. 但是不支持HTML格式, 以及向控制台打印日志. 
最大的好处是, 使用这种日志API不会对 Robot Framework 产生依赖.

.. sourcecode:: python

   import logging

   def my_keyword(arg):
       logging.debug('Got argument %s' % arg)
       do_something()
       logging.info('This is a boring example')

`logging` 模块的日志级别和Robot Framework的相比略有不同, 其中 `DEBUG`, `INFO`, `WARNING` 和 `ERROR` 直接对应Robot Framework相应的日志级别, `CRITICAL` 对应 `ERROR`. 
自定义的日志级别映射为 "和它最接近, 同时低于它" 的标准级别. 例如, 介于 `INFO` 和 `WARNING` 之间的级别最终映射为 `INFO` 级别.

__ http://docs.python.org/library/logging.html

.. Logging during library initialization
库初始化时写日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

库在导入和初始化时也可以写日志. 这部分日志不会和普通日志消息一样写入 `日志文件`_, 而是写入 `syslog`_. 这种日志可以将任何关于库的初始化的debug信息记录下来. 级别设置为 `WARN` 或者 `ERROR` 的日志同时也可在 `测试执行错误`_ 章节中看到.

Libraries can also log during the test library import and initialization.
These messages do not appear in the `log file`_ like the normal log messages,
but are instead written to the `syslog`_. This allows logging any kind of
useful debug information about the library initialization. Messages logged
using the `WARN` or `ERROR` levels are also visible in the `test execution errors`_
section in the log file.

这种日志既可以使用 `标准输出和错误流`__ 的方式, 也可以使用 `编程日志API`_. 下面的例子都做了说明:

Logging during the import and initialization is possible both using the
`standard output and error streams`__ and the `programmatic logging APIs`_.
Both of these are demonstrated below.

Java库在初始化时通过stdout写日志
Java library logging via stdout during initialization:

.. sourcecode:: java

   public class LoggingDuringInitialization {

       public LoggingDuringInitialization() {
           System.out.println("*INFO* Initializing library");
       }

       public void keyword() {
           // ...
       }
   }

Python库在导入时通过logging API写日志:
Python library logging using the logging API during import:

.. sourcecode:: python

   from robot.api import logger

   logger.debug("Importing library")

   def keyword():
       # ...

.. note:: 如果你在初始化阶段写日志, 例如, 在Python的 `__init__` 方法中或者Java的构造函数中, 这些日志按 `测试库作用域`_ 的不同, 可能会记录多次.


.. note:: If you log something during initialization, i.e. in Python
          `__init__` or in Java constructor, the messages may be
          logged multiple times depending on the `test library scope`_.

__ `Logging information`_

.. Returning values
返回值
^^^^^^^^^^^^^^^^

关键字与核心框架间交互的最后一步就是返回值, 该值可以是从被测系统获取, 也可能是其它方式生成的. 这个返回值可以被 `赋值给变量`__, 然后作为其它关键字的输入, 这些关键字可以是属于不同的测试库的. 

The final way for keywords to communicate back to the core framework
is returning information retrieved from the system under test or
generated by some other means. The returned values can be `assigned to
variables`__ in the test data and then used as inputs for other keywords,
even from different test libraries.

在Python和Java方法中, 都使用 `return` 语句来返回值. 一般情况下,  一个值会赋给一个 `标量变量`__, 如下例所示. 该示例还展现了返回值可以是任意对象, 并且使用 `扩展变量语法`_ 来获取对象的属性.


Values are returned using the `return` statement both from
the Python and Java methods. Normally, one value is assigned into one
`scalar variable`__, as illustrated in the example below. This example
also illustrates that it is possible to return any objects and to use
`extended variable syntax`_ to access object attributes.

__ `Return values from keywords`_
__ `Scalar variables`_

.. sourcecode:: python

  from mymodule import MyObject

  def return_string():
      return "Hello, world!"

  def return_object(name):
      return MyObject(name)

.. sourcecode:: robotframework

   *** Test Cases ***
   Returning one value
       ${string} =    Return String
       Should Be Equal    ${string}    Hello, world!
       ${object} =    Return Object    Robot
       Should Be Equal    ${object.name}    Robot

关键字还可以一次返回多个值, 这些值可以一次性的赋值给多个 `标量变量`_, 或者是一个 `列表变量`__, 亦或者是若干标量变量加上一个列表变量. 所有这些用法要求返回的值是Python的列表(lists)或者元组(tuples), 或者是Java中的数组(arrays), 列表(Lists)或迭代器(Iterators).

Keywords can also return values so that they can be assigned into
several `scalar variables`_ at once, into `a list variable`__, or
into scalar variables and a list variable. All these usages require
that returned values are Python lists or tuples or
in Java arrays, Lists, or Iterators.

__ `List variables`_

.. sourcecode:: python

  def return_two_values():
      return 'first value', 'second value'

  def return_multiple_values():
      return ['a', 'list', 'of', 'strings']


.. sourcecode:: robotframework

   *** Test Cases ***
   Returning multiple values
       ${var1}    ${var2} =    Return Two Values
       Should Be Equal    ${var1}    first value
       Should Be Equal    ${var2}    second value
       @{list} =    Return Two Values
       Should Be Equal    @{list}[0]    first value
       Should Be Equal    @{list}[1]    second value
       ${s1}    ${s2}    @{li} =    Return Multiple Values
       Should Be Equal    ${s1} ${s2}    a list
       Should Be Equal    @{li}[0] @{li}[1]    of strings

.. Communication when using threads
使用多线程
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果库使用了多线程, 通常应该只在主线程中与框架通讯. 如果一个工作线程需要发送错误报告或者其它日志, 它应该首先将信息传给主线程. 主线程使用异常或本章介绍的其它机制来与框架通讯.

If a library uses threads, it should generally communicate with the
framework only from the main thread. If a worker thread has, for
example, a failure to report or something to log, it should pass the
information first to the main thread, which can then use exceptions or
other mechanisms explained in this section for communication with the
framework.

当线程在后台运行, 同时其它关键字在运行时这点显得尤为重要. 这种情况下, (子线程)和框架间的通讯是未定义的(undefined), 甚至在最坏的情况下会导致程序崩溃, 或者输出文件损坏. 
如果一个关键字启动了后台任务, 那么要想检查后台线程的状态, 或者搜集相应的信息上报, 需要使用另外的关键字来完成.

This is especially important when threads are run on background while
other keywords are running. Results of communicating with the
framework in that case are undefined and can in the worst case cause a
crash or a corrupted output file. If a keyword starts something on
background, there should be another keyword that checks the status of
the worker thread and reports gathered information accordingly.

非主线程中使用 `编程日志API`_ 提供的普通写日志方法, 内容会被默默地忽略.

Messages logged by non-main threads using the normal logging methods from
`programmatic logging APIs`_  are silently ignored.

不过, 有个单独的 robot后台日志__ 项目, 提供了  `BackgroundLogger` , 拥有和标准 `robot.api.logger` 类似的API. 使用 `BackgroundLogger` , 非主线程的日志消息也会被保存下来.

There is also a `BackgroundLogger` in separate robotbackgroundlogger__ project,
with a similar API as the standard `robot.api.logger`. Normal logging
methods will ignore messages from other than main thread, but the
`BackgroundLogger` will save the background messages so that they can be later
logged to Robot's log.

__ https://github.com/robotframework/robotbackgroundlogger

测试库的分发
Distributing test libraries
---------------------------

测试库的文档
Documenting libraries
^^^^^^^^^^^^^^^^^^^^^

一个测试库如果没有提供文档来说明其中包含了哪些关键字, 以及这些关键字的用途的话, 那么不如说这个测试库是没用的(useless). 为了容易维护, 强烈建议把测试库的文档内容直接写在源代码里, 并从中生成文档. 基本上, 这意味着在Python中要使用 docstrings_, 在Java中使用  Javadoc_. 如下例所示:


A test library without documentation about what keywords it
contains and what those keywords do is rather useless. To ease
maintenance, it is highly recommended that library documentation is
included in the source code and generated from it. Basically, that
means using docstrings_ with Python and Javadoc_ with Java, as in
the examples below.

.. sourcecode:: python

    class MyLibrary:
        """This is an example library with some documentation."""

        def keyword_with_short_documentation(self, argument):
            """This keyword has only a short documentation"""
            pass

        def keyword_with_longer_documentation(self):
            """First line of the documentation is here.

            Longer documentation continues here and it can contain
            multiple lines or paragraphs.
            """
            pass

.. sourcecode:: java

    /**
     *  This is an example library with some documentation.
     */
    public class MyLibrary {

        /**
         * This keyword has only a short documentation
         */
        public void keywordWithShortDocumentation(String argument) {
        }

        /**
         * First line of the documentation is here.
         *
         * Longer documentation continues here and it can contain
         * multiple lines or paragraphs.
         */
        public void keywordWithLongerDocumentation() {
        }

    }

对于如上所示的库文档, Python和Java都有各自的工具来生成API文档. 不过, 这些工具的输出对某些用户来说显得稍微有些专业. 
另一种方式是使用 Robot Framework自带的文档工具 Libdoc_. 这个工具不但可以创建使用静态库API的库文档, 不管是使用Python还是Java, 同时还能处理使用了 `动态库API`_ 和 `混合库API`_

Both Python and Java have tools for creating an API documentation of a
library documented as above. However, outputs from these tools can be slightly
technical for some users. Another alternative is using Robot
Framework's own documentation tool Libdoc_. This tool can
create a library documentation from both Python and Java libraries
using the static library API, such as the ones above, but it also handles
libraries using the `dynamic library API`_ and `hybrid library API`_.

关键字文档的第一行用于特殊用途, 一般包含对该关键字的简短概述. 它在某些情况下被当作是 *短文档* 来使用, 例如在 Libdoc_ 中可作为工具提示, 也可以在日志中展示. 不过在日志中展示对Java静态库不适用, 因为Java源码中的文档会在编译时去掉, 自然也不能在运行时获取到了.  

The first line of a keyword documentation is used for a special
purpose and should contain a short overall description of the
keyword. It is used as a *short documentation*, for example as a tool
tip, by Libdoc_ and also shown in the test logs. However, the latter
does not work with Java libraries using the static API,
because their documentations are lost in compilation and not available
at runtime.

默认情况下, 文档内容被认为是遵从 Robot Framework的 `文档格式`__ 规则的. 这份简单的格式允许使用常用的样式, 如 `*粗体*` 和 `_斜体_`, 表格, 列表, 链接等.
从 Robot Framework 2.7.5版本开始, 还可以使用HTML, 纯文本和 reStructuredText_ 格式.

By default documentation is considered to follow Robot Framework's
`documentation formatting`_ rules. This simple format allows often used
styles like `*bold*` and `_italic_`, tables, lists, links, etc.
Starting from Robot Framework 2.7.5, it is possible to use also HTML, plain
text and reStructuredText_ formats. See `Specifying documentation format`_
section for information how to set the format in the library source code and
Libdoc_ chapter for more information about the formats in general.

.. note:: If you want to use non-ASCII characters in the documentation of
          Python libraries, you must either use UTF-8 as your `source code
          encoding`__ or create docstrings as Unicode.

.. _docstrings: http://www.python.org/dev/peps/pep-0257
.. _javadoc: http://java.sun.com/j2se/javadoc/writingdoccomments/index.html
__ http://www.python.org/dev/peps/pep-0263

库的测试
Testing libraries
^^^^^^^^^^^^^^^^^

所有正式应用的测试库自身都需要彻底的被测试, 以避免其中的bug. 当然, 这些测试应该是自动化的, 这样当库有所改变时可以快速的回归测试.

Any non-trivial test library needs to be thoroughly tested to prevent
bugs in them. Of course, this testing should be automated to make it
easy to rerun tests when libraries are changed.

Python和Java都有出色的单元测试工具, 很适合用来测试自己开发的库.
使用这些单元测试工具来测试库和测试其它代码没什么区别. 熟悉这些工具的开发者无需额外学习新东西即可掌握, 当然, 不熟悉的开发需要先学习一下.

Both Python and Java have excellent unit testing tools, and they suite
very well for testing libraries. There are no major differences in
using them for this purpose compared to using them for some other
testing. The developers familiar with these tools do not need to learn
anything new, and the developers not familiar with them should learn
them anyway.

使用Robot Framework自己来测试这些测试库也很简单, 这种方式对它们来说实际上是端到端的验收测试. 内置_ 库提供了很多有用的关键字用于此类目的.
特别值得一提的, 关键字 :name:`Run Keyword And Expect Error` 就对测试关键字是否能正确地报告错误很有用.

It is also easy to use Robot Framework itself for testing libraries
and that way have actual end-to-end acceptance tests for them. There are
plenty of useful keywords in the BuiltIn_ library for this
purpose. One worth mentioning specifically is :name:`Run Keyword And Expect
Error`, which is useful for testing that keywords report errors
correctly.

到底是使用单元测试还是验收测试的方式取决于具体情况. 如果需要模拟真实的待测系统, 使用单元测试往往比较简单. 另一方面, 验收测试能保证关键字在Robot Framework上运行正常.
当然, 如果很难取舍, 同时使用这两种方法也是可以的.

Whether to use a unit- or acceptance-level testing approach depends on
the context. If there is a need to simulate the actual system under
test, it is often easier on the unit level. On the other hand,
acceptance tests ensure that keywords do work through Robot
Framework. If you cannot decide, of course it is possible to use both
the approaches.

测试库打包
Packaging libraries
^^^^^^^^^^^^^^^^^^^

当测试库开发完, 文档完成, 并且通过了测试, 接下来就是分发给用户. 对于那种只包含单个文件的简单的库, 告知用户将文件拷贝到相应的 `模块搜索路径`_ 就可以了. 更复杂的库需要进行打包, 以便能轻松安装.

After a library is implemented, documented, and tested, it still needs
to be distributed to the users. With simple libraries consisting of a
single file, it is often enough to ask the users to copy that file
somewhere and set the `module search path`_ accordingly. More
complicated libraries should be packaged to make the installation
easier.

因为测试库也是普通的源代码, 所以它们也可以使用普通的打包工具. 对于Python, 一个不错的选择是 distutils_, 包含在Python的标准库中, 或者较新一点的 setuptools_.
使用这些工具一个好处是, 测试库被安装的目标路径是自动包含在 `模块搜索路径`_ 中的. 

Since libraries are normal programming code, they can be packaged
using normal packaging tools. With Python, good options include
distutils_, contained by Python's standard library, and the newer
setuptools_. A benefit of these tools is that library modules are
installed into a location that is automatically in the `module
search path`_.

当使用Java时, 把库打包为JAR包是很自然的选择. 测试前必须先把这个JAR包放到  `模块搜索路径`_, 不过, 创建一个 `启动脚本`_ 来自动化处理这些事情会更轻松.

When using Java, it is natural to package libraries into a JAR
archive. The JAR package must be put into the `module search path`_
before running tests, but it is easy to create a `start-up script`_ that
does that automatically.

废弃关键字
Deprecating keywords
^^^^^^^^^^^^^^^^^^^^

有时候需要将现有的关键字替换为新的, 或者完全删除. 仅仅知会用户这些变更并不总是足够, 更有效的方式是在运行时刻发出警告. 为了达到此目的, Robot Framework 的关键字可以被标记为 *废弃的* (*deprecated*), 这样就可以很容易发现已经过时的关键字, 并把它们删除或替换掉.

Sometimes there is a need to replace existing keywords with new ones
or remove them altogether. Just informing the users about the change
may not always be enough, and it is more efficient to get warnings at
runtime. To support that, Robot Framework has a capability to mark
keywords *deprecated*. This makes it easier to find old keywords from
the test data and remove or replace them.

想要废弃一个关键字的话, 在关键字的文档中以 `*DEPRECATED` 开始, 并且在第一行内以一个 `*` 结束, 注意这里需要区分大小写. 例如,  `*DEPRECATED*`, `*DEPRECATED.*`, 
`*DEPRECATED in version 1.5.*` 都是合法的标记.

Keywords can be deprecated by starting their documentation with text
`*DEPRECATED`, case-sensitive, and having a closing `*` also on the first
line of the documentation. For example, `*DEPRECATED*`, `*DEPRECATED.*`, and
`*DEPRECATED in version 1.5.*` are all valid markers.

当执行了一个废弃的关键字, 一条已废弃警告会被记入日志, 并且这个警告同时会出现在 `控制台和日志文件中的测试执行错误章节`__. 这个警告消息以 `Keyword '<name>' is deprecated.` 开头, 后面是该关键字的 `短文档`__.
例如, 如果下面的关键字被执行, 会有如下所示的警告:

When a deprecated keyword is executed, a deprecation warning is logged and
the warning is shown also in `the console and the Test Execution Errors
section in log files`__. The deprecation warning starts with text `Keyword
'<name>' is deprecated.` and has rest of the `short documentation`__ after
the deprecation marker, if any, afterwards. For example, if the following
keyword is executed, there will be a warning like shown below in the log file.

.. sourcecode:: python

    def example_keyword(argument):
        """*DEPRECATED!!* Use keyword `Other Keyword` instead.

        This keyword does something to given ``argument`` and returns results.
        """
        return do_something(argument)

.. raw:: html

   <table class="messages">
     <tr>
       <td class="time">20080911&nbsp;16:00:22.650</td>
       <td class="warn level">WARN</td>
       <td class="msg">Keyword 'SomeLibrary.Example Keyword' is deprecated. Use keyword `Other Keyword` instead.</td>
     </tr>
   </table>

这个废弃系统对大多数的库都有效, 包括 `用户关键字`__. 唯一的例外是用Java实现的静态关键字, 因为文档会在编译时丢失, 无法在运行时获取到. 对于这些关键字, 可以使用用户关键字作为封装, 然后废弃.

This deprecation system works with most test libraries and also with
`user keywords`__.  The only exception are keywords implemented in a
Java test library that uses the `static library interface`__ because
their documentation is not available at runtime. With such keywords,
it possible to use user keywords as wrappers and deprecate them.

.. note:: Robot Framework 2.9版本之前, 文档必须精确地以 `*DEPRECATED*` 开始,
          在结束星号 `*` 之前不能有任何额外的内容.


.. note:: Prior to Robot Framework 2.9 the documentation must start with
          `*DEPRECATED*` exactly without any extra content before the
          closing `*`.

__ `Errors and warnings during execution`_
__ `Documenting libraries`_
__ `User keyword name and documentation`_
__ `Creating static keywords`_

.. _Dynamic library:

动态库API
Dynamic library API
-------------------

动态库API大部分情况和静态API类似. 例如, 报告关键字状态, 写日志, 以及返回值, 都是以完全相同的方式工作. 最重要的是, 和其它测试库相比, 导入动态库并使用其中的关键字,
完全没有区别. 换句话说, 用户无需知道测试库是使用何种API实现的.

The dynamic API is in most ways similar to the static API. For
example, reporting the keyword status, logging, and returning values
works exactly the same way. Most importantly, there are no differences
in importing dynamic libraries and using their keywords compared to
other libraries. In other words, users do not need to know what APIs their
libraries use.

静态和动态库的唯一区别在于Robot Framework是如何发现库中实现了哪些关键字, 这些关键字的参数和文档信息, 以及这些关键字实际是怎样执行的.
对于静态API, 这些都是通过反射机制(除了Java库的文档), 但是对于动态库, 需要通过几个特殊的方法来实现.


Only differences between static and dynamic libraries are
how Robot Framework discovers what keywords a library implements,
what arguments and documentation these keywords have, and how the
keywords are actually executed. With the static API, all this is
done using reflection (except for the documentation of Java libraries),
but dynamic libraries have special methods that are used for these
purposes.

使用动态API的一个好处是可以更灵活地组织库. 使用静态API时, 所有的关键字必须在一个类或者模块中, 然而对动态API, 举例来说, 你可以将每个关键字都实现为一个单独的类. 这种场景对Python来说不那么重要, 因为Python本身的动态特性和多重继承机制已经有了足够的灵活性, 而且还可以使用 `混合库API`_.

One of the benefits of the dynamic API is that you have more flexibility
in organizing your library. With the static API, you must have all
keywords in one class or module, whereas with the dynamic API, you can,
for example, implement each keyword as a separate class. This use case is
not so important with Python, though, because its dynamic capabilities and
multi-inheritance already give plenty of flexibility, and there is also
possibility to use the `hybrid library API`_.

另一个使用动态API的主要用户场景是可以实现一个库, 这个库仅作为代理, 实际的库可能运行在其它进程, 甚至其它机器上. 这种代理库可以非常轻量, 因为关键字的名称和其它所有信息都是动态的, 所以每次当实际库中新增了关键字, 没必要再去更新代理.

Another major use case for the dynamic API is implementing a library
so that it works as proxy for an actual library possibly running on
some other process or even on another machine. This kind of a proxy
library can be very thin, and because keyword names and all other
information is got dynamically, there is no need to update the proxy
when new keywords are added to the actual library.

本节介绍了动态API是如何在Robot Framework和动态库中工作的. 对Robot Framework来说, 它并不关心这些库实际是如何实现的(例如, `run_keyword` 方法是如何映射到相应的关键字). 实际上, 可能会有很多不同的方式. 
但是, 如果你是使用Java, 在实现自己的系统前不妨先参考下 `JavalibCore <https://github.com/robotframework/JavalibCore>`_. 这个可重用工具的集合支持多种关键字创建方式, 也许其中的某个机制正好符合你的需求.

This section explains how the dynamic API works between Robot
Framework and dynamic libraries. It does not matter for Robot
Framework how these libraries are actually implemented (for example,
how calls to the `run_keyword` method are mapped to a correct
keyword implementation), and many different approaches are
possible. However, if you use Java, you may want to examine
`JavalibCore <https://github.com/robotframework/JavalibCore>`__
before implementing your own system. This collection of
reusable tools supports several ways of creating keywords, and it is
likely that it already has a mechanism that suites your needs.

.. _`Getting dynamic keyword names`:

.. Getting keyword names
获取关键字名称
^^^^^^^^^^^^^^^^^^^^^

动态库通过 `get_keyword_names` 方法来告知它实现了哪些关键字. 当使用Java时, 还可以使用这个方法的别名 `getKeywordNames`, 更符合Java的命名规范. 这个方法不能接受任何参数, 必须返回一个字符串的列表或数组, 这些字符串就是这个库实现的关键字的名称.


Dynamic libraries tell what keywords they implement with the
`get_keyword_names` method. The method also has the alias
`getKeywordNames` that is recommended when using Java. This
method cannot take any arguments, and it must return a list or array
of strings containing the names of the keywords that the library implements.

如果返回的关键字名称包含多个单词, 它们可以以空格或者下划线分隔, 或者使用驼峰法(camelCase)格式. 例如, `['first keyword', 'second keyword']`, `['first_keyword', 'second_keyword']`, 和 `['firstKeyword', 'secondKeyword']` 最后都会被映射为 :name:`First Keyword` and :name:`Second Keyword`.

If the returned keyword names contain several words, they can be returned
separated with spaces or underscores, or in the camelCase format. For
example, `['first keyword', 'second keyword']`,
`['first_keyword', 'second_keyword']`, and
`['firstKeyword', 'secondKeyword']` would all be mapped to keywords
:name:`First Keyword` and :name:`Second Keyword`.

动态库必须总是包含这个方法, 如果没有, 或者调用它时因为某些原因发生了错误, 这个库将被视作静态库.

Dynamic libraries must always have this method. If it is missing, or
if calling it fails for some reason, the library is considered a
static library.

Marking methods to expose as keywords
'''''''''''''''''''''''''''''''''''''
如果一个动态库中包含的方法既有那些最终作为关键字执行的, 也有那些私有的提供辅助功能的, 那么将这些关键字方法打上标记会使 `get_keyword_names` 的实现变得轻松.
装饰器 `robot.api.deco.keyword` 提供了简便的方式. 它为被装饰的方法创建了 `robot_name` 属性. 于是, 在 `get_keyword_names` 中, 可以通过检查每个方法的 `robot_name` 属性来创建关键字的列表. 关于该装饰器的更多内容请参考 `使用自定义关键字名`_.

If a dynamic library should contain both methods which are meant to be keywords
and methods which are meant to be private helper methods, it may be wise to
mark the keyword methods as such so it is easier to implement `get_keyword_names`.
The `robot.api.deco.keyword` decorator allows an easy way to do this since it
creates a custom `robot_name` attribute on the decorated method.
This allows generating the list of keywords just by checking for the `robot_name`
attribute on every method in the library during `get_keyword_names`.  See
`Using a custom keyword name`_ for more about this decorator.

.. sourcecode:: python

   from robot.api.deco import keyword

   class DynamicExample:

       def get_keyword_names(self):
           return [name for name in dir(self) if hasattr(getattr(self, name), 'robot_name')]

       def helper_method(self):
           # ...

       @keyword
       def keyword_method(self):
           # ...

.. _`Running dynamic keywords`:

Running keywords
运行关键字
^^^^^^^^^^^^^^^^

动态库还有一个特殊的 `run_keyword` (别名 `runKeyword`) 方法用来执行关键字.
当动态库中的关键字在测试用例中被调用时, Robot Framework 通过调用这个库的 `run_keyword` 方法使其运行. 这个方法接受2个或者3个参数, 第1个参数是一个字符串, 即要执行的关键字的名称, 这个名称的格式和  `get_keyword_names` 返回的一样. 第2个参数是一个参数的列表或者数组, 其中包含需要传递给该关键字的参数. 第3个可选参数是一个Python字典(dict)或者Java中的map, 其中是要传递给关键字的可能的 `任意关键字参数`_ (`**kwargs`). 

Dynamic libraries have a special `run_keyword` (alias
`runKeyword`) method for executing their keywords. When a
keyword from a dynamic library is used in the test data, Robot
Framework uses the library's `run_keyword` method to get it
executed. This method takes two or three arguments. The first argument is a
string containing the name of the keyword to be executed in the same
format as returned by `get_keyword_names`. The second argument is
a list or array of arguments given to the keyword in the test data.

The optional third argument is a dictionary (map in Java) that gets
possible `free keyword arguments`_ (`**kwargs`) passed to the
keyword. See `free keyword arguments with dynamic libraries`_ section
for more details about using kwargs with dynamic test libraries.

当获取到关键字名称和参数后, 库可以按自己的方式自由地执行这个关键字, 但是它还是使用和静态库相同的机制来和框架通讯. 也就是说, 使用异常来报告状态, 通过写stdout或API来写日志, 使用return语句来返回值.

After getting keyword name and arguments, the library can execute
the keyword freely, but it must use the same mechanism to
communicate with the framework as static libraries. This means using
exceptions for reporting keyword status, logging by writing to
the standard output or by using provided logging APIs, and using
the return statement in `run_keyword` for returning something.

每个动态库都必须包含 `get_keyword_names` 和 `run_keyword` 这两个方法, 其它的方法都是可选的. 
下面的例子展示了一个用Python实现的动态库, 虽然没有实用价值.

Every dynamic library must have both the `get_keyword_names` and
`run_keyword` methods but rest of the methods in the dynamic
API are optional. The example below shows a working, albeit
trivial, dynamic library implemented in Python.

.. sourcecode:: python

   class DynamicExample:

       def get_keyword_names(self):
           return ['first keyword', 'second keyword']

       def run_keyword(self, name, args):
           print "Running keyword '%s' with arguments %s." % (name, args)

获取关键字的参数
Getting keyword arguments
^^^^^^^^^^^^^^^^^^^^^^^^^

如果一个动态库仅仅实现了 `get_keyword_names` 和 `run_keyword` 这两个方法, Robot Framework将无法获取任何关于关键字所需的参数信息. 例如, 上例中的  :name:`First Keyword` 和 :name:`Second Keyword` 都可以接受任意数量的参数.
现实中大部分关键字都预期接受一定个数的参数, 在这种情况下它们将不得不自己检查参数的个数, 所以, 这是个问题.

If a dynamic library only implements the `get_keyword_names` and
`run_keyword` methods, Robot Framework does not have any information
about the arguments that the implemented keywords need. For example,
both :name:`First Keyword` and :name:`Second Keyword` in the example above
could be used with any number of arguments. This is problematic,
because most real keywords expect a certain number of keywords, and
under these circumstances they would need to check the argument counts
themselves.

动态库通过 `get_keyword_arguments` (别名 `getKeywordArguments`) 方法来告知Robot Framework 关键字预期的参数. 这个方法接受关键字的名称作为参数, 返回一个字符串的列表或数组, 每个字符串表示该关键字可接受的参数.

Dynamic libraries can tell Robot Framework what arguments the keywords
it implements expect by using the `get_keyword_arguments`
(alias `getKeywordArguments`) method. This method takes the name
of a keyword as an argument, and returns a list or array of strings
containing the arguments accepted by that keyword.

和静态关键字类似, 动态关键字可以有任意数量的参数, 可以有缺省值, 还可以同时接受可变数量的参数以及任意关键字参数. 
下面的表格说明了使用怎样的语法来表示这些不同的参数类型. 注意, 示例中使用的是Python的列表, Java开发应该用Java的列表或字符串数组替代.

Similarly as static keywords, dynamic keywords can require any number
of arguments, have default values, and accept variable number of
arguments and free keyword arguments. The syntax for how to represent
all these different variables is explained in the following table.
Note that the examples use Python syntax for lists, but Java developers
should use Java lists or String arrays instead.

.. table:: Representing different arguments with `get_keyword_arguments`
   :class: tabular

   +--------------------+----------------------------+------------------------------+----------+
   |    Expected        |      How to represent      |            Examples          | Limits   |
   |    arguments       |                            |                              | (min/max)|
   +====================+============================+==============================+==========+
   | No arguments       | Empty list.                | | `[]`                       | | 0/0    |
   +--------------------+----------------------------+------------------------------+----------+
   | One or more        | List of strings containing | | `['one_argument']`         | | 1/1    |
   | argument           | argument names.            | | `['a1', 'a2', 'a3']`       | | 3/3    |
   +--------------------+----------------------------+------------------------------+----------+
   | Default values     | Default values separated   | | `['arg=default value']`    | | 0/1    |
   | for arguments      | from names with `=`.       | | `['a', 'b=1', 'c=2']`      | | 1/3    |
   |                    | Default values are always  |                              |          |
   |                    | considered to be strings.  |                              |          |
   +--------------------+----------------------------+------------------------------+----------+
   | Variable number    | Last (or second last with  | | `['*varargs']`             | | 0/any  |
   | of arguments       | kwargs) argument has `*`   | | `['a', 'b=42', '*rest']`   | | 1/any  |
   | (varargs)          | before its name.           |                              |          |
   +--------------------+----------------------------+------------------------------+----------+
   | Free keyword       | Last arguments has         | | `['**kwargs']`             | | 0/0    |
   | arguments (kwargs) | `**` before its name.      | | `['a', 'b=42', '**kws']`   | | 1/2    |
   |                    |                            | | `['*varargs', '**kwargs']` | | 0/any  |
   +--------------------+----------------------------+------------------------------+----------+


当使用了 `get_keyword_arguments`, Robot Framework自动计算出有多少位置参数, 以及是否支持自由命名参数. 如果传递了错误的参数给关键字, 会在 `run_keyword` 调用之前就提示错误.

When the `get_keyword_arguments` is used, Robot Framework automatically
calculates how many positional arguments the keyword requires and does it
support free keyword arguments or not. If a keyword is used with invalid
arguments, an error occurs and `run_keyword` is not even called.

通过该方法返回的实际的参数名称和缺省值也同样重要. `命名参数`_ 和 Libdoc_ 需要用到它们.

The actual argument names and default values that are returned are also
important. They are needed for `named argument support`__ and the Libdoc_
tool needs them to be able to create a meaningful library documentation.

如果没有 `get_keyword_arguments` 方法, 或者针对某个关键字调用该方法返回了 `None` 或 `null`, 则该关键字的参数规范就是可以接受所有参数. 这个自动的参数规范是 `[*varargs, **kwargs]` 或者 `[*varargs]`, 取决于 `run_keyword` 是否包含第3个代表 kwargs 的参数.

If `get_keyword_arguments` is missing or returns `None` or
`null` for a certain keyword, that keyword gets an argument specification
accepting all arguments. This automatic argument spec is either
`[*varargs, **kwargs]` or `[*varargs]`, depending does
`run_keyword` `support kwargs`__ by having three arguments or not.

__ `Named argument syntax with dynamic libraries`_
__ `Free keyword arguments with dynamic libraries`_

获取关键字的文档
Getting keyword documentation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

最后一个动态库可实现的特殊方法是 `get_keyword_documentation` (别名 `getKeywordDocumentation`). 顾名思义, 它接受一个关键字名称作为参数, 返回该关键字的文档, 以一个字符串的形式.

The final special method that dynamic libraries can implement is
`get_keyword_documentation` (alias
`getKeywordDocumentation`). It takes a keyword name as an
argument and, as the method name implies, returns its documentation as
a string.

返回的文档用起来和Python静态库的文档字符串没什么差别. 主要的使用场景就是插入到 Libdoc_ 生成的文档中. 并且文档第一行(第一个 `\n` 之前的部分)会写入到日志中.

The returned documentation is used similarly as the keyword
documentation string with static libraries implemented with
Python. The main use case is getting keywords' documentations into a
library documentation generated by Libdoc_. Additionally,
the first line of the documentation (until the first `\n`) is
shown in test logs.

Getting keyword tags
获取关键字的标签
^^^^^^^^^^^^^^^^^^^^

动态库没有其它方法来定义 `关键字标签`_, 除了在文档的最后一行, 以 `Tags:` 作为前缀指定.
今后有可能会添加单独的 `get_keyword_tags` 方法到动态库的API中.

Dynamic libraries do not have any other way for defining `keyword tags`_
than by specifying them on the last row of the documentation with `Tags:`
prefix. Separate `get_keyword_tags` method can be added to the dynamic API
later if there is a need.

Getting general library documentation
获取库的综合文档
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`get_keyword_documentation` 方法还可以被用来指定测试库的总文档. 这部分不在测试执行时使用, 但是它们可以让 Libdoc_ 生成的文档变得更好.

The `get_keyword_documentation` method can also be used for
specifying overall library documentation. This documentation is not
used when tests are executed, but it can make the documentation
generated by Libdoc_ much better.

这种综合性的文档有两种, 一种是关于库的介绍, 另一个是关于库的使用指导. 前一种需要传递 `__intro__` 参数给 `get_keyword_documentation`, 而后一种传递 `__init__`. 这两种文档的差别, 最好是通过 Libdoc_ 实践看看表现.

Dynamic libraries can provide both general library documentation and
documentation related to taking the library into use. The former is
got by calling `get_keyword_documentation` with special value
`__intro__`, and the latter is got using value
`__init__`. How the documentation is presented is best tested
with Libdoc_ in practice.

基于Python的动态库还可以通过代码的文档字符串(docstring)来指定综合文档. 其中类的docstring对应 `__intro__`, `__init__` 方法的对应 `__init__`. 如果通过代码和 `get_keyword_documentation` 方法都能获取到非空的文档, 则最终使用后者.

Python based dynamic libraries can also specify the general library
documentation directly in the code as the docstring of the library
class and its `__init__` method. If a non-empty documentation is
got both directly from the code and from the
`get_keyword_documentation` method, the latter has precedence.

Named argument syntax with dynamic libraries
动态库中的命名参数语法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

从Robot Framework 2.8版本开始, 动态库API开始支持 `命名参数语法`_.  使用该语法基于使用 `get_keyword_arguments` 获取到的参数名称和缺省值.

Starting from Robot Framework 2.8, also the dynamic library API supports
the `named argument syntax`_. Using the syntax works based on the
argument names and default values `got from the library`__ using the
`get_keyword_arguments` method.

大部分情况下, 动态关键字的命名参数语法和其它关键字的没什么区别. 唯一的例外是当关键字有多个参数有缺省值, 而只有后面的几个传了值时, 此时框架会将略过的可选参数按照 `get_keyword_arguments` 中返回的缺省值进行赋值.

For the most parts, the named arguments syntax works with dynamic keywords
exactly like it works with any other keyword supporting it. The only special
case is the situation where a keyword has multiple arguments with default
values, and only some of the latter ones are given. In that case the framework
fills the skipped optional arguments based on the default values returned
by the `get_keyword_arguments` method.

动态库中使用命名参数语法的例子见下面. 所有的例子都使用了关键字  :name:`Dynamic`, 该关键字的参数规范是 `[arg1, arg2=xxx, arg3=yyy]`.
注释部分是调用该关键字的入参.

Using the named argument syntax with dynamic libraries is illustrated
by the following examples. All the examples use a keyword :name:`Dynamic`
that has been specified to have argument specification
`[arg1, arg2=xxx, arg3=yyy]`.
The comment shows the arguments that the keyword is actually called with.

.. sourcecode:: robotframework

   *** Test Cases ***
   Only positional
       Dynamic    a                             # [a]
       Dynamic    a         b                   # [a, b]
       Dynamic    a         b         c         # [a, b, c]

   Named
       Dynamic    a         arg2=b              # [a, b]
       Dynamic    a         b         arg3=c    # [a, b, c]
       Dynamic    a         arg2=b    arg3=c    # [a, b, c]
       Dynamic    arg1=a    arg2=b    arg3=c    # [a, b, c]

   Fill skipped
       Dynamic    a         arg3=c              # [a, xxx, c]

__ `Getting keyword arguments`_

Free keyword arguments with dynamic libraries
动态库里的 Free keyword arguments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

从Robot Framework 2.8.2版本开始， 动态库也可以支持 `free keyword arguments`_ (`**kwargs`). 一个必须的前提条件是  `run_keyword` 方法必须接受三个参数. 其中第3个参数被用来接受kwargs. kwargs在Python中作为字典, 在Java中使用Map传递给关键字.

Starting from Robot Framework 2.8.2, dynamic libraries can also support
`free keyword arguments`_ (`**kwargs`). A mandatory precondition for
this support is that the `run_keyword` method `takes three arguments`__:
the third one will get kwargs when they are used. Kwargs are passed to the
keyword as a dictionary (Python) or Map (Java).

一个关键字接受什么参数取决于 `get_keyword_arguments` 返回的结果. 如果最后返回的参数是以 `**` 开头, 则表示这个关键字可以接受kwargs.

What arguments a keyword accepts depends on what `get_keyword_arguments`
`returns for it`__. If the last argument starts with `**`, that keyword is
recognized to accept kwargs.

下面的例子演示了动态库使用kwargs的情况. 所有的例子都使用了关键字 :name:`Dynamic`, 该关键字被设定的参数规范为 `[arg1=xxx, arg2=yyy, **kwargs]`.
注释部分是调用该关键字的入参.

Using the free keyword argument syntax with dynamic libraries is illustrated
by the following examples. All the examples use a keyword :name:`Dynamic`
that has been specified to have argument specification
`[arg1=xxx, arg2=yyy, **kwargs]`.
The comment shows the arguments that the keyword is actually called with.

.. sourcecode:: robotframework

   *** Test Cases ***
   No arguments
       Dynamic                            # [], {}

   Only positional
       Dynamic    a                       # [a], {}
       Dynamic    a         b             # [a, b], {}

   Only kwargs
       Dynamic    a=1                     # [], {a: 1}
       Dynamic    a=1       b=2    c=3    # [], {a: 1, b: 2, c: 3}

   Positional and kwargs
       Dynamic    a         b=2           # [a], {b: 2}
       Dynamic    a         b=2    c=3    # [a], {b: 2, c: 3}

   Named and kwargs
       Dynamic    arg1=a    b=2           # [a], {b: 2}
       Dynamic    arg2=a    b=2    c=3    # [xxx, a], {b: 2, c: 3}

__ `Running dynamic keywords`_
__ `Getting keyword arguments`_

总结
^^^^^^^

动态库API中的所有特殊方法都列在下表中. 方法名使用了下划线的格式, 但是驼峰命名法同样也可以.

All special methods in the dynamic API are listed in the table
below. Method names are listed in the underscore format, but their
camelCase aliases work exactly the same way.

.. table:: All special methods in the dynamic API
   :class: tabular

   ===========================  =========================  =======================================================
               Name                    Arguments                                  Purpose
   ===========================  =========================  =======================================================
   `get_keyword_names`                                     `Return names`__ of the implemented keywords.
   `run_keyword`                `name, arguments, kwargs`  `Execute the specified keyword`__ with given arguments. `kwargs` is optional.
   `get_keyword_arguments`      `name`                     Return keywords' `argument specifications`__. Optional method.
   `get_keyword_documentation`  `name`                     Return keywords' and library's `documentation`__. Optional method.
   ===========================  =========================  =======================================================

__ `Getting dynamic keyword names`_
__ `Running dynamic keywords`_
__ `Getting keyword arguments`_
__ `Getting keyword documentation`_

如果使用Java, 可以像下面这样正式的声明接口. 不过, *不需要* 这样显示的接口, 因为 Robot Framework 是使用反射直接检测类是否实现了必需的 `get_keyword_names` 和 `run_keyword` 方法(或者以驼峰命名的别名). 另外, `get_keyword_arguments` 和 `get_keyword_documentation` 完全是可选的.

It is possible to write a formal interface specification in Java as
below. However, remember that libraries *do not need* to implement
any explicit interface, because Robot Framework directly checks with
reflection if the library has the required `get_keyword_names` and
`run_keyword` methods or their camelCase aliases. Additionally,
`get_keyword_arguments` and `get_keyword_documentation`
are completely optional.

.. sourcecode:: java

   public interface RobotFrameworkDynamicAPI {

       List<String> getKeywordNames();

       Object runKeyword(String name, List arguments);

       Object runKeyword(String name, List arguments, Map kwargs);

       List<String> getKeywordArguments(String name);

       String getKeywordDocumentation(String name);

   }

.. note:: 除了使用 `List`, 还可以使用数组, 如 `Object[]` 或 `String[]`.


.. note:: In addition to using `List`, it is possible to use also arrays
          like `Object[]` or `String[]`.

使用动态API的一个很好的例子是Robot Framework自带的 `Remote library`_.

A good example of using the dynamic API is Robot Framework's own
`Remote library`_.

混合库API
------------------

顾名思义, 混合库API是介于静态API和动态API之间的混合. 和动态API一样, 混合API只能以类的方式实现.


The hybrid library API is, as its name implies, a hybrid between the
static API and the dynamic API. Just as with the dynamic API, it is
possible to implement a library using the hybrid API only as a class.

Getting keyword names
获取关键字名称
^^^^^^^^^^^^^^^^^^^^^

关键字的名称获取和动态API一样. 库需要有 `get_keyword_names` 或 `getKeywordNames` 方法来返回关键字名称的列表.

Keyword names are got in the exactly same way as with the dynamic
API. In practice, the library needs to have the
`get_keyword_names` or `getKeywordNames` method returning
a list of keyword names that the library implements.

Running keywords
运行关键字
^^^^^^^^^^^^^^^^

混合API中没有用来执行关键字的 `run_keyword` 方法. Robot Framework利用反射来查找实现关键字的方法, 这一点和静态API类似. 使用混合API实现的库既可以自己直接实现这些方法, 或者, 更重要的是, 它还可以动态的处理.

In the hybrid API, there is no `run_keyword` method for executing
keywords. Instead, Robot Framework uses reflection to find methods
implementing keywords, similarly as with the static API. A library
using the hybrid API can either have those methods implemented
directly or, more importantly, it can handle them dynamically.

使用Python时, 可以很简单的使用 `__getattr__` 来动态处理找不到的方法. 对于大多数Python程序员来说, 这个特殊方法应该很熟悉了, 所以也应该能立即明白下面的示例. 如果还不太明白的人, 可以先参考 `Python Reference Manual`__.

In Python, it is easy to handle missing methods dynamically with the
`__getattr__` method. This special method is probably familiar
to most Python programmers and they can immediately understand the
following example. Others may find it easier to consult `Python Reference
Manual`__ first.

__ http://docs.python.org/reference/datamodel.html#attribute-access

.. sourcecode:: python

   from somewhere import external_keyword

   class HybridExample:

       def get_keyword_names(self):
           return ['my_keyword', 'external_keyword']

       def my_keyword(self, arg):
           print "My Keyword called with '%s'" % arg

       def __getattr__(self, name):
           if name == 'external_keyword':
               return external_keyword
           raise AttributeError("Non-existing attribute '%s'" % name)

注意到 `__getattr__` 并不像 `run_keyword` 那样实际执行这个关键字, 它只是返回一个可调用的对象, 最终这个对象被Robot Framework调用执行.

Note that `__getattr__` does not execute the actual keyword like
`run_keyword` does with the dynamic API. Instead, it only
returns a callable object that is then executed by Robot Framework.

另一点需要注意的是, Robot Framework将使用 `get_keyword_names` 返回的名称来查找方法. 也就是说实际的方法名必须和返回的方法名称一致. 例如, 上面的例子中, 如果 `get_keyword_names` 返回的是 `My Keyword` 而不是 `my_keyword` 的话, 最终执行结果会是找不到相应的方法.

Another point to be noted is that Robot Framework uses the same names that
are returned from `get_keyword_names` for finding the methods
implementing them. Thus the names of the methods that are implemented in
the class itself must be returned in the same format as they are
defined. For example, the library above would not work correctly, if
`get_keyword_names` returned `My Keyword` instead of
`my_keyword`.

混合API对Java来说没太大作用, 因为Java没有办法动态处理找不到的方法. 当然, 可以在库的类中实现所有的方法, 但是那样就跟静态API相比没什么优势了.

The hybrid API is not very useful with Java, because it is not
possible to handle missing methods with it. Of course, it is possible
to implement all the methods in the library class, but that brings few
benefits compared to the static API.

Getting keyword arguments and documentation
获取关键字参数和文档
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当使用混合API时, Robot Framework 使用反射来查找实现关键字的方法, 这一点和静态API类似. 当找到方法的引用后, 也可以像静态API一样直接查找该方法的参数定义和文档. 所以, 就没必要和动态API那样存在另外的特殊方法.

When this API is used, Robot Framework uses reflection to find the
methods implementing keywords, similarly as with the static API. After
getting a reference to the method, it searches for arguments and
documentation from it, in the same way as when using the static
API. Thus there is no need for special methods for getting arguments
and documentation like there is with the dynamic API.

总结
^^^^^^^

当使用Python来开发测试库时, 混合API和动态API一样拥有动态的能力. 同时一个巨大的好处是无需要使用特殊方法来获取参数和文档. 
将真正动态的关键字交由 `__getattr__` 处理, 而将其它的都直接在主类中实现, 是一种很实用的做法.

When implementing a test library in Python, the hybrid API has the same
dynamic capabilities as the actual dynamic API. A great benefit with it is
that there is no need to have special methods for getting keyword
arguments and documentation. It is also often practical that the only real
dynamic keywords need to be handled in `__getattr__` and others
can be implemented directly in the main library class.

由于这种清楚的好处, 并且同等的能力, 在使用Python开发时, 混合API在大多数情况下是相对动态API的更好的选择. 一个值得注意的特例是实现一个代理库的情况, 因为真实的关键字必须要在某处被执行, 这个代理只能向前传递关键字名和参数.

Because of the clear benefits and equal capabilities, the hybrid API
is in most cases a better alternative than the dynamic API when using
Python. One notable exception is implementing a library as a proxy for
an actual library implementation elsewhere, because then the actual
keyword must be executed elsewhere and the proxy can only pass forward
the keyword name and arguments. 

使用混合API的一个很好的例子是Robot Framework自带的 Telnet_ 库.

A good example of using the hybrid API is Robot Framework's own
Telnet_ library.

Using Robot Framework's internal modules
使用Robot Framework的内置模块
----------------------------------------

Test libraries implemented with Python can use Robot Framework's
internal modules, for example, to get information about the executed
tests and the settings that are used. This powerful mechanism to
communicate with the framework should be used with care, though,
because all Robot Framework's APIs are not meant to be used by
externally and they might change radically between different framework
versions.

Available APIs
可用的API
^^^^^^^^^^^^^^

从Robot Framework 2.7版本开始, `API文档`_ 单独部署在 `Read the Docs`_ 服务上. 如果你不确定如何使用某个特定的API, 请在 `邮件列表`_ 里提问.

Starting from Robot Framework 2.7, `API documentation`_ is hosted separately
at the excellent `Read the Docs`_ service. If you are unsure how to use
certain API or is using them forward compatible, please send a question
to `mailing list`_.

Using BuiltIn library
使用内置库
^^^^^^^^^^^^^^^^^^^^^

可使用的最安全的API莫过于 BuiltIn_ 库里的关键字方法. 这些关键字极少变动, 并且每次变动前先将老的用法废弃掉. 其中一个最有用的方法是 `replace_variables`, 它允许访问当前可用的变量. 
下面的例子说明了怎样获取一个很有用的 `自动变量`_ `${OUTPUT_DIR}` 的值. 而且还可以在库中使用 `set_test_variable`, `set_suite_variable` 和 `set_global_variable` 来设置新的变量.

The safest API to use are methods implementing keywords in the
BuiltIn_ library. Changes to keywords are rare and they are always
done so that old usage is first deprecated. One of the most useful
methods is `replace_variables` which allows accessing currently
available variables. The following example demonstrates how to get
`${OUTPUT_DIR}` which is one of the many handy `automatic
variables`_. It is also possible to set new variables from libraries
using `set_test_variable`, `set_suite_variable` and
`set_global_variable`.

.. sourcecode:: python

   import os.path
   from robot.libraries.BuiltIn import BuiltIn

   def do_something(argument):
       output = do_something_that_creates_a_lot_of_output(argument)
       outputdir = BuiltIn().replace_variables('${OUTPUTDIR}')
       path = os.path.join(outputdir, 'results.txt')
       f = open(path, 'w')
       f.write(output)
       f.close()
       print '*HTML* Output written to <a href="results.txt">results.txt</a>'

使用 `BuiltIn` 中的方法唯一需要注意的一点, 所有通过 `run_keyword` 执行的方法需要特殊处理一下, 它们必须先调用 `BuiltIn` 模块中的 `register_run_keyword` 方法注册为 *run keywords*. 具体如何使用以及为什么要这么做, 请参阅 `register_run_keyword` 方法的文档说明.

The only catch with using methods from `BuiltIn` is that all
`run_keyword` method variants must be handled specially.
Methods that use `run_keyword` methods have to be registered
as *run keywords* themselves using `register_run_keyword`
method in `BuiltIn` module. This method's documentation explains
why this needs to be done and obviously also how to do it.

.. Extending existing test libraries
扩展已有的测试库
---------------------------------

本章将介绍几种不同的方法来为已有的测试库添加新功能, 这个测试库可以是第三方发布的, 也可以是自己开发的.

This section explains different approaches how to add new
functionality to existing test libraries and how to use them in your
own libraries otherwise.

.. Modifying original source code
修改原始源代码
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果可以直接访问要扩展的测试库的源代码, 直接修改源码是很自然的选择. 该方法的最大问题是, 如果原库代码有升级, 会很难保证不影响到修改的地方. 对于用户来说, 使用相对原库包含了不同功能的测试库, 也容易造成混乱. 同时, 重新打包该库也可能会是个艰巨的任务.

If you have access to the source code of the library you want to
extend, you can naturally modify the source code directly. The biggest
problem of this approach is that it can be hard for you to update the
original library without affecting your changes. For users it may also
be confusing to use a library that has different functionality than
the original one. Repackaging the library may also be a big extra
task.

如果改动和增强是通用的, 并且计划提交给原库的开发者, 那么这种方式会是很好的选择. 如果你的改动被接受了, 并且会包含在未来发布的新版本中, 则上面讨论的所有问题都不存在了. 如果这种改动不那么通用, 或者因为其它原因不能提交, 则使用下面章节中的方法可能会更好.

This approach works extremely well if the enhancements are generic and
you plan to submit them back to the original developers. If your
changes are applied to the original library, they are included in the
future releases and all the problems discussed above are mitigated. If
changes are non-generic, or you for some other reason cannot submit
them back, the approaches explained in the subsequent sections
probably work better.

Using inheritance
使用继承
^^^^^^^^^^^^^^^^^

另一个直接的方式是使用继承来扩展已有库. 该方式由下面的例子说明, 为 SeleniumLibrary_ 添加新的关键字 :name:`Title Should Start With`. 本例中使用Python, 但是同样适用于使用Java代码扩展Java开发的库.

Another straightforward way to extend an existing library is using
inheritance. This is illustrated by the example below that adds new
:name:`Title Should Start With` keyword to the SeleniumLibrary_. This
example uses Python, but you can obviously extend an existing Java
library in Java code the same way.

.. sourcecode:: python

   from SeleniumLibrary import SeleniumLibrary

   class ExtendedSeleniumLibrary(SeleniumLibrary):

       def title_should_start_with(self, expected):
           title = self.get_title()
           if not title.startswith(expected):
               raise AssertionError("Title '%s' did not start with '%s'"
                                    % (title, expected))

相较于直接修改原库代码, 这种方式最大的不同在于, 修改后的新的测试库有一个新的名字. 这样就清晰的告诉别人这是一个自定义的库. 但是一个大问题是, 这两个库将很难同时使用. 首先这两个库的同名关键字会产生 `冲突`__, 另外, 这两个库并没有共享状态(??).

A big difference with this approach compared to modifying the original
library is that the new library has a different name than the
original. A benefit is that you can easily tell that you are using a
custom library, but a big problem is that you cannot easily use the
new library with the original. First of all your new library will have
same keywords as the original meaning that there is always
conflict__. Another problem is that the libraries do not share their
state.

如果你想要从头使用一个新的测试库并且添加自定义的扩展, 这种方式是个不错的选择. 否则, 请继续参考其它章节介绍的方法.

This approach works well when you start to use a new library and want
to add custom enhancements to it from the beginning. Otherwise other
mechanisms explained in this section are probably better.

__ `Handling keywords with same names`_

Using other libraries directly
直接使用其它库
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

因为测试库本质上无非就是类或者模块, 一个简单的方式就是引入这个库, 直接调用其中的方法. 这种方式对于那些静态的且不依赖于库状态的方法很有用. 
前面 `Robot Framework内置库`__ 的例子已经说明了如何使用这种方式.

Because test libraries are technically just classes or modules, a
simple way to use another library is importing it and using its
methods. This approach works great when the methods are static and do
not depend on the library state. This is illustrated by the earlier
example that uses `Robot Framework's BuiltIn library`__.

如果这个库是有状态的, 那么事情可能会不如人意. 因为你的代码中使用的这个库的实例, 和框架使用的并不完全一样, 通过执行关键字产生的状态变化在你的库中是不可见的.
下一节内容说明了如何获取和框架使用的相同的库实例.

If the library has state, however, things may not work as you would
hope.  The library instance you use in your library will not be the
same as the framework uses, and thus changes done by executed keywords
are not visible to your library. The next section explains how to get
an access to the same library instance that the framework uses.

__ `Using Robot Framework's internal modules`_

Getting active library instance from Robot Framework
从框架获取活动的库实例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

内置关键字 :name:`Get Library Instance` 可以用来获取当前活动的测试库的实例. 该关键字返回的实例与框架自己使用的实例完全一样, 这样就没有状态不一致的问题. 虽然该功能可通过关键字使用, 但是在测试库中更典型的用法是import :name:`BuiltIn` 类.
下面的例子说明了如何用该方法实现 `使用继承`_ 例子中实现的关键字 :name:`Title Should Start With`.

BuiltIn_ keyword :name:`Get Library Instance` can be used to get the
currently active library instance from the framework itself. The
library instance returned by this keyword is the same as the framework
itself uses, and thus there is no problem seeing the correct library
state. Although this functionality is available as a keyword, it is
typically used in test libraries directly by importing the :name:`BuiltIn`
library class `as discussed earlier`__. The following example illustrates
how to implement the same :name:`Title Should Start With` keyword as in
the earlier example about `using inheritance`_.

__ `Using Robot Framework's internal modules`_

.. sourcecode:: python

   from robot.libraries.BuiltIn import BuiltIn

   def title_should_start_with(expected):
       seleniumlib = BuiltIn().get_library_instance('SeleniumLibrary')
       title = seleniumlib.get_title()
       if not title.startswith(expected):
           raise AssertionError("Title '%s' did not start with '%s'"
                                % (title, expected))

当测试库有状态时, 这种方式显然比直接引入和使用更好. 相比继承的方式, 最大的好处是可以正常的使用原库, 新的库作为扩展只在需要的时候再用.
下面的例子演示了上例中的代码是如何通过新库 :name:`SeLibExtensions` 提供使用.

This approach is clearly better than importing the library directly
and using it when the library has a state. The biggest benefit over
inheritance is that you can use the original library normally and use
the new library in addition to it when needed. That is demonstrated in
the example below where the code from the previous examples is
expected to be available in a new library :name:`SeLibExtensions`.

.. sourcecode:: robotframework

   *** Settings ***
   Library    SeleniumLibrary
   Library    SeLibExtensions

   *** Test Cases ***
   Example
       Open Browser    http://example      # SeleniumLibrary
       Title Should Start With    Example  # SeLibExtensions

Libraries using dynamic or hybrid API
扩展使用动态或混合API的库
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用了 动态API__ 或者 `混合库API`_ 的测试库往往都有自己的扩展方式. 想要扩展这些库, 需要咨询库的开发者或者参考库的文档或者源代码.

Test libraries that use the dynamic__ or `hybrid library API`_ often
have their own systems how to extend them. With these libraries you
need to ask guidance from the library developers or consult the
library documentation or source code.

__ `dynamic library API`_
