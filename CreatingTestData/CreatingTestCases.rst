.. role:: name(emphasis)
.. role:: setting(emphasis)

.. _creating test cases:

创建测试用例
============

本章介绍整个测试用例的语法. 使用 :ref:`test case files` 将测试用例组织为 :ref:`测试套件 <creating test suites>`, 以及 :ref:`test suite directories` 将在下一章讨论.

.. contents::
   :depth: 2
   :local:

.. _test case syntax:

测试用例语法
------------


基础语法
~~~~~~~~

测试用例表格中的测试用例由可用的关键字构造. 关键字可以从 :ref:`test libraries` 或 :ref:`resource files` 导入, 也可以在当前测试用例文件中的 :ref:`关键字表格 <user keywords>` 创建.

测试用例表格的第一列包含测试用例的名称. 一个用例始于测试用例名, 直到遇到下一个用例名, 或者到表格的结束. 在表格头和第一个测试用例之间不允许有其它数据, 否则将引发错误.

第二列一般情况下是关键字的名称. 当要 :ref:`从关键字获取返回值 <user keyword return values>` 的时候是个特例, 这种情况下第二列, 甚至后续的列都可能是用来接受返回值的变量名称, 关键字名称跟在这些变量后面. 最后, 不论关键字名称位于第几列, 跟在其后的列包含的是要传递给该关键字的参数.


.. _example-tests:

.. sourcecode:: robotframework

   *** Test Cases ***
   Valid Login
       Open Login Page
       Input Username    demo
       Input Password    mode
       Submit Credentials
       Welcome Page Should Be Open

   Setting Variables
       Do Something    first argument    second argument
       ${value} =    Get Some Value
       Should Be Equal    ${value}    Expected value

.. _settings in the Test Case table:

测试用例表格中的设置
~~~~~~~~~~~~~~~~~~~

每个测试用例都可以有自己的相关设置. 设置项名称总是出现在第2列, 它们的值跟在后面的列中.
设置项的名称使用方括号(``[]``)括起来, 以区别于关键字.

下面列出了可设置的项, 后面的章节会进行详细的解释.


:setting:`[Documentation]`
    用于指定 :ref:`测试用例文档 <test case documentation>`.

:setting:`[Tags]`
    用于指定 :ref:`tags`.

:setting:`[Setup]`, :setting:`[Teardown]`
   用于指定 :ref:`test setup and teardown`.

:setting:`[Template]`
   用于指定 :ref:`template keyword`. 测试用例本身将只包含数据, 每行数据都是传递给该关键字的参数, 最终实现数据驱动的测试.

:setting:`[Timeout]`
   用于设置 :ref:`test case timeout`. :ref:`timeouts` 将在独立的章节讨论.

带设置的测试用例示例:

.. sourcecode:: robotframework

   *** Test Cases ***
   Test With Settings
       [Documentation]    Another dummy test
       [Tags]    dummy    owner-johndoe
       Log    Hello, world!


.. Test case related settings in the Setting table
.. _test case related settings:

设置表格中和测试用例相关的设置项
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

下列是设置表格中与测试用例相关的设置项. 这些设置项大部分都是上述用例特定设置的默认值.

:setting:`Force Tags`, :setting:`Default Tags`
   :ref:`tags` 的强制(forced)值和默认值.

:setting:`Test Setup`, :setting:`Test Teardown`
   :ref:`test setup and teardown` 的默认值.

:setting:`Test Template`
   默认的 :ref:`template keyword`.

:setting:`Test Timeout`
   默认的 :ref:`test case timeout`. :ref:`timeouts` 在专门的章节讨论.


.. _using arguments:

使用参数
--------

前面有的例子已经展示了关键字是怎样 *接受* 不同的参数的, 本节将更具体地讨论这个重要的功能点.

如何 *实现* 这些带各种参数的 :ref:`用户关键字 <user keyword arguments>` 和 :ref:`库关键字 <keyword arguments>` 则在其它独立的章节讨论.

关键字可以接受零到多个参数, 并且有些参数还可以有默认值. 总的来说, 
一个关键字可以接受的参数数量和类型取决于它的实现, 一般来讲, 最好的去处是去该关键字的文档中来搜索相关信息.

对本章中的例子而言, 相关的文档可以使用 :ref:`Libdoc` 工具生成. 不过, 也可是使用其它的文档生成工具, 例如 ``javadoc``.

.. hint:: 译注: 调用关键字的传参方式和调用Python函数的语法如出一辙, 
          所以熟悉Python的小伙伴可以快速掠过本节内容.


.. _Mandatory arguments:

必填参数
~~~~~~~~~

