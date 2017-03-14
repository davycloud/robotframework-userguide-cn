.. Advanced features

高级特性
==========

.. contents::
   :depth: 2
   :local:

.. Handling keywords with same names

处理同名关键字
---------------------------------

Robot Framework中的关键字分为 :ref:`库关键字 <Using test libraries>` 和 `用户关键字`_. 前者来自 `标准库`_ 或 `外部库`_, 后者则要么在当前调用的文件中创建, 要么从 `资源文件`_ 中导入. 当用到的关键字变得很多时, 难免会遇到重名的情况, 本节将说明如何处理这种冲突状况.

.. Keyword scopes

关键字的范围
~~~~~~~~~~~~~~

当仅使用关键字名称时, 如果存在若干同名的关键字, Robot Framework将试图决定哪个关键字的优先级最高. 关键字的优先级将由关键字的创建方式决定:

1. 在调用关键字的相同文件中创建. 此关键字拥有最高的优先级.

2. 在资源文件中创建并引入(可以是直接引入,也可以是导入的别的资源文件). 此是第二高优先级.

3. 在外部库中创建. 只有在没有其它同名的用户关键字存在的情况下才会用到. 而且,  
   如果标准库中存在了同名的关键字, 将显示警告.

4. 标准库中创建关键字. 这些关键字的优先级最低.

.. Specifying a keyword explicitly

显式指定关键字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

光靠作用域不能完美的解决重名问题, 因为相同作用域的若干库或者资源中也会有同名的关键字. 此时只能使用 *关键字全名*, 所谓全名就是在关键字名称的前面加上其所在的资源或库的名称作为前缀, 中间使用点(`.`)作为分隔.

对于库中的关键字, 长名称的格式是 :name:`LibraryName.Keyword Name`. 例如, 标准库 OperatingSystem_ 中的关键字 :name:`Run` 可以写作 :name:`OperatingSystem.Run`. 如果库是一个模块或者包, 则必须使用模块或包的全名(例如: :name:`com.company.Library.Some Keyword`). 如果在引用库的时候使用了 `WITH NAME syntax`_, 则前缀名称必须是该自定义的名称.

资源文件中的关键字全名指定格式也是类似. 资源的名称取自资源文件的基础名称(basename)并去掉文件扩展名. 例如, 资源文件 :file:`myresources.html` 中的关键字 :name:`Example` 可以写作 :name:`myresources.Example`. 注意, 这种语法如果遇到多个资源文件的基础名称相同, 则必须修改文件名或者关键字名. 

关键字的全名和关键字的普通名称一样, 同时忽略大小写, 空格和下划线.

.. Specifying explicit priority between libraries and resources

为库和资源显式指定优先级
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果重名冲突的情况比较多, 全部使用全名称格式可能需要不少的工作量. 同时, 全名称格式将难以创建动态的(依赖可用库或资源的)测试用例和用户关键字. 一个针对这两个问题的解决方案是通过一个内置的关键字 :name:`Set Library Search Order` 显式地指定关键字的优先级.

.. note:: 虽然该关键字的名字中包含了 *library*, 但是它不仅作用于库, 还对资源文件有效.

:name:`Set Library Search Order` 接受一个有序列表作为参数, 列表中是库和资源的名称. 当关键字名称遭遇到重名的情况, 将依次在这个列表中的库或资源中查找, 一旦找到即被采用. 如果列表内指定的库和资源没有找到关键字, 则重名冲突造成的执行失败和正常情况一样.

更多的信息和示例请参阅该关键字的文档.

.. Timeouts

超时处理
--------

关键字有可能会遇到执行时间超长或者执行被挂起的情况. Robot Framework允许为 `测试用例`_ 和 `用户关键字`_ 设置超时时长, 如果用例或者关键字没有在指定时长内结束, 则当前还在执行的关键字会被强行终止. 这种情况有可能会导致测试库或系统进入不稳定的状态, 因此, 超时设置只在没有其它更好更安全的办法下才推荐使用. 

