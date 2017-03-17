.. _rebot:
.. _post-processing outputs:

测试输出的处理
==============

测试执行过程中生成的 :ref:`XML output files` 可以使用Rebot工具来处理, 该工具集成到了Robot Framework中. 它在测试执行时自动调用生成测试报告和日志文件, 也可以单独使用, 生成自定义的报告和日志, 或者用来将测试结果组合合并在一起.

.. contents::
   :depth: 2
   :local:

.. Using Rebot

使用Rebot
-----------

.. Synopsis

概要
~~~~~

::

    rebot [options] robot_outputs
    python|jython|ipy -m robot.rebot [options] robot_outputs
    python|jython|ipy path/to/robot/rebot.py [options] robot_outputs
    java -jar robotframework.jar rebot [options] robot_outputs

如上所列, 使用Rebot最常见的用法是调用 :ref:`执行脚本 <runner script>`: ``rebot``. 同时可以使用Python解释器直接调用模块 :ref:`robot.rebot module <executing installed robot module>` 或 :ref:`robot/rebot.py 文件 <executing installed robot directory>`. 最后还可以通过Java调用 :ref:`standalone JAR distribution`.

.. note::
    Robot Framework 3.0 版本之前, 安装的 ``rebot`` 脚本只用于Python解释器,
    Jython和IronPython对应的脚本分别是 ``jyrebot`` 和 ``ipyrebot``.
    目前这两个脚本也还是会安装, 不过未来的计划是逐渐淘汰并最终删除.

.. Specifying options and arguments

指定选项和参数
~~~~~~~~~~~~~~~

使用Rebot的基础语法完全等同于 :ref:`执行测试 <executing test cases>`, 并且大部分的命令行选项也都相同. 主要的区别就是Rebot的执行参数是 :ref:`XML output files` 而不是测试文件或目录.


.. Return codes with Rebot

返回码
~~~~~~

Rebot的返回码和 :ref:`执行测试的返回码 <return codes>` 一样.


.. Creating different reports and logs

创建不同的报告和日志
--------------------

可以调用Rebot生成和测试结束后自动生成的一样的报告和日志, 显然这样做意义不大. 但是如果想要有选择性的生成报告, 例如, 一份全部用例的报告和一份用例子集的报告, 这时就很有用::

   rebot output.xml
   rebot path/to/output_file.xml
   rebot --include smoke --name Smoke_Tests c:\results\output.xml

另一个常见用途是在运行测试时只生成output文件(使用选项 ``--log NONE --report NONE``), 把生成日志和报告文件放在后面再执行. 例如, 测试可以在不同的环境执行, 最终把output文件集中汇总到一个地方, 然后在那里创建报告和日志. 如果使用Jython运行测试生成报告和日志耗时较长也可以这样做. 总之, 在执行时禁止日志和报告生成, 改为事后生成, 可以节省很多时间, 同时也占用更少内存.

.. Combining outputs

联合输出
--------

Rebot的一个重要的特性功能是能够将不同的测试轮次的输出联合(combine)起来. 例如, 同样的测试用例在不同的环境下运行多次, 最终联合所有的output文件汇总输出一份报告.

联合输出非常简单, 只要将需要联合在一起的多个output文件一起作为参数传递即可::

   rebot output1.xml output2.xml
   rebot outputs/*.xml

当联合输出时, 将自动创建一个新的顶层测试套件, 把要联合的套件都作为子套件包含在内. 
这和 :ref:`一次执行多个测试文件或目录 <specifying test data to be executed>` 是一样的, 也会使用 ``&`` 和空格来连接各子套件的名字, 生成这个新顶层套件的名字. 可以想象这个自动生成的名字通常不会很好看, 所以最好还是通过选项 
:option:`--name` 指定一个更有意义的名字::

   rebot --name Browser_Compatibility firefox.xml opera.xml safari.xml ie.xml
   rebot --include smoke --name Smoke_Tests c:\results\*.xml

__ `Specifying test data to be executed`_


合并输出
--------

如果同样的测试再次执行(re-execute), 或者某一个套件分开执行(executed in pieces), 使用上述的联合输出结果创建顶层测试套件是没必要的. 这种情况下, 更好的做法是将所有结果合并(merge)到一起.

使用 :option:`--merge` 选项来告诉Rebot将多个output文件合并来, 该选项不带参数, 其它参数的使用则和正常情况下一样::

   rebot --merge --name Example --critical regression original.xml merged.xml

在实践中合并是如何使用的两个主要场景将在下面章节中讨论.

.. Merging re-executed tests

合并重新执行的用例
~~~~~~~~~~~~~~~~~~

有时候需要重新执行测试用例的一个子集, 例如, 当bug(可能是待测系统的, 也可能是测试本身的bug)修复后的再次执行. 这时可以 :ref:`选择性的执行用例`, 通过名字(:option:`--test` 和 :option:`--suite` 选项), 或通过标签(:option:`--include` 和 :option:`--exclude`), 或通过前次执行状态(:option:`--rerunfailed`).

使用默认的 :ref:`联合输出` 报告的做法在这种情况下有些不妥, 主要问题是组合结果是分开的套件, 并且可能已经修复的失败仍然列在其中. 这种情况使用 :option:`--merge (-R)` 更合适.

这样merge的结果是, 同一个用例, 后面的执行结果将替代前面的. 使用个实际的例子更容易解释清楚, 下面同时用到了 :option:`--rerunfailed` 和 :option:`--merge`::

  robot --output original.xml tests                          # first execute all tests
  robot --rerunfailed original.xml --output rerun.xml tests  # then re-execute failing
  rebot --merge original.xml rerun.xml                       # finally merge results

合并后的测试结果消息将包含一个标注, 提示结果被替代了. 同时该消息还将展示该测试旧的状态和消息.

合并结果必须有相同的顶层测试套件. 那些要合并的用例和套件, 如果在原始output中没找到, 则会追加到结果中. 实际情况在下面章节中讨论.

.. note:: 合并重新执行的结果是Robot Framework 2.8.4新特性功能.
          在Robot Framework 2.8.6之前, 合并使用的是已被废弃的选项 :option:`--rerunmerge` 实现, 同时新的测试和套件在合并输出中会被略过.


.. Merging suites executed in pieces

合并零碎执行的测试套件
~~~~~~~~~~~~~~~~~~~~~~

合并选项 :option:`--merge` 的另一个重要用途是将一个零碎执行的测试套件的结果合并起来. 例如, 一个套件分使用 :option:`--include` 和 :option:`--exclude` 执行::

    robot --include smoke --output smoke.xml tests   # first run some tests
    robot --exclude smoke --output others.xml tests  # then run others
    rebot --merge smoke.xml others.xml               # finally merge results

合并后, 最终的output文件将包含所有测试用例和测试套件的结果. 如果某些用例多次出现, 则后面的会覆盖前面的(如上节). 同样, 这种合并策略要求所有output文件的顶层测试套件是同一个.
