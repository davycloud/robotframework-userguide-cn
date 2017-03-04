.. _test data:

基础语法
================

本章介绍Robot Framework中所有测试数据的语法. 接下来的章节会依次介绍如何创建测试用例, 测试套件等.

.. contents::
   :depth: 2
   :local:

.. Files and directories

文件和目录
---------------------

测试用例的层次结构是按照如下的方式组织的:

- 测试用例在 `test case files`_ 中.
- 一个测试用例文件自动创建 `test suite`_, 该套件包含文件中所有的用例
- 包含测试用例文件的目录组成一个更高层次的测试套件. 目录测试套件 `test suite directory`_  中的用例文件是其子套件
- 目录测试套件可以包含其它的目录, 以此类推.
- 测试套件目录可以包含一个特殊的 `initialization file`_.

除了用例和测试套件, 还有几个核心概念:

- `Test libraries`_ 包含最底层的关键字.
- `Resource files`_ 包含高层次的 `user keywords`_ 和 variables_ .
- `Variable files`_ 提供更灵活的方式来创建变量.

.. _Supported file formats:
文件格式
----------------------

Robot Framework的测试数据以表格的格式定义. 实际标记表格的格式有多种, 包括: HTML格式, tab分隔的TSV格式, 纯文本格式, reStructuredText (reST)格式. 
这些具体格式的细节以及各自的优势和问题, 在下面的章节会一一介绍. 具体使用何种格式取决于实际情况, 如果没有特殊需求的推荐使用纯文本格式.

Robot Framework通过文件的扩展名来选择使用何种解析器. 扩展名不分大小写. 可以识别的扩展名包括: 

* HTML: :file:`.html`, :file:`.htm` 和 :file:`.xhtml`
* TSV: :file:`.tsv`
* 纯文本: :file:`.txt` 和特殊的 :file:`.robot`
* reStructuredText: :file:`.rst` 和 :file:`.rest` 

不同的 `test data templates`_ 可用来简化一定的工作.

.. note:: 特别的文件扩展名 :file:`.robot` 从Robot Framework 2.7.6版本后开始支持.

.. hint:: 译者注: 习惯于使用RIDE的初学者可以略过本章后面每种格式的细节介绍. 

.. _HTML format:
HTML格式
~~~~~~~~~~~

HTML文件可以在表格外有其它格式或文本存在. 这样可以在测试用例文件中添加额外的信息, 使得测试用例文件看起来就跟正式的测试规范文档一样.
使用HTML格式最大的问题是编辑起来不容易, 另一个问题是HTML文件在版本控制系统中使用不便, 因为每次变更除了测试数据的变化还会包含HTML标记的变化.

HTML文件中, 测试数据定义在各自独立的表格中. Robot Framework通过第一个单元格的文字来识别区分这些 `test data tables`_.
所有没被识别的表格最终都会被忽略.

.. table:: Using the HTML format
   :class: example

   ============  ================  =======  =======
      Setting          Value        Value    Value
   ============  ================  =======  =======
   Library       OperatingSystem
   \
   ============  ================  =======  =======

.. table::
   :class: example

   ============  ================  =======  =======
     Variable        Value          Value    Value
   ============  ================  =======  =======
   ${MESSAGE}    Hello, world!
   \
   ============  ================  =======  =======

.. table::
   :class: example

   ============  ===================  ============  =============
    Test Case           Action          Argument      Argument
   ============  ===================  ============  =============
   My Test       [Documentation]      Example test
   \             Log                  ${MESSAGE}
   \             My Keyword           /tmp
   \
   Another Test  Should Be Equal      ${MESSAGE}    Hello, world!
   ============  ===================  ============  =============

.. table::
   :class: example

   ============  ======================  ============  ==========
     Keyword            Action             Argument     Argument
   ============  ======================  ============  ==========
   My Keyword    [Arguments]             ${path}
   \             Directory Should Exist  ${path}
   ============  ======================  ============  ==========