大多数的关键字有部分参数是必须要传递的. 在关键字的文档中, 这部分参数的表示方式是 ``first, second, third``. 这里的参数名称实际上并不重要, 仅作为描述该参数的意义, 但是参数的个数必须完全一致. 少传或者多传都会导致错误发生.

下面的用例用到了关键字 :name:`Create Directory` 和 :name:`Copy File`, 两者都来自于 OperatingSystem_ 库. 它们的参数定义分别是 ``path`` 和 ``source, destination``, 也就是说, 分别需要1个和2个参数. 最后一个关键字, 来自 BuiltIn_ 的 :name:`No Operation` 不接受参数.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Create Directory    ${TEMPDIR}/stuff
       Copy File    ${CURDIR}/file.txt    ${TEMPDIR}/stuff
       No Operation

.. _default values:

缺省值
~~~~~~~

可以为参数指定缺省值. 在关键字文档中, 缺省值通常表示为 ``name=default value``, 即使用等于号把参数名和缺省值连接起来. 

对于使用Java实现的关键字来说, 意味着同一个关键字, 需要使用不同的参数 :ref:`多次实现 <default values with Java>` (实际上就是重载).

所有参数都可以指定缺省值, 不过位置参数 **不可以** 跟在带缺省值的参数的后面.

使用缺省值的情况见下例, 使用了关键字 :name:`Create File`, 可接受的参数是 ``path, content=, encoding=UTF-8``. 可以尝试不带任何参数, 或者多于3个参数来调用该关键字, 将会报错.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Create File    ${TEMPDIR}/empty.txt
       Create File    ${TEMPDIR}/utf-8.txt         Hyvä esimerkki
       Create File    ${TEMPDIR}/iso-8859-1.txt    Hyvä esimerkki    ISO-8859-1


.. _varargs:
.. _variable number of arguments:

可变数量的参数
~~~~~~~~~~~~~~

一个关键字还可以接受任意数量的参数, 也就是说, 参数个数是不确定的. 这种类型的参数一般称之为 *varargs*. varargs可以和必需参数以及带缺省值的参数混合使用, 不过在参数列表中必须排在它们的后面.
在文档中, 此种参数通过在参数名称前加一个星号(``*``)来表示, 例如 ``*varargs``.


例如, OperatingSystem_ 库中的关键字 :name:`Remove Files` 和 :name:`Join Paths`, 参数分别是  ``*paths`` 和 ``base, *parts``. 前者可以接受任意数量的参数, 后者则至少需要一个参数.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Remove Files    ${TEMPDIR}/f1.txt    ${TEMPDIR}/f2.txt    ${TEMPDIR}/f3.txt
       @{paths} =    Join Paths    ${TEMPDIR}    f1.txt    f2.txt    f3.txt    f4.txt


.. _named argument syntax:
.. _named arguments:

命名参数
~~~~~~~~

命名参数语法使 :ref:`default values` 参数用起来更加灵活,  因为它可以明确地标示要传递的值对应的是哪个参数. 这种语法格式和Python语言中的 :ref:`keyword arguments <python kwargs>` 如出一辙.


基础语法
'''''''''

可以使用 ``arg=value`` 的格式来指定传值给某个参数. 当多个参数有默认值时, 这种方式就显得特别有用, 因为我们只需要指定其中一些, 让其余的继续使用默认值.

例如, 如果一个关键字接受参数 ``arg1=a, arg2=b, arg3=c``, 调用时传参 ``arg3=override``, 则参数 ``arg1`` 和 ``arg2`` 使用默认值, 而 ``arg3`` 的值为 ``override``. 这一切说起来颇为啰嗦, 查看下面的 :ref:`named arguments example` 就比较清楚了.

命名参数语法是大小写敏感和空格敏感的. 前者的意思是, 如果参数名为 ``arg``, 则必须使用 ``arg=value``, 而 ``Arg=value`` 或 ``ARG=value`` 都是错误的. 后者的意思是, 等号 ``=`` 的前面不能有空格, ``=`` 后面可以有空格, 这些空格会被视作值的一部分.

当命名参数语法用在 :ref:`user keywords` 时, 参数名称必须去掉 ``${}`` 标示. 例如, ``${arg1}=first, ${arg2}=second`` 必须写成如 ``arg2=override``.

不可以在命名参数后面使用普通的位置参数, 例如 ``| Keyword | arg=value | positional |``.
从Robot Framework 2.8 开始, 这会导致一个明确的错误提示. 
命名参数之间的相对位置则无关紧要.

.. note:: 在Robot Framework 2.8版本之前, 不可以针对没有默认值的参数使用命名参数语法.

.. _named arguments with variables:

带变量的命名参数
''''''''''''''''

