.. _configuring execution:

测试执行的配置项
=================

本章将介绍用于配置 :ref:`测试执行 <test execution>` 或 :ref:`测试输出的处理 <post-processing outputs>` 的各种命令行选项. 与生成output文件相关的选项将在 :ref:`下一章 <created outputs>` 讨论.


.. contents::
   :depth: 2
   :local:

.. _selecting test cases:

选择测试用例
-------------

Robot Framework提供了若干命令行选项用于选择测试用例来执行. 这些选项同样可以在使用 Rebot_ 处理测试输出时使用. 

.. _by test suite and test case names:

根据套件和用例名称
~~~~~~~~~~~~~~~~~~

测试套件和测试用例可以按照它们的名字来选择, 分别用到的选项是 :option:`--suite (-s)` 和 :option:`--test (-t)`. 这些选项都可以多次指定, 用来选择多个套件或用例. 选项的参数是大小写无关并且忽略空格, 同时, 还可以使用 :ref:`简单模式` 来匹配多个名字.

如果同时使用了选项 :option:`--suite` 和 :option:`--test`, 则只有匹配套件内的匹配用例才会被选中.

::

  --test Example
  --test mytest --test yourtest
  --test example*
  --test mysuite.mytest
  --test *.suite.mytest
  --suite example-??
  --suite mysuite --test mytest --test your*

使用选项 :option:`--suite` 和执行相应的测试用例文件或目录差不多是一样的. 
通过选项指定套件的一个最大的好处是可以基于父套件来选择套件. 使用时将父子套件名称之间用点(``.``)来连接. 这种情况, 如果父套件有setup和teardown, 则会执行.

::

  --suite parent.child
  --suite myhouse.myhousemusic --test jack*

通过 :option:`--test` 选项来选择单个的测试用例执行在创建测试用例时很实用, 但是在自动化执行时作用有限. 通常情况下, 通过标签来选择用例更加灵活.

.. _by tag names:

根据标签
~~~~~~~~~

分别使用 :option:`--include (-i)` 和 :option:`--exclude (-e)` 选项可以指定要包含和排除的 :ref:`标签 <tag>` 名字.
使用 :option:`--include`, 只有那些标签匹配上的测试用例才会选中, 而使用 :option:`--exclude` 则相反. 如果两个选项同时出现, 只有那些两者同时满足的用例会被选中.

::

   --include example
   --exclude not_ready
   --include regression --exclude long_lasting

:option:`--include` 和 :option:`--exclude` 都可以指定多次. 这种情况下, 被选中的用例是: 有任意一个标签匹配上任意的包含的标签, 同时, 没有任何标签匹配上排除标签.


除了指定完全匹配某个标签, 还可以使用 :ref:`标签模式 <tag patterns>`, 即使用 ``*`` 和 ``?`` 这些通配符, 以及 ``AND``, ``OR``, 和 ``NOT`` 这些逻辑操作符来组合标签::

   --include feature-4?
   --exclude bug*
   --include fooANDbar
   --exclude xxORyyORzz
   --include fooNOTbar

通过标签来选择用例是非常灵活的机制, 由此可以实现很多有趣的功能:

- 一部分测试子集要在其它测试执行前先执行, 这个子集通常也称作冒烟测试, 可以被打上标签
  ``smoke``, 然后通过 ``--include smoke`` 执行.

- 还未完成的测试用例, 可以在提交到版本控制系统的时候打上标签 ``not_ready``, 执行时指定
  ``--exclude not_ready`` 以避免执行到.

- 敏捷开发中, 测试用例可以打上 ``sprint-<num>`` 标签, 其中 ``<num>`` 表示某次迭代, 
  当所有用例执行完成, 可以针对这轮特定的迭代生成单独的报告
  (例如: ``rebot --include sprint-42 output.xml``).

.. _Re-executing failed test cases:

重新执行失败的用例
~~~~~~~~~~~~~~~~~~

命令行选项 :option:`--rerunfailed (-R)` 可被用来从上次执行的 :ref:`output file` 中抽取所有失败的用例重新执行. 该选项是非常有用的, 例如, 把所有用例全部执行一遍很耗时间, 这样就可以迭代地修复和执行那些失败的用例.