.. _Editing test data:
编辑测试数据
'''''''''''''''''

HTML文件中测试数据的编辑可以使用任意的编辑器, 不过还是推荐使用图形编辑器, 可以直观的看到表格. RIDE_ 支持HTML文件的读写, 不过遗憾的是, 它会把其它格式和表格外的数据丢弃掉.

.. _Encoding and entity references:
编码和实体引用
''''''''''''''''''''''''''''''

HTML文件中实体引用 (例如, `&auml;`) 也是支持的. 并且支持任意的文件编码格式. 不过HTML文件中必须使用META元素来指定编码, 例如::

  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

XHTML 文件则应该使用 XML 格式的序文, 例如::

  <?xml version="1.0" encoding="Big5"?>

如果没有指定编码, Robot Framework 默认使用 ISO-8859-1.

.. _TSV format:
TSV格式
~~~~~~~~~~

TSV 文件可以通过电子表格软件(例如Excel)来编辑, 因为其语法简单, 所以可以很轻松的使用程序自动生成. 同时可以很容易地在普通文本编辑器中编辑, 并且对版本控制系统也比较友好. 不过在这些方面 `plain text format`_ 表现更好.

TSV files can be edited in spreadsheet programs and, because the syntax is
so simple, they are easy to generate programmatically. They are also pretty
easy to edit using normal text editors and they work well in version control,
but the `plain text format`_ is even better suited for these purposes.

TSV文件中, 所有的数据都存在一张大表中, 不同的 `Test data tables`_ 通过星号(`*`)
来识别, 一个或多个星号后面跟着表名, 最后还可以跟一个结束星号(后面的星号是可选的).
所有在第一个被识别的表格之前的数据, 处理方式类似HTML, 会被忽略.

.. table:: Using the TSV format
   :class: tsv-example

   ============  =======================  =============  =============
   \*Setting*    \*Value*                 \*Value*       \*Value*
   Library       OperatingSystem
   \
   \
   \*Variable*   \*Value*                 \*Value*       \*Value*
   ${MESSAGE}    Hello, world!
   \
   \
   \*Test Case*  \*Action*                \*Argument*    \*Argument*
   My Test       [Documentation]          Example test
   \             Log                      ${MESSAGE}
   \             My Keyword               /tmp
   \
   Another Test  Should Be Equal          ${MESSAGE}     Hello, world!
   \
   \
   \*Keyword*    \*Action*                \*Argument*    \*Argument*
   My Keyword    [Arguments]              ${path}
   \             Directory Should Exist   ${path}
   ============  =======================  =============  =============

.. _Editing test data:
编辑测试数据
'''''''''''''''''

可以使用多种电子表格程序来编辑TSV文件. 记得在保存文件时选择"以tab分隔"的格式, 并将文件扩展名设置为 :file:`.tsv`. 同时, 建议关闭所有的自动纠错设置, 并且将所有单元格的格式都设置为文本格式.

TSV文件相对容易编辑, 特别当编辑器支持可视的区分Tab和空格时. RIDE_ 也支持TSV格式.

Robot Framework 解析TSV格式的数据时, 首先按行分割, 然后将行按照制表符分为单元格.
电子表格程序有时会给单元格中的数据裹上引号(例如: `"my value"`), 如果数据本身含有引号, 其中会写两次(例如: `"my ""quoted"" value"`), Robot Framework可以正确的处理这种情况.
如果你是通过软件来创建这些数据, 不用操心太多, 但是如果是通过程序生成的数据, 对于引号的处理要参照软件的实现一样.

.. _Encoding:
文件编码
''''''''

TSV 文件总是按照UTF-8编码来处理, 因为ASCII编码是UTF-8的子集, 所以自然也是支持的.

.. _Plain text format:
纯文本格式
~~~~~~~~~~~~~~~~~

纯文本格式非常容易使用文本编辑器来编辑, 同时在版本控制系统中运行良好. 由于这些优势, 纯文本格式是Robot Framework中最常用的一种数据格式.

纯文本格式技术上讲和  `TSV format`_ 有些类似, 区别在于其中用来分隔单元格的方式不同.
TSV使用制表符(Tab), 而纯文本使用两个或更多的空格, 还可以使用前后都有空格的管道符(即竖线), (:codesc:`\ |\ `).

`test data tables`_ 必须在名称前有一个或者多个星号, 这一点和TSV格式类似. 表头中多余的星号和空格都会忽略, 例如, `*** Settings ***` 和 `*Settings` 效果完全一样.
同样类似于TSV格式, 第一个表格前的所有数据都会被忽略.

在解析纯文本文件时, 制表符(Tab)会自动转换成两个空格, 这样就可以像在TSV文件中那样在纯文本格式中使用Tab作为分隔. 不过需要注意的是, 多个tab在纯文本格式文件也只会被当作一个分隔符, 而在TSV格式中, 每个tab就是一个分隔符.

