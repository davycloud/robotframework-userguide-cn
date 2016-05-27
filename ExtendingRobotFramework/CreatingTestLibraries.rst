创建测试库
=======================

Robot Framework 实际的测试能力都是由测试库提供. 目前已经有很多库存在, 其中甚至有些已经与框架绑定, 但是很多时候仍然需要创建新的测试库. 
得益于Robot Framework简单直接的API, 这一过程并不复杂.

.. contents::
   :depth: 2
   :local:

概述
------------

支持的编程语言
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Robot Framework 自身使用 Python_ 编写, 很自然的, 扩展它的测试库也可以使用相同的语言. 
当使用 Jython_ 运行时, 也可以使用 Java_ 来实现测试库. 
纯的Python代码, 只要其没有使用Jython不支持的模块或语法, 在两种情况下都可以运行. 
当使用Python时, 可以利用 `Python C API`__ 使用C语言来实现测试库, 当然, 使用Python的 ctypes__ 模块调用C代码则更简单.

使用这些原生支持的语言实现的测试库可以同时作为"包装器", 调用其他语言实现的
功能. 一个典型的例子就是 `远程库`_ 的实现. 当然还可以另起单独的进程来执行
外部脚本或者工具.


Libraries implemented using these natively supported languages can
also act as wrappers to functionality implemented using other
programming languages. A good example of this approach is the `Remote
library`_, and another widely used approaches is running external
scripts or tools as separate processes.

.. tip:: `Python Tutorial for Robot Framework Test Library Developers`__
         covers enough of Python language to get started writing test
         libraries using it. It also contains a simple example library
         and test cases that you can execute and otherwise investigate
         on your machine.

__ http://docs.python.org/c-api/index.html
__ http://docs.python.org/library/ctypes.html
__ http://code.google.com/p/robotframework/wiki/PythonTutorial

不同的测试库API
^^^^^^^^^^^^^^^^^^^^^^^^^^

Robot Framework 有三种不同的测试库API.

静态(Static) API

  最简单的办法是实现一个模块(用Python), 或者类(用Python或Java), 其中的
  方法(methods)直接映射为 `关键字名称`_. 关键字接受和方法相同的 `参数`_,
  可以通过抛异常来报告错误, 可以通过往标注输出里写入来写 `log`_, 同时可以
  通过 `return` 来返回结果.

  The simplest approach is having a module (in Python) or a class
  (in Python or Java) with methods which map directly to
  `keyword names`_. Keywords also take the same `arguments`__ as
  the methods implementing them.  Keywords `report failures`__ with
  exceptions, `log`__ by writing to standard output and can `return
  values`__ using the `return` statement.

动态(Dynamic) API
  
  动态库是指一些类, 其中实现了一个方法, 用于获取关键字的名称. 其它的方法则
  实现这个具体的关键字, 并可接受参数. 这样, 要实现的关键字的名称, 可以在
  运行时动态决定. 其它功能, 如报告状态, 打印日志和返回结果都和静态API类似.

  Dynamic libraries are classes that implement a method to get the names
  of the keywords they implement, and another method to execute a named
  keyword with given arguments. The names of the keywords to implement, as
  well as how they are executed, can be determined dynamically at
  runtime, but reporting the status, logging and returning values is done
  similarly as in the static API.

混合(Hybrid) API
  
  静态API和动态API的混合. 类的一个方法用来说明实现了哪些关键字, 这些关键字是
  直接可用的.  除了自动发现关键字的实现, 其它功能都和静态API类似.

  This is a hybrid between the static and the dynamic API. Libraries are
  classes with a method telling what keywords they implement, but
  those keywords must be available directly. Everything else except
  discovering what keywords are implemented is similar as in the
  static API.

所有这些API都将在本章进行详述. 所有功能都是基于静态API的工作原理, 所以首先
讨论静态API的内容. `动态库API`_ 和 `混合库API`_ 与之的区别在随后的小节中
分别讨论.

本章示例大部分都是Python实现, 但是对于Java开发者来说, 理解这些代码并不难.
在某些少数情况下, Python API和Java API有所差异, 此时会分别解释.

__ `Keyword arguments`_
__ `Reporting keyword status`_
__ `Logging information`_
__ `Returning values`_

创建测试库类或模块
-------------------------------------

测试库的实现可以是Python模块, 也可以是Python或Java的类.

测试库名称
^^^^^^^^^^^^^^

测试库引入(import)时需要指定名称, 这个名称和实现测试库的模块名或类名一样.
举个例子, 如果有个Python模块 `MyLibrary` (文件是 :file:`MyLibrary.py`),
这就可作为一个测试库: :name:`MyLibrary`. 类似的, 一个不在任何包里面的Java
类 `YourLibrary`, 就是一个同名的测试库.

Python的类总是在模块里, 如果模块里实现的类的名称和模块名一致, Robot Framework
允许不带类名来引用这个库. 例如, 有一个类 `MyLib` 在文件 :file:`MyLib.py` 中,
引用库时只需使用名称 :name:`MyLib` 即可. 这个机制对于子模块同样有效, 如, 
`parent.MyLib` 模块中有个类 `MyLib`, 使用  :name:`parent.MyLib` 即可导入.
但是, 如果模块名和类名不一样, 则必须同时指明, 如 :name:`mymodule.MyLibrary` 
或者 :name:`parent.submodule.MyLib`.

非默认包里的Java类也必须指定全名, 例如, 包 `com.mycompany.myproject` 里的
类 `MyLib` 导入名称是: :name:`com.mycompany.myproject.MyLib`.

.. note:: 在同名模块中忽略类名只在 Robot Framework 2.8.4 及之后的版本有效.
          老的版本中仍然需要指定全名, 如 :name:`parent.MyLib.MyLib`.



