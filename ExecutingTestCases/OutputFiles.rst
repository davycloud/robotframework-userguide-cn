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

有几个命令行选项可用来配置和调整输出文件中统计相关的内容, 包括 :name:`Statistics by Tag`, :name:`Statistics by Suite` 和 :name:`Test Details by Tag` 表格. 所有这些选项都可同时用于测试执行和后处理输出.

.. Configuring displayed suite statistics

配置测试套件统计
~~~~~~~~~~~~~~~~

当一个层次较深的测试套件被执行后, 要在 :name:`Statistics by Suite` 表中显示所有的测试套件级别会让表格变得很难看. 默认情况下所有的测试套件都显示, 但是可以通过命令行选项 :option:`--suitestatlevel` 来控制, 该选项接受的参数值表示套件的级别::

    --suitestatlevel 3

.. Including and excluding tag statistics

包括和排除标签统计
~~~~~~~~~~~~~~~~~~

如果使用了很多的标签,  :name:`Statistics by Tag` 表也会变得非常拥挤. 命令行选项 :option:`--tagstatinclude` 和 :option:`--tagstatexclude` 可被用来选择哪些标签要展示或不展示. 类似于选项 :option:`--include` and :option:`--exclude` 被用来 :ref:`选择测试用例 <by tag names>`::

   --tagstatinclude some-tag --tagstatinclude another-tag
   --tagstatexclude owner-*
   --tagstatinclude prefix-* --tagstatexclude prefix-13


.. Generating combined tag statistics

生成组合标签统计
~~~~~~~~~~~~~~~~

命令行选项 :option:`--tagstatcombine` 可用来生成标签集合, 将多个标签的统计组合起来. 

标签组合使用 :ref:`tag patterns` 指定, 其中 ``*`` 和 ``?`` 作为通配符, 并可使用 ``AND``, ``OR`` 和 ``NOT`` 操作符将单个标签或模式结合起来.  

下面的例子展示了使用不同的模式来组合标签统计, 下面的图片显示了结果的 :name:`Statistics by Tag` 表的片段::

    --tagstatcombine owner-*
    --tagstatcombine smokeANDmytag
    --tagstatcombine smokeNOTowner-janne*

.. figure:: ./tagstatcombine.png
   :width: 550

   Examples of combined tag statistics

如上所示, 添加的组合统计的名字默认就是给出的模式. 如果这样还不够, 还可以自定义名字, 在模式的后面用冒号(``:``)分隔紧接着跟上名字. 名字中的下划线会被转换为空格::

    --tagstatcombine prio1ORprio2:High_priority_tests

.. Creating links from tag names

标签统计链接
~~~~~~~~~~~~

使用命令行选项 :option:`--tagstatlink` 可以在 :name:`Statistics by Tag` 表格中添加外部链接. 该选项的参数值的格式是 ``tag:link:name``, 其中 ``tag`` 是要添加链接的标签, ``link`` 是要创建的链接, ``name`` 是链接的名字.

``tag`` 可以是单个标签, 但更多时候是使用 :ref:`simple pattern`, ``*`` 匹配任意内容, ``?`` 匹配任意的单个字符. 当 ``tag`` 是一个模式时, 通配符匹配的内容可以在 ``link`` 和 ``title`` 使用 ``%N`` 替代, 其中"N"是从1开始计数的匹配序号.

下面的例子展示了该选项的用法, 下面的图片展示了使用这些选项执行测试后的 :name:`Statistics by Tag` 表的结果片段::


    --tagstatlink mytag:http://www.google.com:Google
    --tagstatlink jython-bug-*:http://bugs.jython.org/issue_%1:Jython-bugs
    --tagstatlink owner-*:mailto:%1@domain.com?subject=Acceptance_Tests:Send_Mail

.. figure:: ./tagstatlink.png
   :width: 550

   Examples of links from tag names

.. Adding documentation to tags

给标签添加文档
~~~~~~~~~~~~~~

标签可以使用命令行选项 :option:`--tagdoc` 给定文档, 该选项的参数格式是 ``tag:doc``. 其中 `tag` 是标签的名字, 也可以是表示多个标签的 :ref:`simple pattern`. `doc` 是要指定的文档. 文档中的下划线自动转为空格, 并且文档中可以包含 :ref:`HTML formatting`.

给定的文档将和匹配的标签在 :name:`Test Details by Tag` 表格中展示, 同时在 :name:`Statistics by Tag` 表格中作为这些标签的提示. 

如果一个标签有多个文档, 则这些文档将使用 ``&`` 连在一起.

