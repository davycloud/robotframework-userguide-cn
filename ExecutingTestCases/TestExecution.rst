.. Test execution

测试执行
==============

本章将介绍解析测试数据后而创建的测试套件是如何执行的, 如何在发生失败后继续执行测试用例, 以及如何优雅地结束整个测试执行.

.. contents::
   :depth: 2
   :local:

.. Execution flow

执行流程
--------------

.. Executed suites and tests

测试套件和用例
~~~~~~~~~~~~~~~~~~~~~~~~~

测试用例总是在测试套件内执行. 从 `测试用例文件`_ 创建的测试套件直接包含测试用例, 而从 目录__ 创建的测试套件包含子套件, 子套件中包含测试用例或者它们自己的子套件. 默认情况下, 一个被执行的测试套件内的所有用例都会运行, 不过可以使用某些选项 :option:`--test`, :option:`--suite`, :option:`--include` 和 :option:`--exclude` 来 `选择用例`__. 其中用例数量为零的测试套件将被忽略.

测试执行始于顶层测试套件. 如果套件中包含用例, 这些用例将逐个执行, 如果套件包含子套件, 则以深度优先的顺序递归地执行. 

当单个用例执行时, 用例内的关键字按顺序执行. 通常任意关键字运行失败都将导致当前用例结束, 不过也可以通过设置 `continue after failures`__.

精确的 `执行顺序`_ 以及 `Setups和Teardowns`_ 如何影响执行将在下面的章节讨论.

__ `Test suite directories`_
__ `Selecting test cases`_
__ `Continue on failure`_


.. Setups and teardowns

Setups和Teardowns
~~~~~~~~~~~~~~~~~~~~

Setups和Teardowns可以用在 `测试套件`__, `测试用例`__ 和 `用户关键字`__ 级别.

__ `Test setup and teardown`_
__ `Suite setup and teardown`_
__ `User keyword teardown`_

.. Suite setup

套件setup
'''''''''''

如果一个测试套件有setup, 它将在所有用例和子套件执行前被执行. 如果该setup通过, 测试流程正常继续. 如果该setup失败, 则该套件内的所有用例(包括其中子套件内的用例)统统都标记为失败. 这些用例, 以及其中可能包含的setup和teardown都不会再执行.

套件的setup通常被用来设置测试环境. 因为一旦套件的setup失败, 其中所有用例不会执行, 所以可以很方便地使用套件的setup来检验测试环境是否已经满足了可测试条件.

.. Suite teardown

套件teardown
''''''''''''''

测试套件的teardown将在套件内所有用例(包括其中子套件内的用例)执行完毕后再执行. 套件teardown的执行条件不受测试状态的影响, 即使套件的setup执行失败, teardown也会执行.

如果teardown失败, 所有的用例都将在随后的测试报告和日志中标记为失败.

套件的teardown经常被用作最后清理测试环境的步骤. 为了确保所有的任务都已经结束, teardown内的所有关键字都会被执行, 即使其中有失败的情况.

__ `Continue on failure`_

.. Test setup

用例setup
''''''''''

用例setup在用例内的关键字执行前被执行. 如果该setup失败, 则所有的关键字都不会再执行. 用例的setup最大的用途是为特定的用例设置测试环境.

.. Test teardown

用例teardown
'''''''''''''

用例teardown在用例执行完毕后被执行. 它的执行也不受用例执行状态的影响, 即不管用例是成功还是失败, teardown总是会执行.

和套件的teardown类似, 用例的teardown也主要用来做清理工作, 同样其中的所有关键字都会执行到, 即使其中有失败的情况.

.. Keyword teardown