::

  robot tests                             # first execute all tests
  robot --rerunfailed output.xml tests    # then re-execute failing

该选项在幕后的工作就是把失败的用例挑选出来, 就和使用 :option:`--test` 一样. 同时它还可以和其它选择用例的选项 :option:`--test`, :option:`--suite`, :option:`--include` 和 :option:`--exclude` 结合使用以达到微调的效果.

如果选项指定的output文件并不是出自当前要执行的测试的话, 将会导致不可预测的结果. 此外, 如果output中没有失败的用例, 则会报错. 使用特殊的 ``NONE`` 作为output文件值的话, 其效果等同于不指定该选项.

.. tip:: 重新执行的结果和初始的结果可以通过命令行选项 :option:`--merge` :ref:`合并 <merging outputs>`

.. note:: 重新执行失败的用例是Robot Framework 2.8版本的新特性功能.
          在Robot Framework 2.8.4版本之前的选项名是 :option:`--runfailed`.
          旧的名字仍在用, 但是将在以后去除掉.

.. _When no tests match selection:

当没有用例匹配选择的情况
~~~~~~~~~~~~~~~~~~~~~~~~

默认情况下, 如果没有任何用例匹配选择标准, 测试执行将以失败告终, 报错如::

    [ ERROR ] Suite 'Example' with includes 'xxx' contains no test cases.

因为没有输出文件生成, 所以此种行为在自动化执行和处理测试的时候将会是个麻烦. 幸好有另一个命令行选项 :option:`--RunEmptySuite` 可被用来强制要求测试套件在这种情况下也正常执行. 

该选项也可用在执行空目录或者空的测试用例文件, 效果都是一样.

相似的情形在使用 Rebot_ 处理输出文件时也存在, 例如没有任何用例匹配到过滤规则, 或者output文件中就没有任何用例. 默认在此种情况下, Rebot的执行也会报错. 选项 :option:`--ProcessEmptySuite` 可用来改变这个行为. 该选项和在测试运行时使用的 :option:`--RunEmptySuite` 作用一样.

.. note:: :option:`--ProcessEmptySuite` 是 Robot Framework 2.7.2版本新加功能.

.. _Setting criticality:

设置关键性
----------

测试执行的最终结果取决于关键(critical)用例的结果. 如果任意一个关键用例失败, 则整个测试被认为失败. 反之, 非关键(non-critical)测试用例的失败不会影响整个测试的状态.

所有的测试用例默认都是关键的, 不过可以通过选项 :option:`--critical (-c)` 和 :option:`--noncritical (-n)` 来设置. 这些选项基于标签来指定哪些用例是关键的, 类似于 :option:`--include` 和 :option:`--exclude` :ref:`通过标签选择用例 <by tag names>`.

如果只使用 :option:`--critical`, 标签匹配上的用例是关键的. 如果单使用 :option:`--noncritical`, 则标签没有匹配上的用例是关键的. 最后, 如果两个都设置了, 则只有那些既匹配了critical标签, 又没有匹配non-critical标签的用例被视作关键的.

这两个选项和 :option:`--include` 和 :option:`--exclude` 一样也支持 :ref:`标签模式` 的用法. 即大小写无关, 空格和下划线无关, 支持 ``*`` 和 ``?`` 通配符, ``AND``, ``OR`` 和 ``NOT`` 运算符的模式匹配规则.

::

  --critical regression
  --noncritical not_ready
  --critical iter-* --critical req-* --noncritical req-6??

设置关键性的最常用处是在测试用例未完全就绪时, 或者测试执行时测试特性仍在开发中. 当然, 你可以使用 :option:`--exclude` 选项将这些用例在执行时排除在外, 但是将它们作为非关键的用例进行执行可以让你看到它们何时转为pass.

