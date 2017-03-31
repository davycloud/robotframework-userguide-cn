.. role:: name(emphasis)
.. role:: setting(emphasis)

.. _listener interface:

监听器接口
==========

Robot Framework提供了一个监听器(listener)接口可以用来接收测试执行过程中的通知. 监听器可被用作, 例如, 提供外部测试监控, 当测试失败时发送邮件, 或者和其它系统之间通信.
监听器API的3.0版本同时还支持在测试执行过程中修改测试用例和结果.

所谓监听器就是实现了特殊方法的类或者模块, 可以用Python和Java来实现. 监控整个测试执行的监听器必须在命令行中指定. 此外, :ref:`测试库可以注册监听器 <test libraries as listeners>`, 使其可以在测试库运行时接收到通知.


.. contents::
   :depth: 2
   :local:

.. Taking listeners into use

启用监听器
----------

监听器通过在命令行中设置选项 :option:`--listener` 来启用, 该选项的参数是监听器的名字.
监听器的名字和 :ref:`test library names` 类似, 来自于实现该监听器的类名或模块名. 

指定的监听器必须位于 :ref:`module search path` 内. 当然, 和 :ref:`导入测试库类似 <using physical path to library>`, 你也可以指定监听器文件的绝对或者相对路径. 该选项可以多次使用来启用多个监听器, 如::

   robot --listener MyListener tests.robot
   robot --listener com.company.package.Listener tests.robot
   robot --listener path/to/MyListener.py tests.robot
   robot --listener module.Listener --listener AnotherListener tests.robot

从命令行还可以为监听器的类传递参数. 参数跟在监听器的名字(或路径)后面, 使用冒号(``:``)隔开. 如果监听器使用的是Windowns格式的绝对路径, 磁盘驱动器后面的冒号不会被当作分隔.

从Robot Framework 2.8.7版本之后, 还可以使用分号(``;``)作为分隔符. 这在监听器的参数本身含有冒号时非常有用, 不过在类UNIX操作系统中, 这需要使用双引号把整个包起来. 示例如下::

   robot --listener listener.py:arg1:arg2 tests.robot
   robot --listener "listener.py;arg:with:colons" tests.robot
   robot --listener C:\Path\Listener.py;D:\data;E:\extra tests.robot

.. _listener interface versions:

监听器接口版本
--------------

存在两种监听器接口版本. 监听器版本2自从Robot Framework 2.1开始支持, 版本3从Robot Framework 3.0版本及之后开始支持. 

监听器实现时必须设置属性 ``ROBOT_LISTENER_API_VERSION`` 为2或3, 这个值可以是整型也可以是字符串. 此外还有更古老的监听器版本1, 但是从Robot Framework 3.0版本之后即不再支持.

监听器版本2和3之间最大的区别在于前者只能从测试执行中获取信息, 但不能影响测试执行. 而版本3的接口直接获取Robot Framework自用的测试数据和结果对象, 从而可以修改执行情况和测试结果. 更多监听器的应用请参考 :ref:`listener examples`.

版本2和3两者间的另一个差别在于前者同时支持Python和Java, 而后者只支持Python.

.. Listener interface methods

监听器接口的方法
----------------

Robot Framework 在测试执行开始时实例化监听器类, 而作为模块实现的监听器则直接使用. 在测试执行过程中, 不同的监听器方法在测试套件, 测试用例和关键字开始和结束时被调用. 当导入测试库或资源文件或变量文件时, 当输出文件就绪时, 以及当整个测试执行结束时, 都有相应的方法会被调用. 一个监听器不必实现所有的接口, 只需要实现它实际用到的方法.

监听器版本2和3的接口方法基本一样, 不过接受的参数不同. 这些方法以及它们的参数将在下面的章节中详细说明. 

所有方法名字中含下划线连接的, 也可以使用对应的 *camelCase* 命名. 例如, ``start_suite`` 方法和 ``startSuite`` 是一样的.

.. Listener version 2

监听器版本2
~~~~~~~~~~~

下面的表格中列出了监听器API版本2中的方法. 所有测试执行进度相关的方法的签名都是 ``method(name, attributes)``, 其中 ``attributes`` 是一个字典, 其中包含了事件的详细信息. 对接受到的信息, 除了不能做直接的修改, 监听器方法可以做任意的事情. 如果有修改的需求, 则应该选择 :ref:`listener version 3`.

