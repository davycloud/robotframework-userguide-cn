.. _creating test suites:
.. _test suites:

创建测试套件
====================

Robot Framework测试用例存在测试用例文件中, 这些文件又可以组织在文件夹中. 这些文件和文件夹组成了测试用例集的层次结构.

.. contents::
   :depth: 2
   :local:

.. _test case files:

测试用例文件
---------------

Robot Framework 的测试用例使用测试用例表格在测试文件中创建. 这个文件自动创建了一个测试集, 包含其中所有的测试用例. 理论上, 一个文件中的用例数量没有上限, 但是一般建议不要超过10个, 除非是使用 `data-driven approach`_ 的时候.

下面在设置表格中的配置项可以用来自定义一个测试用例集:

`Documentation`:setting:
   用来指定 `test suite documentation`_
`Metadata`:setting:
   使用键值对来指定 `free test suite metadata`_ 
`Suite Setup`:setting:, `Suite Teardown`:setting:
   用来指定 `suite setup and teardown`_.

.. note:: 所有的设置名称后面都可以选择性的添加一个冒号,  
          例如 :setting:`Documentation:` 这样可以增加可读性, 特别是使用纯文本格式时.

__ `Test case syntax`_

.. _test suite directory:
.. _test suite directories:

测试套件文件夹
----------------------

测试用例文件可以组织在文件夹中, 这些文件夹就是更高层次的测试用例集. 文件夹类型的测试套件不直接包含用例, 仅包含文件用例集, 不过这些文件夹还可能继续保存在其它文件下面, 由此形成一个更高层次的用例套件. 以此类推, 这个组织结构在层次上没有限制, 所有的测试用例可以按需组织.

当一个测试目录被执行, 其中的文件和子目录会按如下规则递归的处理:

- 以点(:file:`.`)或下划线(:file:`_`)开头的文件/目录名被忽略.
- 目录 :file:`CVS` (注意大小写)被忽略.
- 文件的扩展名不在 `可识别扩展名`__ 之列的(:file:`.html`, :file:`.xhtml`, :file:`.htm`, :file:`.tsv`, :file:`.txt`, :file:`.rst`, 或 :file:`.rest`) 被忽略(大小写无关).
- 处理其它文件和目录.
  
如果被处理的文件或者目录下不包含任何用例, 则也会默默的跳过.

__ `Supported file formats`_

.. _Warning on invalid files:

非法文件警告
~~~~~~~~~~~~~~~~~~~~~~~~

一般情况下, 其中不包含合法测试用例表格的文件会默默忽略, 但同时把消息写入 syslog_. 可以在命令行中指定选项 :option:`--warnonskippedfiles`, 将该消息作为警告处理, 最终出现在 `test execution errors`__.

__ `Errors and warnings during execution`_

.. _initialization file:

初始化文件
~~~~~~~~~~~~~~~~~~~~

目录形式的测试用例集也可以和文件形式的用例集一样有类似的设置. 但是由于目录本身没法保存相关信息, 必须将其保存在一个特殊的测试套件初始化文件中.

初始化文件的文件名的格式总是 :file:`__init__.ext`, 其中扩展名和用例文件的扩展名一样, 也必须要是可识别的. (例如: :file:`__init__.robot` 或 :file:`__init__.html`). 这种命名格式借鉴自Python, :file:`__init__.py` 将目录变为一个模块(module).

初始化文件中除了不能包含测试用例表格, 以及不支持某些设置项, 其它结构和语法和用例文件一样.

初始化文件中创建或者导入的变量和关键字在下层的测试套件中 *不可用*. 想要共享变量和(或)关键字, 可以放在 `resource files`_, 再由测试用例文件导入.

初始化文件的最大用途是指定用例集相关的设置. 指定设置的方式类似 `test case files`_, 同时也可以指定某些 `test case related settings`__.

下面将解释说明如何在初始化文件中使用不同的设置项.

`Documentation`:setting:, `Metadata`:setting:, `Suite Setup`:setting:, `Suite Teardown`:setting:
   这些测试套件相关的设置和测试用例文件中的设置一样.

`Force Tags`:setting:
   为下面的所有用例指定标签.

`Test Setup`:setting:, `Test Teardown`:setting:, `Test Timeout`:setting:
   为下面的测试用例设置默认的 setup/teardown 或 超时动作. 测试用例可以单独设置以覆盖这里的配置.