:ref:`variables` 不但可以用于参数的值, 也可以用于参数名称. 如果变量值是一个单独的 :ref:`scalar variable`, 则将按原值传递给关键字. 变量的值可以是任何的对象, 不仅限于字符串.
例如, 调用关键字时使用 ``arg=${object}``, 则变量 ``${object}`` 最终会传递给关键字.

如果将变量用在命名参数的名称上, 变量需要在匹配参数名称之前就被解析出来. 这是2.8.6版本后才有的新功能.

命名参数语法需要在调用关键字时使用字面的等号(``=``), 也就是说, 不能把 ``=`` 放在一个变量里, 例如 ``foo=bar``, 是不可能触发命名参数语法的.

当封装关键字的时候, 这点必须特别注意. 如果上层的关键字接受 :ref:`variable number of arguments`, 也就是 `` @{args}``, 该关键字中调用另一个关键字时把所有的参数都通过 `` @{args}`` 传递过去, 则其中可能包含的 ``named=arg`` 将不会被识别.
具体请看下面的例子.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Run Program    shell=True    # This will not come as a named argument to Run Process

   *** Keywords ***
   Run Program
       [Arguments]    @{args}
       Run Process    program.py    @{args}    # Named arguments are not recognized from inside @{args}

如果关键字需要接受并传递任意的命名参数, 它必须改用 :ref:`free keyword arguments`.
参见 :ref:`kwargs examples`, 这个例子展示了一个封装后的关键字如何既传递位置参数, 又传递命名参数.

.. _Escaping named arguments syntax:

转义命名参数语法
''''''''''''''''

只有在参数等号前面的部分匹配上了某个参数名称时, 命名参数语法才起作用. 
假如有一个位置参数需要传入一个值 ``foo=quux``, 假设同时有一个参数名是 ``foo``, 此时, 大多数情况也许会触发语法错误, 还可能将 ``quux`` 值传递给了参数 ``foo``, 当然也有可能是如我们预期的那样(位置参数接受到了完整的字符串值), 取决于参数的位置顺序等实际情况. 这时候, 需要使用反斜杠转义符来将其中的等号 :ref:`转义 <escaping>`, 如 ``foo\=quux``. 这样, 位置参数将会明确的接受到值 ``foo=quux``.

注意, 如果关键字没有参数名称是 ``foo``, 则这里的转义不是必需的. 不过, 由于这种写法会更明确, 所以推荐的做法是不管什么情况下都转义.


.. _Where named arguments are supported:

支持命名参数的地方
''''''''''''''''''

如前所述, 命名参数语法可以用于关键字. 此外, 它还可以用在 :ref:`importing libraries`.

:ref:`user keywords` 以及大部分的 :ref:`test libraries` 也都支持命名参数. 唯一的例外是基于Java的 :ref:`static library API` 库.
使用 :ref:`Libdoc` 生成的库文档中会显示该库是否支持命名参数.

.. note:: Robot Framework2.8版本之前, 使用 :ref:`dynamic library API` 
          的库也不支持命名参数语法.

.. _named arguments example:

命名参数示例
''''''''''''

下面的例子展示了命名参数语法如何应用在库关键字和用户关键字上, 以及在导入 :ref:`Telnet` 测试库时的应用.

.. sourcecode:: robotframework

   *** Settings ***
   Library    Telnet    prompt=$    default_log_level=DEBUG

   *** Test Cases ***
   Example
       Open connection    10.0.0.42    port=${PORT}    alias=example
       List files    options=-lh
       List files    path=/tmp    options=-l

   *** Keywords ***
   List files
       [Arguments]    ${path}=.    ${options}=
       Execute command    ls ${options} ${path}

.. _free keyword arguments:

任意命名参数
~~~~~~~~~~~~

Robot Framework 2.8加入了对 `Python风格的命名参数 <python kwargs>` (``**kwargs``)的支持. 这意味着关键字可以接受任意数量的 ``name=value`` 语法格式的参数.

Kwargs和 :ref:`命名参数 <named arguments with variables_>` 类似, 也支持变量. 实际应用中, 变量既可以作为参数名, 也可以作为参数值, 但是必须明确的使用转义符.
例如, ``foo=${bar}`` 和 ``${foo}=${bar}`` 都是合法的, 只要这些变量本身没问题. 一个额外的限制是, 作为参数名称的变量值必须是字符串.

.. note:: 支持在参数名称中使用变量是Robot Framework 2.8.6才加入的新功能.

最初 ``**kwargs`` 只可用于Python编写的库, 后来Robot Framework 2.8.2扩展到 :ref:`dynamic library API`, 2.8.3版本继续扩展到支持基于的Java测试库以及 :ref:`remote library interface`. 最终, 用户关键字在2.9版本也开始支持 :ref:`kwargs <Kwargs with user keywords>`. 

也就是说, 当前最新的版本中, 所有的关键字都可以使用 ``**kwargs``.

.. _python kwargs: http://docs.python.org/2/tutorial/controlflow.html#keyword-arguments

.. _kwargs examples:

Kwargs示例
'''''''''''