.. table:: Methods in the listener API 2
   :class: tabular

   +------------------+------------------+----------------------------------------------------------------+
   |    Method        |    Arguments     |                          Documentation                         |
   +==================+==================+================================================================+
   | start_suite      | name, attributes | Called when a test suite starts.                               |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `id`: Suite id. `s1` for the top level suite, `s1-s1`        |
   |                  |                  |   for its first child suite, `s1-s2` for the second            |
   |                  |                  |   child, and so on. New in RF 2.8.5.                           |
   |                  |                  | * `longname`: Suite name including parent suites.              |
   |                  |                  | * `doc`: Suite documentation.                                  |
   |                  |                  | * `metadata`: `Free test suite metadata`_ as a dictionary/map. |
   |                  |                  | * `source`: An absolute path of the file/directory the suite   |
   |                  |                  |   was created from. New in RF 2.7.                             |
   |                  |                  | * `suites`: Names of the direct child suites this suite has    |
   |                  |                  |   as a list.                                                   |
   |                  |                  | * `tests`: Names of the tests this suite has as a list.        |
   |                  |                  |   Does not include tests of the possible child suites.         |
   |                  |                  | * `totaltests`: The total number of tests in this suite.       |
   |                  |                  |   and all its sub-suites as an integer.                        |
   |                  |                  | * `starttime`: Suite execution start time.                     |
   +------------------+------------------+----------------------------------------------------------------+
   | end_suite        | name, attributes | Called when a test suite ends.                                 |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `id`: Same as in `start_suite`.                              |
   |                  |                  | * `longname`: Same as in `start_suite`.                        |
   |                  |                  | * `doc`: Same as in `start_suite`.                             |
   |                  |                  | * `metadata`: Same as in `start_suite`.                        |
   |                  |                  | * `source`: Same as in `start_suite`.                          |
   |                  |                  | * `starttime`: Same as in `start_suite`.                       |
   |                  |                  | * `endtime`: Suite execution end time.                         |
   |                  |                  | * `elapsedtime`: Total execution time in milliseconds as       |
   |                  |                  |   an integer                                                   |
   |                  |                  | * `status`: Suite status as string `PASS` or `FAIL`.           |
   |                  |                  | * `statistics`: Suite statistics (number of passed             |
   |                  |                  |   and failed tests in the suite) as a string.                  |
   |                  |                  | * `message`: Error message if suite setup or teardown          |
   |                  |                  |   has failed, empty otherwise.                                 |
   +------------------+------------------+----------------------------------------------------------------+
   | start_test       | name, attributes | Called when a test case starts.                                |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `id`: Test id in format like `s1-s2-t2`, where               |
   |                  |                  |   the beginning is the parent suite id and the last part       |
   |                  |                  |   shows test index in that suite. New in RF 2.8.5.             |
   |                  |                  | * `longname`: Test name including parent suites.               |
   |                  |                  | * `doc`: Test documentation.                                   |
   |                  |                  | * `tags`: Test tags as a list of strings.                      |
   |                  |                  | * `critical`: `yes` or `no` depending is test considered       |
   |                  |                  |   critical or not.                                             |
   |                  |                  | * `template`: The name of the template used for the test.      |
   |                  |                  |   An empty string if the test not templated.                   |
   |                  |                  | * `starttime`: Test execution execution start time.            |
   +------------------+------------------+----------------------------------------------------------------+
   | end_test         | name, attributes | Called when a test case ends.                                  |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `id`: Same as in `start_test`.                               |
   |                  |                  | * `longname`: Same as in `start_test`.                         |
   |                  |                  | * `doc`: Same as in `start_test`.                              |
   |                  |                  | * `tags`: Same as in `start_test`.                             |
   |                  |                  | * `critical`: Same as in `start_test`.                         |
   |                  |                  | * `template`: Same as in `start_test`.                         |
   |                  |                  | * `starttime`: Same as in `start_test`.                        |
   |                  |                  | * `endtime`: Test execution execution end time.                |
   |                  |                  | * `elapsedtime`: Total execution time in milliseconds as       |
   |                  |                  |   an integer                                                   |
   |                  |                  | * `status`: Test status as string `PASS` or `FAIL`.            |
   |                  |                  | * `message`: Status message. Normally an error                 |
   |                  |                  |   message or an empty string.                                  |
   +------------------+------------------+----------------------------------------------------------------+
   | start_keyword    | name, attributes | Called when a keyword starts.                                  |
   |                  |                  |                                                                |
   |                  |                  | `name` is the full keyword name containing                     |
   |                  |                  | possible library or resource name as a prefix.                 |
   |                  |                  | For example, `MyLibrary.Example Keyword`.                      |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `type`: String `Keyword` for normal keywords, `Setup` or     |
   |                  |                  |   `Teardown` for the top level keyword used as setup/teardown, |
   |                  |                  |   `For` for for loops, and `For Item` for individual for loop  |
   |                  |                  |   iterations. **NOTE:** Keyword type reporting was changed in  |
   |                  |                  |   RF 3.0. See issue `#2248`__ for details.                     |
   |                  |                  | * `kwname`: Name of the keyword without library or             |
   |                  |                  |   resource prefix. New in RF 2.9.                              |
   |                  |                  | * `libname`: Name of the library or resource the               |
   |                  |                  |   keyword belongs to, or an empty string when                  |
   |                  |                  |   the keyword is in a test case file. New in RF 2.9.           |
   |                  |                  | * `doc`: Keyword documentation.                                |
   |                  |                  | * `args`: Keyword's arguments as a list of strings.            |
   |                  |                  | * `assign`: A list of variable names that keyword's            |
   |                  |                  |   return value is assigned to. New in RF 2.9.                  |
   |                  |                  | * `tags`: `Keyword tags`_ as a list of strings. New in RF 3.0. |
   |                  |                  | * `starttime`: Keyword execution start time.                   |
   +------------------+------------------+----------------------------------------------------------------+
   | end_keyword      | name, attributes | Called when a keyword ends.                                    |
   |                  |                  |                                                                |
   |                  |                  | `name` is the full keyword name containing                     |
   |                  |                  | possible library or resource name as a prefix.                 |
   |                  |                  | For example, `MyLibrary.Example Keyword`.                      |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `type`: Same as with `start_keyword`.                        |
   |                  |                  | * `kwname`: Same as with `start_keyword`.                      |
   |                  |                  | * `libname`: Same as with `start_keyword`.                     |
   |                  |                  | * `doc`: Same as with `start_keyword`.                         |
   |                  |                  | * `args`: Same as with `start_keyword`.                        |
   |                  |                  | * `assign`: Same as with `start_keyword`.                      |
   |                  |                  | * `tags`: Same as with `start_keyword`.                        |
   |                  |                  | * `starttime`: Same as with `start_keyword`.                   |
   |                  |                  | * `endtime`: Keyword execution end time.                       |
   |                  |                  | * `elapsedtime`: Total execution time in milliseconds as       |
   |                  |                  |   an integer                                                   |
   |                  |                  | * `status`: Keyword status as string `PASS` or `FAIL`.         |
   +------------------+------------------+----------------------------------------------------------------+
   | log_message      | message          | Called when an executed keyword writes a log message.          |
   |                  |                  |                                                                |
   |                  |                  | `message` is a dictionary with the following contents:         |
   |                  |                  |                                                                |
   |                  |                  | * `message`: The content of the message.                       |
   |                  |                  | * `level`: `Log level`_ used in logging the message.           |
   |                  |                  | * `timestamp`: Message creation time in format                 |
   |                  |                  |   `YYYY-MM-DD hh:mm:ss.mil`.                                   |
   |                  |                  | * `html`: String `yes` or `no` denoting whether the message    |
   |                  |                  |   should be interpreted as HTML or not.                        |
   |                  |                  |                                                                |
   |                  |                  | Starting from RF 3.0, this method is not called if the message |
   |                  |                  | has level below the current `threshold level <Log levels_>`_.  |
   +------------------+------------------+----------------------------------------------------------------+
   | message          | message          | Called when the framework itself writes a syslog_ message.     |
   |                  |                  |                                                                |
   |                  |                  | `message` is a dictionary with the same contents as with       |
   |                  |                  | `log_message` method.                                          |
   +------------------+------------------+----------------------------------------------------------------+
   | library_import   | name, attributes | Called when a library has been imported.                       |
   |                  |                  |                                                                |
   |                  |                  | `name` is the name of the imported library. If the library     |
   |                  |                  | has been imported using the `WITH NAME syntax`_, `name` is     |
   |                  |                  | the specified alias.                                           |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `args`: Arguments passed to the library as a list.           |
   |                  |                  | * `originalname`: The original library name when using the     |
   |                  |                  |   WITH NAME syntax, otherwise same as `name`.                  |
   |                  |                  | * `source`: An absolute path to the library source. `None`     |
   |                  |                  |   with libraries implemented with Java or if getting the       |
   |                  |                  |   source of the library failed for some reason.                |
   |                  |                  | * `importer`: An absolute path to the file importing the       |
   |                  |                  |   library. `None` when BuiltIn_ is imported well as when       |
   |                  |                  |   using the :name:`Import Library` keyword.                    |
   |                  |                  |                                                                |
   |                  |                  | New in Robot Framework 2.9.                                    |
   +------------------+------------------+----------------------------------------------------------------+
   | resource_import  | name, attributes | Called when a resource file has been imported.                 |
   |                  |                  |                                                                |
   |                  |                  | `name` is the name of the imported resource file without       |
   |                  |                  | the file extension.                                            |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `source`: An absolute path to the imported resource file.    |
   |                  |                  | * `importer`: An absolute path to the file importing the       |
   |                  |                  |   resource file. `None` when using the :name:`Import Resource` |
   |                  |                  |   keyword.                                                     |
   |                  |                  |                                                                |
   |                  |                  | New in Robot Framework 2.9.                                    |
   +------------------+------------------+----------------------------------------------------------------+
   | variables_import | name, attributes | Called when a variable file has been imported.                 |
   |                  |                  |                                                                |
   |                  |                  | `name` is the name of the imported variable file with          |
   |                  |                  | the file extension.                                            |
   |                  |                  |                                                                |
   |                  |                  | Contents of the attribute dictionary:                          |
   |                  |                  |                                                                |
   |                  |                  | * `args`: Arguments passed to the variable file as a list.     |
   |                  |                  | * `source`: An absolute path to the imported variable file.    |
   |                  |                  | * `importer`: An absolute path to the file importing the       |
   |                  |                  |   resource file. `None` when using the :name:`Import           |
   |                  |                  |   Variables` keyword.                                          |
   |                  |                  |                                                                |
   |                  |                  | New in Robot Framework 2.9.                                    |
   +------------------+------------------+----------------------------------------------------------------+
   | output_file      | path             | Called when writing to an `output file`_ is ready.             |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | log_file         | path             | Called when writing to a `log file`_ is ready.                 |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | report_file      | path             | Called when writing to a `report file`_ is ready.              |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | xunit_file       | path             | Called when writing to an `xunit file`_ is ready.              |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | debug_file       | path             | Called when writing to a `debug file`_ is ready.               |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | close            |                  | Called when the whole test execution ends.                     |
   |                  |                  |                                                                |
   |                  |                  | With `library listeners`_ called when the library goes out     |
   |                  |                  | of scope.                                                      |
   +------------------+------------------+----------------------------------------------------------------+