.. _Space separated format:
空格分隔的格式
''''''''''''''''''''''

空格的数量是不定的, 最少需要2个. 这样可以将数据对齐的更好看点. 当使用文本编辑器时, 这点相对TSV格式来说是一大优势, 因为TSV的对齐是没法控制的.

.. sourcecode:: robotframework

   *** Settings ***
   Library       OperatingSystem

   *** Variables ***
   ${MESSAGE}    Hello, world!

   *** Test Cases ***
   My Test
       [Documentation]    Example test
       Log    ${MESSAGE}
       My Keyword    /tmp

   Another Test
       Should Be Equal    ${MESSAGE}    Hello, world!

   *** Keywords ***
   My Keyword
       [Arguments]    ${path}
       Directory Should Exist    ${path}

因为空格被用作了分隔符, 所以所有空单元格必须要 escaped__ 才行. 可以使用 `${EMPTY}` 变量, 也可以使用一个反斜杠(`\\`). 
其它测试数据中 `handling whitespace`_ 没什么不同, 该转义的还是需要转义.

Because space is used as separator, all empty cells must be escaped__
with `${EMPTY}` variable or a single backslash. Otherwise
`handling whitespace`_ is not different than in other test data
because leading, trailing, and consecutive spaces must always be
escaped.

__ Escaping_

.. tip:: 关键字和参数之间推荐使用4个空格隔开.

.. _pipe separated format:

竖线加空格的分隔方式
'''''''''''''''''''''''''''''''

使用空格分隔的最大的问题是, 视觉上分隔关键字和参数有时候会比较困难. 特别是关键字中包含空格, 同时包含很多参数, 参数中也可能包含了空格.
这种情况下, 使用竖线加空格的方式来划定分界线更好, 使得单元格的边界视觉上更清晰, 容易区分.

.. sourcecode:: robotframework

   | *Setting*  |     *Value*     |
   | Library    | OperatingSystem |

   | *Variable* |     *Value*     |
   | ${MESSAGE} | Hello, world!   |

   | *Test Case*  | *Action*        | *Argument*   |
   | My Test      | [Documentation] | Example test |
   |              | Log             | ${MESSAGE}   |
   |              | My Keyword      | /tmp         |
   | Another Test | Should Be Equal | ${MESSAGE}   | Hello, world!

   | *Keyword*  |
   | My Keyword | [Arguments] | ${path}
   |            | Directory Should Exist | ${path}

一个纯文本文件中的测试数据既可以使用只有空格的分隔符, 也可以使用 空格+竖线 的分隔符, 但是一行之内只能使用其中的一种.
竖线加空格的数据行, 由必需的行首竖线开始, 行末的竖线则可有可无. 竖线的前后必须有至少一个空格(除了行首和行末的情况), 竖线不需要对齐, 不过对齐会使数据显得更清楚.

使用了竖线后就不用再转义空的单元格了(除了行末  `trailing empty cells`__). 唯一需要注意的是, 测试数据中的前后带空格的竖线必须使用反斜杠转义:

.. sourcecode:: robotframework

   | *** Test Cases *** |                 |                 |                      |
   | Escaping Pipe      | ${file count} = | Execute Command | ls -1 *.txt \| wc -l |
   |                    | Should Be Equal | ${file count}   | 42                   |

__ Escaping_

.. _Editing and encoding:
编辑和编码
''''''''''''''''''''

相对于HTML和TSV, 使用纯文本格式的最大好处是可以很容易的使用普通文本编辑器进行编辑. 很多编辑器和IDE(例如Eclipse, Emacs, Vim, and TextMate)都有相应的插件, 可以对Robot Framework的测试数据实现语法高亮显示, 有的还有更高级的特性, 比如关键字自动补全. RIDE_ 同样支持纯文本格式.

类似于TSV文件, 纯文本格式总是使用UTF-8编码.

.. _Recognized extensions:
可识别的文件扩展名
'''''''''''''''''''''

从Robot Framework 2.7.6版本开始, 可以将纯文本格式的数据文件保存为特定的  :file:`.robot` 扩展名, 而不是普通的  :file:`.txt`. 新的扩展名可以更容易的区分测试数据文件和其它的文本文件.

.. _reStructuredText format:
reStructuredText格式
~~~~~~~~~~~~~~~~~~~~~~~

reStructuredText_ (reST) 是一个易读的纯文本标记语言, 广泛用在Python项目的文档编写上(包括Python自己的官方文档, 也包括本手册). reST一般最终会编译输出为HTML格式, 同时还支持其它多种格式.

在Robot Framework中使用reST可以让你在一个简明的文本格式文件中, 混合使用丰富格式的文档和测试数据. 因为也是文本格式, 所以使用文本编辑器, 对比工具, 和版本控制都很方便.
实践中, reST结合了很多纯文本和HTML格式的优势.