第1个例子中, 使用了 Process_ 中的 :name:`Run Process` 关键字. 该关键字的参数签名是: ``command, *arguments, **configuration``, 分别的含义是: ``command`` 表示要执行的命令, 命令所需的参数通过 :ref:`variable number of arguments` 即 ``*arguments`` 指定, 最后是可选的配置参数 ``**configuration``.
例子中同时展示了kwargs中变量的用法,  就和 :ref:`使用命名参数 <named arguments with variables>` 一样.

.. sourcecode:: robotframework

   *** Test Cases ***
   Using Kwargs
       Run Process    program.py    arg1    arg2    cwd=/home/user
       Run Process    program.py    argument    shell=True    env=${ENVIRON}

关于如何在自定义的测试库中使用 kwargs 语法, 请参阅: :ref:`creating test libraries` 中的  :ref:`free keyword arguments (**kwargs)` 部分内容.

第2个例子中, 我们封装了一个 :ref:`user keyword` 来简化调用上述的运行 ``program.py`` 过程.
封装后的关键字 :name:`Run Program` 接受任意数量的参数和kwargs, 并将它们传递给底层的关键字 :name:`Run Process`, 只是固定其中的 ``command`` 参数.

.. sourcecode:: robotframework

   *** Test Cases ***
   Using Kwargs
       Run Program    arg1    arg2    cwd=/home/user
       Run Program    argument    shell=True    env=${ENVIRON}

   *** Keywords ***
   Run Program
       [Arguments]    @{arguments}    &{configuration}
       Run Process    program.py    @{arguments}    &{configuration}

.. _Arguments embedded to keyword names:

关键字名称中嵌入参数
~~~~~~~~~~~~~~~~~~~

在关键字的名称中嵌入参数是一种全新的指定参数的方式. 这种语法同时被 :ref:`测试库关键字 <embedding arguments into keyword names>` 和 :ref:`用户关键字 <Embedding arguments into keyword name>` 支持.


.. _failures:

失败
----

.. _when test case fails:

测试用例何时失败
~~~~~~~~~~~~~~~~

一个测试用例中用到的任何一个关键字发生失败, 则该用例也执行失败. 正常情况下, 这表明该测试用例的执行被结束, 如果设置了 :ref:`test teardown`, 则也会执行. 测试流程继续往下执行下一个用例. 

特殊情况下, 如果当发生错误时不想结束测试用例, 可以使用特殊的 :ref:`可继续的失败 <continue on failure>`.

.. _Error messages:

错误信息
~~~~~~~~

测试用例的错误信息直接来源于失败的关键字. 有些错误信息由关键字直接生成, 有的关键字则允许用户来配置.

在某些情况下, 例如设置了可继续执行的失败, 一个测试用例可能会发生多次的失败, 这时最终的错误信息将由各自的错误信息组合起来. 
超长的错误信息在 :ref:`reports` 中会自动截断, 以保持报告的可读性. 完整的信息总是可以在 :ref:`log` 文件中找到.

默认情况下错误信息就是普通的文本, 不过从Robot Framework 2.8版本开始, 错误信息中可以 :ref:`包含HTML格式 <HTML in error messages>`. 通过在错误信息的开始部分指定 ``*HTML*`` 标记即可启用该功能(该标记本身在最终的错误信息中不会展示).

下面第2个例子中展示了如何使用自定义的HTML格式的消息.

.. sourcecode:: robotframework

   *** Test Cases ***
   Normal Error
       Fail    This is a rather boring example...

   HTML Error
       ${number} =    Get Number
       Should Be Equal    ${number}    42    *HTML* Number is not my <b>MAGIC</b> number.

.. _test case documentation:
.. _test case name and documentation:

测试用例名称和文档
-----------------

测试用例的名称直接在测试用例表格中指定: 即表格中输入的用例名称是什么就是什么.
一个测试套件中的用例名称必须唯一. 

在测试用例中, 可以通过 :ref:`automatic variable` ``${TEST_NAME}`` 来指代当前用例的名称. 这个变量在测试执行的任何阶段都可以访问到, 包括用户关键字, 以及测试准备和测试结束阶段.