下面是监听器方法和参数的Java interface定义. 其中 ``java.util.Map attributes`` 内容的定义同样参考上表. 记住, 一个监听器 *不必* 实现所有的方法.

.. sourcecode:: java

   public interface RobotListenerInterface {
       public static final int ROBOT_LISTENER_API_VERSION = 2;
       void startSuite(String name, java.util.Map attributes);
       void endSuite(String name, java.util.Map attributes);
       void startTest(String name, java.util.Map attributes);
       void endTest(String name, java.util.Map attributes);
       void startKeyword(String name, java.util.Map attributes);
       void endKeyword(String name, java.util.Map attributes);
       void logMessage(java.util.Map message);
       void message(java.util.Map message);
       void outputFile(String path);
       void logFile(String path);
       void reportFile(String path);
       void debugFile(String path);
       void close();
   }

__ https://github.com/robotframework/robotframework/issues/2248

.. _listener version 3:

监听器版本3
~~~~~~~~~~~~~~~~~~

监听器版本3大多数方法和 `监听器版本2`_ 一样, 不过这些方法的和测试执行相关的参数不同. 该API获取到了Robot Framework框架自己在运行时刻的实际模型对象(model objects), 监听器既可以从这些对象中查询所需信息, 也可以直接做出修改.

监听器版本3从Robot Framework 3.0版本开始支持. 初始时并不支持版本2中所有的方法. 主要原因是 `suitable model objects are not available internally`__. ``close`` 方法和输出文件相关的方法在两个版本中完全一样.

