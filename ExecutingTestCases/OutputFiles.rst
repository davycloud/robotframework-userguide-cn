.. _created outputs:

输出文件
========

测试执行完成后会创建多个输出文件, 所有这些文件都和测试结果相关. 本章将讨论什么输出文件会被创建, 如何配置创建地点, 以及如何调整输出内容.

.. contents::
   :depth: 2
   :local:

.. _different output files:

不同输出文件
------------

本节将说明有哪几种不同的输出文件, 以及如何配置它们输出的目录. 

输出文件都是通过命令行选项来配置, 选项的参数就是输出文件的路径. 一个特殊值 ``NONE`` (大小写无关)可被用来禁止输出某种文件.


.. _output directory:

输出目录
~~~~~~~~

所有的输出文件都可设置为绝对路径, 也就是可以分别指定创建到不同的地方. 此外, 路径也可以是相对输出目录的相对路径. 

默认的输出目录是测试执行开始的地方, 可以通过选项 :option:`--outputdir (-d)` 指定. 该选项指定的路径, 同样既可以是绝对路径, 也可以是相对执行路径的位置. 

不管输出文件的路径是以何种方式给出, 其所在的文件夹都会自动被创建(如果不存在的话).

.. _output.xml:
.. _output file:

输出文件(Output.xml)
~~~~~~~~~~~~~~~~~~~~~

.. note:: 译注:
          本小节讨论的是特指XML格式的输出文件, 为了区分泛指的输出文件(日志,报告等), 
          接下来当特指XML格式的输出文件时, 将不做翻译, 直接称Output文件.


Output文件包含了测试执行的所有结果, 以XML格式保存. :ref:`日志 <log file>`, :ref:`报告 <report file>` 和 :ref:`xUnit <xunit file>` 这些都是基于XML文件创建,  并且还可以使用 Rebot_ 进一步组合和后处理.


.. tip:: 从Robot Framework 2.8, 生成 :ref:`报告 <report file>` 和 :ref:`xUnit <xunit file>` 文件不再需要处理output文件.
         这样在测试执行时禁用 :ref:`日志 <log file>` 的话就可以减少内存使用.

命令行选项 :option:`--output (-o)` 决定了output文件创建的路径, 如果不是绝对路径的话, 该路径是相对于 :ref:`output directory`. 默认的output文件名称是 :file:`output.xml`.

当使用Rebot :ref:`post-processing outputs` 时, 除非明确使用 :option:`--output` 选项, 否则不会创建新的output文件.

为选项 :option:`--output` 指定特殊值 ``NONE`` 可以禁止output文件生成. 在Robot Framework 2.8版本之前, 这同样会禁止日志和报告文件的生成, 不过现今的版本不会这样了. 如果想禁用所有, 必须明确的分别指定 ``--output NONE --report NONE --log NONE``.

.. _log file:

日志文件
~~~~~~~~

日志文件以HTML格式记录了测试用例执行的细节, 以层次的结构展示测试套件, 测试用例和关键字的细节. 当每次需要详细地研究测试结果时, 日志文件几乎都是必需的. 此外, 尽管日志文件也包含了统计, 更高层次的概览信息也是参考报告文件比较好.

命令行选项 :option:`--log (-l)` 指定了日志文件创建的位置. 除非使用了特殊值 ``NONE``, 日志文件总是会创建, 其默认名称是 :file:`log.html`.

.. figure:: src/ExecutingTestCases/:ref:`日志 <log file>`passed.png
   :target: src/ExecutingTestCases/:ref:`日志 <log file>`passed.html
   :width: 500

   An example of beginning of a log file

.. figure:: src/ExecutingTestCases/:ref:`日志 <log file>`failed.png
   :target: src/ExecutingTestCases/:ref:`日志 <log file>`failed.html
   :width: 500

   An example of a log file with keyword details visible

.. _report file:

报告文件
~~~~~~~~

报告(Report)文件也是HTML格式, 包含测试执行结果的概况. 其中有基于标签和测试套件的统计结果, 还有所有执行的测试用例列表. 

当同时生成日志文件和报告文件时, 报告文件内会有指向日志文件的链接, 可以轻松的导航到更详细的信息. 

当所有 :ref:`critical tests` 通过时, 报告页面的背景是绿色, 反之则是红色, 这使得通过报告可以轻松了解到测试执行的总体状态.