.. note:: Dropping class names with submodules works only in Robot Framework
          2.8.4 and newer. With earlier versions you need to include also
          the class name like :name:`parent.MyLib.MyLib`.

.. tip:: If the library name is really long, for example when the Java
         package name is long, it is recommended to give the library a
         simpler alias by using the `WITH NAME syntax`_.

给测试库提供参数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

实现为类的测试库都可以接受参数. 这些参数在Setting表中指定, 跟在库名后面,
当Robot Framework创建测试库的实例时, 这些参数传给构造函数. 

实现为模块的测试库不可以接受参数, 试图给其传递参数的行为将导致报错.

库所需的参数的个数和库的构造函数的参数个数一样. 默认参数和不定数量参数的处理
类似于`keyword arguments`_. Java库不支持不定数量的参数. 

传递给库的参数, 包括库名本身, 都可以使用变量. 也就是说, 可以在命令行中修改.
例如:

The number of arguments needed by the library is the same
as the number of arguments accepted by the library's
constructor. The default values and variable number of arguments work
similarly as with `keyword arguments`_, with the exception that there
is no variable argument support for Java libraries. Arguments passed
to the library, as well as the library name itself, can be specified
using variables, so it is possible to alter them, for example, from the
command line.

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

测试库的作用域
^^^^^^^^^^^^^^^^^^^^^

用类实现的库可以有内部状态, 这些状态可以被关键字或构造函数修改. 因为这些状态
会影响到关键字实际的行为, 所以, 保证一个测试用例不会意外地影响到另一个用例显
得非常重要. 这种依赖行为有可能造成非常难定位的bug. 例如, 添加了新的测试用例,
而这些用例使用库的方式并不一致.

Robot Framework 为了保证测试用例之间的独立性, 默认情况下, 它为每个测试用例
创建新的测试库实例. 然而, 这种方式不总是我们想要的, 比如有时测试用例需要共享
某个状态的时候. 此外, 那些无状态的库显然也不需要每次都创建新实例.

实例化测试库类的方式可以通过一个特别的属性  `ROBOT_LIBRARY_SCOPE` 来控制.
这个属性是一个字符串, 可以有以下三种取值:

`TEST CASE`
  为每个测试用例创建新的实例. 如果有suite setup和suite teardown的话, 同样
  也会新建. 这是默认行为.

`TEST SUITE`
  为每个测试集创建新的实例. 最底层的测试集, 也就是由测试用例文件组成的测试集,
  拥有属于自己的测试库实例, 高层的测试集, 都有属于自己的测试库实例.

`GLOBAL`
  整个测试执行过程中只有一个实例被创建. 所有的测试集和测试用例共享这个实例.
  通过模块创建的测试库都是全局的.

.. note:: 如果一个测试库被导入多次, 每次使用不同的参数, 则不管有没有定义
          作用域, 每次都会新建一个实例.

当有状态的测试库定义了作用域为 `TEST SUITE` 或 `GLOBAL` , 建议测试库要
包含能清除这些状态的关键字. 这样, 在测试集 setup 或 teardown 时, 可以
调用这些关键字以保证下面的测试用例从一个明确的已知状态开始.

例如, :name:`SeleniumLibrary` 使用了 `GLOBAL` 作用域, 使得不同的测试
用例使用相同的浏览器, 而不是每次重新打开. 同时, 它还提供了关键字
:name:`Close All Browsers` 关闭所有浏览器.

下面是使用了 `TEST SUITE` 作用域的Python库的示例:

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

使用了 `GLOBAL` 作用域的Java库的示例:

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

__ `Providing arguments to test libraries`_

指定测试库的版本
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当一个测试库投入使用, Robot Framework 会尝试获取它的版本号, 将该信息
写入到 日志_ 中以供调试. 库文档工具 Libdoc_ 在生成文档时也会写入该信息.

版本号信息在属性 `ROBOT_LIBRARY_VERSION` 中定义, 类似 `测试库的作用域`_ 
中的 `ROBOT_LIBRARY_SCOPE`. 如果 `ROBOT_LIBRARY_VERSION` 属性不存在,
则会尝试从 `__version__` 属性获取. 这些属性必须是类或者模块的属性.
对于Java库, 这个属性必须声明为 `static final`.

使用 `__version__` 的Python模块示例:

.. sourcecode:: python

    __version__ = '0.1'

    def keyword():
        pass

使用 `ROBOT_LIBRARY_VERSION` 的Java类示例:

.. sourcecode:: java

    public class VersionExample {

        public static final String ROBOT_LIBRARY_VERSION = "1.0.2";

        public void keyword() {
        }
    }

指定文档格式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

从 Robot Framework 2.7.5版本开始, 库文档工具 Libdoc_ 开始支持多种格式.
如果不想使用 Robot Framework's 自己的 `文档格式`_, 可以通过在源码中定义
属性 `ROBOT_LIBRARY_DOC_FORMAT` 来指定格式, 就跟指定 作用域__ 和 版本__ 
一样.

文档格式可指定的值包括: `ROBOT` (默认), `HTML`, `TEXT` (纯文本),
和 `reST` (reStructuredText_). 这些值不区分大小写. 如果要使用 `reST`
格式需要安装 docutils_ 模块.

下面使用Python和Java设置文档格式的例子分别使用了 reStructuredText 和 HTML
格式. 关于给测试库写文档的更多内容, 请参考 `Documenting libraries`_ 和 
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

__ `测试库的作用域`_
__ `指定测试库的版本`_


作为监听器的测试库 Library acting as listener
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`监听器接口`_ 可以让外部的监听器在测试执行过程中得到关于执行状态的通知.
例如, 当测试集, 测试用例和关键字开始执行或结束. 有时候这些通知对测试库
来说很有用, 可以使用 `ROBOT_LIBRARY_LISTENER` 注册一个自定义的监听器.
该属性的值应该是要使用的监听器的示例, 有可能是测试库本身. 更多内容和示例,
请参考 `Test libraries as listeners`_ section.