__ https://github.com/robotframework/robotframework/issues/1208#issuecomment-164910769

.. table:: Methods in the listener API 3
   :class: tabular

   +------------------+------------------+----------------------------------------------------------------+
   |    Method        |    Arguments     |                          Documentation                         |
   +==================+==================+================================================================+
   | start_suite      | data, result     | Called when a test suite starts.                               |
   |                  |                  |                                                                |
   |                  |                  | `data` and `result` are model objects representing             |
   |                  |                  | the `executed test suite <running.TestSuite_>`_ and `its       |
   |                  |                  | execution results <result.TestSuite_>`_, respectively.         |
   +------------------+------------------+----------------------------------------------------------------+
   | end_suite        | data, result     | Called when a test suite ends.                                 |
   |                  |                  |                                                                |
   |                  |                  | Same arguments as with `start_suite`.                          |
   +------------------+------------------+----------------------------------------------------------------+
   | start_test       | data, result     | Called when a test case starts.                                |
   |                  |                  |                                                                |
   |                  |                  | `data` and `result` are model objects representing             |
   |                  |                  | the `executed test case <running.TestCase_>`_ and `its         |
   |                  |                  | execution results <result.TestCase_>`_, respectively.          |
   +------------------+------------------+----------------------------------------------------------------+
   | end_test         | data, result     | Called when a test case ends.                                  |
   |                  |                  |                                                                |
   |                  |                  | Same arguments as with `start_test`.                           |
   +------------------+------------------+----------------------------------------------------------------+
   | start_keyword    | N/A              | Not implemented in RF 3.0.                                     |
   +------------------+------------------+----------------------------------------------------------------+
   | end_keyword      | N/A              | Not implemented in RF 3.0.                                     |
   +------------------+------------------+----------------------------------------------------------------+
   | log_message      | message          | Called when an executed keyword writes a log message.          |
   |                  |                  | `message` is a model object representing the `logged           |
   |                  |                  | message <result.Message_>`_.                                   |
   |                  |                  |                                                                |
   |                  |                  | This method is not called if the message has level below       |
   |                  |                  | the current `threshold level <Log levels_>`_.                  |
   +------------------+------------------+----------------------------------------------------------------+
   | message          | message          | Called when the framework itself writes a syslog_ message.     |
   |                  |                  |                                                                |
   |                  |                  | `message` is same object as with `log_message`.                |
   +------------------+------------------+----------------------------------------------------------------+
   | library_import   | N/A              | Not implemented in RF 3.0.                                     |
   +------------------+------------------+----------------------------------------------------------------+
   | resource_import  | N/A              | Not implemented in RF 3.0.                                     |
   +------------------+------------------+----------------------------------------------------------------+
   | variables_import | N/A              | Not implemented in RF 3.0.                                     |
   +------------------+------------------+----------------------------------------------------------------+
   | output_file      | path             | Called when writing to an `output file`_ is ready.             |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | log_file         | path             | Called when writing to a `log file`_ is ready.                 |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | report_file      | path             | Called when writing to a `report file`_ is ready.              |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | xunit_file       | path             | Called when writing to an `xunit file`_ is ready.              |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | debug_file       | path             | Called when writing to a `debug file`_ is ready.               |
   |                  |                  |                                                                |
   |                  |                  | `path` is an absolute path to the file.                        |
   +------------------+------------------+----------------------------------------------------------------+
   | close            |                  | Called when the whole test execution ends.                     |
   |                  |                  |                                                                |
   |                  |                  | With `library listeners`_ called when the library goes out     |
   |                  |                  | of scope.                                                      |
   +------------------+------------------+----------------------------------------------------------------+