通常用户在设计和实现库时, 应该仔细设计以避免出现关键字挂起的情况, 或者实现自身的超时处理机制.

.. Test case timeout

测试用例的超时
~~~~~~~~~~~~~~~~~

测试用例的超时设置可以通过设置表格中的 :setting:`Test Timeout` 设置项, 或者用例表格中的 :setting:`[Timeout]` 设置项. 前者是为当前用例集下的所有的测试用例设定一个默认的超时时长, 而后者则只应用当前单个用例, 并且会覆盖可能存在的默认值.

使用空白的 :setting:`[Timeout]` 设置意味着测试永不超时, 即使已经设置了 :setting:`Test Timeout`. 除了空白还可以使用 `NONE`, 结果一样.

不管在哪里定义超时, 跟在设置项名称后面的第一个格子中包含的就是超时的时长. 该时长必须使用Robot Framework中的 `时间格式`_, 可以是直接的秒数, 也可以是诸如 `1 minute 30 seconds` 这种格式. 值得注意的是, 框架本身总是会有时间消耗的, 所以不建议将超时时长设置短于1秒. 


当超时发生时, 默认的错误提示信息是 `Test timeout <time> exceeded`. 用户可以自定义错误消息, 只需要将错误消息跟在超时时长的后面格子中. 这里的消息设置和文档类似, 可以跨多个单元格.

超时值和错误消息中都可以包含变量.

如果有超时, 运行中的关键字被终止, 当前用例执行失败. 不过, 作为 `test teardown`_ 运行的关键字不会被中断, 因为teardown操作一般都是重要的清理动作. 如果有必要的话, 可以通过设置 `用户关键字的超时`_ 来中断这些关键字.

.. sourcecode:: robotframework

   *** Settings ***
   Test Timeout    2 minutes

   *** Test Cases ***
   Default Timeout
       [Documentation]    Timeout from the Setting table is used
       Some Keyword    argument

   Override
       [Documentation]    Override default, use 10 seconds timeout
       [Timeout]    10
       Some Keyword    argument

   Custom Message
       [Documentation]    Override default and use custom message
       [Timeout]    1min 10s    This is my custom error
       Some Keyword    argument

   Variables
       [Documentation]    It is possible to use variables too
       [Timeout]    ${TIMEOUT}
       Some Keyword    argument

   No Timeout
       [Documentation]    Empty timeout means no timeout even when Test Timeout has been used
       [Timeout]
       Some Keyword    argument

   No Timeout 2
       [Documentation]    Disabling timeout with NONE works too and is more explicit.
       [Timeout]    NONE
       Some Keyword    argument

.. User keyword timeout

用户关键字的超时
~~~~~~~~~~~~~~~~~~~~

在关键字表格中通过设置项 :setting:`[Timeout]` 可以为用户关键字设定超时. 使用的语法格式, 包括时长的值的格式和自定义错误都和 `测试用例的超时`_ 完全一样. 

稍有不同的地方在于当超时发生且没有自定义错误提示信息时, 默认的错误提示信息是 `Keyword timeout <time> exceeded`.

从Robot Framework3.0版本开始, 超时设置可以由一个变量来指定, 既而该变量可以是由参数来指定. 以前的版本中已经支持使用全局变量来指定超时时长.

.. sourcecode:: robotframework

   *** Keywords ***
   Timed Keyword
       [Documentation]    Set only the timeout value and not the custom message.
       [Timeout]    1 minute 42 seconds
       Do Something
       Do Something Else

   Wrapper With Timeout
       [Arguments]    @{args}
       [Documentation]    This keyword is a wrapper that adds a timeout to another keyword.
       [Timeout]    2 minutes    Original Keyword didn't finish in 2 minutes
       Original Keyword    @{args}

   Wrapper With Customizable Timeout
       [Arguments]    ${timeout}    @{args}
       [Documentation]    Same as the above but timeout given as an argument.
       [Timeout]    ${timeout}
       Original Keyword    @{args}