配置项 :setting:`[Documentation]` 用来为用例设置一段文档说明. 这个说明会显示在命令行的输出中, 以及后续的测试日志和测试报告中. 文档中可以使用简单的 :ref:`HTML formatting`, 也可以使用 :ref:`variables`, 使文档更加的动态.

如果一个文档被分为多列, 则同一行中每个单元格中的内容最终以空格连接. 如果文档 :ref:`被分为多行 <Dividing test data to several rows>`, 则最终的文档 :ref:`以换行符拼接起来 <newlines in test data>`. 如果某行结尾已经有了换行符, 或者使用了 :ref:`转义的反斜杠 <escaping>`, 则不会再添加换行符.

.. sourcecode:: robotframework

   *** Test Cases ***
   Simple
       [Documentation]    Simple documentation
       No Operation

   Formatting
       [Documentation]    *This is bold*, _this is italic_  and here is a link: http://robotframework.org
       No Operation

   Variables
       [Documentation]    Executed at ${HOST} by ${USER}
       No Operation

   Splitting
       [Documentation]    This documentation    is split    into multiple columns
       No Operation

   Many lines
       [Documentation]    Here we have
       ...                an automatic newline
       No Operation

测试用例拥有一个清楚的, 描述性的名称是非常重要的, 这种情况下一般就不再需要文档说明了.
如果用例的逻辑比较复杂, 以至于必须使用文档才能说清楚, 这往往意味着该用例中的关键字有待改进, 需要使用更好的名称, 而不是添加额外的文档.
最后, 诸如环境或用户信息等这类元数据, 最好使用 :ref:`tags` 来指定. 

.. _tagging test cases

.. _test case tags:
.. _tags:

测试用例的标签
--------------

Robot Framework的标签功能是一个简单而强大的分类机制. 标签本身就是任意的文本, 它们被用于如下目的:

- 标签在 :ref:`reports`, :ref:`logs` 以及测试数据中展示, 显示关于测试用例的元数据信息.
- 用例的 :ref:`执行统计 <configuring statistics>` (total, passed, failed 就是自动基于标签收集的).
- 使用标签, 可以 :ref:`包含或排除 <by tag names>` 测试用例来执行.
- 使用标签, 可以指定何种用例是 :ref:`critical`.


本节仅介绍如何针对测试用例设置标签, 几种不同的方式列在下面, 这些方法可以自然地组合.

:setting:`Force Tags` 在Setting表中设置
  包含该设置的测试用例文件中所有用例都被指定打上这些标签. 如果这是用在
  `test suite initialization file`, 则下面的所有子测试套件都被打上这些标签.

:setting:`Default Tags` 在Setting表中设置
   没有单独设置 :setting:`[Tags]` 的用例将被打上这些默认标签.
   默认标签不支持在测试套件的初始化文件中指定.

:setting:`[Tags]` in the Test Case table
  每个测试用例各自要打的标签. 如果设置了, 就不再包含 :setting:`Default Tags`,
  所以, 可以通过设置一个空值来覆盖默认标签, 亦可以使用 `NONE`.

:option:`--settag` 命令行选项
  所有通过包含该选项的命令执行的测试用例, 除了已有的标签, 都会再加上选项中指定的标签.

`Set Tags`:name:, `Remove Tags`:name:, `Fail`:name: and `Pass Execution`:name: 关键字
  这些内置的关键字可以用来在测试执行过程中动态的操纵用例的标签.

标签本身就是任意的文本, 但是它们会被标准化: **去除所有空格, 全部转为小写**.
如果一个用例打上相同的标签多次, 仅保留第一个.
标签可以使用变量来创建, 只要变量存在即可.

.. sourcecode:: robotframework

   *** Settings ***
   Force Tags      req-42
   Default Tags    owner-john    smoke

   *** Variables ***
   ${HOST}         10.0.1.42

   *** Test Cases ***
   No own tags
       [Documentation]    This test has tags owner-john, smoke and req-42.
       No Operation

   With own tags
       [Documentation]    This test has tags not_ready, owner-mrx and req-42.
       [Tags]    owner-mrx    not_ready
       No Operation

   Own tags with variables
       [Documentation]    This test has tags host-10.0.1.42 and req-42.
       [Tags]    host-${HOST}
       No Operation

   Empty own tags
       [Documentation]    This test has only tag req-42.
       [Tags]
       No Operation

   Set Tags and Remove Tags Keywords
       [Documentation]    This test has tags mytag and owner-john.
       Set Tags    mytag
       Remove Tags    smoke    req-*

.. _Reserved tags:

保留的标签
~~~~~~~~~~