.. Listeners logging

监听器的日志
-----------------

监听器可以使用Robot Framework提供的 :ref:`programmatic logging APIs`. 不过有些地方有所限制, 监听器方法打印日志消息的差别在下表中说明.

.. table:: How listener methods can log
   :class: tabular

   +----------------------+---------------------------------------------------+
   |         Methods      |                   Explanation                     |
   +======================+===================================================+
   | start_keyword,       | Messages are logged to the normal `log file`_     |
   | end_keyword,         | under the executed keyword.                       |
   | log_message          |                                                   |
   +----------------------+---------------------------------------------------+
   | start_suite,         | Messages are logged to the syslog_. Warnings are  |
   | end_suite,           | shown also in the `execution errors`_ section of  |
   | start_test, end_test | the normal log file.                              |
   +----------------------+---------------------------------------------------+
   | message              | Messages are normally logged to the syslog. If    |
   |                      | this method is used while a keyword is executing, |
   |                      | messages are logged to the normal log file.       |
   +----------------------+---------------------------------------------------+
   | Other methods        | Messages are only logged to the syslog.           |
   +----------------------+---------------------------------------------------+

.. note:: 为了避免递归调用, 监听器打印的日志不会发送给再发送给监听器的 
          ``log_message`` 和 ``message`` 方法.