创建静态关键字
------------------------

什么方法被认作关键字 What methods are considered keywords
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当使用静态库API时, Robot Framework 利用反射机制获得类或模块实现的公有\
方法(public methods). 所有以下划线(_)开始的方法被排除, 在Java库里, 只
在 `java.lang.Object` 里实现的方法也被忽略. 所有没被忽略的方法都被视为
关键字. 例如, 下面的Python和Java示例实现了关键字  :name:`My Keyword`.

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

当库是Python模块实现的, 可以使用Python的 `__all__` 属性来限制到底
哪些方法是关键字. 如果使用了 `__all__`, 只有列在其中的才会被当作
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
^^^^^^^^^^^^

测试数据(test data)中使用的关键字名称, 与方法名称对比, 最终确定是哪个
方法实现了这个关键字. 名称的比较是忽略大小写的, 并且其中的空格和下划线
也都忽略掉. 例如, 方法名 `hello` 最终可以映射为的关键字名称可以是:
:name:`Hello`, :name:`hello` 甚至 :name:`h e l l o`. 反过来也类似,
例如, `do_nothing` 和 `doNothing` 这两个方法都可被当作 :name:`Do Nothing`
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
位置在 `module search path`_.

.. sourcecode:: robotframework

   *** Settings ***
   Library    MyLibrary

   *** Test Cases ***
   My Test
       Do Nothing
       Hello    world

使用自定义的关键字名称
Using a custom keyword name
'''''''''''''''''''''''''''

如果一个方法不想使用默认映射的关键字名称, 也可以明确指定为自定义的关键字名称.
这是通过设置方法的  `robot_name` 属性来实现的. 可以使用装饰器方法
 `robot.api.deco.keyword` 方便快捷的设置. 例如:

.. sourcecode:: python

  from robot.api.deco import keyword

  @keyword('Login Via User Panel')
  def login(username, password):
      # ...

.. sourcecode:: robotframework

   *** Test Cases ***
   My Test
       Login Via User Panel    ${username}    ${password}

如果使用装饰器时不带任何参数, 则这个装饰器对改变关键字名称不起作用, 但是仍然
会创建  `robot_name` 属性. 这种情况对 `标记方法为关键字`_ , 同时又不改变
关键字名称的时候很有用. 

Using this decorator without an argument will have no effect on the exposed
keyword name, but will still create the `robot_name` attribute.  This can be useful
for `Marking methods to expose as keywords`_ without actually changing
keyword names.

设置自定义的关键字名称还使得库关键字可以接受使用 `嵌入参数`__ 语法的参数.

__ `Embedding arguments into keyword names`_

关键字标签
^^^^^^^^^^^^^^^

从 Robot Framework 2.9 版本开始, 库关键字和  `用户关键字`__ 可以有标签.
库关键字通过设置方法的 `robot_tags` 属性实现, 该属性的值是要设置的标签的列表.
装饰器 `robot.api.deco.keyword` 可以按如下的方法来方便的指定这个属性:

.. sourcecode:: python

  from robot.api.deco import keyword

  @keyword(tags=['tag1', 'tag2'])
  def login(username, password):
      # ...

  @keyword('Custom name', ['tags', 'here'])
  def another_example():
      # ...

设置标签的另一个方法是在 `关键字文档`__ 的最后一行给出, 以 `Tags:` 作为前缀
开始, 后面跟着按逗号分开的标签. 例如: 

.. sourcecode:: python

  def login(username, password):
      """Log user in to SUT.

      Tags: tag1, tag2
      """
      # ...

__ `User keyword tags`_
__ `Documenting libraries`_

关键字参数
~~~~~~~~~~~~~~~~~

对于静态和混合API, 关于一个关键字的参数表信息是直接从实现它的方法上获取的.
而使用了 `动态库API`_ 的库则使用其它的方式来传递这些信息, 所以本章不适用
于动态库.

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

.. note:: 使用静态库API实现的Java库有一个很大的限制, 即不支持 `命名参数语法`_. 如果
          你觉得这是一个障碍, 那要么改使用Python来实现, 要么切换到 `动态库API`_

关键字的缺省值
Default values to keywords
~~~~~~~~~~~~~~~~~~~~~~~~~~

和函数类似, 关键字的有些参数有时需要有缺省值. Python 和 Java 对于处理方法的缺省值
使用不同的语法, 

It is often useful that some of the arguments that a keyword uses have
default values. Python and Java have different syntax for handling default
values to methods, and the natural syntax of these languages can be
used when creating test libraries for Robot Framework.

Python中的缺省值
Default values with Python
''''''''''''''''''''''''''

Python中, 方法总是只有一个实现, 但是方法的签名中可能会指定缺省值.
这种语法对Python程序员来说应该非常熟悉, 例如:

In Python a method has always exactly one implementation and possible
default values are specified in the method signature. The syntax,
which is familiar to all Python programmers, is illustrated below:

.. sourcecode:: python

   def one_default(arg='default'):
       print "Argument has value %s" % arg

   def multiple_defaults(arg1, arg2='default 1', arg3='default 2'):
       print "Got arguments %s, %s and %s" % (arg1, arg2, arg3)

上例中的第一个关键字, 可以接受 0 个或 1 个参数, 当 0 个参数时, 参数 `arg`
使用缺省值 `default`; 当有 1 个参数时, 参数 `arg` 就使用这个传入的值; 
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

Java中的缺省值
Default values with Java
''''''''''''''''''''''''

Java中, 一个方法可以有多个实现, 分别是不同的签名. Robot Framework 将所有
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

可变数量的参数
Variable number of arguments (`*varargs`)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robot Framework 还支持接受任何数量的参数的关键字. 类似于缺省值,
实际的语法在Python和Java中有所差异.

Robot Framework supports also keywords that take any number of
arguments. Similarly as with the default values, the actual syntax to use
in test libraries is different in Python and Java.

Python中的可变数量的参数
''''''''''''''''''''''''''''''''''''''''

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