通常情况下, 用户在各自的环境下可以任意使用自定义的标签. 不过, 某些标签名称对Robot Framework来说有特殊的意义, 如果用于其它目的可能会引发不可预测的结果.
所有的特殊标签现在(并且在将来也会)都以前缀 ``robot-`` 开始. 为了避免出现问题, 用户应该避免使用带 ``robot-`` 前缀的标签, 除非是刻意要使用特殊功能.

目前(3.0版本), 唯一的特殊标签只有 ``robot-exit``, 当使用 :ref:`stopping test execution gracefully` 时, 自动添加该标签到相应的测试. 

当然, 将来也许会增加更多的特殊标签.

.. _test setup:
.. _test teardown:
.. _test setup and teardown:

Setup和Teardown
---------------

和很多其他测试自动化框架类似, Robot Framework也有setup和teardown的功能. 简而言之, setup在测试用例之前执行, 而teardown在测试用例之后执行. 

在Robot Framework中, setup和teardown都是带参数的普通关键字而已, 并且各自只能指定一个关键字. 如果涉及到多个步骤, 只能创造一个更高层的 :ref:`user keywords`. 另一种解决方案是使用 :ref:`BuiltIn` 关键字 :name:`Run Keywords` 来执行多个关键字.

Teardown在以下两个方面比较特殊. 首先, 它在测试用例执行失败的时候也会被执行, 所以常常用来作为测试环境的清理工作, 因为不管测试结果如何, 这些清理任务都需要做. 其次, teardown中所有的关键字都会被执行, 哪怕其中有的执行失败. 这种 :ref:`continue on failure` 机制也可以用来普通关键字上, 但是在teardown中, 这是个默认打开的.

为测试用例设置setup或teardown的最简单的方式是在测试用例文件中, 在设置表中指定  :setting:`Test Setup` 和 :setting:`Test Teardown`.

每个单独的用例也可以指定自身的setup或teardown, 在测试用例表中设置 :setting:`[Setup]` 或 :setting:`[Teardown]` 即可. 如果用例文件的设置表中已经设置, 则此处设置会覆盖前者.
亦或者设置 :setting:`[Setup]` 或 :setting:`[Teardown]` 项为空(可以是空白, 也可以使用特殊的 ``NONE`` 值), 则表示当前用例没有setup/teardown.

.. sourcecode:: robotframework

   *** Settings ***
   Test Setup       Open Application    App A
   Test Teardown    Close Application

   *** Test Cases ***
   Default values
       [Documentation]    Setup and teardown from setting table
       Do Something

   Overridden setup
       [Documentation]    Own setup, teardown from setting table
       [Setup]    Open Application    App B
       Do Something

   No teardown
       [Documentation]    Default setup, no teardown at all
       Do Something
       [Teardown]

   No teardown 2
       [Documentation]    Setup and teardown can be disabled also with special value NONE
       Do Something
       [Teardown]    NONE

   Using variables
       [Documentation]    Setup and teardown specified using variables
       [Setup]    ${SETUP}
       Do Something
       [Teardown]    ${TEARDOWN}

Setup或teardown中指定的关键字名称可以使用变量代替. 这样在不同的环境下, 只需在执行时通过命令行选项指定关键字名称, 就可以使用不同的setup或者teardown.

.. note:: :ref:`测试套件可以有自己的setup和teardown <Suite setup and teardown>`. 
           一个测试套件的setup在其中所有的用例以及所有子套件之前被执行, 而它的
           teardown则在最后.

.. _template keyword: 
.. _test templates:

测试模板
--------

测试模板将普通的 :ref:`keyword-driven` 测试转为 :ref:`data-driven` 测试.

关键字驱动的测试用例的内容包含若干步骤, 每个步骤都涉及带若干参数调用关键字, 而包含模板的测试用例的内容只有调用模板关键字所需的参数.

相比较于在一个测试用例(或一个测试用例文件中的多个测试)中多次调用同一个关键字, 这样只需一个测试用例即可搞定.

模板关键字既可以接受普通的位置参数和命名参数, 也可以接受嵌在关键字名称中的参数. 不同于其他设置, 模板的指定不可以使用变量.


基础用法
~~~~~~~~

下面的例子展示了一个接受位置参数的关键字, 是如何作为模板使用的. 其中这两个用例在功能上是完全相同的.

.. sourcecode:: robotframework

   *** Test Cases **
   Normal test case
       Example keyword    first argument    second argument

   Templated test case
       [Template]    Example keyword
       first argument    second argument

如例中所示, 可以对单个测试用例通过设置 :setting:`[Template]` 指定一个模板. 另一种方式是在测试文件的设置表中设置 :setting:`Test Template`, 这种情况下, 该文件中的所有测试用例都会应用该模板.

