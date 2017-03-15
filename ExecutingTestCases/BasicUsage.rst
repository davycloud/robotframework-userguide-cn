.. Basic usage

基本用法
===========

Robot Framework 中的测试用例通过命令行执行, 执行的结果默认生成一个XML格式的 `output file`_ 和HTML格式的 report_ and log_. 执行完毕后, 这些输出文件可以通过Rebot工具整合或者进行 `后期处理`__.

__ `Post-processing outputs`_

.. contents::
   :depth: 2
   :local:

.. _executing test cases:

开始执行测试
------------

.. Synopsis

概要
~~~~

::

    robot [options] data_sources
    python|jython|ipy -m robot [options] data_sources
    python|jython|ipy path/to/robot/ [options] data_sources
    java -jar robotframework.jar [options] data_sources

测试的执行通常是使用 ``robot`` 这个 `runner script`_ 启动. 此外还可以通过Python解释器直接运行安装的 `robot module`__ 或 `robot directory`__. 最后还可以使用 `standalone JAR distribution`_.

Test execution is normally started using the ``robot`` `runner script`_.
Alternatively it is possible to execute the installed `robot module`__ or
`robot directory`__ directly using the selected interpreter. The final
alternative is using the `standalone JAR distribution`_.

.. note:: Robot Framework 3.0版本之前并没有 ``robot`` 脚本, 而是针对不同的解释器,
          Python, Jython和IronPython, 分别使用 ``pybot``, ``jybot`` 和 ``ipybot``. 这些脚本目前还保留, 不过计划在未来的版本中逐渐废弃.

不管用什么方式执行, 待测试数据的路径(可以是多个)总是作为参数跟在命令后面. 此外, 还可以指定不同的命令行选项, 用来控制测试执行或者结果输出.

__ `Executing installed robot module`_
__ `Executing installed robot directory`_

.. Specifying test data to be executed

指定待执行的测试数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robot Framework 中的测试用例是通过 文件_ 和 目录_ 的方式创建和组织的, 所以要执行时把这些文件或目录的路径传给执行脚本就可以了. 路径可以是绝对路径, 不过大多数情况下, 使用的还是相对路径. 

指定的文件或者目录将被用来创建顶层测试套件, 除非通过命令行选项 :option:`--name` 指定名字, 否则这个测试套件的名字也就是 `文件或者目录名`__. 

下面的例子给出了几种可能的执行方式, 注意到这些例子中只使用了 ``robot`` 脚本, 实际其它执行方法也是类似的.

__ `Test case files`_
__ `Test suite directories`_
__ `Setting the name`_
__ `Test suite name and documentation`_

::

   robot tests.robot
   robot path/to/my_tests/
   robot c:\robot\tests.robot

还可以一次给出多个测试用例文件或目录, 之间用空格隔开. 这种情况下, Robot Framework自动创建顶层测试套件, 这些指定的文件和目录都成为它的子套件. 这个顶层的测试套件的名字由各子套件的名字使用 ` & ` 拼接而成. 例如, 下面的例子中的顶层套件的名字是 :name:`My Tests & Your Tests`. 可以想见这个自动创建的名字将会非常冗长, 所以大多数情况下, 最好还是通过 :option:`--name` 命令行选项来指定一个名字, 如下面的第二个例子::

   robot my_tests.robot your_tests.robot
   robot --name Example path/to/tests/pattern_*.robot

.. Using command line options

命令行选项
--------------------------

Robot Framework提供了为数不少的命令行选项用来控制测试用例的执行和输出. 本节将介绍这些选项的语法和到底有哪些选项可用. 具体如何使用它们将在本章的其它小节讨论.

.. Using options

使用选项
~~~~~~~~~~~~~

当要使用命令行选项时, 这些选项必须总是出现在执行脚本和数据源之间. 例如::

   robot -L debug my_tests.robot
   robot --include smoke --variable HOST:10.0.0.42 path/to/tests/