测试执行时设置的关键性没有在任何地方保存. 如果要在Rebot :ref:`post-processing outputs` 时也保持相同的关键性, 需要同时也使用相同的 :option:`--critical` 和/或 :option:`--noncritical` 选项::

  # Use rebot to create new log and report from the output created during execution
  robot --critical regression --outputdir all tests.robot
  rebot --name Smoke --include smoke --critical regression --outputdir smoke all/output.xml

  # No need to use --critical/--noncritical when no log or report is created
  robot --log NONE --report NONE tests.robot
  rebot --critical feature1 output.xml


.. _Setting metadata:

设置元数据
----------

.. _Setting the name:

设置名字
~~~~~~~~

Robot Framework 解析测试数据时, :ref:`测试套件的名字 <test suite name and documentation>` 是根据用例文件和目录创建而来的. 而顶层的测试套件名字可以通过命令行选项 :option:`--name (-N)` 指定. 给定名字中的下划线将自动转为空格, 并且其中的单词将转为首字母大写.


.. _Setting the documentation:

设置文档
~~~~~~~~~~

除了可以在测试数据中 :ref:`定义文档 <test suite name and documentation>`, 顶层测试套件的文档还可以通过命令行选项 :option:`--doc (-D)` 给出. 文档中的下划线将转为空格, 并且文档中可以包含简单的 :ref:`HTML formatting`.


.. _setting free metadata:

设置自由元数据
~~~~~~~~~~~~~~

:ref:`Free test suite metadata` 可以通过命令行选项 :option:`--metadata (-M)` 给出. 该选项的参数格式是 ``name:value``, 其中 ``name`` 是要设置的元数据的名字, ``value`` 是值.
名字和值中包含的下划线会被转为空格, 并且值可以包含简单的 :ref:`HTML formatting`.

该选项可以出现多次以设置多个元数据

.. _Setting tags:

设置标签
~~~~~~~~~~~~

命令行选项 :option:`--settag (-G)` 可用来为所有测试用例设置给定的标签. 该选项也可以使用多次以设置多个标签.

.. _module search path:
.. _Configuring where to search libraries and other extensions:

配置模块搜索路径
-----------------

当Robot Framework导入 :ref:`test library <specifying library to import>`, :ref:`listener <setting listeners>`, 或其它基于Python的扩展的时候, 它要用Python解释器从系统中导入包含扩展内容的模块(module).
其中查找模块的一系列的位置被称之为 *模块搜索路径*, 本节将介绍几种不同的方法来配置. 当导入基于Java的库或Jython的扩展时, 除了正常的模块搜索路径, 还要加上Java的类路径(classpash).

Robot Framework在导入 :ref:`资源和变量文件` 时, 如果指定的路径不能直接匹配到文件路径, 则也会用到Python的模块搜索路径.

模块搜索路径的设置正确, 测试库和扩展才能被找到, 这是测试执行成功的必要条件. 如果你想要通过下面的方法自定义模块搜索路径, 则创建一个自定义的 :ref:`start-up script` 是个不错的选择.


.. _Locations automatically in module search path:

自动包含在模块搜索路径中的位置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Python解释器的标准库和安装的第三方库最终都是在模块搜索路径内. 也就是说, 使用 :ref:`Python打包系统打包 <packaging libraries>` 的测试库是自动安装到模块搜索路径内的, 可以直接导入.



``PYTHONPATH``, ``JYTHONPATH`` and ``IRONPYTHONPATH``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Python, Jython和IronPython分别从环境变量 ``PYTHONPATH``, ``JYTHONPATH`` 和
``IRONPYTHONPATH`` 中读取需要加入到模块搜索路径中的位置. 
无论用到哪个环境变量, 如果你想要指定多个位置, 需要在类UNIX系统中使用冒号分隔(``/opt/libs:$HOME/testlibs``), 而在Windows系统中使用分号分隔(``D:\libs;%HOMEPATH%\testlibs``).

环境变量可以在系统级别设置为永久生效, 也可以针对某个用户有效. 同时还可以使用命令来临时设置, 这点可以在 :ref:`start-up scripts` 得到非常好的应用.

.. note:: 在Robot Framework 2.9之前, 当使用Jython和IronPython运行时, 
          ``PYTHONPATH`` 环境变量中的内容被框架自己加入到模块搜索路径中.
          现在则不会了, 必须分别使用 ``JYTHONPATH`` 和 ``IRONPYTHONPATH``.