如果用例单独设置了 :setting:`[Template]`, 则会覆盖文件中的 :setting:`Test Template`, 进而可以为 :setting:`[Template]` 设置空值(空格或 ``NONE``), 表示当前用例没有模板, 即使测试文件设置表中已有设置.

如果一个模板用例的内容有多行数据, 该模板会逐行应用于这些数据. 也就是说, 该模板关键字会被调用多次, 每次使用其中一行的数据作为参数.

模板测试用例在执行过程中, 如果有某一轮次执行失败也不会影响下面轮次继续执行. 也就是说, 对于普通测试用例需要单独设置的 :ref:`continue on failure` 特性, 对于模板测试用例来说是自动启用的.

.. sourcecode:: robotframework

   *** Settings ***
   Test Template    Example keyword

   *** Test Cases ***
   Templated test case
       first round 1     first round 2
       second round 1    second round 2
       third round 1     third round 2

对模板来说, 各种参数, 包括 :ref:`default values`, :ref:`varargs`, :ref:`named arguments` 和 :ref:`free keyword arguments`, 都和平常一样使用. 参数中的 :ref:`variables` 也同样支持.

.. _Templates with embedded arguments:

使用内嵌参数的模板
~~~~~~~~~~~~~~~~~

从Robot Framework 2.8.2版本开始, 模板支持一种变种的 :ref:`embedded argument syntax`.

对于模板来说, 如果模板关键字名称中包含变量, 则这些变量会被视作参数的占位符, 最终在执行时被实际参数替代. 最终的关键字也不再需要其它的参数了. 如下例所示:

.. sourcecode:: robotframework

   *** Test Cases ***
   Normal test case with embedded arguments
       The result of 1 + 1 should be 2
       The result of 1 + 2 should be 3

   Template with embedded arguments
       [Template]    The result of ${calculation} should be ${expected}
       1 + 1    2
       1 + 2    3

   *** Keywords ***
   The result of ${calculation} should be ${expected}
       ${result} =    Calculate    ${calculation}
       Should Be Equal    ${result}     ${expected}

当模板中使用了嵌入式的参数, 模板关键字中参数的个数必须和传入的参数个数一致, 但是参数的名称不一定非要和原关键字保持一致. 甚至还可以添加或减少参数, 如下例所示:

.. sourcecode:: robotframework

   *** Test Cases ***
   Different argument names
       [Template]    The result of ${foo} should be ${bar}
       1 + 1    2
       1 + 2    3

   Only some arguments
       [Template]    The result of ${calculation} should be 3
       1 + 2
       4 - 1

   New arguments
       [Template]    The ${meaning} of ${life} should be 42
       result    21 * 2

在模板中使用嵌入式参数的主要好处是参数名称都被明确指定. 当使用普通参数时, 也可以通过给列命名来达成同样的效果. 这会在后续章节 :ref:`data-driven style` 中介绍.

.. _Templates with for loops:

带for循环的模板
~~~~~~~~~~~~~~~

如果模板用例中使用了 :ref:`for loops`, 该模板会应用于for循环中每一次的循环. 并且遇到错误也会继续执行. 也就是说, 用例中每一步骤中的每次循环都会被执行.

最好还是来看例子:

.. sourcecode:: robotframework

   *** Test Cases ***
   Template and for
       [Template]    Example keyword
       :FOR    ${item}    IN    @{ITEMS}
       \    ${item}    2nd arg
       :FOR    ${index}    IN RANGE    42
       \    1st arg    ${index}

.. _Different test case styles:

不同的测试用例风格
-----------------

有几种不同的测试用例的写法. 用来描述某些 *工作流* 的用例一般是关键字驱动或行为驱动型. 数据驱动型则是针对同一个工作流, 使用不同的输入数据.

.. _Keyword-driven style:

关键字驱动型
~~~~~~~~~~~~

针对工作流的测试, 例如 :name:`Valid Login` 流程, 由若干关键字和相应的参数组成. 

如 :ref:`前例 <example-tests>` 所示, 关键字驱动型的用例的通常结构是, 系统先进入到一个初始状态(:name:`Open Login Page`), 然后对系统进行某些操作(:name:`Input Name`, :name:`Input Password`, :name:`Submit Credentials`), 最后校验系统的表现是否符合预期(:name:`Welcome Page Should Be Open`).


.. _Data-driven style:

数据驱动型
~~~~~~~~~~

数据驱动型的测试方法中, 测试用例仅使用一个高级别的关键字, 通常是创建的 :ref:`user keyword`, 该关键字中则隐藏了实际的流程. 