示例::

    --tagdoc mytag:My_documentation
    --tagdoc regression:*See*_http://info.html
    --tagdoc owner-*:Original_author

.. Removing and flattening keywords

Remove和Flatten关键字
----------------------

:ref:`output files` 大部分的内容来自于关键字和它们的日志消息. 当创建更高层的报告时, 日志文件完全可以忽略, 这时关键字和它们的消息只是占用无谓的空间. 日志文件本身也有可能会变得非常庞大, 特别是其中包含了 :ref:`for loops`, 或者是其它重复执行多次某个关键字的结构.

这种情况下, 命令行选项 :option:`--removekeywords` 和 :option:`--flattenkeywords` 可被用来丢弃或压平(flatten)非必要的关键字.

这两个选项都可用在 :ref:`executing test cases` 和 :ref:`post-processing outputs`. 当用在测试执行时, 只影响日志文件, 不影响XML输出文件. 而用在 ``rebot`` 命令时, 日志文件和可能新建的XML文件都会影响.

.. _removing keywords:

Remove关键字
~~~~~~~~~~~~~

选项 :option:`--removekeywords` 将关键字连同它们的消息一起删除. 该选项可以有以下几种操作模式, 并且可以指定多次来启用多种模式. 包含了 :ref:`错误和警告 <errors and warnings>` 的关键字不会删除, 除非使用的是 ``ALL`` 模式.

``ALL``
   无条件地删除所有关键字的数据

``PASSED``
   删除成功通过的测试用例的关键字. 大多数情况下, 使用该选项创建的日志文件包含了足够信息来调查可能的故障.

``FOR``
   删除 :ref:`for loops` 所有通过的迭代, 除了最后一个.

``WUKS``
   删除在内置关键字 :name:`Wait Until Keyword Succeeds` 之中失败的关键字, 除了最后一个.

``NAME:<pattern>``
   删除所有匹配模式的关键字数据, 不管该关键字的状态是什么. 模式匹配针对关键字的全名, 也就是说可能包括了库名或资源文件名的前缀. 模式不区分大小写, 并且忽略空格和下划线, 并且支持 :ref:`simple patterns`, 可使用 ``*`` 和 ``?`` 做通配符.

``TAG:<pattern>``
   删除所有标签匹配上给定模式的关键字数据. 标签对大小写和空格都不敏感, 并且可以使用 :ref:`tag patterns`, 也就是说可以使用 ``*`` 和 ``?`` 做通配符, 以及 ``AND``, ``OR`` 和 ``NOT`` 操作符. :ref:`库关键字的标签 <keyword tags>` 和 :ref:`user keyword tags` 都可以.


示例::

   rebot --removekeywords all --output removed.xml output.xml
   robot --removekeywords passed --removekeywords for tests.robot
   robot --removekeywords name:HugeKeyword --removekeywords name:resource.* tests.robot
   robot --removekeywords tag:huge tests.robot

删除关键字是在解析完 :ref:`output file` 并且生成了内部模型之后. 所以不会像 :ref:`flattening keywords` 那样减少对内存的占用.

.. note:: 在执行测试时可用选项 :option:`--removekeywords` 
          以及 `FOR` 和 `WUKS` 两种模式是在 Robot Framework 2.7 新增.

.. note:: `NAME:<pattern>` 模式在 Robot Framework 2.8.2 版本新增, 
          `TAG:<pattern>` 在 2.9 版本新增.


.. _flattening keywords:

Flatten关键字
~~~~~~~~~~~~~~

选项 :option:`--flattenkeywords` 将匹配的关键字"压平", 也就是说, 只保留该关键字全部的日志消息, 包括所有子关键字(递归)的日志, 而子关键字自身则都被丢弃. 

.. this means that matching keywords get all log messages from their child
.. keywords, recursively, and child keywords are discarded otherwise. Flattening
.. supports the following modes:

Flatten支持以下几种模式:

``FOR``
   Flatten :ref:`for loops` fully.

``FORITEM``
   Flatten individual for loop iterations.

``NAME:<pattern>``
   压平匹配模式的关键字. 模式匹配的规则和 :ref:`removing keywords` 和 `NAME:<pattern>` 模式一样.

``TAG:<pattern>``

   压平标签匹配上的关键字, 规则和 :ref:`removing keywords` 的 `TAG:<pattern>` 模式一样.

示例::

   robot --flattenkeywords name:HugeKeyword --flattenkeywords name:resource.* tests.robot
   rebot --flattenkeywords foritem --output flattened.xml original.xml

压平关键字是在 :ref:`output file` 初始解析之前就已经完成, 所以可以节约大量的内存, 特别是当关键字嵌套比较深的时候.