用户关键字的超时可以在其执行的过程中应用. 如果整个关键字的执行时长长于指定的超时时长, 则当前正在执行的关键字会被终止. 用户关键字的超时在测试用例的teardown中同样生效, 而测试用例中的超时则不会影响teardown.

如果用例和关键字(包括嵌套调用的关键字)都设置了超时, 则其中所余时间最短的将首先触发超时.

.. _for loop:

FOR循环
---------

在自动化测试中, 将某些操作重复执行若干次是一个很常见的需求. 在Robot Framework中, 测试库中可以有任意形式的循环结构, 大多数时候循环操作本应该就在测试库中实现. 

Robot Framework也提供了for循环的语法, 这在重复执行来自不同测试库中的关键字的时候很有用.

for循环既可用于测试用例, 也可以在用户关键字中使用. 除非场景特别简单, 不然还是推荐在用户关键字中使用, 这样可以隐藏for循环带来的复杂度. 基本的for循环语法 `FOR item IN sequence` 借鉴于Python, 不过其它脚本如Perl也有类似的语法. 

.. Normal for loop

普通for循环
~~~~~~~~~~~~~~~

普通的for循环语法中, 每次迭代都从列表中取一个值赋给变量. 语法以 `:FOR` 开始, 注意开始的冒号是必需的, 以便和其它普通关键字区分开. 跟在后面单元格中的是循环变量, 接下来的格子则必须是 `IN`, 后面的格子(可能是多个)里则包含的是待迭代的值. 这些值中可以包含 变量_, 包括 列表变量_.

for循环中使用的关键字跟在下面的行中, 必须向右缩进一格. 当使用的是 `plain text format`_, 缩进单元格必须使用 `escaped with a backslash`__, 而其它的数据格式则只需要保持空白就行. for循环结束于正常缩进(即不再缩进)的行, 或者是整个表格的结尾. 

.. sourcecode:: robotframework

   *** Test Cases ***
   Example 1
       :FOR    ${animal}    IN    cat    dog
       \    Log    ${animal}
       \    Log    2nd keyword
       Log    Outside loop

   Example 2
       :FOR    ${var}    IN    one    two
       ...     ${3}    four    ${last}
       \    Log    ${var}

上面 :name:`Example 1` 将迭代执行两次, 第一次循环变量 `${animal}` 被赋值 `cat`, 接下来是 `dog`. 循环体包含了两次 :name:`Log` 关键字调用. 第二个例子中, 循环值 `分成了多行`__, 循环迭代了5次.

在for循环中使用 `列表变量`_ 更方便. 如下面的例子, `@{ELEMENTS}` 是任意长度的列表, 每次迭代会依次对列表中的元素调用 :name:`Start Element`.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       :FOR    ${element}    IN    @{ELEMENTS}
       \    Start Element  ${element}

.. Nested for loops

for循环的嵌套
~~~~~~~~~~~~~~~~

Robot Framework的for语法并不支持嵌套, 不过可以通过用户关键字封装for循环, 然后在另一个for循环中调用.

.. sourcecode:: robotframework

   *** Keywords ***
   Handle Table
       [Arguments]    @{table}
       :FOR    ${row}    IN    @{table}
       \    Handle Row    @{row}

   Handle Row
       [Arguments]    @{row}
       :FOR    ${cell}    IN    @{row}
       \    Handle Cell    ${cell}

__ `Dividing test data to several rows`_
__ Escaping_

.. Using several loop variables

使用多个循环变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

和Python的for语句类似, 循环变量可以有多个. 该语法和正常的循环语句一样, 只是在 `:FOR` 和 `IN` 之间有多个循环变量, 每个变量占一格. 循环变量的个数可以是任意个, 但是它们必须能够被值的个数整除.

如果有很多值需要迭代, 通常会把它们在循环变量的下面组织对齐, 以提高可读性, 如下面例子中第一个循环:

.. sourcecode:: robotframework

   *** Test Cases ***
   Three loop variables
       :FOR    ${index}    ${english}    ${finnish}    IN
       ...     1           cat           kissa
       ...     2           dog           koira
       ...     3           horse         hevonen
       \    Add to dictionary    ${english}    ${finnish}    ${index}
       :FOR    ${name}    ${id}    IN    @{EMPLOYERS}
       \    Create    ${name}    ${id}

.. For-in-range loop

for-in-range 循环
~~~~~~~~~~~~~~~~~

前面的for循环总是迭代一个序列, 这是最常见的形式, 但是有时候, 针对某个特定次数的for循环也很有用. Robot Framework提高了特殊的 `FOR index IN RANGE limit` 语法来实现这种目的. 同样, 该语法借鉴于Python.

和普通的for循环类似, for-in-range循环同样始于 `:FOR`, 后面跟循环变量. 只是这种情况下, 循环变量只能有一个, 该变量将包含当前循环的下标(index). 循环变量后的格子中必须包含 `IN RANGE`, 后面的格子包含的是循环的限定范围.

最简单的情况是只给出循环的上限, 这种情况下, 循环下标从0开始, 逐次递加1, 直到上限为止(不包括上限). 还可以同时给出起始值(start)和结束值(end), 这种情况下, 循环从start开始, 逐次递加1, 直到end-1. 再复杂一点的情况是通过第3个参数指定每次递进的值(step), 该值可以为负数. 

对上下限值可以使用简单的算术操作, 如加法和减法, 这在这些值是变量的时候特别有用.

从Robot Framework 2.8.7版本开始, start, end 和 step 都可以使用浮点数.

.. sourcecode:: robotframework

   *** Test Cases ***
   Only upper limit
       [Documentation]    Loops over values from 0 to 9
       :FOR    ${index}    IN RANGE    10
       \    Log    ${index}

   Start and end
       [Documentation]  Loops over values from 1 to 10
       :FOR    ${index}    IN RANGE    1    11
       \    Log    ${index}

   Also step given
       [Documentation]  Loops over values 5, 15, and 25
       :FOR    ${index}    IN RANGE    5    26    10
       \    Log    ${index}

   Negative step
       [Documentation]  Loops over values 13, 3, and -7
       :FOR    ${index}    IN RANGE    13    -13    -10
       \    Log    ${index}

   Arithmetics
       [Documentation]  Arithmetics with variable
       :FOR    ${index}    IN RANGE    ${var}+1
       \    Log    ${index}

   Float parameters
       [Documentation]  Loops over values 3.14, 4.34, and 5.34
       :FOR    ${index}    IN RANGE    3.14    6.09    1.2
       \    Log    ${index}

.. For-in-enumerate loop

for-in-enumerate 循环
~~~~~~~~~~~~~~~~~~~~~

有时候循环迭代某个列表的时候, 同时又想跟踪当前元素在列表中的位置, 这时候就可以用到Robot Framework的 `FOR index ... IN ENUMERATE ...` 语法. 该语法源于 `Python built-in function <https://docs.python.org/2/library/functions.html#enumerate>`_.

For-in-enumerate循环和普通for循环一样, 只是在循环变量的前面增加一个额外的索引变量, 循环变量后面跟着的是 `IN ENUMERATE` 而不是 `IN`. 索引值从`0`开始.

例如, 下面例子中两个测试用例做得是同一件事:

.. sourcecode:: robotframework

   *** Variables ***
   @{LIST}         a    b    c

   *** Test Cases ***
   Manage index manually
       ${index} =    Set Variable    -1
       : FOR    ${item}    IN    @{LIST}
       \    ${index} =    Evaluate    ${index} + 1
       \    My Keyword    ${index}    ${item}

   For-in-enumerate
       : FOR    ${index}    ${item}    IN ENUMERATE    @{LIST}
       \    My Keyword    ${index}    ${item}