.. _listener examples:

监听器示例
----------

本章包含了几个监听器的示例. 第一个例子仅仅只是用来获取Robot Framework的信息, 其它则展示了如何修改测试执行和创建测试结果.

.. Getting information

获取信息
~~~~~~~~~~~~~~~~~~~

第一个例子用Python模块的方式实现, 并且使用的是 `监听器版本2`_.

.. sourcecode:: python

   """Listener that stops execution if a test fails."""

   ROBOT_LISTENER_API_VERSION = 2

   def end_test(name, attrs):
       if attrs['status'] == 'FAIL':
           print 'Test "%s" failed: %s' % (name, attrs['message'])
           raw_input('Press enter to continue.')

假设上面的代码示例保存为文件 :file:`PauseExecution.py`, 则可以在命令行中使用::

   robot --listener path/to/PauseExecution.py tests.robot

相同的例子还可以使用新的 `监听器版本3`_ 来实现, 命令行中使用的方式则完全一样.

.. sourcecode:: python

   """Listener that stops execution if a test fails."""

   ROBOT_LISTENER_API_VERSION = 3

   def end_test(data, result):
       if not result.passed:
           print 'Test "%s" failed: %s' % (result.name, result.message)
           raw_input('Press enter to continue.')

下面的例子仍然使用Python, 但是略微复杂一点. 监听器将获取到的所有信息按照简单格式, 写入到一个临时目录下的文本文件中. 文件名可以从命令行中获取, 如果不指定则使用默认值. 注意, 在真实场景下, 针对这样的功能, 通过命令行选项 :option:`--debugfile` 来指定生成 `debug file`_ 更加实用. 

.. sourcecode:: python

   import os.path
   import tempfile


   class PythonListener:
       ROBOT_LISTENER_API_VERSION = 2

       def __init__(self, filename='listen.txt'):
           outpath = os.path.join(tempfile.gettempdir(), filename)
           self.outfile = open(outpath, 'w')

       def start_suite(self, name, attrs):
           self.outfile.write("%s '%s'\n" % (name, attrs['doc']))

       def start_test(self, name, attrs):
           tags = ' '.join(attrs['tags'])
           self.outfile.write("- %s '%s' [ %s ] :: " % (name, attrs['doc'], tags))

       def end_test(self, name, attrs):
           if attrs['status'] == 'PASS':
               self.outfile.write('PASS\n')
           else:
               self.outfile.write('FAIL: %s\n' % attrs['message'])

        def end_suite(self, name, attrs):
            self.outfile.write('%s\n%s\n' % (attrs['status'], attrs['message']))

        def close(self):
            self.outfile.close()

下面是上例用Java语言编写, 实现和上面Python代码相同的功能.