.. note:: Flattening keywords is a new feature in Robot Framework 2.8.2, `FOR`
          and `FORITEM` modes were added in 2.8.5 and `TAG:<pattern>` in 2.9.

.. _setting start and end time of execution:

设置执行的起始和结束时间
------------------------

.. hint:: 译注: 没太明白这个选项有什么作用, 待测试后理解了再补充.



当使用Rebot来 :ref:`combining outputs` 时, 可以通过设定选项 :option:`--starttime` 和 :option:`--endtime` 来分别指定组合测试套件的开始和结束时间.

默认情况下, 组合的测试套件不包含这两个值. 当给定了这两个值之后, 整个执行的消耗时间也就基于此来计算. 否则只能通过累加子套件的耗时来计算.

该选项也可以用在单个测试套件上, 同样也会影响到测试套件的消耗时间.

时间戳的格式为必须是 ``YYYY-MM-DD hh:mm:ss.mil``, 所有的分隔符都是可选的, 毫秒到小时的部分也是可以忽略的. 例如, ``2008-06-11 17:59:20.495`` 和 ``20080611-175920.495`` 和 ``20080611175920495`` 都是等价的, 仅 ``20080611`` 也可以.

示例::

   rebot --starttime 20080611-17:59:20.495 output1.xml output2.xml
   rebot --starttime 20080611-175920 --endtime 20080611-180242 *.xml
   rebot --starttime 20110302-1317 --endtime 20110302-11418 myoutput.xml

.. _pre-Rebot modifier:
.. _programmatic modification of results:

结果的编程式修改
-------------------

如果内置的修改测试结果的功能无法满足需求,  Robot Framework 2.9 以及更新的版本提供了编程式地自定义修改途径. 使用 :option:`--prerebotmodifier` 选项来创建一个模型修改器(model modifier).

该功能的原理和使用 :option:`--prerunmodifier` 选项来启用 :ref:`programmatic modification of test data` 几乎一样. 明显的区别在于这次修改器操作的是 :ref:`result model` 而不是 :ref:`running model`.

例如, 下面的修改器将所有虽然执行通过但是耗时超过一定限制的测试用例标记为失败:

.. sourcecode:: python

    from robot.api import SuiteVisitor


    class ExecutionTimeChecker(SuiteVisitor):

        def __init__(self, max_seconds):
            self.max_milliseconds = float(max_seconds) * 1000

        def visit_test(self, test):
            if test.status == 'PASS' and test.elapsedtime > self.max_milliseconds:
                test.status = 'FAIL'
                test.message = 'Test execution took too long.'

假设上面的修改器是保存在文件 :file:`ExecutionTimeChecker.py` 之中, 则像这样指定::

    # Specify modifier as a path when running tests. Maximum time is 42 seconds.
    robot --prerebotmodifier path/to/ExecutionTimeChecker.py:42 tests.robot

    # Specify modifier as a name when using Rebot. Maximum time is 3.14 seconds.
    # ExecutionTimeChecker.py must be in the module search path.
    rebot --prerebotmodifier ExecutionTimeChecker:3.14 output.xml

如果需要应用多个模型修改器, 则 :option:`--prerebotmodifier` 可以指定多次.

当执行测试时, :option:`--prerunmodifier` 和 :option:`--prerebotmodifier` 可以同时使用.


.. _system log:

系统日志
----------

Robot Framework 有自己的系统日志, 其中包含的信息主要关于:

   - 被处理和被跳过的测试数据文件
   - 导入的测试库, 资源文件和变量文件
   - 执行的测试套件和测试用例
   - 创建的输出

通常情况下用户永远不需要知道这些信息, 只有在定位测试库的问题, 亦或者是Robot Framework自身的问题时, 才显得有用.

系统日志默认并不会创建, 通过设置环境变量 ``ROBOT_SYSLOG_FILE`` 的值为包含日志的路径名来启用该日志.

系统之日和普通的日志文件拥有一样的 :ref:`log levels`, 例外在于没有 ``FAIL`` 级别, 取而代之的是 ``ERROR`` 级别. 系统日志级别通过环境变量 ``ROBOT_SYSLOG_LEVEL`` 来指定.

可能的 :ref:`未预期的错误和警告 <errors and warnings during execution>` 除了写入控制台和普通日志文件外也会写入系统日志.

.. sourcecode:: bash

   #!/bin/bash

   export ROBOT_SYSLOG_FILE=/tmp/syslog.txt
   export ROBOT_SYSLOG_LEVEL=DEBUG

   robot --name Syslog_example path/to/tests