`Default Tags`:setting:, `Test Template`:setting:
   不支持.

.. sourcecode:: robotframework

   *** Settings ***
   Documentation    Example suite
   Suite Setup      Do Something    ${MESSAGE}
   Force Tags       example
   Library          SomeLibrary

   *** Variables ***
   ${MESSAGE}       Hello, world!

   *** Keywords ***
   Do Something
       [Arguments]    ${args}
       Some Keyword    ${arg}
       Another Keyword

__ `Test case related settings in the Setting table`_

.. _Test suite name and documentation:

测试套件名称和文档
---------------------------------

测试套件的名称由文件或目录名称构造. 文件的扩展名被去掉, 名字中的下划线被空格替换, 全小写的单词首字母会变大写. 例如: :file:`some_tests.html` 变为 :name:`Some Tests`, :file:`My_test_directory` 转变为 :name:`My Test Directory`.

文件或目录名称可以包含前缀来控制测试集的 `execution order`_. 前缀和后面的基础名称用两个下划线分隔, 当构造用例集名称时, 前缀和下划线都被去掉.
例如, 文件 :file:`01__some_tests.txt` 和 :file:`02__more_tests.txt` 创建的测试用例集名称分别是 :name:`Some Tests` 和 :name:`More Tests`, 并且前者会先执行.

测试套件的文档通过在Setting表格中设置 :setting:`Documentation` 来指定. 该设置项可以在用例文件中, 以及目录套件的初始化文件中设置. 测试套件的文档表现形式和创建方式于 `test case documentation`_ 一般无二.

.. sourcecode:: robotframework

   *** Settings ***
   Documentation    An example test suite documentation with *some* _formatting_.
   ...              See test documentation for more documentation examples.

高层测试套件的名称和文档都可以在执行的时候, 通过命令行选项 :option:`--name` 和 :option:`--doc` 分别覆盖. 详见 `Setting metadata`_.

.. _Free test suite metadata:

测试套件的metadata
------------------------

除了文档外, 测试套件还可以设置其他的元数据(metadata). 这些元数据通过在 Setting 表格中使用 :setting:`Metadata` 设置项来指定. 设置的元数据会在测试报告和日志文件中展示.

元数据的键和值所在的列跟在 :setting:`Metadata` 后面. 值的处理和文档类似, 也就是说, 可以分割为 `into several cells`__ (由空格拼接), 或者 `into several rows`__(由换行拼接), 简单的  `HTML formatting`_ 甚至 variables_ 都支持.

__ `Dividing test data to several rows`_
__ `Newlines in test data`_

.. sourcecode:: robotframework

   *** Settings ***
   Metadata    Version        2.0
   Metadata    More Info      For more information about *Robot Framework* see http://robotframework.org
   Metadata    Executed At    ${HOST}

对于高层的测试用例集, 可以通过命令行选项 :option:`--metadata` 来设置元数据. 具体细节请参考 `Setting metadata`_.

.. _suite setup and teardown:

套件的Setup和Teardown
------------------------

测试套件和测试用例一样可以设置Setup和Teardown. 测试套件的Setup在其中所有测试用例和子套件运行之前被执行, Teardown则是在之后执行.

每一层测试套件都可以有Setup和Teardown, 目录形式的测试套件需要在 `初始化文件`_ 中设置.

__ `Test setup and teardown`_

和测试用例类似, 测试套件的Setup和Teardown也都是可接受参数的关键字. 它们在 Setting 表格中通过 :setting:`Suite Setup` 和 :setting:`Suite Teardown` 各自指定. 关键字的参数跟在设置名称的后面.

如果一个测试套件的Setup执行失败了, 该套件下的所有子套件和用例会立即置为失败状态, 实际上并不会执行. 利用这种特性, 可以来检验用例执行的必要前置条件是否满足.

测试套件的Teardown一般是用来在所有测试用例执行完毕后, 执行必要的清理操作. 不同于用例, 即使setup执行失败了, teardown也会执行. 如果teardown执行失败, 所有的测试用例也会被标记为失败, 不管这些用例自己执行的结果如何. 注意, teardown中的所有关键字, 即使其中某些执行失败, 最终都会被执行.

Setup和Teardown中的关键字名称可以使用变量. 这种特性使得可以在不同的环境中, 通过命令行指定不同的变量, 就可以执行不同的setup和teardown过程.