这种测试对于要针对某个相同的测试场景使用不同的输入/输出时非常有用. 虽然可以在每个测试用例中都重复调用一次相同的关键字, 但是使用 :ref:`test template` 功能更加简便, 因为关键字只需要指定一次. 

.. sourcecode:: robotframework

   *** Settings ***
   Test Template    Login with invalid credentials should fail

   *** Test Cases ***                USERNAME         PASSWORD
   Invalid User Name                 invalid          ${VALID PASSWORD}
   Invalid Password                  ${VALID USER}    invalid
   Invalid User Name and Password    invalid          invalid
   Empty User Name                   ${EMPTY}         ${VALID PASSWORD}
   Empty Password                    ${VALID USER}    ${EMPTY}
   Empty User Name and Password      ${EMPTY}         ${EMPTY}

.. tip:: 如上例所示, 给列命名使得测试用例更易读易懂. 这种方法之所以可行,
         是因为对用例表的表头那一行, 除了第一格中的数据, 其它的内容都是 :ref:`被忽略的 <ignored data>`

上例中包含了6个独立的测试, 每次都是非法 用户名/密码 的组合情况, 而下面的例子中, 则展示了如何使用 :ref:`test templates`, 在一个用例中测试所有组合情况. 当使用模板时, 所有的轮次都会被执行, 所以从功能上讲, 两者没有区别. 

如上例所示, 每个组合都单独命名为一个测试用例, 很容易看出来测试目的. 不过, 如果是大量的组合情况, 也会带来统计上的混乱. 所以, 到底使用何种风格, 取决于具体的应用场景和个人喜好.

.. sourcecode:: robotframework

   *** Test Cases ***
   Invalid Password
       [Template]    Login with invalid credentials should fail
       invalid          ${VALID PASSWORD}
       ${VALID USER}    invalid
       invalid          whatever
       ${EMPTY}         ${VALID PASSWORD}
       ${VALID USER}    ${EMPTY}
       ${EMPTY}         ${EMPTY}



.. _behavior-driven style:

行为驱动型
~~~~~~~~~~

还可以按照需求的格式来编写测试用例, 让非技术型的项目成员也能理解. 这种 *可执行的需求* 是所谓的 :ref:`验收测试驱动开发 <atdd>` (Acceptance Test Driven Development, ATDD) 或 :ref:`以示例表示规范 <specification by example>` 的基石.

写出这种需求/测试的一种方式是 *Given-When-Then* 样式, 该样式由 :ref:`行为驱动开发 <bdd>` (Behavior Driven Development, BDD) 所普及.
当按照这种样式编写测试用例时, 初始的状态通常由 :name:`Given` 起始的关键字开始, 其中的操作由 :name:`When` 开头的关键字描述, 而预期结果则由 :name:`Then` 开头的关键字处理. 如果某个步骤需要多个操作, 则使用 :name:`And` 或 :name:`But` 将大家连贯起来.

.. sourcecode:: robotframework

   *** Test Cases ***
   Valid Login
       Given login page is open
       When valid username and password are inserted
       and credentials are submitted
       Then welcome page should be open

.. _atdd: http://testobsessed.com/2008/12/08/acceptance-test-driven-development-atdd-an-overview
.. _specification by example: http://en.wikipedia.org/wiki/Specification_by_example
__ http://en.wikipedia.org/wiki/Behavior_Driven_Development


.. _ignore given/when/then/and/but prefixes:

忽略 :name:`Given/When/Then/And/But` 前缀
'''''''''''''''''''''''''''''''''''''''''

以下前缀 :name:`Given`, :name:`When`, :name:`Then`, :name:`And` and :name:`But` 在搜索匹配关键字时, 如果没有全名匹配则会被忽略. 这个规则同时适用于用户关键字和库关键字. 例如, 上例中的 :name:`Given login page is open` 实现为用户关键字的时候, 关键字名称带不带 :name:`Given` 前缀都可以.

忽略前缀可以使得同一个关键字带上不同的前缀使用. 例如 :name:`Welcome page should be open` 还可以用作 :name:`And welcome page should be open`.

.. note:: 前缀 :name:`But` 在 Robot Framework 2.8.7 版本中新增支持.


.. _Embedding data to keywords:

关键字中嵌入数据
'''''''''''''''

当编写具体的例子时, 可以在关键字的实现中传递实际的数据是很有用的. 用户关键字通过 :ref:`embedding arguments into keyword name` 提供支持.

.. When writing concrete examples it is useful to be able pass actual data to
.. keyword implementations. User keywords support this by allowing `embedding
.. arguments into keyword name`_.