和普通的for循环一样, 一次迭代可以处理多个值, 只要列表元素的总数可以整除一次迭代的变量个数(当然索引变量是不算在内的).

.. sourcecode:: robotframework

   *** Test Case ***
   For-in-enumerate with two values per iteration
       :FOR    ${index}    ${english}    ${finnish}    IN ENUMERATE
       ...    cat      kissa
       ...    dog      koira
       ...    horse    hevonen
       \    Add to dictionary    ${english}    ${finnish}    ${index}

For-in-enumerate 循环是 Robot Framework 2.9版本新增功能.

.. For-in-zip loop

for-in-zip循环
~~~~~~~~~~~~~~~

有时候需要将几个相关的列表并在一起处理, Robot Framework 使用 `FOR ... IN ZIP ...` 语法来处理, 该方法来源于 `Python built-in zip function <https://docs.python.org/2/library/functions.html#zip>`_.

来看个例子, 下面两个用例的作用是一样的:

.. sourcecode:: robotframework

   *** Variables ***
   @{NUMBERS}      ${1}    ${2}    ${5}
   @{NAMES}        one     two     five

   *** Test Cases ***
   Iterate over two lists manually
       ${length}=    Get Length    ${NUMBERS}
       : FOR    ${idx}    IN RANGE    ${length}
       \    Number Should Be Named    ${NUMBERS}[${idx}]    ${NAMES}[${idx}]

   For-in-zip
       : FOR    ${number}    ${name}    IN ZIP    ${NUMBERS}    ${NAMES}
       \    Number Should Be Named    ${number}    ${name}

和其它循环的语法类似, for-in-zip要求跟在循环变量后面格子中的是 `IN ZIP`.

for-in-zip循环的值必须是列表或数组类型的序列, 并且循环变量的数量必须和列表的数量相同. 而迭代的停止取决于其中最短的那个列表.

注意, for-in-zip后面用到的列表通常都是以 `scalar variables`_ 的形式给出, 如 `${list}`. 如果是 `list variable`_ 形式, 则要求这个列表中的元素本身也是列表. (这里需要对着两种变量的格式理解充分)

For-in-zip 循环是 Robot Framework 2.9版本新增功能.

.. Exiting for loop

退出for循环
~~~~~~~~~~~~~~~~

通常for循环在所有元素都迭代完成后自然结束, 也有可能当其中的关键字执行失败而退出. 如果需要提前退出循环, 可以调用 BuiltIn_ 关键字 :name:`Exit For Loop` 和 :name:`Exit For Loop If`. 它们的作用类似于编程语言中的 `break` 语句.

:name:`Exit For Loop` 和 :name:`Exit For Loop If` 可以直接在for循环内使用, 也可以在for循环中调用的关键字中使用. 这两种情况都可以让测试跳过循环继续往下执行. 不可以在for循环的外面使用了这两个关键字, 否则会引起错误.

.. sourcecode:: robotframework

   *** Test Cases ***
   Exit Example
       ${text} =    Set Variable    ${EMPTY}
       :FOR    ${var}    IN    one    two
       \    Run Keyword If    '${var}' == 'two'    Exit For Loop
       \    ${text} =    Set Variable    ${text}${var}
       Should Be Equal    ${text}    one

上例中, 可以使用 :name:`Exit For Loop If` 来替代  :name:`Exit For Loop` 加 :name:`Run Keyword If` 的用法. 更多的信息和示例请参阅这些关键字的文档.

.. note:: :name:`Exit For Loop If` 在Robot Framework 2.8版本新增.

.. Continuing for loop

继续for循环
~~~~~~~~~~~~~~~~~~~

除了退出整个for循环, 有时候需要的是略过本次迭代而进入下一轮迭代. 这时可以使用 BuiltIn_ 关键字 :name:`Continue For Loop` 和 :name:`Continue For Loop If`, 和编程语言中的  `continue` 语句类似.