关键字teardown
''''''''''''''''

`用户关键字`_ 没有setup, 只能设置teardown, 其作用和其它类型的teardown一样. 关键字的teardown在关键字执行后被执行, 不管关键字执行状态如何, teardown最终会完全执行即使其中有关键字失败的情况.

.. Execution order

执行顺序
~~~~~~~~~~~~~~~

一个测试套件文件内的测试用例的执行顺序就是其在文件中定义的顺序. 高层套件内的多个子套件的执行顺序是子套件文件或目录名称按字母顺序, 不分大小写.
如果通过命令行指定多个文件和(或)目录, 则它们将以给出的顺序执行.

如果想让某个目录内的测试套件按特定顺序执行, 可以为文件或目录名称加上前缀, 例如: :file:`01` 和 :file:`02`. 如果将前缀和基础名称中间用2个下划线连接, 则最终生成的测试套件名称不会包含前缀, 例如::

   01__my_suite.html -> My Suite
   02__another_suite.html -> Another Suite

如果按照字母顺序排列还有问题, 还有一个解决方案是将它们按想要的顺序依次分别列出. 显然在命令行中列出会导致启动命令超长, 不过使用 `argument files`_ 就刚好解决问题, 其中每行列出一个文件.

除了固定的顺序外, 还可以使用 :option:`--randomize` 选项使 `执行顺序随机化`__.

__ `Randomizing execution order`_

.. Passing execution

跳过执行
~~~~~~~~~~~~~~~~~

通常情况下, 用例以及setup和teardown执行通过的标准是其中包含的所有的关键字都执行无错. 从Robot Framework 2.8版本开始, 还可以通过 BuiltIn_ 关键字 :name:`Pass Execution` 和 :name:`Pass Execution If` 以PASS状态结束执行, 同时跳过剩下的关键字.

关键字 :name:`Pass Execution` and :name:`Pass Execution If` 在不同条件下的行为如下:

- 当在 `setup 或 teardown`__ (不管是套件的,用例的还是关键字的)中使用, 将使setup或 
  teardown通过. 如果开始的关键字有teardown, 将会执行. 用例的执行和状态不受影响. 

- 当在测试用例中(setup和teardown之外)使用, 当前用例直接pass. 
  如果用例或关键字有teardown, teardown仍会被执行.

- 如果 `可继续的失败`__ 在这些关键字之前发生,
  而且在随后的teardown执行中发生了失败, 则整个执行结果标记为失败.

- 调用这两个关键字时必须要给出中断执行的理由, 同时还可以修改测试用例的标签.
  更多的细节和示例请参考 `它们的文档`__.

在测试用例以及setup或teardown的中间跳过执行需要谨慎. 在最坏的情况下, 这可能会导致测试跳过了可能发生问题的地方(因为最终状态是PASS, 所以不会引起注意). 如果是因为外部因素导致测试无法继续, 更安全的做法是将用例置为 `non-critical`__, 然后失败(fail).

__ `Setups and teardowns`_
__ `Continue on failure`_
__ `BuiltIn`_
__ `Setting criticality`_

.. Continue on failure

失败后继续
-------------------

通常测试用例在任意关键字失败后都会立即终止. 这种行为可以缩短测试执行时间, 防止后续的关键字挂住(hanging), 避免引发其它问题. 然而这么做也有缺点, 因为有时候后续的关键字可以给予更多的系统状态相关的信息. 因此Robot Framework提供了若干特性功能, 使得在发生失败后能继续执行.

.. :name:`Run Keyword And Ignore Error` and :name:`Run Keyword And Expect Error` keywords

通过内置关键字
~~~~~~~~~~~~~~~~~~~

内置关键字 :name:`Run Keyword And Ignore Error` 和 :name:`Run Keyword And Expect Error` 可以处理关键字执行失败的情况, 这样测试就不会立即终止. 

但是使用这些关键字会增加额外的复杂度, 所以下面的特性功能更值得考虑.

.. Special failures from keywords

特殊的失败类型
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`库关键字`_ 是通过抛异常来报告失败, 所以可以使用特殊的异常来告诉框架, 发生了这个失败是可以继续执行的. 如何创建此种异常在 `测试库API`__ 章节中说明.

当用例结束时, 如果中间发生了一次或多次可继续的失败, 当前用例将标记为失败. 如果是多次失败, 则最终的错误信息里将把所有失败都列出来::

  Several failures occurred:

  1) First error message.

  2) Second error message ...

如果在一个可继续的失败后又发生了一个正常失败, 用例会结束, 同时所有的失败都会被列入最后的错误信息中. 

如果要将关键字的返回值赋给变量, 遇到该关键字失败, 则最终返回值将总是 `None`.

__ `Continuing test execution despite of failures`_

.. :name:`Run Keyword And Continue On Failure` keyword

关键字 :name:`Run Keyword And Continue On Failure`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

内置关键字 :name:`Run Keyword And Continue On Failure` 将任意失败都转为可继续的.  框架处理这些失败的方式和库关键字发起的可继续失败一样.

.. Execution continues on teardowns automatically

teardown总是继续
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

为了确保所有的清理任务都被照顾到, `用例和套件的teardown`_ 在失败发生时总是会自动继续执行. 也就是说, teardown中的所有关键字, 不管什么层次, 最终总是会全部执行到.

__ `Setups and teardowns`_

.. All top-level keywords are executed when tests have templates

用例模板所有顶层关键字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当使用 `测试模板`_ 时, 所有的数据行都会被执行到, 以确保所有的数据组合都被测试到. 这种用法仅限于顶层关键字, 也就是说, 如果这些关键字中间发生了不可继续执行的错误, 这个过程还是会和正常的一样结束.

.. Stopping test execution gracefully

优雅地结束测试执行
----------------------------------

有时候需要在所有测试结束前中断执行, 同时还需要生成日志和报告. 下面就介绍几种不同的方法, 在所有这些情况中, 剩下的用例都会标记为失败.

从Robot Framework 2.9版本开始, 由于前面发生致命错误而自动置为失败的用例在报告中将被打上 `robot-exit` 标签, 同时其它的用例 `NOT robot-exit` `combined tag pattern`__  标签, 这样可以轻松的看出哪些用例被跳过了. 注意导致退出发生的那个用例本身不会打上 `robot-exit` 标签.

Starting from Robot Framework 2.9 the tests that are automatically failed get
`robot-exit` tag and the generated report will include `NOT robot-exit`
`combined tag pattern`__ to easily see those tests that were not skipped. Note
that the test in which the exit happened does not get the `robot-exit` tag.

__ `Generating combined tag statistics`_

.. Pressing `Ctrl-C`

按下 `Ctrl-C`
~~~~~~~~~~~~~~~~~

当测试运行时, 在控制台中按下 `Ctrl-C` 会使得测试中断执行. 如果测试是运行在Python上的, 执行将立即停止, 而使用Jython时, 将在当前正在运行的关键字结束后再结束.

如果很快地再次按下 `Ctrl-C`, 整个执行将立即停止, 并且报告和日志文件都不会生成.

.. Using signals

使用信号
~~~~~~~~~~~~~

只有在类Unix系统中才能使用信号 `INT` 和 `TERM` 来终止测试执行. 这些信号可以在命令行中通过使用 ``kill`` 命令发送给进程.

发送信号的方式在使用Jython时和 按`Ctrl-C` 有同样的限制. 并且类似地, 再次发送信号将强行终止整个执行.

.. Using keywords

使用关键字
~~~~~~~~~~~~~~

测试执行的停止还可以通过调用关键字来实现. BuiltIn_ 关键字 :name:`Fatal Error` 用于这种目的. 用户自定义的关键字可以使用 `fatal exceptions`__.

__ `Stopping test execution`_

.. Stopping when first test case fails

用例失败即停
~~~~~~~~~~~~~~~~~~~

如果设置了选项 :option:`--exitonfailure`, 则任意一个 `critical test`_ 失败都将导致整个测试执行停止, 同样地, 其它剩下的用例会被标记为失败.

.. Stopping on parsing or execution error

解析或执行错误
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robot Framework分开对待关键字执行 *失败* (failures)和执行 *错误* (error)的情况, 
失败是指关键字执行没有通过, 而错误是指非法设置或者库导入错误. 
默认情况下, 这些错误会显示在 `test execution errors`__, 但是这些错误本身是不会使用例失败和影响执行的(当然是在错误和用例不相关的时候). 如果设置了选项 :option:`--exitonerror`, 则一旦有错误发生, 整个执行将停止, 并且剩下的所有测试用例会标记为失败. 因为解析错误发生在测试执行前, 也就意味着实际上没有用例会真正运行.

.. note:: :option:`--exitonerror` is new in Robot Framework 2.8.6.

__ `Errors and warnings during execution`_

.. Handling teardowns

处理teardown
~~~~~~~~~~~~~~~~~~

默认情况下, 已经开始运行的用例或套件的teardown总是会执行, 即使测试执行使用了上述的方法来中止了. 这样做可以保证不错过清理任务.

如果设置了选项 :option:`--skipteardownonexit` 则可以关闭这个特性, 当执行停止时跳过teardown的执行. 在有些时候, 例如清理任务耗时非常长, 这将非常有用. 