Java中的可变数量的参数
''''''''''''''''''''''''''''''''''''''
Robot Framework 支持 `Java可变数量参数的语法`__. 下面的例子和上面Python的例子
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

Robot Framework 从 2.8.3 版本开始, 还支持另一种方式来实现相同的结果, 即
使用数组或者 `java.util.List` 作为最后一个参数, 或者倒数第二个参数(如果使用了
`任意关键字参数`_). 例如, 下面的示例和上面的功能是相同的:

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
更新版本的 Robot Framework 支持在 `库的构造器`__ 中使用varargs.

__ http://docs.oracle.com/javase/1.5.0/docs/guide/language/varargs.html
__ `Providing arguments to test libraries`_

任意关键字参数
Free keyword arguments (`**kwargs`)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robot Framework 2.8版本增加了任意关键字参数的支持, 使用的是Python的 `**kwargs` 语法.
如何在测试用例中使用这种语法的讨论在 `创建测试用例`_ 章节下的 `Free keyword arguments`_ 小节中. 
本章我们来看一下如何在测试库中使用它.

Robot Framework 2.8 added the support for free keyword arguments using Python's
`**kwargs` syntax. How to use the syntax in the test data is discussed
in `Free keyword arguments`_ section under `Creating test cases`_. In this
section we take a look at how to actually use it in custom test libraries.