:name:`Continue For Loop` 和 :name:`Continue For Loop If` 可以直接在for循环内使用, 也可以在for循环中调用的关键字中使用. 两种情况下都可以使得本次迭代被跳过, 进入下一次迭代. 如果本次迭代就是最后一次, 则整个循环结束. 同样, 在循环外调用这些关键字是错误的.

.. sourcecode:: robotframework

   *** Test Cases ***
   Continue Example
       ${text} =    Set Variable    ${EMPTY}
       :FOR    ${var}    IN    one    two    three
       \    Continue For Loop If    '${var}' == 'two'
       \    ${text} =    Set Variable    ${text}${var}
       Should Be Equal    ${text}    onethree

关于这些关键字更多的信息和示例请参阅它们在 BuiltIn_ 库的文档

.. note:: :name:`Continue For Loop` 和 :name:`Continue For Loop If` 
          都是在Robot Framework 2.8版本新增.

.. Removing unnecessary keywords from outputs

去除输出中不必要的关键字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

拥有多次迭代的for循环可以产生大量的输出信息, 从而造成 output_ 和 log_ 文件大小的显著增加. 从Robot Framework 2.7版本开始, 可以使用命令行选项 :option:`--RemoveKeywords FOR` 从输出中 `remove unnecessary keywords`__.

__ `Removing and flattening keywords`_

.. Repeating single keyword

重复执行单个关键字
~~~~~~~~~~~~~~~~~~~~~~~~

当每次循环只需要重复调用一个关键字的时候, 使用for循环显得有点小题大做. 这时候可以使用 BuiltIn_ 关键字 :name:`Repeat Keyword`. 该关键字的第一个参数要重复的次数, 后面是要重复的关键字, 以及它的参数. 重复次数的值可以加上后缀 `times` 或者 `x` 以提高可读性.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Repeat Keyword    5    Some Keyword    arg1    arg2
       Repeat Keyword    42 times    My Keyword
       Repeat Keyword    ${var}    Another Keyword    argument

.. Conditional execution

条件执行
---------------------

通常来说, 不建议在测试用例中使用条件判断的逻辑, 甚至在用户关键字中也不要用, 因为这会使得用例和关键字变得难以理解和维护. 这种逻辑应该放在测试库中, 这样就可以很自然地使用编程语言的语法结构来实现. 

然而, 总会有些时候会发现条件判断逻辑是有用的, 虽然Robot Framework并没有提供if/else的语法结构, 但是我们可以通过其它几种方式来实现相同的效果.

- 在 `测试用例`__ 和 `测试套件`__ 的setup或teardown中的关键字名称可以使用变量来代替. 
  这样利于根据条件来改变关键字, 如通过命令行.

- BuiltIn_ 关键字 :name:`Run Keyword` 把其它关键字作为参数来调用, 自然也可以是变量.
  这个变量的值就可以动态的确立, 如根据前面的关键字结果或者是命令行.

- BuiltIn_ 关键字 :name:`Run Keyword If` 和 :name:`Run Keyword Unless` 只在
  特定的表达式结果是true或false的情况才调用指定的关键字. 所以简单的 if/else 结构,
  完全可以由它们来完成. 详细的例子可以参考关键字的文档.

- 另一个 BuiltIn_ 关键字 :name:`Set Variable If` 可以根据条件表达式的结果, 动态的
  为变量赋值.

- 还有几个 BuiltIn_ 关键字可以在测试用例/套件执行失败或成功的时候才调用指定的关键字.

__ `Test setup and teardown`_
__ `Suite setup and teardown`_


.. Parallel execution of keywords

关键字的并发执行
------------------------------

如果有并发执行的需求, 必须在测试库中实现, 并且代码应该运行在后台. 通常这意味着测试库应该有一个关键字如 :name:`Start Something` 来启动执行, 并且立即返回, 然后通过另一个关键字如 :name:`Get Results From Something`, 等待获取执行的结果并返回. 

可以参考 OperatingSystem_ 库中的 :name:`Start Process` 和 :name:`Read Process Output` 实现.