.. Short and long options

短选项和长选项
~~~~~~~~~~~~~~~~~~~~~~

选项总是有一个较长的名字, 如 :option:`--name`, 最常用的那些选项同时还有一个短名字, 例如 :option:`-N`. 此外, 长选项名称可以被截断, 只要它仍是唯一无歧义的即可. 例如, `--logle DEBUG` 可以生效, 但 `--lo log.html` 不行, 因为前者唯一匹配上了 :option:`--loglevel`, 而后者匹配了多个选项. 

短选项和截短的选项在手动执行测试用例时都挺实用, 不过在 `start-up scripts`_ 中推荐使用长选项名称, 因为它们更易懂.

长选项的格式是大小写无关的, 这有益于写出更易读的选项名字. 例如, :option:`--SuiteStatLevel` 等价于 :option:`--suitestatlevel`, 但是前者显然更清楚易读.

.. Setting option values

设置选项值
~~~~~~~~~~~~~~~~~~~~~

大多数的选项需要在选项名称后面给定一个值. 长选项和短选项都接受在空格后面跟上选项的值, 例如,  `--include tag` 或 `-i tag`. 同时对于长选项, 还可以使用等号(`=`), 例如 `--include=tag`, 而对于短选项, 中间的分隔符则是可用省略的, 如 `-itag`.

有的选项可用被指定多次. 例如, `--variable VAR1:value --variable VAR2:another` 设置了两个变量. 如果一个选项只能接受一个值而被指定了多次, 则生效的将是最后的那个.

.. Disabling options accepting no values

禁用不带值的选项
~~~~~~~~~~~~~~~~~~~~~~~~~~

不接受值的选项可用通过在选项名前加上(或去掉)前缀 `no` 来禁用. 最后的那个选项优先级最高. 例如, `--dryrun --dryrun --nodryrun --nostatusrc --statusrc` 最终不会激活dry-run模式, 并且将返回正常的状态rc(即生效的是 `--nodryrun --statusrc`).

.. note:: 在选项前加/减 `no` 前缀是Robot Framework 2.9版本才有的新特性功能.
          早期版本中要禁用不带值的选项是通过再次使用一样的选项(??).

.. Simple patterns

简单模式
~~~~~~~~~~~~~~~

很多的命令行选项都可以接受所谓 *简单模式* 的参数, 这种类似 `glob patterns`__ 的匹配规则如下:

- `*` 匹配任意字符串, 空字符也不例外.
- `?` 匹配单个字符串.
- 除非指定, 否则模式匹配是大小写, 空白, 以及下划线无关的.

例如::

   --test Example*     # Matches tests with name starting 'Example', case insensitively.
   --include f??       # Matches tests with a tag that starts with 'f' or 'F' and is three characters long.

__ http://en.wikipedia.org/wiki/Glob_(programming)

.. Tag patterns

标签模式
~~~~~~~~~~~~

大多数标签(tag)相关的选项名可以以 *标签模式* 接受参数. 这种模式和 `简单模式`_ 类似, 同时支持 `AND`, `OR` 和 `NOT` 运算符. 这些操作符用来将多个标签或模式组合起来. 具体看下面的例子.

Most tag related options accept arguments as *tag patterns*. They have all the
same characteristics as `simple patterns`_, but they also support `AND`,
`OR` and `NOT` operators explained below. These operators can be
used for combining two or more individual tags or patterns together.

`AND` 或 `&`
   只有所有单个的模式匹配了, 整个模式才匹配. `AND` 和 `&` 是等价的::

      --include fooANDbar     # Matches tests containing tags 'foo' and 'bar'.
      --exclude xx&yy&zz      # Matches tests containing tags 'xx', 'yy', and 'zz'.