Python中的任意关键字参数
''''''''''''''''''''''''''''''''''

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

基本上, 所有以 `命名参数语法`_ `name=value` 形式跟在关键字调用最后面, 且不匹配其它任何参数的参数, 将以 kwargs 传入给关键字. 
如果想要避免一个字面字符串被当作任意关键字参数, 则其中的等号 `=` 必须被 `转义`_ , 例如 `foo=quux` 要写作 `foo\=quux`.

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

要查看真实测试库中相同的示例的, 参考 Process_ 库中的关键字 :name:`Run Process` 和 :name:`Start Keyword`.

For a real world example of using a signature exactly like in the above
example, see :name:`Run Process` and :name:`Start Keyword` keywords in the
Process_ library.

__ 转义_

Java中的任意关键字参数
''''''''''''''''''''''''''''''''

从Robot Framework 2.8.3版本开始, Java测试库也开始支持这种语法. Java语言本身是不支持kwargs语法的, 但是关键字可以利用 `java.util.Map` 类型作为最后一个参数, 来接受 kwargs.

如果一个Java关键字接受 kwargs, Robot Framework 会自动将关键字调用的末尾所有形如  `name=value` 的参数打包放入一个 `Map` , 然后传递给关键字方法. 例如, 下面的例子中的关键字使用起来和前面的Python示例完全一样:

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

.. note:: 和 `Java中的varargs`__ 一样, kwargs的关键字也只能有一个方法签名. 

__ `Java中的可变数量的参数`_

参数类型
~~~~~~~~~~~~~~

正常情况下, 关键字的参数以字符串的形式传递给 Robot Framework. 如果关键字需要其它的类型, 可以使用 变量_ 或者在关键字的内部将字符串转换为所需的类型. 
使用了 `Java关键字`_, 基础类型会自动的强制转换.

Normally keyword arguments come to Robot Framework as strings. If
keywords require some other types, it is possible to either use
variables_ or convert strings to required types inside keywords. With
`Java keywords`__ base types are also coerced automatically.

__ `Java中的参数类型`_

Python中的参数类型
''''''''''''''''''''''''''

因为Python的参数并没有任何的类型信息, 所以使用Python的库时不可能自动的将字符串转换为其它类型. 调用Python方法实现的关键字, 只要参数的数量正确, 调用就总是能够成功, 只不过如果参数不兼容, 后面的执行会失败. 幸运的是, 在Python中转换参数类型是很简单的事情:

Because arguments in Python do not have any type information, there is
no possibility to automatically convert strings to other types when
using Python libraries. Calling a Python method implementing a keyword
with a correct number of arguments always succeeds, but the execution
fails later if the arguments are incompatible. Luckily with Python it
is simple to convert arguments to suitable types inside keywords:

.. sourcecode:: python

  def connect_to_host(address, port=25):
      port = int(port)
      # ...

Java中的参数类型
''''''''''''''''''''''''

Java方法的参数都有类型, 而且所有基础类型会自动处理. 这意味着, test data 中的字符串类型的参数, 在运行时刻强制转换为正确的类型. 可以转换的类型有:

Arguments to Java methods have types, and all the base types are
handled automatically. This means that arguments that are normal
strings in the test data are coerced to correct type at runtime. The
types that can be coerced are:

- 整数型 (`byte`, `short`, `int`, `long`)
- 浮点数 (`float` 和 `double`)
- 布尔型 (`boolean`)
- 上述类型的对象版本, 如. `java.lang.Integer`

Java的关键字方法可能会有多个签名, 强制转换只有在有相同的或兼容的签名才会发生. 下面的例子中, 关键字  `doubleArgument` 和 `compatibleTypes` 可以强制转换, 但是 `conflictingTypes` 会发生冲突.

The coercion is done for arguments that have the same or compatible
type across all the signatures of the keyword method. In the following
example, the conversion can be done for keywords `doubleArgument`
and `compatibleTypes`, but not for `conflictingTypes`.

.. sourcecode:: java

   public void doubleArgument(double arg) {}

   public void compatibleTypes(String arg1, Integer arg2) {}
   public void compatibleTypes(String arg2, Integer arg2, Boolean arg3) {}

   public void conflictingTypes(String arg1, int arg2) {}
   public void conflictingTypes(int arg1, String arg2) {}

对于数值型的类型, 如果测试数据中的字符串包含数字, 就可以强制转换, 对于布尔型, 则必须包含字符串 `true` 或者 `false`. 
强制转换只在测试数据的原始值是字符串的情况下才会发生, 当然还可以使用包含了正确数据类型的变量. 要应对冲突的方法签名, 使用变量是唯一的选择.

The coercion works with the numeric types if the test data has a
string containing a number, and with the boolean type the data must
contain either string `true` or `false`. Coercion is only
done if the original value was a string from the test data, but it is
of course still possible to use variables containing correct types with
these keywords. Using variables is the only option if keywords have
conflicting signatures.

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

从 Robot Framework 2.8 版本开始, 参数类型的强制转换在 `Java库的构造函数`__ 中也起作用.

Starting from Robot Framework 2.8, argument type coercion works also with
`Java library constructors`__.

__ `Providing arguments to test libraries`_

使用装饰器
~~~~~~~~~~~~~~~~

当编写静态关键字时, 有时候使用Python的装饰器修改它们会很方便. 但是, 装饰器修改了函数的签名, 这会让 Robot Framework 在判断关键字能接受什么参数时产生混乱. 特别是用 Libdoc_ 创建库文档和使用 RIDE_ 时. 为了避免这种情况, 要么就不要用装饰器, 要么使用方便的 `装饰器模块`__ 创建保留签名的装饰器. 

When writing static keywords, it is sometimes useful to modify them with
Python's decorators. However, decorators modify function signatures,
and can confuse Robot Framework's introspection when determining which
arguments keywords accept. This is especially problematic when creating
library documentation with Libdoc_ and when using  RIDE_. To avoid this
issue, either do not use decorators, or use the handy `decorator module`__
to create signature-preserving decorators.

.. 要翻墙啊...
__ http://micheles.googlecode.com/hg/decorator/documentation.html

关键字中嵌入参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

库关键字还能接受使用 `嵌入参数语法`__ 传递的参数. 装饰器 `robot.api.deco.keyword` 被用来创建 `自定义关键字名称`__, 其中包括所需语法.

Library keywords can also accept arguments which are passed using
`Embedded Argument syntax`__.  The `robot.api.deco.keyword` decorator
can be used to create a `custom keyword name`__ for the keyword
which includes the desired syntax.

__ `Embedding arguments into keyword name`_
__ `Using a custom keyword name`_

.. sourcecode:: python

    from robot.api.deco import keyword

    @keyword('Add ${quantity:\d+} Copies Of ${item} To Cart')
    def add_copies_to_cart(quantity, item):
        # ...

.. sourcecode:: robotframework

   *** Test Cases ***
   My Test
       Add 7 Copies Of Coffee To Cart

Communicating with Robot Framework
----------------------------------

与Robot Framework通讯
----------------------------------

当实现关键字的方法被调用后, 它可以使用任何机制去和被测系统通讯. 同时, 它还可以发送消息给 Robot Framework的日志文件, 返回可以保存到变量中的结果, 最重要的, 报告该关键字是否通过了(passed).

After a method implementing a keyword is called, it can use any
mechanism to communicate with the system under test. It can then also
send messages to Robot Framework's log file, return information that
can be saved to variables and, most importantly, report if the
keyword passed or not.

报告关键字状态
~~~~~~~~~~~~~~~~~~~~~~~~

使用异常(exceptions)即可报告关键字状态. 如果一个方法的执行抛出了一个异常, 这个关键字的状态就是 `FAIL`, 如果正常返回, 则状态是 `PASS`.

Reporting keyword status is done simply using exceptions. If an executed
method raises an exception, the keyword status is `FAIL`, and if it
returns normally, the status is `PASS`.

错误消息会写入日志和报告文件. 控制台也会显示异常类型和异常消息. 一般的异常(如 `AssertionError`, `Exception`, 和 `RuntimeError`), 只显示异常消息; 其它的异常, 消息的格式是 `异常类型: 异常消息`.

The error message shown in logs, reports and the console is created
from the exception type and its message. With generic exceptions (for
example, `AssertionError`, `Exception`, and
`RuntimeError`), only the exception message is used, and with
others, the message is created in the format `ExceptionType:
Actual message`.

从 Robot Framework 2.8.2 版本开始, 也可以让自己的异常类型和一般异常一样, 失败消息中没有异常类型作为前缀. 要实现这个效果, 为自定义异常类添加一个特殊属性 `ROBOT_SUPPRESS_NAME`, 并将值置为 `True`.

Starting from Robot Framework 2.8.2, it is possible to avoid adding the
exception type as a prefix to failure message also with non generic exceptions.
This is done by adding a special `ROBOT_SUPPRESS_NAME` attribute with
value `True` to your exception.

Python:

.. sourcecode:: python

    class MyError(RuntimeError):
        ROBOT_SUPPRESS_NAME = True

Java:

.. sourcecode:: java

    public class MyError extends RuntimeException {
        public static final boolean ROBOT_SUPPRESS_NAME = true;
    }

这种情况下, 异常消息的内容要尽量明确, 提供足够的信息给用户.

In all cases, it is important for the users that the exception message is as
informative as possible.

错误消息中使用HTML
HTML in error messages
''''''''''''''''''''''

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
~~~~~~~~~~~~~~~~~~~~~~~

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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
~~~~~~~~~~~~~~~~~~~

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

Logging example
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

日志API
Programmatic logging APIs
~~~~~~~~~~~~~~~~~~~~~~~~~

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

Logging during library initialization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Libraries can also log during the test library import and initialization.
These messages do not appear in the `log file`_ like the normal log messages,
but are instead written to the `syslog`_. This allows logging any kind of
useful debug information about the library initialization. Messages logged
using the `WARN` or `ERROR` levels are also visible in the `test execution errors`_
section in the log file.

Logging during the import and initialization is possible both using the
`standard output and error streams`__ and the `programmatic logging APIs`_.
Both of these are demonstrated below.

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