Using reST with Robot Framework allows you to mix richly formatted documents
and test data in a concise text format that is easy to work with
using simple text editors, diff tools, and source control systems.
In practice it combines many of the benefits of plain text and HTML formats.

当使用reST文件时, 可以有两种方式来定义测试数据. 要么使用 `code blocks`__ 的方式, 其中定义测试用例仍然采用 `plain text format`_ 中的格式. 要么使用和在 `HTML format`_ 中一样使用 tables__ 格式. 

When using reST files with Robot Framework, there are two ways to define the
test data. Either you can use `code blocks`__ and define test cases in them
using the `plain text format`_ or alternatively you can use tables__ exactly
like you would with the `HTML format`_.

.. note:: 在Robot Framework中使用reST文件需要安装Python的 docutils_ 模块.

__ `Using code blocks`_
__ `Using tables`_

.. _Using code blocks:
代码块
'''''''''''''''''

reStructuredText文档以一种称之为代码块的方式来表示一段代码示例. 当reST文档转换成HTML或其它格式时, 代码块中的内容使用 Pygments_ 进行语法高亮.
标准reST语法中, 代码块使用 `code` 指令(directive), 但是 Sphinx_ 使用 `code-block` 或 `sourcecode`. 其中代码的语言名作为参数提供给指令. 例如, 下面的代码块分别包含Python和Robot Framework的例子:

.. sourcecode:: rest

    .. code:: python

       def example_keyword():
           print 'Hello, world!'

    .. code:: robotframework

       *** Test Cases ***
       Example Test
           Example Keyword

当Robot Framework解析reST文件时, 首先开始查找可能包含了测试数据的 `code`, `code-block` 或 `sourcecode` 代码块. 如果找到了, 其中的数据会被写入内存中的文件并执行. 代码块之外的其它数据被忽略.

代码块内的测试数据内容必须使用 `plain text format`_ 定义的.

如下例所示, 空格分隔和竖线分隔都是支持的.

.. sourcecode:: rest

    Example
    -------

    This text is outside code blocks and thus ignored.

    .. code:: robotframework

       *** Settings ***
       Library       OperatingSystem

       *** Variables ***
       ${MESSAGE}    Hello, world!

       *** Test Cases ***
       My Test
           [Documentation]    Example test
           Log    ${MESSAGE}
           My Keyword    /tmp

       Another Test
           Should Be Equal    ${MESSAGE}    Hello, world!

    这段话在代码块之外, 所以会被忽略. 上面使用了空格分隔的格式.
    下面的代码块使用的是竖线分隔的格式.

    .. code:: robotframework

       | *** Keyword ***  |                        |         |
       | My Keyword       | [Arguments]            | ${path} |
       |                  | Directory Should Exist | ${path} |

.. note:: Escaping_ using the backslash character works normally in this format.
          No double escaping is needed like when using reST tables.

.. note:: 使用代码块的方式是Robot Framework 2.8.2才出现的新特性.

.. _Using tables:
使用表格
''''''''''''

如果reStructuredText文档没有包含Robot Framework数据的代码块, 则和 `HTML format`_ 一样检查是否有包含数据的表格. 这种情况下, Robot Framework在内存中将文档转换为HTML, 然后按照正常HTML文件的方式继续解析.

Robot Framework靠第一个单元格的内容来标示 `test data tables`_, 被识别的表格外的其它内容都被忽略. 
下面的例子展示了4种测试数据表格, 既使用了简单的表格语法, 也使用了网格(grid)表格语法:

.. sourcecode:: rest

    Example
    -------

    This text is outside tables and thus ignored.

    ============  ================  =======  =======
      Setting          Value         Value    Value
    ============  ================  =======  =======
    Library       OperatingSystem
    ============  ================  =======  =======


    ============  ================  =======  =======
      Variable         Value         Value    Value
    ============  ================  =======  =======
    ${MESSAGE}    Hello, world!
    ============  ================  =======  =======


    =============  ==================  ============  =============
      Test Case          Action          Argument      Argument
    =============  ==================  ============  =============
    My Test        [Documentation]     Example test
    \              Log                 ${MESSAGE}
    \              My Keyword          /tmp
    \
    Another Test   Should Be Equal     ${MESSAGE}    Hello, world!
    =============  ==================  ============  =============

    Also this text is outside tables and ignored. Above tables are created
    using the simple table syntax and the table below uses the grid table
    approach.

    +-------------+------------------------+------------+------------+
    |   Keyword   |         Action         |  Argument  |  Argument  |
    +-------------+------------------------+------------+------------+
    | My Keyword  | [Arguments]            | ${path}    |            |
    +-------------+------------------------+------------+------------+
    |             | Directory Should Exist | ${path}    |            |
    +-------------+------------------------+------------+------------+

.. note:: 使用简单表格时, 第一列的空单元格需要转义. 例子中用的是反斜杠 :codesc:`\\`, 
          还可以使用 `..`.

.. note:: 因为反斜杠在reST中是转义字符, 如果要指定并显示一个反斜杠自身, 
          需要额外再加一个反斜杠. 例如, 换行字符必须写作 `\\n`. 因为反斜杠在Robot Framework的数据中也被用作转义符, 所以要在测试数据中表示一个字面上的反斜杠需要二次转义, 如 `c:\\\\temp`.


.. note:: Empty cells in the first column of simple tables need to be escaped.
          The above example uses :codesc:`\\` but `..` could also be used.

.. note:: Because the backslash character is an escape character in reST,
          specifying a backslash so that Robot Framework will see it requires
          escaping it with an other backslash like `\\`. For example,
          a new line character must be written like `\\n`. Because
          the backslash is used for escaping_ also in Robot Framework data,
          specifying a literal backslash when using reST tables requires double
          escaping like `c:\\\\temp`.

每次运行时都要把reST文件转换为HTML文件, 显然这会带来额外的损耗. 如果想规避这个问题, 最好是使用其它外部工具先将reST文件转换为HTML, 让Robot Framework使用生成后的文件.

.. _Editing and encoding:
编辑和编码
''''''''''''''''''''

reStructuredText文件可以使用任何文本编辑器, 很多编辑器和IDE都提供了语法高亮功能. 但是, RIDE_ 并不支持reST格式.

如果reST文件中包含non-ASCII字符, 则文件需要保存为UTF-8编码格式.

.. _Syntax errors in reST source files:
reST源文件中的语法错误
''''''''''''''''''''''''''''''''''