命令行选项 :option:`--report (-r)` 指定了报告文件创建的位置. 除非使用了特殊值 ``NONE``, 报告文件总是会创建, 其默认名称是 :file:`report.html`.

.. figure:: src/ExecutingTestCases/:ref:`报告 <report file>`passed.png
   :target: src/ExecutingTestCases/:ref:`报告 <report file>`passed.html
   :width: 500

   An example report file of successful test execution

.. figure:: src/ExecutingTestCases/:ref:`报告 <report file>`failed.png
   :target: src/ExecutingTestCases/:ref:`报告 <report file>`failed.html
   :width: 500

   An example report file of failed test execution

.. _xunit:
.. _xunit file:
.. _XUnit compatible result file:

兼容XUnit的结果文件
~~~~~~~~~~~~~~~~~~~~

XUnit结果文件包含了兼容 :ref:`xUnit <http://en.wikipedia.org/wiki/XUnit>` 的XML格式的测试执行概况. 这些文件可以作为那些处理xUnit报告的外部工具的输入. 例如, 持续集成工具 :ref:`Jenkins <http://jenkins-ci.org>` 服务器就支持基于xUnit相容的结果生成统计.

.. tip:: Jenkins also has a separate :ref:`Robot Framework plugin <https://wiki.jenkins-ci.org/display/JENKINS/Robot+Framework+Plugin>`.

XUnit输出文件只在明确的使用了命令行选项 :option:`--xunit (-x)` 之后才会创建. 该选项需要指定生成xUnit文件的路径, 相对于 :ref:`output directory`.

因为xUnit报告没有所谓 :ref:`non-critical tests <setting criticality>` 的概念, 所有的测试都会被标记为通过或失败, 而没有关键的和非关键的之分. 如果这样处理有问题, 可以使用选项 :option:`--xunitskipnoncritical` 将非关键的用例标记为略过. 被略过的测试将会获得一个包含了实际状态以及可能的测试用例发出的消息(message), 整个消息的格式类似于 ``FAIL: Error message``.

.. note:: :option:`--xunitskipnoncritical` 是Robot Framework 2.8才有的新选项.


.. _debug file:

调试文件
~~~~~~~~~~

调试(Debug)文件是纯文本文件, 在测试执行过程中被写入. 所有的测试库产生的消息都会被写入, 同时还包括测试套件, 测试用例以及关键字的启动和结束信息. 调试文件可被用来监控测试执行. 例如使用 :ref:`fileviewer.py <https://bitbucket.org/robotframework/robottools/src/master/fileviewer/>` 工具, 或者在类UNIX系统中, ``tail -f`` 命令即可.

调试文件只在明确的使用了选项 :option:`--debugfile (-b)` 后才会被创建.

.. _timestamping output files:

给输出文件打时间戳
~~~~~~~~~~~~~~~~~~

本章提到的所有输出文件都可以自动打上时间戳, 使用 :option:`--timestampoutputs (-T)` 选项, 时间戳的格式为 ``YYYYMMDD-hhmmss``, 位于文件扩展名和基础名之间. 

例如, 下面的例子中的输出文件分别是 :file:`output-20080604-163225.xml` 和 :file:`mylog-20080604-163225.html`::

   robot --timestampoutputs --log mylog.html --report NONE tests.robot

.. _setting titles:

设置标题
~~~~~~~~

:ref:`日志 <log file>` 和 :ref:`报告 <report file>` 文件的标题(title)由顶层测试套件名加上 :name:`Test Log` 或 :name:`Test Report` 组成. 自定义的标题可以通过命令行选项 :option:`--logtitle` 和 :option:`--reporttitle` 分别指定.

Example::

   robot --logtitle Smoke_Test_Log --reporttitle Smoke_Test_Report --include smoke my_tests/

.. _setting background colors:

设置背景色
~~~~~~~~~~

默认情况下, :ref:`report file` 在所有 :ref:`关键测试 <setting criticality>` 都通过的时候背景色为绿色, 否则背景是红色. 这些颜色可以通过命令行选项 :option:`--reportbackground` 自定义, 该选项接受以冒号分隔的两个或三个颜色参数::

   --reportbackground blue:red
   --reportbackground green:yellow:red
   --reportbackground #00E:#E00