.. sourcecode:: java

   import java.io.*;
   import java.util.Map;
   import java.util.List;


   public class JavaListener {
       public static final int ROBOT_LISTENER_API_VERSION = 2;
       public static final String DEFAULT_FILENAME = "listen_java.txt";
       private BufferedWriter outfile = null;

       public JavaListener() throws IOException {
           this(DEFAULT_FILENAME);
       }

       public JavaListener(String filename) throws IOException {
           String tmpdir = System.getProperty("java.io.tmpdir");
           String sep = System.getProperty("file.separator");
           String outpath = tmpdir + sep + filename;
           outfile = new BufferedWriter(new FileWriter(outpath));
       }

       public void startSuite(String name, Map attrs) throws IOException {
           outfile.write(name + " '" + attrs.get("doc") + "'\n");
       }

       public void startTest(String name, Map attrs) throws IOException {
           outfile.write("- " + name + " '" + attrs.get("doc") + "' [ ");
           List tags = (List)attrs.get("tags");
           for (int i=0; i < tags.size(); i++) {
              outfile.write(tags.get(i) + " ");
           }
           outfile.write(" ] :: ");
       }

       public void endTest(String name, Map attrs) throws IOException {
           String status = attrs.get("status").toString();
           if (status.equals("PASS")) {
               outfile.write("PASS\n");
           }
           else {
               outfile.write("FAIL: " + attrs.get("message") + "\n");
           }
       }

       public void endSuite(String name, Map attrs) throws IOException {
           outfile.write(attrs.get("status") + "\n" + attrs.get("message") + "\n");
       }

       public void close() throws IOException {
           outfile.close();
       }
   }

.. Modifying execution and results

变更执行和结果
~~~~~~~~~~~~~~

下面的例子展示了如何使用 `监听器版本3`_ 来变更执行的测试套件和用例, 以及执行的结果.

.. Modifying executed suites and tests

变更执行的测试套件和用例
''''''''''''''''''''''''

改变正在执行的内容需要修改作为第一参数传递给 ``start_suite``
和 ``start_test`` 方法的模型对象, 该对象中包含了正在执行的 `test suite <running.TestSuite_>`_ 对象或 `test case <running.TestCase_>`_ 对象. 

下面的例子展示了在每个执行的测试套件中新加入一个测试, 以及在每个测试中新加入一个关键字. 

.. sourcecode:: python

   ROBOT_LISTENER_API_VERSION = 3

   def start_suite(suite, result):
       suite.tests.create(name='New test')

   def start_test(test, result):
       test.keywords.create(name='Log', args=['Keyword added by listener!'])

试图在 ``end_suite`` 或 ``end_test`` 方法中修改执行是无效的, 原因很简单, 因为测试套件或用例已经执行完成了. 试图在 ``start_suite`` 或 ``start_test`` 方法中修改测试的名称, 文档或者其它类似的元数据也是无效的, 因为这些信息对应的对象已经创建过了. 只有变更测试套件的用例或子测试集或者关键字才有效.

这个API和 :ref:`pre-run modifier` API 很相似, 都是在整个测试执行开始前修改测试套件和用例. 而使用监听器的最大好处是这个修改可以基于测试结果或者其他条件动态执行. 这会带来一些比较有趣的结果, 比如在基于模型的测试中.

尽管监听器接口并不是建立在 Robot Framework 内部的 :ref:`visitor interface` 之上的, 监听器仍然可以使用观察者接口. 例如, 在 :ref:`pre-run modifier` 中用到的 ``SelectEveryXthTest`` 在这里可以这样用:

.. sourcecode:: python

   from SelectEveryXthTest import SelectEveryXthTest

   ROBOT_LISTENER_API_VERSION = 3

   def start_suite(suite, result):
       selector = SelectEveryXthTest(x=2)
       suite.visit(selector)

.. Modifying results

变更测试结果
''''''''''''

改变测试执行的结果需要修改作为第二参数传递给 ``start_suite``
和 ``start_test`` 方法的结果对象, 分别是 `test suite <result.TestSuite_>`_ 对象或 `test case <result.TestCase_>`_ 对象. 此外还可以修改传给  :ref:`log_message` 方法的 `message <result.Message_>`_ 对象. 示例请参看下面的监听器类.