如果一个reStructuredText文档中的语法有错误(比如表格格式不正确), 则解析会失败, 其中的用例也不会执行. 当执行单个reST文件时, Robot Framework会在控制台显示错误信息. 但是在执行一个目录时, 这种解析错误一般会被忽略.

从Robot Framework2.9.2版本开始, 当运行测试时, 低于 `SEVERE` 级别的错误会被忽略, 这样做是为了避免恼人的非标准指令或标记引起的错误. 这有可能隐藏真正的错误, 但是正常处理这些文件时还是可以发现的(译注: 这里说的处理应该是指用其它工具解析reST文件).

.. _Test data tables:
测试数据表格
----------------

测试数据按结构划分有4种类型, 如下表所列. 这些测试数据表格由表格中第一个单元格标示. 4种表格的名称分别是 `Settings`, `Variables`, `Test Cases`, 和 `Keywords`. 匹配时不区分大小写, 同时单数形式如 `Setting` 和 `Test Case` 也可接受.

.. table:: Different test data tables
   :class: tabular

   +--------------+--------------------------------------------+
   |    Table     |                 Used for                   |
   +==============+============================================+
   | Settings     | | 1) Importing `test libraries`_,          |
   |              |   `resource files`_ and `variable files`_. |
   |              | | 2) Defining metadata for `test suites`_  |
   |              |   and `test cases`_.                       |
   +--------------+--------------------------------------------+
   | Variables    | Defining variables_ that can be used       |
   |              | elsewhere in the test data.                |
   +--------------+--------------------------------------------+
   | Test Cases   | `Creating test cases`_ from available      |
   |              | keywords.                                  |
   +--------------+--------------------------------------------+
   | Keywords     | `Creating user keywords`_ from existing    |
   |              | lower-level keywords                       |
   +--------------+--------------------------------------------+

.. _Rules for parsing the data:

数据的解析规则
--------------------------

.. _comment:

.. _Ignored data:

被忽略的数据
~~~~~~~~~~~~

当Robot Framework解析测试数据时, 以下数据都会被忽略:

- 所有在第一格中没有适配 `可识别的表格名称`__ 的表格
- 第一行中除了第一格之外的其它任何数据
- 第一个表格之前的所有数据, 如果所使用的文档格式允许表格之间存在数据, 这些也会被忽略
- 所有空行, 这意味着可以使用空行提供表格的可读性
- 行末的空单元格, 除非已经 被转义__.
- 所有的单个反斜杠(:codesc:`\\`), 如果其不是用于转义的话
- 如果井字符(`#`)是一个单元格的第一个字符, 则所有跟在后面的字符都被忽略, 也就是说可以使用 `#` 在测试数据中添加注释. 
- HTML/reST格式中其它的标记和格式

如果数据被Robot Framework忽略, 则这些数据不会出现在任何后续的报告中, 并且大部分使用Robot Framework的其它工具也会忽略它们. 
如果要添加在输出中可见的信息, 请将它们放在文档中, 或测试用例和套件的其它元数据(metadata)中, 或使用内置的关键字 :name:`Log` 和 :name:`Comment` 记入日志.

When Robot Framework ignores some data, this data is not available in
any resulting reports and, additionally, most tools used with Robot
Framework also ignore them. To add information that is visible in
Robot Framework outputs, place it to the documentation or other metadata of
test cases or suites, or log it with the BuiltIn_ keywords :name:`Log` or
:name:`Comment`.

__ `Test data tables`_
__ `Prevent ignoring empty cells`_

.. _Handling whitespace:
如何处理空格
~~~~~~~~~~~~~~~~~~~

Robot Framework处理空格的方式和HTML源代码处理空格的方式一样:

- 换行, 回车, 以及制表符, 都转换为空格.
- 单元格领头的空格和末尾的空格都会被忽略
- 多个连续的空格被压缩为一个空格

此外, 非中断空格(non-breaking spaces)被替换为普通空格, 以避免引发难以定位的错误.

如果需要领头的空格, 或末尾的空格, 或连续的空格, 它们都 `必须被转义`__. 
换行, 回车, 制表符, 和非中断空格则可以分别使用 `escape sequences`_ `\n`, `\r`, `\t`, and `\xA0`.

- Newlines, carriage returns, and tabs are converted to spaces.
- Leading and trailing whitespace in all cells is ignored.
- Multiple consecutive spaces are collapsed into a single space.

In addition to that, non-breaking spaces are replaced with normal spaces.
This is done to avoid hard-to-debug errors
when a non-breaking space is accidentally used instead of a normal space.

If leading, trailing, or consecutive spaces are needed, they `must be
escaped`__. Newlines, carriage returns, tabs, and non-breaking spaces can be
created using `escape sequences`_ `\n`, `\r`, `\t`, and `\xA0` respectively.

__ `Prevent ignoring spaces`_

.. _Escaping:
字符转义
~~~~~~~~

Robot Framework的测试数据使用反斜杠(:codesc:`\\`)作为转义字符, 此外还增加了 `built-in variables`_ `${EMPTY}` 和 `${SPACE}` 经常用来作为转义. 不同的转义策略在下面的小节中详细讨论.