当指定两个颜色时, 第一个用来替代默认的绿色, 第二个用来替代默认的红色. 这样就可以使用蓝色替代绿色背景, 对色盲人群来说更容易区分.

如果指定三个颜色, 第一个在所有用例都成功时使用, 第二个在只有非关键用例失败的时候, 而最后一个在有关键用例失败的时候使用. 这样, 如果想识别出非关键测试失败的情况, 就可以单独为此指定个颜色, 如黄色.

颜色值是针对HTML页面的 ``body`` 元素的 ``background`` CSS属性. 该值可以是HTML支持的颜色名称(如: ``red``), 十六进制的值(如: ``#f00`` 或 ``#ff0000``), 或者是一个RGB值(如: ``rgb(255,0,0)``). 默认的绿色和红色分别对应的十六进制值是 ``#9e9`` 和 ``#f66``.

.. _Log levels:

日志级别
----------

.. _available log levels:

可用的日志级别
~~~~~~~~~~~~~~

:ref:`log file` 中的消息可以有不同的日志级别. 这些消息有些是Robot Framework自己写入, 有的是被执行的关键字打印的不同的级别 :ref:`日志消息 <logging information>`. 可用的日志级别包括:

``FAIL``
   当关键字失败时使用. 只能由Robot Framework自己使用.

``WARN``
   用来展示警告. 警告消息同样会出现在 :ref:`控制台以及日志文件的测试执行错误区 <errors and warnings during execution>`,
   不过它们不会影响到测试用例的状态.

``INFO``
   默认的消息级别. 默认情况下日志文件中不会显示低于此级别的消息.

``DEBUG``
   用于调试目的. 当需要记录测试库内部执行过程时很有用. 当关键字失败时, 代码失败的地方会自动使用该级别打印traceback信息.

``TRACE``
   更详细的调试级别. 使用该级别时, 关键字的参数和返回值会自动写入日志.


.. _setting log level:

设置日志级别
~~~~~~~~~~~~

默认情况下, 低于 ``INFO`` 级别的日志消息不会写日志, 不过这个阈值可以通过命令行选项 :option:`--loglevel (-L)` 修改. 该选项接受任意的日志级别作为参数. 还可以使用 ``NONE`` 这个特殊值来禁止所有日志.

在使用Rebot :ref:`post-processing outputs` 时也可以使用 :option:`--loglevel` 选项. 这样可以做到, 例如, 在测试执行时使用 ``TRACE`` 级别, 产生详细的日志信息, 但是要在随后普通查看时使用 ``INFO`` 级别生成更小的日志文件. 默认情况下所有执行阶段产生的消息在Rebot处理时也都包含. 反之则不行, 执行阶段省略的消息不可能再恢复.

在测试数据中使用 BuiltIn_ 关键字 :name:`Set Log Level` 也可以改变日志级别. 该关键字的参数和 :option:`--loglevel` 一样, 并且会返回原来的日志级别以备后续(如teardown)恢复. 

.. _visible log level:

可见的日志级别
~~~~~~~~~~~~~~

自Robot Framework 2.7.2版本开始, 如果日志文件中包含 ``DEBUG`` 或 ``TRACE`` 级别的消息, 则在右上角会出现一个下拉框, 让用户选择低于某个级别的日志不可见. 这在使用 ``TRACE`` 级别运行测试时特别有用.

.. figure:: ./visible_log_level.png
   :target: ./visible_log_level.html
   :width: 500

   An example log showing the visible log level drop down

默认情况下, 下拉框被设置选中最低级别, 以显示日志文件中的所有消息. 默认的可视日志级别可通过选项 :option:`--loglevel` 更改, 将其值设置为如下的格式::

   --loglevel DEBUG:INFO

可视日志级别跟在正常的日志级别之后, 用冒号分隔, 上例中, 测试运行的日志级别是 ``DEBUG``, 但是默认的可视级别是 ``INFO``.

.. Splitting logs

分割日志
--------

正常情况下, 日志文件就是一个单个HTML文件. 随着测试用例的数量增长, 文件的大小也越来越大, 以至于不方便(甚至不可能)用浏览器打开. 因此, 可以使用选项 :option:`--splitlog` 来将日志文件分割成若干部分到外部文件, 当浏览器需要时再加载.

分割日志最大的好处是每个独立的日志部分都很小, 所以即使整个测试数据很庞大, 打开和浏览日志文件也会很容易. 一个小缺点是, 当日志文件增长时, 全部文件大小会较多.

技术上讲, 每个测试用例相关的测试数据保存在一个JavaScript文件中, 该文件位于主日志文件相同目录内. 这些文件的命名如 :file:`log-42.js`, 其中 :file:`log` 主日志文件的基础名称, 而 :file:`42` 是递增的序号.

.. note:: 当拷贝日志文件时, 必须将所有的 :file:`log-*.js` 文件都带上.


.. _configuring statistics:

配置统计
--------

There are several command line options that can be used to configure
and adjust the contents of the :name:`Statistics by Tag`, :name:`Statistics
by Suite` and :name:`Test Details by Tag` tables in different output
files. All these options work both when executing test cases and when
post-processing outputs.

Configuring displayed suite statistics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a deeper suite structure is executed, showing all the test suite
levels in the :name:`Statistics by Suite` table may make the table
somewhat difficult to read. By default all suites are shown, but you can
control this with the command line option :option:`--suitestatlevel` which
takes the level of suites to show as an argument::

    --suitestatlevel 3

Including and excluding tag statistics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When many tags are used, the :name:`Statistics by Tag` table can become
quite congested. If this happens, the command line options
:option:`--tagstatinclude` and :option:`--tagstatexclude` can be
used to select which tags to display, similarly as
:option:`--include` and :option:`--exclude` are used to `select test
cases`__::

   --tagstatinclude some-tag --tagstatinclude another-tag
   --tagstatexclude owner-*
   --tagstatinclude prefix-* --tagstatexclude prefix-13

__ `By tag names`_

Generating combined tag statistics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The command line option :option:`--tagstatcombine` can be used to
generate aggregate tags that combine statistics from multiple
tags. The combined tags are specified using :ref:`tag patterns` where
`*` and `?` are supported as wildcards and `AND`,
`OR` and `NOT` operators can be used for combining
individual tags or patterns together.

The following examples illustrate creating combined tag statistics using
different patterns, and the figure below shows a snippet of the resulting
:name:`Statistics by Tag` table::

    --tagstatcombine owner-*
    --tagstatcombine smokeANDmytag
    --tagstatcombine smokeNOTowner-janne*

.. figure:: src/ExecutingTestCases/tagstatcombine.png
   :width: 550

   Examples of combined tag statistics

As the above example illustrates, the name of the added combined statistic
is, by default, just the given pattern. If this is not good enough, it
is possible to give a custom name after the pattern by separating them
with a colon (`:`). Possible underscores in the name are converted
to spaces::

    --tagstatcombine prio1ORprio2:High_priority_tests

Creating links from tag names
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can add external links to the :name:`Statistics by Tag` table by
using the command line option :option:`--tagstatlink`. Arguments to this
option are given in the format `tag:link:name`, where `tag`
specifies the tags to assign the link to, `link` is the link to
be created, and `name` is the name to give to the link.

`tag` may be a single tag, but more commonly a `simple pattern`_
where `*` matches anything and `?` matches any single
character. When `tag` is a pattern, the matches to wildcards may
be used in `link` and `title` with the syntax `%N`,
where "N" is the index of the match starting from 1.

The following examples illustrate the usage of this option, and the
figure below shows a snippet of the resulting :name:`Statistics by
Tag` table when example test data is executed with these options::

    --tagstatlink mytag:http://www.google.com:Google
    --tagstatlink jython-bug-*:http://bugs.jython.org/issue_%1:Jython-bugs
    --tagstatlink owner-*:mailto:%1@domain.com?subject=Acceptance_Tests:Send_Mail

.. figure:: src/ExecutingTestCases/tagstatlink.png
   :width: 550

   Examples of links from tag names

Adding documentation to tags
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Tags can be given a documentation with the command line option
:option:`--tagdoc`, which takes an argument in the format
`tag:doc`. `tag` is the name of the tag to assign the
documentation to, and it can also be a :ref:`simple pattern` matching
multiple tags. `doc` is the assigned documentation. Underscores
in the documentation are automatically converted to spaces and it
can also contain :ref:`HTML formatting`.

The given documentation is shown with matching tags in the :name:`Test
Details by Tag` table, and as a tool tip for these tags in the
:name:`Statistics by Tag` table. If one tag gets multiple documentations,
they are combined together and separated with an ampersand.

Examples::

    --tagdoc mytag:My_documentation
    --tagdoc regression:*See*_http://info.html
    --tagdoc owner-*:Original_author

Removing and flattening keywords
--------------------------------

Most of the content of :ref:`output files` comes from keywords and their
log messages. When creating higher level reports, log files are not necessarily
needed at all, and in that case keywords and their messages just take space
unnecessarily. Log files themselves can also grow overly large, especially if
they contain :ref:`for loops` or other constructs that repeat certain keywords
multiple times.

In these situations, command line options :option:`--removekeywords` and
:option:`--flattenkeywords` can be used to dispose or flatten unnecessary keywords.
They can be used both when :ref:`executing test cases` and when :ref:`post-processing
outputs`. When used during execution, they only affect the log file, not
the XML output file. With `rebot` they affect both logs and possibly
generated new output XML files.

Removing keywords
~~~~~~~~~~~~~~~~~

The :option:`--removekeywords` option removes keywords and their messages
altogether. It has the following modes of operation, and it can be used
multiple times to enable multiple modes. Keywords that contain `errors
or warnings`__ are not removed except when using the `ALL` mode.

`ALL`
   Remove data from all keywords unconditionally.

`PASSED`
   Remove keyword data from passed test cases. In most cases, log files
   created using this option contain enough information to investigate
   possible failures.

`FOR`
   Remove all passed iterations from :ref:`for loops` except the last one.

`WUKS`
   Remove all failing keywords inside BuiltIn_ keyword
   :name:`Wait Until Keyword Succeeds` except the last one.

`NAME:<pattern>`
   Remove data from all keywords matching the given pattern regardless the
   keyword status. The pattern is
   matched against the full name of the keyword, prefixed with
   the possible library or resource file name. The pattern is case, space, and
   underscore insensitive, and it supports :ref:`simple patterns` with `*`
   and `?` as wildcards.

`TAG:<pattern>`
   Remove data from keywords with tags that match the given pattern. Tags are
   case and space insensitive and they can be specified using `tag patterns`_
   where `*` and `?` are supported as wildcards and `AND`, `OR` and `NOT`
   operators can be used for combining individual tags or patterns together.
   Can be used both with `library keyword tags`__ and :ref:`user keyword tags`.

Examples::

   rebot --removekeywords all --output removed.xml output.xml
   robot --removekeywords passed --removekeywords for tests.robot
   robot --removekeywords name:HugeKeyword --removekeywords name:resource.* tests.robot
   robot --removekeywords tag:huge tests.robot

Removing keywords is done after parsing the :ref:`output file` and generating
an internal model based on it. Thus it does not reduce memory usage as much
as :ref:`flattening keywords`.

__ `Errors and warnings`_
__ `Keyword tags`_

.. note:: The support for using :option:`--removekeywords` when executing tests
          as well as `FOR` and `WUKS` modes were added in Robot
          Framework 2.7.

.. note:: `NAME:<pattern>` mode was added in Robot Framework 2.8.2 and
          `TAG:<pattern>` in 2.9.

Flattening keywords
~~~~~~~~~~~~~~~~~~~

The :option:`--flattenkeywords` option flattens matching keywords. In practice
this means that matching keywords get all log messages from their child
keywords, recursively, and child keywords are discarded otherwise. Flattening
supports the following modes:

`FOR`
   Flatten :ref:`for loops` fully.

`FORITEM`
   Flatten individual for loop iterations.

`NAME:<pattern>`
   Flatten keywords matching the given pattern. Pattern matching rules are
   same as when :ref:`removing keywords` using `NAME:<pattern>` mode.

`TAG:<pattern>`
   Flatten keywords with tags matching the given pattern. Pattern matching
   rules are same as when :ref:`removing keywords` using `TAG:<pattern>` mode.

Examples::

   robot --flattenkeywords name:HugeKeyword --flattenkeywords name:resource.* tests.robot
   rebot --flattenkeywords foritem --output flattened.xml original.xml

Flattening keywords is done already when the :ref:`output file` is parsed
initially. This can save a significant amount of memory especially with
deeply nested keyword structures.

.. note:: Flattening keywords is a new feature in Robot Framework 2.8.2, `FOR`
          and `FORITEM` modes were added in 2.8.5 and `TAG:<pattern>` in 2.9.

Setting start and end time of execution
---------------------------------------

When :ref:`combining outputs` using Rebot, it is possible to set the start
and end time of the combined test suite using the options :option:`--starttime`
and :option:`--endtime`, respectively. This is convenient, because by default,
combined suites do not have these values. When both the start and end time are
given, the elapsed time is also calculated based on them. Otherwise the elapsed
time is got by adding the elapsed times of the child test suites together.

It is also possible to use the above mentioned options to set start and end
times for a single suite when using Rebot.  Using these options with a
single output always affects the elapsed time of the suite.

Times must be given as timestamps in the format `YYYY-MM-DD
hh:mm:ss.mil`, where all separators are optional and the parts from
milliseconds to hours can be omitted. For example, `2008-06-11
17:59:20.495` is equivalent both to `20080611-175920.495` and
`20080611175920495`, and also mere `20080611` would work.

Examples::

   rebot --starttime 20080611-17:59:20.495 output1.xml output2.xml
   rebot --starttime 20080611-175920 --endtime 20080611-180242 *.xml
   rebot --starttime 20110302-1317 --endtime 20110302-11418 myoutput.xml

.. _pre-Rebot modifier:

Programmatic modification of results
------------------------------------

If the provided built-in features to modify results are are not enough,
Robot Framework 2.9 and newer provide a possible to do custom modifications
programmatically. This is accomplished by creating a model modifier and
activating it using the :option:`--prerebotmodifier` option.

This functionality works nearly exactly like :ref:`programmatic modification of
test data` that can be enabled with the :option:`--prerunmodifier` option.
The obvious difference is that this time modifiers operate with the
`result model`_, not the :ref:`running model`. For example, the following modifier
marks all passed tests that have taken more time than allowed as failed:

.. sourcecode:: python

    from robot.api import SuiteVisitor


    class ExecutionTimeChecker(SuiteVisitor):

        def __init__(self, max_seconds):
            self.max_milliseconds = float(max_seconds) * 1000

        def visit_test(self, test):
            if test.status == 'PASS' and test.elapsedtime > self.max_milliseconds:
                test.status = 'FAIL'
                test.message = 'Test execution took too long.'

If the above modifier would be in file :file:`ExecutionTimeChecker.py`, it
could be used, for example, like this::

    # Specify modifier as a path when running tests. Maximum time is 42 seconds.
    robot --prerebotmodifier path/to/ExecutionTimeChecker.py:42 tests.robot

    # Specify modifier as a name when using Rebot. Maximum time is 3.14 seconds.
    # ExecutionTimeChecker.py must be in the module search path.
    rebot --prerebotmodifier ExecutionTimeChecker:3.14 output.xml

If more than one model modifier is needed, they can be specified by using
the :option:`--prerebotmodifier` option multiple times. When executing tests,
it is possible to use :option:`--prerunmodifier` and
:option:`--prerebotmodifier` options together.

System log
----------

Robot Framework has its own plain-text system log where it writes
information about

   - Processed and skipped test data files
   - Imported test libraries, resource files and variable files
   - Executed test suites and test cases
   - Created outputs

Normally users never need this information, but it can be
useful when investigating problems with test libraries or Robot Framework
itself. A system log is not created by default, but it can be enabled
by setting the environment variable ``ROBOT_SYS:ref:`日志 <log file>`FILE`` so
that it contains a path to the selected file.

A system log has the same :ref:`log levels` as a normal log file, with the
exception that instead of `FAIL` it has the `ERROR`
level. The threshold level to use can be altered using the
``ROBOT_SYS:ref:`日志 <log file>`LEVEL`` environment variable like shown in the
example below.  Possible `unexpected errors and warnings`__ are
written into the system log in addition to the console and the normal
log file.

.. sourcecode:: bash

   #!/bin/bash

   export ROBOT_SYS:ref:`日志 <log file>`FILE=/tmp/syslog.txt
   export ROBOT_SYS:ref:`日志 <log file>`LEVEL=DEBUG

   robot --name Sys:ref:`日志 <log file>`example path/to/tests

__ `Errors and warnings during execution`_