Python library logging using the logging API during import:

.. sourcecode:: python

   from robot.api import logger

   logger.debug("Importing library")

   def keyword():
       # ...

.. note:: If you log something during initialization, i.e. in Python
          `__init__` or in Java constructor, the messages may be
          logged multiple times depending on the `test library scope`_.

__ `Logging information`_

Returning values
~~~~~~~~~~~~~~~~

The final way for keywords to communicate back to the core framework
is returning information retrieved from the system under test or
generated by some other means. The returned values can be `assigned to
variables`__ in the test data and then used as inputs for other keywords,
even from different test libraries.

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

Communication when using threads
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a library uses threads, it should generally communicate with the
framework only from the main thread. If a worker thread has, for
example, a failure to report or something to log, it should pass the
information first to the main thread, which can then use exceptions or
other mechanisms explained in this section for communication with the
framework.

This is especially important when threads are run on background while
other keywords are running. Results of communicating with the
framework in that case are undefined and can in the worst case cause a
crash or a corrupted output file. If a keyword starts something on
background, there should be another keyword that checks the status of
the worker thread and reports gathered information accordingly.

Messages logged by non-main threads using the normal logging methods from
`programmatic logging APIs`_  are silently ignored.

There is also a `BackgroundLogger` in separate robotbackgroundlogger__ project,
with a similar API as the standard `robot.api.logger`. Normal logging
methods will ignore messages from other than main thread, but the
`BackgroundLogger` will save the background messages so that they can be later
logged to Robot's log.

__ https://github.com/robotframework/robotbackgroundlogger

Distributing test libraries
---------------------------

Documenting libraries
~~~~~~~~~~~~~~~~~~~~~

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

Both Python and Java have tools for creating an API documentation of a
library documented as above. However, outputs from these tools can be slightly
technical for some users. Another alternative is using Robot
Framework's own documentation tool Libdoc_. This tool can
create a library documentation from both Python and Java libraries
using the static library API, such as the ones above, but it also handles
libraries using the `dynamic library API`_ and `hybrid library API`_.

The first line of a keyword documentation is used for a special
purpose and should contain a short overall description of the
keyword. It is used as a *short documentation*, for example as a tool
tip, by Libdoc_ and also shown in the test logs. However, the latter
does not work with Java libraries using the static API,
because their documentations are lost in compilation and not available
at runtime.

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

Testing libraries
~~~~~~~~~~~~~~~~~

Any non-trivial test library needs to be thoroughly tested to prevent
bugs in them. Of course, this testing should be automated to make it
easy to rerun tests when libraries are changed.

Both Python and Java have excellent unit testing tools, and they suite
very well for testing libraries. There are no major differences in
using them for this purpose compared to using them for some other
testing. The developers familiar with these tools do not need to learn
anything new, and the developers not familiar with them should learn
them anyway.

It is also easy to use Robot Framework itself for testing libraries
and that way have actual end-to-end acceptance tests for them. There are
plenty of useful keywords in the BuiltIn_ library for this
purpose. One worth mentioning specifically is :name:`Run Keyword And Expect
Error`, which is useful for testing that keywords report errors
correctly.

Whether to use a unit- or acceptance-level testing approach depends on
the context. If there is a need to simulate the actual system under
test, it is often easier on the unit level. On the other hand,
acceptance tests ensure that keywords do work through Robot
Framework. If you cannot decide, of course it is possible to use both
the approaches.

Packaging libraries
~~~~~~~~~~~~~~~~~~~

After a library is implemented, documented, and tested, it still needs
to be distributed to the users. With simple libraries consisting of a
single file, it is often enough to ask the users to copy that file
somewhere and set the `module search path`_ accordingly. More
complicated libraries should be packaged to make the installation
easier.

Since libraries are normal programming code, they can be packaged
using normal packaging tools. With Python, good options include
distutils_, contained by Python's standard library, and the newer
setuptools_. A benefit of these tools is that library modules are
installed into a location that is automatically in the `module
search path`_.

When using Java, it is natural to package libraries into a JAR
archive. The JAR package must be put into the `module search path`_
before running tests, but it is easy to create a `start-up script`_ that
does that automatically.

Deprecating keywords
~~~~~~~~~~~~~~~~~~~~

Sometimes there is a need to replace existing keywords with new ones
or remove them altogether. Just informing the users about the change
may not always be enough, and it is more efficient to get warnings at
runtime. To support that, Robot Framework has a capability to mark
keywords *deprecated*. This makes it easier to find old keywords from
the test data and remove or replace them.

Keywords can be deprecated by starting their documentation with text
`*DEPRECATED`, case-sensitive, and having a closing `*` also on the first
line of the documentation. For example, `*DEPRECATED*`, `*DEPRECATED.*`, and
`*DEPRECATED in version 1.5.*` are all valid markers.

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

This deprecation system works with most test libraries and also with
`user keywords`__.  The only exception are keywords implemented in a
Java test library that uses the `static library interface`__ because
their documentation is not available at runtime. With such keywords,
it possible to use user keywords as wrappers and deprecate them.

.. note:: Prior to Robot Framework 2.9 the documentation must start with
          `*DEPRECATED*` exactly without any extra content before the
          closing `*`.

__ `Errors and warnings during execution`_
__ `Documenting libraries`_
__ `User keyword name and documentation`_
__ `Creating static keywords`_

.. _Dynamic library:

Dynamic library API
-------------------

The dynamic API is in most ways similar to the static API. For
example, reporting the keyword status, logging, and returning values
works exactly the same way. Most importantly, there are no differences
in importing dynamic libraries and using their keywords compared to
other libraries. In other words, users do not need to know what APIs their
libraries use.