.. _Escaping special characters:
转义特殊字符
'''''''''''''''''''''''''''

反斜杠可以用来转义特殊字符, 这样我们就能使用它们的字面值了.

.. table:: Escaping special characters
   :class: tabular

   ===========  ================================================================  ==============================
    Character                              Meaning                                           Examples
   ===========  ================================================================  ==============================
   `\$`         Dollar sign, never starts a `scalar variable`_.                   `\${notvar}`
   `\@`         At sign, never starts a `list variable`_.                         `\@{notvar}`
   `\%`         Percent sign, never starts an `environment variable`_.            `\%{notvar}`
   `\#`         Hash sign, never starts a comment_.                               `\# not comment`
   `\=`         Equal sign, never part of `named argument syntax`_.               `not\=named`
   `\|`         Pipe character, not a separator in the `pipe separated format`_.  `| Run | ps \| grep xxx |`
   `\\`         Backslash character, never escapes anything.                      `c:\\temp, \\${var}`
   ===========  ================================================================  ==============================

.. _escape sequence:
.. _escape sequences:

.. _Forming escape sequences:
转义序列
''''''''''''''''''''''''

反斜杠还可以用来创建特殊的转义序列, 这些转义序列所代表的意义很难, 或者不可能, 在测试数据中通过普通字符表示.

The backslash character also allows creating special escape sequences that are
recognized as characters that would otherwise be hard or impossible to create
in the test data.

.. table:: Escape sequences
   :class: tabular

   =============  ====================================  ============================
      Sequence                  Meaning                           Examples
   =============  ====================================  ============================
   `\n`           Newline character.                    `first line\n2nd line`
   `\r`           Carriage return character             `text\rmore text`
   `\t`           Tab character.                        `text\tmore text`
   `\xhh`         Character with hex value `hh`.        `null byte: \x00, ä: \xE4`
   `\uhhhh`       Character with hex value `hhhh`.      `snowman: \u2603`
   `\Uhhhhhhhh`   Character with hex value `hhhhhhhh`.  `love hotel: \U0001f3e9`
   =============  ====================================  ============================

.. note:: 在测试数据中的所有字符串, 包括 `\x02` 这种字符, 都是Unicode. 
          如果有需要的话, 必须明确地转换. 转换的方法可以使用 BuiltIn_ 关键字
          :name:`Convert To Bytes` 或 String_ 库中的 :name:`Encode String To Bytes`. 或者在Python代码中使用类似于 `str(value)` 和 `value.encode('UTF-8')` 的方法.

.. note:: 如果在 `\x` `\u` 或 `\U` 转义符后面跟了非法的十六进制数, 
          则解析的结果是保留原始的字符, 反斜杠除外. 例如, `\xAX` (X不是十六进制), `\U00110000` (值太大了) 解析的结果分别是 `xAX` 和 `U00110000`.
          不过, 这种情况可能会在将来有所改变.

.. note:: 如果需要用与操作系统无关的换行符(Windows中是 `\r\n`, 其它系统是`\n`),
          可以使用 `Built-in variable`_ `${\n}`.

.. note:: 跟在 `\n` 后面的未经转义的空格会被忽略. 也就是说, `two lines\nhere` 和
          `two lines\n here` 是等价的. 这样做的动机是在使用HTML格式时, 能包裹包含换行符的长行, 不过同样的逻辑对其它格式也一样. 该规则的一个特殊情况是在 `extended variable syntax`_ 中的空白符不会忽略.

.. note:: `\x`, `\u` and `\U` 转义序列在Robot Framework 2.8.2版本新引入.

.. _Prevent ignoring empty cells:
避免忽略空单元格
''''''''''''''''''''''''''''

如果需要使用空值, 例如作为关键字的参数, 必须明确地转义以避免被框架 忽略__. 不过使用哪种数据格式, 空的收尾单元格必须被转义. 当使用 `space separated format`_ 时, 所有的空值都必须被转义.

If empty values are needed as arguments for keywords or otherwise, they often
need to be escaped to prevent them from being ignored__. Empty trailing cells
must be escaped regardless of the test data format, and when using the
`space separated format`_ all empty values must be escaped.

空的单元格既可以使用反斜杠转义, 也可以使用 `built-in variable`_ `${EMPTY}`. 特别推荐使用后者, 因为更清楚易懂. 一个特殊情况是在 `space separated format`_ 中使用 `for loops`_ 时, 缩进的单元格中应使用反斜杠.
所有这些情况都在下面的例子中进行了说明, 先是HTML格式, 然后是空格分隔的纯文本格式:

.. table::
   :class: example

   ==================  ============  ==========  ==========  ================================
        Test Case         Action      Argument    Argument                Argument
   ==================  ============  ==========  ==========  ================================
   Using backslash     Do Something  first arg   \\
   Using ${EMPTY}      Do Something  first arg   ${EMPTY}
   Non-trailing empty  Do Something              second arg  # No escaping needed in HTML
   For loop            :FOR          ${var}      IN          @{VALUES}
   \                                 Log         ${var}      # No escaping needed here either
   ==================  ============  ==========  ==========  ================================

.. sourcecode:: robotframework

   *** Test Cases ***
   Using backslash
       Do Something    first arg    \
   Using ${EMPTY}
       Do Something    first arg    ${EMPTY}
   Non-trailing empty
       Do Something    ${EMPTY}     second arg    # Escaping needed in space separated format
   For loop
       :FOR    ${var}    IN    @{VALUES}
       \    Log    ${var}                         # Escaping needed here too

__ `Ignored data`_