``--pythonpath`` 选项
~~~~~~~~~~~~~~~~~~~~~

Robot Framework提供了一个单独的命令行选项 :option:`--pythonpath (-P)` 用来将位置加入到模块搜索路径. 虽然该选项名称中包含的是python, 它对Jython和IronPython也同样有用.

不管在何种操作系统下, 多个位置都使用冒号来分隔, 或者可以多次使用该选项. 
给定的路径名称可以使用glob的模式匹配多个路径, 但是通配符必须要 :ref:`转义 <escaping complicated characters>`.


例如::

   --pythonpath libs
   --pythonpath /opt/testlibs:mylibs.zip:yourlibs
   --pythonpath mylib.jar --pythonpath lib/STAR.jar --escape star:STAR


程序设置 ``sys.path``
~~~~~~~~~~~~~~~~~~~~~~~

Python解释器把模块搜索路径以字符串列表的形式存储在 :ref:`sys.path <https://docs.python.org/2/library/sys.html#sys.path> <>`. 该属性可以在程序执行过程中动态地更新, 改动将在下次需要导入某个模块的时候起效.

.. _Java classpath:

Java的类路径
~~~~~~~~~~~~~~

当用Jython导入Java实现的库时, 这些库的位置既可以是在Jython的模块搜索路径, 也可以是在 :ref:`Java classpath <https://docs.oracle.com/javase/8/docs/technotes/tools/findingclasses.html>`. 设置classpath的最常见方式是通过环境变量 ``CLASSPATH``, 具体和 ``PYTHONPATH``, ``JYTHONPATH`` 或 ``IRONPYTHONPATH`` 类似.

或者, 可以使用Java的选项 :option:`-cp`. 该选项不属于 ``robot`` :ref:`runner script`, 但是可以在使用Jython时通过选项前缀 :option:`-J` 来启用, 例如::

  jython -J-cp example.jar -m robot.run tests.robot

当使用独立的JAR包时, classpath的设置有些许不同, 因为 `java -jar` 命令既不支持环境变量 ``CLASSPATH`` 也不支持 :option:`-cp` 选项. 有两种不同的方法来解决::

  java -cp lib/testlibrary.jar:lib/app.jar:robotframework-2.9.jar org.robotframework.RobotFramework tests.robot
  java -Xbootclasspath/a:lib/testlibrary.jar:lib/app.jar -jar robotframework-2.9.jar tests.robot


.. _Setting variables:

设置变量
--------

:ref:`variables` 既可以使用 :option:`--variable (-v)` 选项 :ref:`个别 <setting variables in command line>` 设置, 也可以使用 :option:`--variablefile (-V)` 选项通过 :ref:`variable file` 批量设置. 关于变量和变量文件在其它章节介绍, 下面的例子展示了如何使用这些命令行选项::

  --variable name:value
  --variable OS:Linux --variable IP:10.0.0.42
  --variablefile path/to/variables.py
  --variablefile myvars.py:possible:arguments:here
  --variable ENVIRONMENT:Windows --variablefile c:\resources\windows.py


.. _Dry run:

空运行(Dry run)
--------------

Robot Framework支持所谓的 *空运行* 模式, 这种模式下测试用例的运行和正常一样, 只是测试库中的关键字不执行. 该模式可以用于验证测试数据, 如果空运行通过, 则数据应该是语法正确的. 使用 :option:`--dryrun` 选项即可启用该模式.

空运行模式的执行可能会因为以下原因而失败:

  * Using keywords that are not found.
  * Using keywords with wrong number of arguments.
  * Using user keywords that have invalid syntax.

除了这些失败, 正常的 :ref:`执行错误 <errors and warnings during execution>` 也会有提示. 例如, 当测试库或资源文件无法导入时.

.. note:: 空运行模式不校验变量. 这点限制可能会在未来版本中有所提升.


.. _Randomizing execution order:

执行顺序随机化
--------------

测试执行的顺序可以通过选项 :option:`--randomize <what>[:<seed>]` 随机化, 这其中的 ``<what>`` 可能是以下几种:

``tests``
    每个测试套件内的用例按随机顺序执行

``suites``
    所有的测试套件按随机顺序执行, 但是套件内的测试用例还是按照它们定义的顺序执行.

``all``
    测试套件和测试用例都按照随机顺序执行.

``none``
    测试套件和测试用例都不会随机执行. 该值可被用来覆盖掉前面设置的 :option:`--randomize` 选项.

从Robot Framework 2.8.5版本开始, 还可以给定一个自定义的随机种子(seed)来初始化随机生成器. 该种子作为选项 :option:`--randomize` 的值给出, 格式为 `<what>:<seed>`, 必须是整数. 如果没有给定种子, 则随机生成. 

被执行的顶层测试套件自动获得了名为 :name:`Randomized` 的 :ref:`元数据 <free test suite metadata>`, 可通过其知晓什么被随机化以及随机种子是多少.

Examples::

    robot --randomize tests my_test.robot
    robot --randomize all:12345 path/to/tests


.. _pre-run modifier:
.. _Programmatic modification of test data:

测试数据编程修改
----------------

如果Robot Framework内置的在执行前修改测试数据的功能不够用, 从2.9版本开始, 可以通过编程的方法来自定义修改. 这可以通过创建一个模型修改器(model modifier)并使用选项 :option:`--prerunmodifier` 来激活使用它.

模型修改器被实现为观察者(visitor), 可以遍历可执行的测试套件结构, 并且按需修改. 观察者接口在 :ref:`Robot Framework API documentation <visitor interface_>` 中有所说明. 使用它可以修改 :ref:`test suites <running.TestSuite>`, :ref:`test cases <running.TestCase>` 和 :ref:`keywords <running.Keyword>`. 

下面的例子展示了如何使用模型修改器, 以及该功能的强大之处.

.. sourcecode:: python

   ../code_examples/select_every_xth_test.py

当在命令行中使用 :option:`--prerunmodifier` 选项来指定一个模型修改器时, 既可以使用修改器的类名, 也可以是修改器的源文件. 如果是类名, 则包含该类的模块必须在 :ref:`模块搜索路径 <module search path>`, 并且如果模块名和类名不同, 则名称还必须包含模块名, 如 ``module.ModifierClass``. 如果是文件路径, 则类名必须和文件名一样. 大部分情况和 :ref:`指定导入库 <specifying library to import>` 一样.

如果一个修改器需要参数, 如上例, 则参数值跟在修改器的名字或路径后面给出, 使用冒号(`:`)或分号(`;`)来分隔. 如果同时出现了冒号和分号, 则最先出现的那个符号被视为分隔符.

例如, 如果上述模型修改器在文件 :file:`SelectEveryXthTest.py` 中定义, 则可以这样用::

    # 通过文件路径指定修改器. 每次隔1个来运行测试.
    robot --prerunmodifier path/to/SelectEveryXthTest.py:2 tests.robot

    # 通过名称指定修改器. 从第2个开始, 每次隔2个运行的测试用例.
    # SelectEveryXthTest.py 必须在模块搜索路径内.
    robot --prerunmodifier SelectEveryXthTest:3:1 tests.robot

如果需要使用多个模型修改器, 则可以通过多次使用 :option:`--prerunmodifier` 选项来指定. 如果类似的变更需要在创建测试结果前执行,  则可以使用 :option:`--prerebotmodifier` 选项来启用 :ref:`编程修改结果`.

If more than one model modifier is needed, they can be specified by using
the :option:`--prerunmodifier` option multiple times. If similar modifying
is needed before creating results, `programmatic modification of results`_
can be enabled using the :option:`--prerebotmodifier` option.

__ `Specifying library to import`_

.. _Controlling console output:

控制台输出
-----------

有多个命令行选项可用来设置测控制台的报告输出.

.. _console output type:

控制台输出类型
~~~~~~~~~~~~~~~

大致的控制台输出类型通过 :option:`--console` 选项来设置. 支持以下大小写无关的几个值:

``verbose``
    每个测试套件和测试用例分别报告. 这是默认的情形.