`OR`
   任意单个的模式匹配, 整个模式即匹配::

      --include fooORbar      # Matches tests containing either tag 'foo' or tag 'bar'.
      --exclude xxORyyORzz    # Matches tests containing any of tags 'xx', 'yy', or 'zz'.

.. If used multiple times, none of the patterns after the first `NOT` must not match

`NOT`
   整个模式在左边的模式匹配, 同时右边模式不匹配的情况下才算匹配. 如果使用多次, 则第一个 `NOT` 后面的模式都不能匹配::

      --include fooNOTbar     # Matches tests containing tag 'foo' but not tag 'bar'.
      --exclude xxNOTyyNOTzz  # Matches tests containing tag 'xx' but not tag 'yy' or tag 'zz'.

   从Robot Framework 2.9开始, 整个模式也可以以 `NOT` 开始, 这种情况下, `NOT` 后面的模式都不匹配的时候代表整个模式匹配::

      --include NOTfoo        # Matches tests not containing tag 'foo'
      --include NOTfooANDbar  # Matches tests not containing tags 'foo' and 'bar'

上面的操作符可以同时混合使用, 操作符的优先级从高到低是: `AND`, `OR` 和 `NOT`::

    --include xANDyORz      # Matches tests containing either tags 'x' and 'y', or tag 'z'.
    --include xORyNOTz      # Matches tests containing either tag 'x' or 'y', but not tag 'z'.
    --include xNOTyANDz     # Matches tests containing tag 'x', but not tags 'y' and 'z'.

虽然标签匹配本身是大小写无关的, 但是所有的操作符都是大小写敏感的, 必须都是全大写字母. 如果标签本身(也许是单词里面)恰巧包含了 `AND`, `OR` 或 `NOT`, 它们需要转换成小写字母以避免无意中的操作符运算::

    --include port          # Matches tests containing tag 'port', case-insensitively
    --include PORT          # Matches tests containing tag 'P' or 'T', case-insensitively
    --exclude handoverORportNOTnotification

.. note:: `OR` 操作符是 Robot Framework 2.8.4 版本才新加入的.

环境变量 ``ROBOT_OPTIONS`` 和 ``REBOT_OPTIONS`` 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

环境变量 ``ROBOT_OPTIONS`` 和 ``REBOT_OPTIONS`` 分别被用来指定 `测试执行`_ 和 `结果后处理`__ 的默认选项. 选项名称和值必须以 空格分开的列表 形式给出, 它们将出现在其它所有显式给出的命令行选项之前. 

这些环境变量的主要作用是用来设置某些选项的全局的默认值, 避免每次执行测试或使用Rebot时重复输入这些选项.

.. sourcecode:: bash

   export ROBOT_OPTIONS="--critical regression --tagdoc 'mytag:Example doc with spaces'"
   robot tests.robot
   export REBOT_OPTIONS="--reportbackground green:yellow:red"
   rebot --name example output.xml

.. note:: 环境变量 ``ROBOT_OPTIONS`` 和 ``REBOT_OPTIONS`` 是 Robot Framework 
          2.8.2版本后加入的功能.

          使用引号将包含空格的值括起来, 是 Robot Framework 2.9.2新增功能.

__ `Post-processing outputs`_

.. Test results

测试结果
------------

.. Command line output

命令行输出
~~~~~~~~~~~~~~~~~~~

测试执行的最直观的输出就是命令行的输出显示. 所有被执行的测试套件和测试用例, 以及它们执行的结果, 都实时地显示出来. 下面的例子展示了一个只包含两个测试用例的简单测试集的执行输出情况::

   ==============================================================================
   Example test suite
   ==============================================================================
   First test :: Possible test documentation                             | PASS |
   ------------------------------------------------------------------------------
   Second test                                                           | FAIL |
   Error message is displayed here
   ==============================================================================
   Example test suite                                                    | FAIL |
   2 critical tests, 1 passed, 1 failed
   2 tests total, 1 passed, 1 failed
   ==============================================================================
   Output:  /path/to/output.xml
   Report:  /path/to/report.html
   Log:     /path/to/log.html