.. _Prevent ignoring spaces:
避免忽略空格
'''''''''''''''''''''''

因为领头的, 收尾的, 以及连续的空格在单元格中都是被 忽略的__, 如果有需要的话, 例如作为关键字的参数, 必须被转义. 和避免忽略空单元格类似, 既可以使用反斜杠, 也可以使用  `built-in variable`_ `${SPACE}`.

.. table:: Escaping spaces examples
   :class: tabular

   ==================================  ==================================  ==================================
        Escaping with backslash             Escaping with `${SPACE}`                      Notes
   ==================================  ==================================  ==================================
   :codesc:`\\ leading space`          `${SPACE}leading space`
   :codesc:`trailing space \\`         `trailing space${SPACE}`            Backslash must be after the space.
   :codesc:`\\ \\`                     `${SPACE}`                          Backslash needed on both sides.
   :codesc:`consecutive \\ \\ spaces`  `consecutive${SPACE * 3}spaces`     Using `extended variable syntax`_.
   ==================================  ==================================  ==================================

如上例所示, 使用 `${SPACE}` 变量是测试数据更容易理解. 当需要不止一个空格时, 结合 `extended variable syntax`_ 使用时, 显得尤其方便.

__ `Handling whitespace`_

.. _Dividing test data to several rows:
测试数据分为多行
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果数据太多不方便放在一行, 可以另起一行, 下面一行的开头使用省略号(`...`)来表示继续.
在测试用例和关键字表格中, 省略号的前面必须至少有一个空的单元格(因为第一列只有用例名称). 
在设置和变量表格中, 可以直接放在设置或变量名的下方. 
在所有类型的表格中, 省略号前面的空单元格都会被忽略.

此外, 某些设置只接受一个值(主要是文档), 这个值也可以分开写在多列. 当解析完毕, 最终将用空格将多列的值拼接起来. 从Robot Framework 2.7版本开始, 这些值如果分为多行, 将 `使用换行符将多行拼接起来`__.

上面讨论的语法都通过下面的例子来解释说明.
前3个表格中的数据没有分割, 接下来的3个表格展示了如何将数据分割为多行以占用更少的列数.

__ `Newlines in test data`_

.. table:: 没有进行分割的数据
   :class: example

   ============  =======  =======  =======  =======  =======  =======
     Setting      Value    Value    Value    Value    Value    Value
   ============  =======  =======  =======  =======  =======  =======
   Default Tags  tag-1    tag-2    tag-3    tag-4    tag-5    tag-6
   ============  =======  =======  =======  =======  =======  =======

.. table::
   :class: example

   ==========  =======  =======  =======  =======  =======  =======
    Variable    Value    Value    Value    Value    Value    Value
   ==========  =======  =======  =======  =======  =======  =======
   @{LIST}     this     list     has      quite    many     items
   ==========  =======  =======  =======  =======  =======  =======

.. table::
   :class: example

   +-----------+-----------------+---------------+------+-------+------+------+-----+-----+
   | Test Case |     Action      |   Argument    | Arg  |  Arg  | Arg  | Arg  | Arg | Arg |
   +===========+=================+===============+======+=======+======+======+=====+=====+
   | Example   | [Documentation] | Documentation |      |       |      |      |     |     |
   |           |                 | for this test |      |       |      |      |     |     |
   |           |                 | case.\\n This |      |       |      |      |     |     |
   |           |                 | can get quite |      |       |      |      |     |     |
   |           |                 | long...       |      |       |      |      |     |     |
   +-----------+-----------------+---------------+------+-------+------+------+-----+-----+
   |           | [Tags]          | t-1           | t-2  | t-3   | t-4  | t-5  |     |     |
   +-----------+-----------------+---------------+------+-------+------+------+-----+-----+
   |           | Do X            | one           | two  | three | four | five | six |     |
   +-----------+-----------------+---------------+------+-------+------+------+-----+-----+
   |           | ${var} =        | Get X         | 1    | 2     | 3    | 4    | 5   | 6   |
   +-----------+-----------------+---------------+------+-------+------+------+-----+-----+

.. table:: 分割为多行的测试数据
   :class: example

   ============  =======  =======  =======
     Setting      Value    Value    Value
   ============  =======  =======  =======
   Default Tags  tag-1    tag-2    tag-3
   ...           tag-4    tag-5    tag-6
   ============  =======  =======  =======

.. table::
   :class: example

   ==========  =======  =======  =======
    Variable    Value    Value    Value
   ==========  =======  =======  =======
   @{LIST}     this     list     has
   ...         quite    many     items
   ==========  =======  =======  =======

.. table::
   :class: example

   ===========  ================  ==============  ==========  ==========
    Test Case       Action           Argument      Argument    Argument
   ===========  ================  ==============  ==========  ==========
   Example      [Documentation]   Documentation   for this    test case.
   \            ...               This can get    quite       long...
   \            [Tags]            t-1             t-2         t-3
   \            ...               t-4             t-5
   \            Do X              one             two         three
   \            ...               four            five        six
   \            ${var} =          Get X           1           2
   \                              ...             3           4
   \                              ...             5           6
   ===========  ================  ==============  ==========  ==========