``dotted``
    仅用点号 `.` 表示测试通过的用例, `f` 表示失败的非重要用例, `F` 表示失败的重要用例, `x` 表示由于 :ref:`测试执行退出 <stopping test execution gracefully>` 而跳过的用例. 失败的重要用例在执行后单独列出. 这种输出类型可以很容易地分辨出那些执行失败的测试用例, 即使用例数量很多. 

``quiet``
    除了 :ref:`错误和警告` 没有其它输出.

``none``
    没有任何输出. 当创建自定义输出时(例如, 监听器_)很有用.


选项 :option:`--dotted (-.)` 和 :option:`--quiet` 分别是 `--console dotted` 和 `--console quiet` 的快捷选项.

Examples::

    robot --console quiet tests.robot
    robot --dotted tests.robot

.. note:: :option:`--console`, :option:`--dotted` 和 :option:`--quiet` 是 
          Robot Framework 2.9新增特性. 早期版本的输出总是相当于当前的 `verbose` 模式.

.. _Console width:

控制台宽度
~~~~~~~~~~

使用选项 :option:`--consolewidth (-W)` 来设置控制台输出的宽度. 默认的值是78个字符.

.. tip:: 在很多类UNIX系统上, 可以方便地利用环境变量 `$COLUMNS`,
         例如, `--consolewidth $COLUMNS`.

.. note:: 在Robot Framework 2.9之前, 该功能通过 :option:`--monitorwidth` 选项
          启用, 目前已经废弃并去除. 而短选项 :option:`-W` 在所有版本中都一样用.

.. _Console colors:

控制台颜色
~~~~~~~~~~

选项 :option:`--consolecolors (-C)` 用来设置是否在控制台输出中使用颜色. 除了在Windows中是使用的Windows API, 其它系统中颜色是通过 :ref:`ANSI colors <http://en.wikipedia.org/wiki/ANSI_escape_code>` 实现的. 在Jython中不能调用Windows的这些API, 所以在Windows中使用Jython是不支持颜色的.

该选项支持以下大小写无关的几个值:

``auto``
    当输出到控制台时启用颜色, 但是当重定向到文件或其它地方则没有颜色. 这是默认的情形.

``on``
    当输出重定向时也使用颜色. 在Windows中没有效果.
    Colors are used also when outputs are redirected. Does not work on Windows.

``ansi``
    和 `on` 一样. 但是在Windows中也同样使用ANSI颜色. 这在重定向输出到某个程序, 而该程序可以理解ANSI时会非常有用. 这是Robot Framework 2.7.5出现的新功能.

``off``
    不使用颜色.

.. note:: 在Robot Framework 2.9之前, 该功能通过 :option:`--monitorcolors` 选项
          启用, 目前已经废弃并去除. 而短选项 :option:`-C` 在所有版本中都一样用.


.. _Console markers:

控制台标记
~~~~~~~~~~~

从Robot Framework 2.7版本开始, 当控制台使用 :ref:`verbose输出 <console output type>` 时, 当测试用例中的顶层关键字执行结束时, 控制台中会显示特殊的标记 `.` (成功) 和
`F` (失败). 这些标记让我们可以从高层次跟踪测试的执行情况, 并且它们在测试用例执行结束后自动清除掉.

从Robot Framework 2.7.4版本开始, 通过 :option:`--consolemarkers (-K)` 选项可以配置合适使用这些标记. 该选项支持以下大小写无关的几个值:

``auto``
    标记在标准输出到控制台时启用, 但是当重定向到文件和其它地方时不出现. 这是默认的情形.

``on``
    始终使用标记.

``off``
    禁用标记.

.. note:: 在Robot Framework 2.9之前, 该功能通过 :option:`--monitormarkers` 选项
          启用, 目前已经废弃并去除. 而短选项 :option:`-K` 在所有版本中都一样用.


.. _setting listeners:

设置监听器
----------

监听器_ 本用来监控测试执行. 当在命令行中使用它们时, 通过选项 :option:`--listener` 来指定. 该选项的值既可以是监听器的文件路径, 也可以是监听器的名字. 详情参见 :ref:`Listener interface` 章节.