从Robot Framework2.7版本开始, 控制台上在用例执行中还会有顶层关键字执行结束的通知. 一个绿色的点表示执行通过, 而红色的F表示失败. 这些标示在行末显示, 并且在测试最终完成后被结果状态覆盖. 如果将控制台输出重定向到文件, 则这些标示不会出现.

.. Generated output files

生成的输出文件
~~~~~~~~~~~~~~~~~~~~~~

命令行输出内容非常有限, 所以通常需要单独的输出文件用来检查测试的结果. 如上例所示, 默认生成3个输出文件. 首先是一个XML格式的文件, 包含了关于测试执行的所有信息. 第二个文件是一个高层的报告文件, 第三个文件则是详细的日志文件. 这些文件和其它可能是输出文件将在 `Different output files`_ 中详细讨论.

.. Return codes

返回码
~~~~~~~~~~~~

执行脚本通过返回码和系统进行通讯, 上报整个测试执行的情况. 当执行成功开始, 并且没有 `critical test`_ 失败, 则返回码为0. 

所有可能的返回码如下表所示.

.. table:: Possible return codes
   :class: tabular

   ========  ==========================================
      RC                    Explanation
   ========  ==========================================
   0         All critical tests passed.
   1-249     Returned number of critical tests failed.
   250       250 or more critical failures.
   251       Help or version information printed.
   252       Invalid test data or command line options.
   253       Test execution stopped by user.
   255       Unexpected internal error.
   ========  ==========================================

返回码总是应该在执行后轻易获取到, 这样就能轻松的自动判断整个执行的状态. 例如, 在bash shell中, 返回码保存在特殊的变量 `$?` 中, 而在Windows中, 是在 `%ERRORLEVEL%` 中. 如果你用到了其它外部工具来执行测试, 请参考它们的文档来了解如何获取返回码.

如果设置了命令行选项 :option:`--NoStatusRC`, 则返回码在有关键失败发生时也将是0. 这在某些场景下会很有用, 例如, 在持续集成(continuous integration)中, CI服务器需要在决定整个测试执行的状态之前, 对测试结果进行后处理.

.. note:: 有些返回码在 Rebot_ 有用到.

.. Errors and warnings during execution

执行过程中的错误和警告
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

执行过程中可能会发生一些未预料到的问题, 例如导入库失败, 导入资源文件失败, 或者是关键字已经 废弃__. 取决于问题的严重性, 它们被分类为错误和警告. 

错误和警告信息会同时写入控制台(如果使用的是标准错误流的话), 并出现在日志文件的单独的 *Test Execution Errors* 章节中, 同时还会写入Robot Framework的 `系统日志`_. 除了Robot Framework自己产生的错误和警告信息, 测试库也可以生成 `errors and warnings`_.

下面的例子展示了错误和警告在日志文件中的样子.

.. raw:: html

   <table class="messages">
     <tr>
       <td class="time">20090322&nbsp;19:58:42.528</td>
       <td class="error level">ERROR</td>
       <td class="msg">Error in file '/home/robot/tests.robot' in table 'Setting' in element on row 2: Resource file 'resource.robot' does not exist</td>
     </tr>
     <tr>
       <td class="time">20090322&nbsp;19:58:43.931</td>
       <td class="warn level">WARN</td>
       <td class="msg">Keyword 'SomeLibrary.Example Keyword' is deprecated. Use keyword `Other Keyword` instead.</td>
     </tr>
   </table>

__ `Deprecating keywords`_

.. Escaping complicated characters

转义特殊字符
-------------------------------

由于空格被用来分隔选项, 所以想在选项值中使用空格就会产生问题. 有的选项, 例如 :option:`--name`, 会自动将下划线转换为空格, 但在其他选项中, 空格则必须要被转义. 除此之外还有很多特殊字符也无法简单的在命令行中使用. 