.. sourcecode:: python

    class ResultModifier(object):
        ROBOT_LISTENER_API_VERSION = 3

        def __init__(self, max_seconds=10):
            self.max_milliseconds = float(max_seconds) * 1000

       def start_suite(self, data, suite):
           suite.doc = 'Documentation set by listener.'
           # Information about tests only available via data at this point.
           smoke_tests = [test for test in data.tests if 'smoke' in test.tags]
           suite.metadata['Smoke tests'] = len(smoke_tests)

        def end_test(self, data, test):
            if test.status == 'PASS' and test.elapsedtime > self.max_milliseconds:
                test.status = 'FAIL'
                test.message = 'Test execution took too long.'

        def log_message(self, msg):
            if msg.level == 'WARN' and not msg.html:
                msg.message = '<b style="font-size: 1.5em">%s</b>' % msg.message
                msg.html = True

这里存在的一个限制是无法修改当前测试套件或用例的名称, 因为它们在监听器被调用时已经被写入到 :ref:`output.xml` 中了. 同样的原因, 在 ``end_suite`` 方法中想要修改已经执行结束的测试用例也不会有效果.

这个API和 :ref:`pre-Rebot modifier` API 很相似, 都可以在报告和日志生成之前修改测试结果. 两者之间最大的区别在于监听器还修改已创建的 :file:`output.xml` 文件.

.. _library listeners:
.. _test libraries as listeners:

作为监听器的测试库
------------------

有时候, :ref:`test libraries` 可能会想要获取测试执行过程的通知消息. 这样, 测试库可以在测试套件或整个测试执行结束时自动执行一些特定的清理操作, 

.. note:: 该功能新增于 Robot Framework 2.8.5.

.. Registering listener

注册监听器
~~~~~~~~~~

测试库可以使用 ``ROBOT_LIBRARY_LISTENER`` 属性来注册一个监听器. 该属性值应该是要使用的监听器的实例. 可以是完全独立的一个监听器, 也可是测试库自身实现. 如果是测试库同时充当监听器, 为了避免监听器的方法被暴露为关键字, 可以使用下划线作前缀. 例如, 使用 ``_end_suite`` 或 ``_endSuite`` 来替代 ``end_suite`` 或 ``endSuite``.

下面的例子展示了如何使用一个外部监听器以及如何让测试库充当监听器:

.. sourcecode:: java

   import my.project.Listener;

   public class JavaLibraryWithExternalListener {
       public static final Listener ROBOT_LIBRARY_LISTENER = new Listener();
       public static final String ROBOT_LIBRARY_SCOPE = "GLOBAL";
       public static final int ROBOT_LISTENER_API_VERSION = 2;

       // actual library code here ...
   }

.. sourcecode:: python

   class PythonLibraryAsListenerItself(object):
       ROBOT_LIBRARY_SCOPE = 'TEST SUITE'
       ROBOT_LISTENER_API_VERSION = 2

       def __init__(self):
           self.ROBOT_LIBRARY_LISTENER = self

       def _end_suite(self, name, attrs):
           print 'Suite %s (%s) ending.' % (name, attrs['id'])

       # actual library code here ...

如例所示, 测试库充当监听器同样需要通过设置属性 ``ROBOT_LISTENER_API_VERSION`` 来指定 `监听器接口版本`_.

从版本2.9开始, `ROBOT_LIBRARY_LISTENER` 属性可以赋值为监听器实例的列表(或类似序列), 其中所有的监听器都会被注册.

.. Called listener methods

被调用的监听器方法
~~~~~~~~~~~~~~~~~~

作为监听器的测试库, 在其被导入的测试套件内会收到该套件内所有事件通知. 也就是说, ``start_suite``, ``end_suite``, ``start_test``, ``end_test``, ``start_keyword``, ``end_keyword``, ``log_message``, and ``message`` 这些方法在该测试套件内都会被调用.

如果一个测试库每次实例化的时候都新建一个监听器的实例, 则实际用到的那个监听器取决于 :ref:`test library scope`. 除了前面提到的那些方法, 当测试库离开作用域时, 还会调用 `close` 方法.

所有这些方法的更多信息参见上面的 `监听器接口的方法`_ 说明.