Only differences between static and dynamic libraries are
how Robot Framework discovers what keywords a library implements,
what arguments and documentation these keywords have, and how the
keywords are actually executed. With the static API, all this is
done using reflection (except for the documentation of Java libraries),
but dynamic libraries have special methods that are used for these
purposes.

One of the benefits of the dynamic API is that you have more flexibility
in organizing your library. With the static API, you must have all
keywords in one class or module, whereas with the dynamic API, you can,
for example, implement each keyword as a separate class. This use case is
not so important with Python, though, because its dynamic capabilities and
multi-inheritance already give plenty of flexibility, and there is also
possibility to use the `hybrid library API`_.

Another major use case for the dynamic API is implementing a library
so that it works as proxy for an actual library possibly running on
some other process or even on another machine. This kind of a proxy
library can be very thin, and because keyword names and all other
information is got dynamically, there is no need to update the proxy
when new keywords are added to the actual library.

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

Getting keyword names
~~~~~~~~~~~~~~~~~~~~~

Dynamic libraries tell what keywords they implement with the
`get_keyword_names` method. The method also has the alias
`getKeywordNames` that is recommended when using Java. This
method cannot take any arguments, and it must return a list or array
of strings containing the names of the keywords that the library implements.

If the returned keyword names contain several words, they can be returned
separated with spaces or underscores, or in the camelCase format. For
example, `['first keyword', 'second keyword']`,
`['first_keyword', 'second_keyword']`, and
`['firstKeyword', 'secondKeyword']` would all be mapped to keywords
:name:`First Keyword` and :name:`Second Keyword`.

Dynamic libraries must always have this method. If it is missing, or
if calling it fails for some reason, the library is considered a
static library.

Marking methods to expose as keywords
'''''''''''''''''''''''''''''''''''''

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
~~~~~~~~~~~~~~~~

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

After getting keyword name and arguments, the library can execute
the keyword freely, but it must use the same mechanism to
communicate with the framework as static libraries. This means using
exceptions for reporting keyword status, logging by writing to
the standard output or by using provided logging APIs, and using
the return statement in `run_keyword` for returning something.

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

Getting keyword arguments
~~~~~~~~~~~~~~~~~~~~~~~~~

If a dynamic library only implements the `get_keyword_names` and
`run_keyword` methods, Robot Framework does not have any information
about the arguments that the implemented keywords need. For example,
both :name:`First Keyword` and :name:`Second Keyword` in the example above
could be used with any number of arguments. This is problematic,
because most real keywords expect a certain number of keywords, and
under these circumstances they would need to check the argument counts
themselves.

Dynamic libraries can tell Robot Framework what arguments the keywords
it implements expect by using the `get_keyword_arguments`
(alias `getKeywordArguments`) method. This method takes the name
of a keyword as an argument, and returns a list or array of strings
containing the arguments accepted by that keyword.

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

When the `get_keyword_arguments` is used, Robot Framework automatically
calculates how many positional arguments the keyword requires and does it
support free keyword arguments or not. If a keyword is used with invalid
arguments, an error occurs and `run_keyword` is not even called.

The actual argument names and default values that are returned are also
important. They are needed for `named argument support`__ and the Libdoc_
tool needs them to be able to create a meaningful library documentation.

If `get_keyword_arguments` is missing or returns `None` or
`null` for a certain keyword, that keyword gets an argument specification
accepting all arguments. This automatic argument spec is either
`[*varargs, **kwargs]` or `[*varargs]`, depending does
`run_keyword` `support kwargs`__ by having three arguments or not.

__ `Named argument syntax with dynamic libraries`_
__ `Free keyword arguments with dynamic libraries`_

Getting keyword documentation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The final special method that dynamic libraries can implement is
`get_keyword_documentation` (alias
`getKeywordDocumentation`). It takes a keyword name as an
argument and, as the method name implies, returns its documentation as
a string.

The returned documentation is used similarly as the keyword
documentation string with static libraries implemented with
Python. The main use case is getting keywords' documentations into a
library documentation generated by Libdoc_. Additionally,
the first line of the documentation (until the first `\n`) is
shown in test logs.

Getting keyword tags
~~~~~~~~~~~~~~~~~~~~

Dynamic libraries do not have any other way for defining `keyword tags`_
than by specifying them on the last row of the documentation with `Tags:`
prefix. Separate `get_keyword_tags` method can be added to the dynamic API
later if there is a need.

Getting general library documentation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `get_keyword_documentation` method can also be used for
specifying overall library documentation. This documentation is not
used when tests are executed, but it can make the documentation
generated by Libdoc_ much better.

Dynamic libraries can provide both general library documentation and
documentation related to taking the library into use. The former is
got by calling `get_keyword_documentation` with special value
`__intro__`, and the latter is got using value
`__init__`. How the documentation is presented is best tested
with Libdoc_ in practice.

Python based dynamic libraries can also specify the general library
documentation directly in the code as the docstring of the library
class and its `__init__` method. If a non-empty documentation is
got both directly from the code and from the
`get_keyword_documentation` method, the latter has precedence.

Named argument syntax with dynamic libraries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting from Robot Framework 2.8, also the dynamic library API supports
the `named argument syntax`_. Using the syntax works based on the
argument names and default values `got from the library`__ using the
`get_keyword_arguments` method.

For the most parts, the named arguments syntax works with dynamic keywords
exactly like it works with any other keyword supporting it. The only special
case is the situation where a keyword has multiple arguments with default
values, and only some of the latter ones are given. In that case the framework
fills the skipped optional arguments based on the default values returned
by the `get_keyword_arguments` method.

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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting from Robot Framework 2.8.2, dynamic libraries can also support
`free keyword arguments`_ (`**kwargs`). A mandatory precondition for
this support is that the `run_keyword` method `takes three arguments`__:
the third one will get kwargs when they are used. Kwargs are passed to the
keyword as a dictionary (Python) or Map (Java).