使用反斜杠或者引号来转义这些复杂的字符并不总是有效的, 所以Robot Framework有自己的通用转义机制. 另外一种方法则是使用 `参数文件`_, 将所有选项以纯文本的格式写入其中. 这两种方法在测试执行, 测试结果处理, 以及使用其他第三方支持工具时, 都有相同或相似的功能.

Robot Framework的命令行转义机制中, 特殊字符可以自由地选用替代字符来转义. 使用命令行选项 :option:`--escape (-E)`, 该选项的参数格式是 `what:with`, 其中 `what` 是待转义字符的名称, `with` 是要替代它的普通字符. 可用这种方法转义的字符如下表所列:

.. table:: Available escapes
   :class: tabular

   =========  =============  =========  =============
   Character   Name to use   Character   Name to use
   =========  =============  =========  =============
   &          amp            (          paren1
   '          apos           )          paren2
   @          at             %          percent
   \\         bslash         \|         pipe
   :          colon          ?          quest
   ,          comma          "          quot
   {          curly1         ;          semic
   }          curly2         /          slash
   $          dollar         \          space
   !          exclam         [          square1
   >          gt             ]          square2
   #          hash           \*         star
   <          lt             \          \
   =========  =============  =========  =============

看看下面的例子会更容易理解. 第一个例子中, 选项metadata `X` 最终的值是 `Value with spaces`, 而第二个例子中, 变量 `${VAR}` 被赋值为 `"Hello, world!"`::

    --escape space:_ --metadata X:Value_with_spaces
    -E space:SP -E quot:QU -E comma:CO -E exclam:EX -v VAR:QUHelloCOSPworldEXQU

注意所有的命令行参数, 包括测试数据的路径, 都会被转义. 所以, 必须小心地选择转义字符的顺序.

.. Argument files

参数文件
--------------

参数文件就是把所有或部分命令行选项和参数放在一个外部文件中, 这样可以避免命令行中的字符问题. 如果用到选项或参数很多, 使用参数文件也可以避免命令行变得太长.

参数文件通过选项 :option:`--argumentfile (-A)` 指定, 同时仍然可以使用其它的命令行选项.

.. Argument file syntax

参数文件的语法
~~~~~~~~~~~~~~~~~~~~

参数文件可以同时包含命令行选项和待测试数据的路径, 一条选项或者一个数据源占一行. 短选项和长选项都可以, 不过推荐使用更容易看懂的长选项. 参数文件中可以无需转义使用任意字符, 但每行开头和结尾的空格会被忽略. 此外, 空行和以井号(#)开始的行也会被忽略::

   --doc This is an example (where "special characters" are ok!)
   --metadata X:Value with spaces
   --variable VAR:Hello, world!
   # This is a comment
   path/to/my/tests

上例中, 选项名称和值之间使用单个空格隔开. 在 Robot Framework 2.7.6 或更高版本中, 还可以使用等号(=)或者任意的空格来隔开. 例如, 下面3行的作用是等价的::

    --name An Example
    --name=An Example
    --name       An Example

如果参数文件包含了non-ASCII字符, 则文件必须以UTF-8编码保存. 

.. Using argument files

使用参数文件
~~~~~~~~~~~~~~~~~~~~

参数文件既可以单独使用(其中包含所有用到的选项和测试数据路径), 也可以和其它选项和路径一起使用. 

当参数文件和其它参数一起使用时, 参数文件中的参数列表会出现在参数文件选项所在的位置. 也就是说, 在参数文件选项之前的选项, 会被参数文件中的选项覆盖掉, 而在其之后的则相反. 同时参数文件选项 :option:`--argumentfile` 还可以多次指定, 甚至递归地使用::

   robot --argumentfile all_arguments.robot
   robot --name Example --argumentfile other_options_and_paths.robot
   robot --argumentfile default_options.txt --name Example my_tests.robot
   robot -A first.txt -A second.txt -A third.txt tests.robot

.. Reading argument files from standard input

从标准输入中读取参数文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

特殊的参数文件名 `STDIN` 可以用来从标准输入中读取参数文件. 这在使用脚本生成参数文件时很有用, 例如::

   generate_arguments.sh | robot --argumentfile STDIN
   generate_arguments.sh | robot --name Example --argumentfile STDIN tests.robot

.. Getting help and version information

获取帮助和版本信息
------------------------------------

当执行测试用例或后处理输出的命令时, 都可以通过选项 :option:`--help (-h)` 来获取命令行的帮助信息. 帮助信息包括一个综合的概述和可用的命令行选项简介.

所有的可执行脚本还提供了获取版本信息的选项 :option:`--version`, 版本信息包含了Python(或Jython)的版本和平台类型::

   $ robot --version
   Robot Framework 3.0 (Jython 2.7.0 on java1.7.0_45)

   C:\>rebot --version
   Rebot 3.0 (Python 2.7.10 on win32)

.. _start-up script:
.. _start-up scripts:

.. Creating start-up scripts

创建启动脚本
-------------------------

测试用例通常都是通过持续集成系统或者其他系统来自动化地执行. 这时通常需要有一个脚本来启动测试执行, 有可能还需要通过某种手段在结束后执行结果处理. 并且这些脚本在手动执行时也会非常有用, 特别是当有大量的命令行选项要设置或者准备整个测试环境的过程很复杂的时候.

在类UNIX系统中, shell脚本提供简单而强大的机制来创建自定义的启动脚本. Windows环境下的批处理文件也可以胜任, 不过功能限制较多且往往更加难懂. 一个平台无关的选择是使用Python或者其他高层次的编程语言. 不管选择何种语言, 都推荐使用长选项名称, 因为它们相对短选项名更加清晰易懂.

下面第一个例子中, 相同的web测试用例针对不同的浏览器分别执行, 并且在执行后将输出文件合并在一起. 使用shell脚本很容易实现, 实际上只要把所需的命令逐个列出即可:

.. sourcecode:: bash

   #!/bin/bash
   robot --variable BROWSER:Firefox --name Firefox --log none --report none --output out/fx.xml login
   robot --variable BROWSER:IE --name IE --log none --report none --output out/ie.xml login
   rebot --name Login --outputdir out --output login.xml out/fx.xml out/ie.xml

使用Windows批处理文件来实现上面相同的功能也不会很复杂. 重要的是记住在Windows系统中, ``robot`` 和 ``rebot`` 命令都是通过批处理文件来实现的, 所以在另一个批处理文件中必须使用 ``call`` 来调用. 否则整个执行会在第一个批处理文件结束的时候终止.

.. sourcecode:: bat

   @echo off
   call robot --variable BROWSER:Firefox --name Firefox --log none --report none --output out\fx.xml login
   call robot --variable BROWSER:IE --name IE --log none --report none --output out\ie.xml login
   call rebot --name Login --outputdir out --output login.xml out\fx.xml out\ie.xml

下面的例子, 测试执行前要把 :file:`lib` 目录下的jar文件放到 ``CLASSPATH`` 环境变量中. 在这些例子中, 启动脚本要求测试数据的路径以参数的形式提供. 同时, 虽然脚本中已经设置了不少选项, 仍然还可以自由地使用命令行选项.  所有这一切通过bash脚本实现起来都很直接:

.. sourcecode:: bash

   #!/bin/bash

   cp=.
   for jar in lib/*.jar; do
       cp=$cp:$jar
   done
   export CLASSPATH=$cp

   robot --ouputdir /tmp/logs --suitestatlevel 2 $*

而使用Windows批处理文件来实现则相对要麻烦一点. 困难的地方在于在For循环中设置包含JAR包的变量, 除非使用一个辅助函数, 否则无法完成.

.. sourcecode:: bat

   @echo off

   set CP=.
   for %%jar in (lib\*.jar) do (
       call :set_cp %%jar
   )
   set CLASSPATH=%CP%

   robot --ouputdir c:\temp\logs --suitestatlevel 2 %*

   goto :eof

   :: Helper for setting variables inside a for loop
   :set_cp
       set CP=%CP%;%1
   goto :eof

.. Modifying Java startup parameters

修改Java启动参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

使用Jython的时候有时需要修改Java的启动参数. 最常见的场景就是增大JVM最大内存数. 有两个简单的方法来配置JVM选项:

1. 设置 ``JYTHON_OPTS`` 环境变量. 可以是在操作系统层面修改以使其永久生效, 
   也可以是在自定义启动脚本中设定, 则只在当次执行时有效.

2. 通过选项 :option:`-J` 把所需的Java参数传递给Jython, 进而传给Java. 
   这在直接 `executing installed robot module`_ 时特别简单::
   
      jython -J-Xmx1024m -m robot tests.robot

.. Debugging problems

调试问题
------------------

测试用例执行失败如果是因为被测系统的问题则意味着该用例找到了一个bug, 但是还有种不幸的情况是测试用例本身就是错误的(buggy).

失败的错误消息会在 `command line output`_ 和出现在 `report file`_ 中, 有时候仅凭这些错误信息就可定位问题. 更多的时候, 还需要参考 `log files`_, 因为其中的日志包含了其它信息, 并指出实际执行失败的关键字.

当错误是由于被测应用引起的, 错误信息和日志信息应该足够能理解错误的原因. 如果不能的话, 则表示测试库没有提供 `enough information`__, 需要进一步加强. 这种情况下手动再次执行测试也许能揭示更多错误相关的信息.

由测试用例自身或其使用的关键字所引起的错误有时会难以调试. 如果错误信息比较明显, 例如, 提示关键字的参数数量不对, 则修复该问题也会很容易. 然而如果一个关键字的错误以意料不到的方式发生, 则定位问题的根源会相当困难. 首先需要检查的地方是日志文件中的 `execution errors`_. 例如, 测试库导入失败的错误可以很好的解释为什么会出现关键字缺失的问题.

如果日志文件不能提供足够的信息, 可以调低 `log level`_ 再次执行测试. 例如, 以 `DEBUG` 级别保存的回溯(tracebacks)信息可以指明错误发生的代码行, 这个信息对于定位个别库关键字问题非常宝贵.

然而日志记录的回溯没有包含Robot Framework框架本身的方法调用信息. 如果你怀疑错误是由于框架的bug引起的, 可以设置环境变量 ``ROBOT_INTERNAL_TRACES`` 为任意的非空值, 这样就可以显示内部跟踪(internal traces). 这个功能是在Robot Framework2.9.2版本后加入的.

如果日志文件信息仍然不足, 则可以启用 syslog_, 看看其中会有什么有用的信息提供. 还可以在测试用例中添加其它关键字来辅助调试下. BuiltIn_ 关键字 :name:`Log` 和 :name:`Log Variables` 是很有用的工具. 

如果所有招数都不管用, 可以考虑从 `mailing lists`_ 或其它地方寻求帮助了.

__ `Communicating with Robot Framework`_

.. Using the Python debugger (pdb)

使用Python调试器(pdb)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Python标准库中的 pdb__ 模块可以用来在测试中设置中断并交互式地进行debug. 典型的使用方法是在Python代码中想中断的地方插入:

.. sourcecode:: python

   import pdb; pdb.set_trace()

这种方法对Robot Framework不管用, 因为标准输出流在关键字执行时被重定向了. 使用下列的代码即可:

.. sourcecode:: python

   import sys, pdb; pdb.Pdb(stdout=sys.__stdout__).set_trace()

__ http://docs.python.org/2/library/pdb.html