What arguments a keyword accepts depends on what `get_keyword_arguments`
`returns for it`__. If the last argument starts with `**`, that keyword is
recognized to accept kwargs.

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

Summary
~~~~~~~

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

.. note:: In addition to using `List`, it is possible to use also arrays
          like `Object[]` or `String[]`.

A good example of using the dynamic API is Robot Framework's own
`Remote library`_.

Hybrid library API
------------------

The hybrid library API is, as its name implies, a hybrid between the
static API and the dynamic API. Just as with the dynamic API, it is
possible to implement a library using the hybrid API only as a class.

Getting keyword names
~~~~~~~~~~~~~~~~~~~~~

Keyword names are got in the exactly same way as with the dynamic
API. In practice, the library needs to have the
`get_keyword_names` or `getKeywordNames` method returning
a list of keyword names that the library implements.

Running keywords
~~~~~~~~~~~~~~~~

In the hybrid API, there is no `run_keyword` method for executing
keywords. Instead, Robot Framework uses reflection to find methods
implementing keywords, similarly as with the static API. A library
using the hybrid API can either have those methods implemented
directly or, more importantly, it can handle them dynamically.

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

Note that `__getattr__` does not execute the actual keyword like
`run_keyword` does with the dynamic API. Instead, it only
returns a callable object that is then executed by Robot Framework.

Another point to be noted is that Robot Framework uses the same names that
are returned from `get_keyword_names` for finding the methods
implementing them. Thus the names of the methods that are implemented in
the class itself must be returned in the same format as they are
defined. For example, the library above would not work correctly, if
`get_keyword_names` returned `My Keyword` instead of
`my_keyword`.

The hybrid API is not very useful with Java, because it is not
possible to handle missing methods with it. Of course, it is possible
to implement all the methods in the library class, but that brings few
benefits compared to the static API.

Getting keyword arguments and documentation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When this API is used, Robot Framework uses reflection to find the
methods implementing keywords, similarly as with the static API. After
getting a reference to the method, it searches for arguments and
documentation from it, in the same way as when using the static
API. Thus there is no need for special methods for getting arguments
and documentation like there is with the dynamic API.

Summary
~~~~~~~

When implementing a test library in Python, the hybrid API has the same
dynamic capabilities as the actual dynamic API. A great benefit with it is
that there is no need to have special methods for getting keyword
arguments and documentation. It is also often practical that the only real
dynamic keywords need to be handled in `__getattr__` and others
can be implemented directly in the main library class.

Because of the clear benefits and equal capabilities, the hybrid API
is in most cases a better alternative than the dynamic API when using
Python. One notable exception is implementing a library as a proxy for
an actual library implementation elsewhere, because then the actual
keyword must be executed elsewhere and the proxy can only pass forward
the keyword name and arguments.

A good example of using the hybrid API is Robot Framework's own
Telnet_ library.

Using Robot Framework's internal modules
----------------------------------------

Test libraries implemented with Python can use Robot Framework's
internal modules, for example, to get information about the executed
tests and the settings that are used. This powerful mechanism to
communicate with the framework should be used with care, though,
because all Robot Framework's APIs are not meant to be used by
externally and they might change radically between different framework
versions.

Available APIs
~~~~~~~~~~~~~~

Starting from Robot Framework 2.7, `API documentation`_ is hosted separately
at the excellent `Read the Docs`_ service. If you are unsure how to use
certain API or is using them forward compatible, please send a question
to `mailing list`_.

Using BuiltIn library
~~~~~~~~~~~~~~~~~~~~~

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

The only catch with using methods from `BuiltIn` is that all
`run_keyword` method variants must be handled specially.
Methods that use `run_keyword` methods have to be registered
as *run keywords* themselves using `register_run_keyword`
method in `BuiltIn` module. This method's documentation explains
why this needs to be done and obviously also how to do it.

Extending existing test libraries
---------------------------------

This section explains different approaches how to add new
functionality to existing test libraries and how to use them in your
own libraries otherwise.

Modifying original source code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have access to the source code of the library you want to
extend, you can naturally modify the source code directly. The biggest
problem of this approach is that it can be hard for you to update the
original library without affecting your changes. For users it may also
be confusing to use a library that has different functionality than
the original one. Repackaging the library may also be a big extra
task.

This approach works extremely well if the enhancements are generic and
you plan to submit them back to the original developers. If your
changes are applied to the original library, they are included in the
future releases and all the problems discussed above are mitigated. If
changes are non-generic, or you for some other reason cannot submit
them back, the approaches explained in the subsequent sections
probably work better.

Using inheritance
~~~~~~~~~~~~~~~~~

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

A big difference with this approach compared to modifying the original
library is that the new library has a different name than the
original. A benefit is that you can easily tell that you are using a
custom library, but a big problem is that you cannot easily use the
new library with the original. First of all your new library will have
same keywords as the original meaning that there is always
conflict__. Another problem is that the libraries do not share their
state.

This approach works well when you start to use a new library and want
to add custom enhancements to it from the beginning. Otherwise other
mechanisms explained in this section are probably better.

__ `Handling keywords with same names`_

Using other libraries directly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because test libraries are technically just classes or modules, a
simple way to use another library is importing it and using its
methods. This approach works great when the methods are static and do
not depend on the library state. This is illustrated by the earlier
example that uses `Robot Framework's BuiltIn library`__.

If the library has state, however, things may not work as you would
hope.  The library instance you use in your library will not be the
same as the framework uses, and thus changes done by executed keywords
are not visible to your library. The next section explains how to get
an access to the same library instance that the framework uses.

__ `Using Robot Framework's internal modules`_

Getting active library instance from Robot Framework
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Test libraries that use the dynamic__ or `hybrid library API`_ often
have their own systems how to extend them. With these libraries you
need to ask guidance from the library developers or consult the
library documentation or source code.

__ `dynamic library API`_
