.. _variable:

变量
=========

.. contents::
   :depth: 2
   :local:

.. Introduction

概述
------------

变量是Robot Framework的一个内部特性, 在大多数的测试数据中都可以使用.

大部分情况下, 变量是用在关键字的参数, 不过所有的设置项也都支持变量表示设置值. 一般情况下关键字的名称是不能使用变量替代的, 不过, 可以通过 BuiltIn_ 关键字 :name:`Run Keyword` 来实现这个效果.

Robot Framework 的变量可以是 scalars_, lists_ 或 `dictionaries`__ , 分别使用语法格式 `${SCALAR}`, `@{LIST}` and `&{DICT}`来定义. 此外, `环境变量`_ 可以直接使用语法 `%{ENV_VAR}` 来获取.

变量在下面的情况中很有用:

- 当测试数据中的字符串经常变化时. 使用变量的话就只需要在一个地方修改.

- 当创建系统无关和平台无关的测试数据时. 使用变量替代硬编码的字符串可以非常方便(例如, 
  `${RESOURCES}`  替代 `c:\\resources`, `${HOST}` 替代 `10.0.0.1:8080`). 因为变量值可以在测试执行时 `通过命令行选项设置`_, 所以修改系统相关的变量非常容易(例如, `--variable HOST:10.0.0.2:1234 --variable RESOURCES:/opt/resources`).

- 当需要传递对象而不是字符串参数给关键字时. 这种情况下只能使用变量实现.

- 当不同的关键字(这些关键字可能来自不同的库)之间需要通信时.
  可以将一个关键字的返回值先赋值给一个变量, 然后再作为参数传递给另外一个关键字

- 当某个值太长或者太复杂时. 例如, 使用 `${URL}` 比直接使用
  `http://long.domain.name:8080/path/to/service?foo=1&bar=2&zap=42` 简短很多.

如果某个变量不存在, 则使用该变量的关键字会失败. 如果要在字面字符串中表示变量的语法格式, 则必须使用反斜杠进行转义, 例如 `\\${NAME}`.

__ `Scalar variables`_
__ `List variables`_
__ `Dictionary variables`_
__ `Setting variables in command line`_
__ Escaping_

.. Variable types

变量的类型
--------------

本节介绍不同的变量类型. 关于变量如何创建在下一节讨论.

Robot Framework 变量, 与关键字类似, 是不区分大小写的, 同时其中的下划线和空格也会被忽略.
推荐使用大写字母来表示全局变量(如 `${PATH}` 或 `${TWO WORDS}`), 小写字母来表示局部变量(如 `${my var}` 或 `${myVar}`). 谨记, 关于大小写的使用风格要保持一致.

变量名称的构成包括: 一个表示变量类型的标识符(`$`, `@`, `&`, `%`), 一对花括号(`{`, `}`), 以及包含在花括号中的变量名.

不同于某些编程语言中变量的语法, 这里的花括号是不能省略的. 花括号内的变量名可以是任何字符, 但是推荐仅使用26个字母, 阿拉伯数字, 下划线和空格的组合. 同时, 如果使用了 `extended variable syntax`_ 还有更多的要求.

.. _scalar variable:

.. Scalar variables

标量
~~~~~~~~~~~~~~~~

当使用标量变量时, 变量被其赋值所替代. 最常用的标量赋值是字符串, 实际上标量可以是任何对象, 包括列表,字典等. 标量的语法格式对于大部分用户来说应该很熟悉, 这种格式在其它编程语言, 如shell脚本和Perl语言中, 都有使用.

下面的例子介绍了如何使用标量变量. 假设变量 `${GREET}` 和 `${NAME}` 在当前作用域内可用, 且分别被赋值为 `Hello` 和 `world`. 例子中两个测试用例是等价的.

.. sourcecode:: robotframework

   *** Test Cases ***
   Constants
       Log    Hello
       Log    Hello, world!!

   Variables
       Log    ${GREET}
       Log    ${GREET}, ${NAME}!!

当一个标量变量在测试数据中独占一个单元格, 该变量被其赋值完全替代, 这个值可以是任何的对象.

如果单元格内还有其它内容(例如字符串或其它变量), 则变量的值会首先转换为Unicode字符串, 然后再和单元格内的其它内容拼接起来. 将对象转为字符串也就意味着要调用Python对象的 `__unicode__` 方法(如果没有则调用 `__str__`), 或者Java对象的 `toString` 方法.

.. note:: 变量作为关键字的参数使用 `named arguments`_ 语法时, 例如, 
          `argname=${var}`, 这时变量值会原样传递, 不做字符串转换.

下面的例子展示了当一个变量独占单元格和非独占时两者之间的区别. 首先, 假定变量 `${STR}` 赋值为字符串 `Hello, world!`, `${OBJ}` 赋值为下面Java对象实例:

.. sourcecode:: java

 public class MyObj {

     public String toString() {
         return "Hi, tellus!";
     }
 }

以下是测试用例:

.. sourcecode:: robotframework

   *** Test Cases ***
   Objects
       KW 1    ${STR}
       KW 2    ${OBJ}
       KW 3    I said "${STR}"
       KW 4    You said "${OBJ}"

当这个用例执行时, 不同的关键字接收到的参数解释如下:

- :name:`KW 1` 接收到字符串 `Hello, world!`
- :name:`KW 2` 接收到MyObj的对象实例 `${OBJ}`
- :name:`KW 3` 接收到字符串 `I said "Hello, world!"`
- :name:`KW 4` 接收到字符串 `You said "Hi, tellus!"`

.. note:: 如果变量不能表示为Unicode, 则这种转换显然会失败. 当发生这种情况时,
          例如, 用变量表示字节序列, 如果想要拼接在一起 `${byte1}${byte2}` 传给关键字.
          这时的变通方案是创建一个包含所有值的变量(如 `${bytes}`)然后独占一个单元格, 这样避免发生转换.


.. _list variable:

.. List variables

列表
~~~~~~~~~~~~~~

当变量作为标量使用, 如 `${EXAMPLE}`, 变量值按原样使用. 如果这个变量的值是一个列表, 或者类似列表的其它序列, 还可以将该变量作为列表变量使用, 格式为 `@{EXAMPLE}`. 这种情况下, 列表中的元素会各自作为参数传递. 

通过一个例子来解释会比较容易理解. 假设有一个变量 `@{USER}` 值是 `['robot', 'secret']`, 下例中两个测试用例是等价的:

.. sourcecode:: robotframework

   *** Test Cases ***
   Constants
       Login    robot    secret

   List Variable
       Login    @{USER}

Robot Framework 将变量存储在一个内部结构中, 同时允许按照标量, 列表或字典的类型来使用. 按照列表来使用要求该值是一个Python列表或者类似列表的对象.

Robot Framework不允许字符串作为字符列表使用, 但是其它的序列对象如元组或字典是可以的.

Robot Framework 2.9版本之前, 标量和列表变量是分开存储的, 但是两者可以互换使用, 即列表变量作为标量使用, 标量变量作为列表使用. 当一个标量和列表变量同名但是不同值时, 这将引起很多混乱.

.. Using list variables with other data

列表变量和其它数据混用
''''''''''''''''''''''''''''''''''''

列表变量可以和其它参数混用, 其中可能还包含其它的列表参数.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Keyword    @{LIST}    more    args
       Keyword    ${SCALAR}    @{LIST}    constant
       Keyword    @{LIST}    @{ANOTHER}    @{ONE MORE}

如果一个列表变量在单元格内和其它内容(字符串或其它变量)混用, 则最终的值会是该变量的字符串表示, 就跟标量变量的处理方式一样.

.. Accessing individual list items

获取列表项
'''''''''''''''''''''''''''''''

使用下标语法 `@{NAME}[index]` 可以获取到列表变量中指定项的值, 其中 `index` 是要获取的项的下标. 下标从0开始, 负数的下标等同于从列表末尾向前数. 下标长度超过列表范围会导致错误. 下标值自动转换为整数, 同样支持变量表示. 获取到的列表项基本等同于一个标量变量.


.. sourcecode:: robotframework

   *** Test Cases ***
   List Variable Item
       Login    @{USER}[0]    @{USER}[1]
       Title Should Be    Welcome @{USER}[0]!

   Negative Index
       Log    @{LIST}[-1]

   Index As Variable
       Log    @{LIST}[${INDEX}]

.. Using list variables with settings

在Setting中使用列表变量
''''''''''''''''''''''''''''''''''

列表变量可以在某些 settings__ 中使用.

列表变量可以用在库和变量文件导入时的参数, 不过库和变量文件自身的名称不能是列表变量. Setup和Teardown中的关键字的参数也可以使用列表变量, 但是关键字名称不可以. 不过这些名称都支持使用标量型变量. 标签相关的设置可以自由使用列表变量.

.. sourcecode:: robotframework

   *** Settings ***
   Library         ExampleLibrary      @{LIB ARGS}    # This works
   Library         ${LIBRARY}          @{LIB ARGS}    # This works
   Library         @{NAME AND ARGS}                   # This does not work
   Suite Setup     Some Keyword        @{KW ARGS}     # This works
   Suite Setup     ${KEYWORD}          @{KW ARGS}     # This works
   Suite Setup     @{KEYWORD}                         # This does not work
   Default Tags    @{TAGS}                            # This works

__ `All available settings in test data`_

.. _dictionary variable:

.. Dictionary variables

字典变量
~~~~~~~~~~~~~~~~~~~~

如上所述, 包含列表的变量可以作为 `列表变量`_, 将其中的项分别传递给关键字. 类似的, 一个变量包含Python的字典, 或者类似字典的对象, 可以当作字典变量使用, 如 `&{EXAMPLE}`.

在实践中, 这意味着字典中的项可以作为 `named arguments`_ 传给关键字. 假设有个字典变量 `&{USER}` 中有值 `{'name': 'robot', 'password': 'secret'}`, 则下面两个用例的效果是等价的.

.. sourcecode:: robotframework

   *** Test Cases ***
   Constants
       Login    name=robot    password=secret

   Dict Variable
       Login    &{USER}

字典型变量是 Robot Framework 2.9 新增的特性.

.. Using dictionary variables with other data

字典变量和其它数据混用
''''''''''''''''''''''''''''''''''''''''''

字典变量可以和其它变量一起使用, 包括其它字典变量. 因为 `named argument syntax`_  要求位置参数必须在命名参数之前, 所以字典变量后面只能跟命名参数或者其它的字典.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Keyword    &{DICT}    named=arg
       Keyword    positional    @{LIST}    &{DICT}
       Keyword    &{DICT}    &{ANOTHER}    &{ONE MORE}

如果一个字典变量在单元格内和其它内容(字符串或其它变量)混用,  最终的值会是变量的字符串表示, 就跟把变量当作标量变量的处理结果一样.

.. Accessing individual dictionary items

获取字典中的项
'''''''''''''''''''''''''''''''''''''

可以通过 `&{NAME}[key]` 这样的语法格式获取字典中某项的值, 其中 `key` 是键的名称. 
键名当作字符串处理, 非字符串的键可以用变量代替. 通过这种方式获取到的值可作为标量变量使用.

如果键是字符串, 还可以使用另一种语法格式 `${NAME.key}`. 更多细节说明请参考 `Creating dictionary variables`_

.. sourcecode:: robotframework

   *** Test Cases ***
   Dict Variable Item
       Login    &{USER}[name]    &{USER}[password]
       Title Should Be    Welcome &{USER}[name]!

   Key As Variable
       Log Many    &{DICT}[${KEY}]    &{DICT}[${42}]

   Attribute Access
       Login    ${USER.name}    ${USER.password}
       Title Should Be    Welcome ${USER.name}!

.. Using dictionary variables with settings

在Setting中使用字典变量
''''''''''''''''''''''''''''''''''''''''

字典变量除了在import, setup, teardown中充当关键字的参数使用, 不能在其它设置项中使用.

.. sourcecode:: robotframework

   *** Settings ***
   Library        ExampleLibrary    &{LIB ARGS}
   Suite Setup    Some Keyword      &{KW ARGS}     named=arg

.. _environment variable:

.. Environment variables

环境变量
~~~~~~~~~~~~~~~~~~~~~

Robot Framework使用 `%{ENV_VAR_NAME}` 这种语法格式来使用环境变量. 环境变量的值只能是字符串.

Robot Framework allows using environment variables in the test
data using the syntax `%{ENV_VAR_NAME}`. They are limited to string
values.

在测试执行前已设置的操作系统环境变量在执行过程中都是可用的, 同时还可以使用关键字 :name:`Set Environment Variable` 创建新的环境变量, 或者 :name:`Delete Environment Variable` 删除某个环境变量, 这两个关键字都是来自于 OperatingSystem_ 库. 因为环境变量是全局的, 所以在一个测试用例中设置的环境变量可以在后续执行的另一个测试用例中使用. 不过, 测试执行中改变的环境变量在测试执行完成后即恢复原状, 即不会真正改变系统的环境变量.

.. sourcecode:: robotframework

   *** Test Cases ***
   Env Variables
       Log    Current user: %{USER}
       Run    %{JAVA_HOME}${/}javac

.. Java system properties

Java系统属性
~~~~~~~~~~~~~~~~~~~~~~

当使用Jython运行测试时, 可以使用 `环境变量`_ 的语法来获取 `Java系统属性`__. 如果一个环境变量的名称和一个系统属性重名, 则最终返回的是环境变量的值.

.. sourcecode:: robotframework

   *** Test Cases ***
   System Properties
       Log    %{user.name} running tests on %{os.name}

__ http://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html

.. Creating variables

创建变量
------------------

测试中可用的变量来源于各种不同的地方.

.. Variable table

变量表格
~~~~~~~~~~~~~~

变量最常见的源头就是在 `test case files`_ 和 `resource files`_ 中的变量表格. 变量表格让变量和其它测试数据都创建在同一个地方, 而且语法也很简单, 因此使用起来非常方便. 不足之处在于这里变量的值只能是字符串, 并且不能动态创建. 要规避这些不足之处, 可以使用 `variable files`_.

.. Creating scalar variables

创建标量
'''''''''''''''''''''''''

最简单的变量赋值操作就是将字符串赋值给一个标量变量. 

在变量表格中的第一列指定变量名称(包括 `${}`), 在第二列放上变量的值. 如果第二列为空, 则表示变量的值是空字符串. 值同时也可以是其它已经定义的变量.

.. sourcecode:: robotframework

   *** Variables ***
   ${NAME}         Robot Framework
   ${VERSION}      2.0
   ${ROBOT}        ${NAME} ${VERSION}

如果想要更明确的标示赋值操作, 可以在变量名称后面加上一个赋值等号 `=`, 这不是必需的.

.. sourcecode:: robotframework

   *** Variables ***
   ${NAME} =       Robot Framework
   ${VERSION} =    2.0

如果一个标量变量的值很长, 可以分割到多列甚至 多行__. 默认情况下, 各个单元格中的值最终会使用空格拼接起来, 不过可以在第一格中使用 `SEPARATOR=<sep>` 来指定连接符.

.. sourcecode:: robotframework

   *** Variables ***
   ${EXAMPLE}      This value is joined    together with a space
   ${MULTILINE}    SEPARATOR=\n    First line
   ...             Second line     Third line

上面的这种拼接方式是Robot Framework 2.9版本的新特性. 在2.8版本中, 这种格式会引发一个语法错误. 而在更早的版本中, 这会创建一个列表值.

__ `Dividing test data to several rows`_

.. Creating list variables

创建列表
'''''''''''''''''''''''

创建列表变量同样很简单. 变量名同样位于变量表格的第一列, 值位于后续的列. 一个列表变量可以有任意多的值, 包括0个值. 如果值比较多, 同样可以 `分为多行`__.

__ `Dividing test data to several rows`_

.. sourcecode:: robotframework

   *** Variables ***
   @{NAMES}        Matti       Teppo
   @{NAMES2}       @{NAMES}    Seppo
   @{NOTHING}
   @{MANY}         one         two      three      four
   ...             five        six      seven

.. Creating dictionary variables

创建字典
'''''''''''''''''''''''''''''

字典变量的创建方式类似列表. 不同之处在于字典的项需要使用 `name=value` 的语法格式, 或者其它的字典变量. 如果有多个项重名, 只保留最后那个. 如果项中包含字面的等号, 则该等号必须使用反斜杠进行 转义__, 如 `\=`.

.. sourcecode:: robotframework

   *** Variables ***
   &{USER 1}       name=Matti    address=xxx         phone=123
   &{USER 2}       name=Teppo    address=yyy         phone=456
   &{MANY}         first=1       second=${2}         ${3}=third
   &{EVEN MORE}    &{MANY}       first=override      empty=
   ...             =empty        key\=here=value

字典变量相较于普通的Python字典有两个额外的属性(properties).

首先, 字典的项可以作为属性(attributes)获取, 也就是说使用 `extended variable syntax`_ 如 `${VAR.key}`. 前提是该key是一个合法的属性名且不会匹配上任何其它普通的属性. 例如, `&{USER}[name]` 同样可以通过 `${USER.name}` 获取(注意到这里是 `$` ), 但是 `${MANY.3}` 就不可以.

另一个特别之处在于字典变量中的项是有顺序的. 也就是说字典总是会按定义时的顺序迭代. 这在把字典当作  `list variables`_ 使用时(例如在 `for loops`_ )很有用. 当字典被当作列表迭代时, 实际返回的值是字典的键. 例如,  `@{MANY}` 变量的值是 `['first', 'second', 3]`.

__ Escaping_

.. Variable file

变量文件
~~~~~~~~~~~~~

变量文件是创建不同类型变量的强大武器. 使用变量文件可以给变量赋值为任意的对象, 同时还可以动态地创建变量. 关于变量文件的语法以及如何使用请参见 `Resource and variable files`_.

.. Setting variables in command line

命令行中设置变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

变量可以在命令行中通过选项 :option:`--variable (-v)` 单个设置, 也可以通过选项 :option:`--variablefile (-V)` 设置变量文件. 通过命令行设置的变量对所有执行的测试文件是全局可见的, 不过如果局部的变量表格或者局部导入的变量文件中存在重名的变量, 则这些变量也会被命令行中指定的值所覆盖.

设置单个变量的选项格式是 :option:`--variable name:value`, 其中 `name` 是变量名, 不带 `${}`, `value`是变量的值. 有多个变量的话就使用这个选项多次. 这种方式只能定义标量变量. 很多特殊字符必须使用选项 :option:`--escape` 经过 转义_ 才能表示. 

__ `Escaping complicated characters`_

.. sourcecode:: bash

   --variable EXAMPLE:value
   --variable HOST:localhost:7272 --variable USER:robot
   --variable ESCAPED:Qquotes_and_spacesQ --escape quot:Q --escape space:_

在上例中, 变量值分别是:

- `${EXAMPLE}` 值为 `value`
- `${HOST}` 和 `${USER}` 值分别为 `localhost:7272` 和 `robot`
- `${ESCAPED}` 值为 `"quotes and spaces"`

在命令行中指定 `variable files`_ 的选项格式是 :option:`--variablefile path/to/variables.py`, `Taking variable files into use`_ 章节中介绍更多细节. 

如果变量同时在命令行的变量文件中和单独指定, 则单独指定的变量有更高的 优先级__

__ `Variable priorities and scopes`_

.. Return values from keywords

关键字的返回值
~~~~~~~~~~~~~~~~~~~~~~~~~~~

关键字的返回值可以赋值给变量, 这样不同的关键字之间就可以交互了.

这种方式定义的变量和其它变量基本相同, 只是其作用域仅限于它们被创建的 `local scope`_. 也就是说 *不可能* 在一个测试用例里得到这样一个返回值变量, 然后在另一个用例中使用. 因为自动化测试用例通常需要保持相互独立, 而不应该互相依赖. 如果用例中定义的变量可以在其它用例使用, 这将导致很难定位的错误. 但是如果确实有这种需求, 也可以通过下节介绍的 BuiltIn_ 中的相关关键字来实现.

.. Assigning scalar variables

赋值给标量
''''''''''''''''''''''''''

关键字返回的任何值都可以赋值给 `scalar variable`_. 如下例所示, 语法非常简单:

.. sourcecode:: robotframework

   *** Test Cases ***
   Returning
       ${x} =    Get X    an argument
       Log    We got ${x}!

上例中, 关键字 :name:`Get X` 的返回值首先赋值给变量 `${x}`, 然后又传给关键字 :name:`Log`. 变量名称后面的等号(`=`)并不是强制要求的, 不过这种写法可以是赋值操作显得更明确. 
这种创建局部变量的方法同时适用于测试用例和用户关键字. 

注意, 虽然值是赋给了标量变量, 但是其本身如果是一个列表(或类似列表), 则它也可以当做 `list variable`_ 使用, 如果是一个类似字典的对象, 可以当做 `dictionary variable`_ 使用.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       ${list} =    Create List    first    second    third
       Length Should Be    ${list}    3
       Log Many    @{list}

.. Assigning list variables

赋值给列表变量
''''''''''''''''''''''''

如果关键字返回一个列表或者类似列表的对象, 则可以赋给 `list variable`_:

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       @{list} =    Create List    first    second    third
       Length Should Be    ${list}    3
       Log Many    @{list}

因为Robot Framework所有的变量都存储在相同的命名空间, 赋值给标量变量还是列表变量其实没有太多的差别. 最主要的差别就是当创建列表变量时, Robot Framework 自动校验值是否为列表或类似列表, 并且新建一个列表来保存返回的值. 当赋值给标量变量时, 返回值不会校验, 完全按照返回对象的类型保存值.

.. Assigning dictionary variables

赋值给字典变量
''''''''''''''''''''''''''''''

如果关键字返回一个字典或者类似字典的对象, 则可以赋给 `dictionary variable`_:

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       &{dict} =    Create Dictionary    first=1    second=${2}    ${3}=third
       Length Should Be    ${dict}    3
       Do Something    &{dict}
       Log    ${dict.first}

因为Robot Framework所有的变量都存储在相同的命名空间, 所以也可先把字典值赋值给标量变量, 后面再有需要时当作字典使用.

虽然如此, 但显式的创建字典变量也有实际的好处. 首先, Robot Framework会校验返回值确实是字典或者类似字典的对象. 

另一个更大的好处是, 值会被转换保存为一个特殊的字典, 就像在变量表格中 `创建字典变量`_ 的那样, 可以通过获取属性值的语法 `${dict.first}` 获取其中的值. 同时, 这个字典的顺序是固定的. 当然, 如果初始字典是无序的, 结果字典的顺序也是随机的.

.. Assigning multiple variables

赋值多个变量
''''''''''''''''''''''''''''

如果一个关键字返回列表或类似列表的对象, 还可以一次性将其中的值同时赋值给多个变量. 可以是多个标量, 也可以是标量和列表混合, 如下例所示:

.. sourcecode:: robotframework

   *** Test Cases ***
   Assign Multiple
       ${a}    ${b}    ${c} =    Get Three
       ${first}    @{rest} =    Get Three
       @{before}    ${last} =    Get Three
       ${begin}    @{middle}    ${end} =    Get Three

假设关键字 :name:`Get Three` 返回一个列表 `[1, 2, 3]`, 会创建的变量如下:

- `${a}`, `${b}` and `${c}` 值分别是 `1`, `2`, and `3`.
- `${first}` 值为 `1`, `@{rest}` 值为 `[2, 3]`.
- `@{before}` 值为 `[1, 2]`, `${last}` 值为 `3`.
- `${begin}` 值为 `1`, `@{middle}` 值为 `[2]`, ${end} 值为 `3`.

如果返回的列表的元素个数多于或者少于可供赋值的标量, 将会报错. 另外, 待赋值的变量中最多只能有一个列表变量, 而字典变量只能单独赋值.

It is an error if the returned list has more or less values than there are
scalar variables to assign. Additionally, only one list variable is allowed
and dictionary variables can only be assigned alone.

同时为多个变量赋值的特性功能在Robot Framework 2.9版本中有所变动. 早期版本中, 列表变量只被允许出现在待赋值变量的最后, 现在则可以是任意位置. 此外, 如果返回的值个数多于标量变量的个数, 最后一个标量会自动变为列表以包含剩下所有的值.

Additionally, it was possible to return more values than scalar variables.
In that case the last scalar variable was magically turned into a list
containing the extra values.

.. note:: 译注, 这段存疑, 和前面矛盾了.实际测试结果是会报错

.. Using :name:`Set Test/Suite/Global Variable` keywords

:name:`Set Test/Suite/Global Variable` 关键字
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

BuiltIn_ 测试库提供了几个可以在测试执行时动态设置变量的关键字: :name:`Set Test Variable`, :name:`Set Suite Variable` 和 :name:`Set Global Variable`. 如果作用域内已经存在同名变量, 则会覆盖变量的值否则创建新的变量.

通过关键字 :name:`Set Test Variable` 设置的变量在当前测试用例的作用域内处处可用. 例如, 在一个测试用例中的一个用户关键字中设置了一个变量, 该变量会在这个测试用例步骤可见, 同时当前用例中的其它用户关键字也可以使用这个变量. 这个关键字创建的变量在其它测试用例中不可用. 

通过关键字  :name:`Set Suite Variable` 创建的变量在当前执行的测试套件内处处可见. 使用这个方式创建变量和在测试数据文件的 `Variable table`_ 中定义变量, 以及从 `variable files`_ 导入变量的效果一样. 这些变量对其它的测试套件, 包括子套件, 都不可见.

通过关键字 :name:`Set Global Variable` 创建的变量在设置之后全局可见. 这种方式创建的变量和在 `creating from the command line`__ 中使用选项 :option:`--variable` 或 :option:`--variablefile` 定义变量效果一样. 因为这个关键字会改变任意地方的变量, 所以必须谨慎使用.

.. note:: 关键字 :name:`Set Test/Suite/Global Variable` 直接在 `作用域`__
          内设置变量, 没有返回值. 而 :name:`Set Variable` 设置局部变量, 并且 返回__.

__ `Setting variables in command line`_
__ `Variable scopes`_
__ `Return values from keywords`_

.. _built-in variable:

.. Built-in variables

内置变量
------------------

Robot Framework 提供了若干的内置变量, 这些变量在测试中自动可用.

.. Operating-system variables

操作系统变量
~~~~~~~~~~~~~~~~~~~~~~~~~~

操作系统相关的内置变量使得编写针对不同操作系统的测试数据变的轻松.

.. table:: Available operating-system-related built-in variables
   :class: tabular

   +------------+------------------------------------------------------------------+
   |  Variable  |                      Explanation                                 |
   +============+==================================================================+
   | ${CURDIR}  | An absolute path to the directory where the test data            |
   |            | file is located. This variable is case-sensitive.                |
   +------------+------------------------------------------------------------------+
   | ${TEMPDIR} | An absolute path to the system temporary directory. In UNIX-like |
   |            | systems this is typically :file:`/tmp`, and in Windows           |
   |            | :file:`c:\\Documents and Settings\\<user>\\Local Settings\\Temp`.|
   +------------+------------------------------------------------------------------+
   | ${EXECDIR} | An absolute path to the directory where test execution was       |
   |            | started from.                                                    |
   +------------+------------------------------------------------------------------+
   | ${/}       | The system directory path separator. `/` in UNIX-like            |
   |            | systems and :codesc:`\\` in Windows.                             |
   +------------+------------------------------------------------------------------+
   | ${:}       | The system path element separator. `:` in UNIX-like              |
   |            | systems and `;` in Windows.                                      |
   +------------+------------------------------------------------------------------+
   | ${\\n}     | The system line separator. :codesc:`\\n` in UNIX-like systems and|
   |            | :codesc:`\\r\\n` in Windows. New in version 2.7.5.               |
   +------------+------------------------------------------------------------------+

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Create Binary File    ${CURDIR}${/}input.data    Some text here${\n}on two lines
       Set Environment Variable    CLASSPATH    ${TEMPDIR}${:}${CURDIR}${/}foo.jar

.. Number variables

数字变量
~~~~~~~~~~~~~~~~

变量的语法可以用来创建整型整数和浮点型数字. 如下例所示. 因为 Robot Framework默认传递的是字符串, 显式的传递数字对那些预期接受参数是数字(而不是数字字符串)的关键字来说很有用.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example 1A
       Connect    example.com    80       # Connect gets two strings as arguments

   Example 1B
       Connect    example.com    ${80}    # Connect gets a string and an integer

   Example 2
       Do X    ${3.14}    ${-1e-4}        # Do X gets floating point numbers 3.14 and -0.0001

使用 `0b`, `0o` 和 `0x` 前缀还可以创建二进制, 八进制 和十六进制的数字. 注意这里的语法不区分大小写.

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       Should Be Equal    ${0b1011}    ${11}
       Should Be Equal    ${0o10}      ${8}
       Should Be Equal    ${0xff}      ${255}
       Should Be Equal    ${0B1010}    ${0XA}

.. Boolean and None/null variables

布尔值和None/null值
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

布尔值和Python中的 `None`, 以及Java中的 `null` 也可以使用类似数字变量的语法来表示.

.. sourcecode:: robotframework

   *** Test Cases ***
   Boolean
       Set Status    ${true}               # Set Status gets Boolean true as an argument
       Create Y    something   ${false}    # Create Y gets a string and Boolean false

   None
       Do XYZ    ${None}                   # Do XYZ gets Python None as an argument

   Null
       ${ret} =    Get Value    arg        # Checking that Get Value returns Java null
       Should Be Equal    ${ret}    ${null}

这些变量都不区分大小写, 例如 `${True}` 和 `${true}` 是一样的. 同样, `${None}` 和 `${null}` 也是同义的, 因为当使用Jython解释器运行时, Jython会视情况自动转换.

.. Space and empty variables

空格和空字符串/列表/字典
~~~~~~~~~~~~~~~~~~~~~~~~~

变量 `${SPACE}` 和 `${EMPTY}` 分别用来创建空格和空字符串. 使用这些变量相对于使用反斜杠 `escape spaces or empty cells`__ 来说容易的多. 同时还可以使用 `extended variable syntax`_ 表示连续的多个空格, 例如 `${SPACE * 5}`.

下面的例子中, 关键字 :name:`Should Be Equal` 接收到两个等价的入参, 可以看出使用变量的形式比使用反斜杠看上去容易理解的多.

.. sourcecode:: robotframework

   *** Test Cases ***
   One Space
       Should Be Equal    ${SPACE}          \ \

   Four Spaces
       Should Be Equal    ${SPACE * 4}      \ \ \ \ \

   Ten Spaces
       Should Be Equal    ${SPACE * 10}     \ \ \ \ \ \ \ \ \ \ \

   Quoted Space
       Should Be Equal    "${SPACE}"        " "

   Quoted Spaces
       Should Be Equal    "${SPACE * 2}"    " \ "

   Empty
       Should Be Equal    ${EMPTY}          \

同样还可以使用 `列表变量`_ 的格式 `@{EMPTY}` 表示空列表, `字典变量`_ 的格式 `&{EMPTY}` 表示空字典. 

在某些情况下, 它们会很有用. 比如, 当使用 `test templates`_ 且 `template keyword is used without arguments`__ 时; 或者想要覆盖不同作用域中的列表或字典变量时. 注意, 没法改变 `@{EMPTY}` 或 `&{EMPTY}` 的值.

There is also an empty `list variable`_ `@{EMPTY}` and an empty `dictionary
variable`_ `&{EMPTY}`. Because they have no content, they basically
vanish when used somewhere in the test data. They are useful, for example,
with `test templates`_ when the `template keyword is used without
arguments`__ or when overriding list or dictionary variables in different
scopes. Modifying the value of `@{EMPTY}` or `&{EMPTY}` is not possible.

.. sourcecode:: robotframework

   *** Test Cases ***
   Template
       [Template]    Some keyword
       @{EMPTY}

   Override
       Set Global Variable    @{LIST}    @{EMPTY}
       Set Suite Variable     &{DICT}    &{EMPTY}

.. note:: `@{EMPTY}` 在Robot Framework 2.7.4版本可用, `&{EMPTY}` 在2.9版本后可用.

__ Escaping_
__ https://groups.google.com/group/robotframework-users/browse_thread/thread/ccc9e1cd77870437/4577836fe946e7d5?lnk=gst&q=templates#4577836fe946e7d5

.. Automatic variables

自动变量
~~~~~~~~~~~~~~~~~~~

Robot Framework还提供了若干的自动变量. 这些变量在测试执行过程中有不同的值, 有些还是全局可用的. 改变这些变量的值不会影响其初始值, 不过其中某些可用通过 `BuiltIn`_ 库中的关键字进行动态修改.

.. table:: Available automatic variables
   :class: tabular

   +------------------------+-------------------------------------------------------+------------+
   |        Variable        |                    Explanation                        | Available  |
   +========================+=======================================================+============+
   | ${TEST NAME}           | The name of the current test case.                    | Test case  |
   +------------------------+-------------------------------------------------------+------------+
   | @{TEST TAGS}           | Contains the tags of the current test case in         | Test case  |
   |                        | alphabetical order. Can be modified dynamically using |            |
   |                        | :name:`Set Tags` and :name:`Remove Tags` keywords.    |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${TEST DOCUMENTATION}  | The documentation of the current test case. Can be set| Test case  |
   |                        | dynamically using using :name:`Set Test Documentation`|            |
   |                        | keyword. New in Robot Framework 2.7.                  |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${TEST STATUS}         | The status of the current test case, either PASS or   | `Test      |
   |                        | FAIL.                                                 | teardown`_ |
   +------------------------+-------------------------------------------------------+------------+
   | ${TEST MESSAGE}        | The message of the current test case.                 | `Test      |
   |                        |                                                       | teardown`_ |
   +------------------------+-------------------------------------------------------+------------+
   | ${PREV TEST NAME}      | The name of the previous test case, or an empty string| Everywhere |
   |                        | if no tests have been executed yet.                   |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${PREV TEST STATUS}    | The status of the previous test case: either PASS,    | Everywhere |
   |                        | FAIL, or an empty string when no tests have been      |            |
   |                        | executed.                                             |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${PREV TEST MESSAGE}   | The possible error message of the previous test case. | Everywhere |
   +------------------------+-------------------------------------------------------+------------+
   | ${SUITE NAME}          | The full name of the current test suite.              | Everywhere |
   +------------------------+-------------------------------------------------------+------------+
   | ${SUITE SOURCE}        | An absolute path to the suite file or directory.      | Everywhere |
   +------------------------+-------------------------------------------------------+------------+
   | ${SUITE DOCUMENTATION} | The documentation of the current test suite. Can be   | Everywhere |
   |                        | set dynamically using using :name:`Set Suite          |            |
   |                        | Documentation` keyword. New in Robot Framework 2.7.   |            |
   +------------------------+-------------------------------------------------------+------------+
   | &{SUITE METADATA}      | The free metadata of the current test suite. Can be   | Everywhere |
   |                        | set using :name:`Set Suite Metadata` keyword.         |            |
   |                        | New in Robot Framework 2.7.4.                         |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${SUITE STATUS}        | The status of the current test suite, either PASS or  | `Suite     |
   |                        | FAIL.                                                 | teardown`_ |
   +------------------------+-------------------------------------------------------+------------+
   | ${SUITE MESSAGE}       | The full message of the current test suite, including | `Suite     |
   |                        | statistics.                                           | teardown`_ |
   +------------------------+-------------------------------------------------------+------------+
   | ${KEYWORD STATUS}      | The status of the current keyword, either PASS or     | `User      |
   |                        | FAIL. New in Robot Framework 2.7                      | keyword    |
   |                        |                                                       | teardown`_ |
   +------------------------+-------------------------------------------------------+------------+
   | ${KEYWORD MESSAGE}     | The possible error message of the current keyword.    | `User      |
   |                        | New in Robot Framework 2.7.                           | keyword    |
   |                        |                                                       | teardown`_ |
   +------------------------+-------------------------------------------------------+------------+
   | ${LOG LEVEL}           | Current `log level`_. New in Robot Framework 2.8.     | Everywhere |
   +------------------------+-------------------------------------------------------+------------+
   | ${OUTPUT FILE}         | An absolute path to the `output file`_.               | Everywhere |
   +------------------------+-------------------------------------------------------+------------+
   | ${LOG FILE}            | An absolute path to the `log file`_ or string NONE    | Everywhere |
   |                        | when no log file is created.                          |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${REPORT FILE}         | An absolute path to the `report file`_ or string NONE | Everywhere |
   |                        | when no report is created.                            |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${DEBUG FILE}          | An absolute path to the `debug file`_ or string NONE  | Everywhere |
   |                        | when no debug file is created.                        |            |
   +------------------------+-------------------------------------------------------+------------+
   | ${OUTPUT DIR}          | An absolute path to the `output directory`_.          | Everywhere |
   +------------------------+-------------------------------------------------------+------------+

测试套件相关的变量 `${SUITE SOURCE}`, `${SUITE NAME}`, `${SUITE DOCUMENTATION}` 和 `&{SUITE METADATA}` 在测试库和变量文件被导入时即可访问. 除了在 Robot Framework 2.8 和 2.8.1 版本里. 不过, 上表中其它的某些自动变量在导入时刻还没有解析.

.. Variable priorities and scopes

变量的优先级和作用域
------------------------------

不同来源的变量拥有不同的优先级, 并作用于不同的作用域.

.. Variable priorities

变量的优先级
~~~~~~~~~~~~~~~~~~~

*通过命令行设置的变量*
  
   对于所有那些在测试执行前指定的变量来说, 通过 `命令行设置`__ 的变量拥有最高优先级.
   这些变量有可能会覆盖在测试用例文件的变量表格中定义的变量, 或者导入的资源文件或变量文件中的变量.

   单独设定的变量(:option:`--variable` 选项)可能会覆盖通过 `variable files`_ (:option:`--variablefile` 选项)定义的变量. 如果同名的变量单独设置多次, 则只生效最后那个. 这种行为使得我们可以在 `start-up script`_ 中设置缺省的变量值, 并在命令行调用时看情况予以覆盖. 

   注意, 如果多个变量文件中有同名参数, 第一个文件中定义的那个变量有最高优先级.

__ `Setting variables in command line`_

*在用例文件的变量表格中定义的变量*

   在测试用例文件的 `变量表格`_ 中创建的变量在该文件中的所有用例内可用. 这些变量有可能会覆盖在导入的资源文件或变量文件中定义的同名变量.

   变量表格中创建的变量在文件中所有其它表格中也是可用的. 也就是说, 它们可以被用在Setting表格中, 用来导入其它文件.

*导入的资源和变量文件中的变量*

   在所有测试数据中创建的变量中, 从 `resource and variable files`_ 导入的变量的优先级最低. 资源文件和变量文件中的变量的优先级相同. 如果多个资源文件和(或)变量文件有同名变量, 则生效的是第一个被导入文件中的变量.

   如果一个资源文件中继续导入其它的资源文件或变量文件, 则其自身变量表格中的变量优先级高于它导入的变量. 而最终只导入这一个资源文件, 就可以访问所有这些文件中所定义的变量.

   注意资源文件和变量文件中的变量不可用于导入它们的文件的变量表格中, 这是因为变量表格在设置表格(即文件导入的地方)之前处理.

*测试执行中定义的变量*

   通过 `return values from keywords`_ 或者 `using Set Test/Suite/Global Variable keywords`_ 在测试执行过程中设置的变量总是覆盖可能存在的同名变量.
   从这点上来说, 这些变量拥有最高的优先级. 但是, 从另一方面来看, 这些变量不会影响到它们作用域之外的变量.

*内置变量*

   `内置变量`_, 如 `${TEMPDIR}` 和 `${TEST_NAME}`, 在所有变量中拥有最高优先级. 它们不能被变量表格或者命令行选项所覆盖, 不过即使这样, 它们还是可以在测试执行过程中被重置. 一个例外是 `number variables`_, 它们总是被动态解析. 虽然也是可以被覆盖的, 但是强烈不建议这样做. 此外, `${CURDIR}` 也比较特殊, 因为它在测试数据处理前就已经被替代.

.. Variable scopes

变量的作用域
~~~~~~~~~~~~~~~

取决于变量创建的地方和方式, 它们可以拥有 全局作用域, 测试套件作用域, 测试用例作用域 或者局部作用域.

.. Global scope

全局作用域
''''''''''''

全局作用域的变量在测试数据中处处可用. 全局变量一般是从命令行设置, 通过 :option:`--variable` 和 :option:`--variablefile` 选项. 还可以使用 BuiltIn_ 关键字 :name:`Set Global Variable` 在测试执行中创建或修改全局变量. 此外, `built-in variables`_ 都是全局的.

推荐使用大写字母来表示全局变量.

.. Test suite scope

测试套件作用域
''''''''''''''''

测试套件内定义或导入的变量在该测试套件作用域内处处可见. 这些变量可以是通过变量表格创建, 也可能是来自导入的 `resource and variable files`_, 也可以使用 BuiltIn_ 关键字 :name:`Set Suite Variable` 在测试执行中创建或修改.

测试套件作用域 *不是* 递归的, 即高层测试套件内的变量在低层的测试套件内 *不可用*. 如果有必要, 使用 `resource and variable files`_ 来共享变量.

因为这些变量在测试套件内基本可当作全局性的, 所以同样推荐使用大写字母来表示.

.. Test case scope

测试用例作用域
'''''''''''''''

测试用例作用域的变量在测试用例内部, 包括用例内所有的用户关键字内, 都是可见的. 用例作用域的变量都是通过使用 BuiltIn_ 关键字 :name:`Set Test Variable` 在测试用例中创建.

该作用域内的变量同样也推荐使用大写字母表示.

.. Local scope

局部作用域
'''''''''''

测试用例和用户关键字拥有一个局部作用域, 对其它用例和关键字都是不可见的. 局部变量通过执行关键字并获取其 `return values`__ 来创建, 作为 arguments__ 传递给用户关键字.

推荐使用小写字母来表示局部变量.

.. note:: 在 Robot Framework 2.9 版本之前, 局部作用域内的变量会
          `泄露到低层的用户关键字中`__. 这个绝不能视为是有意的特性, 而应该在早期版本中也显式的设置并传递变量. 

__ `Setting variables in command line`_
__ `Return values from keywords`_
__ `User keyword arguments`_
__ https://github.com/robotframework/robotframework/issues/532

.. Advanced variable features

变量的高级特性
--------------------------

.. Extended variable syntax

变量扩展语法
~~~~~~~~~~~~~~~~~~~~~~~~

扩展的变量语法支持获取变量对象的属性值(例如, `${object.attribute}`), 甚至还可以执行对象的方法(例如, `${obj.getName()}`). 这种语法对标量和列表都可用, 但是大部分时候还是用于前者.

变量扩展语法是一个强大的特性功能, 但是应该谨慎使用. 获取变量的属性一般没有问题, 相对来说, 使用一个变量来保存拥有多个属性的对象总好于使用多个变量. 不过另一方面, 调用对象的方法(特别是方法还需要参数的时候)会使得测试数据变得复杂难懂. 如果必须这么做, 建议将调用方法的代码移到测试库中去做.

下面的例子展示了使用变量扩展语法的大多数场景. 首先假定我们有如下的 `variable file`_ 和测试用例:

.. sourcecode:: python

   class MyObject:

       def __init__(self, name):
           self.name = name

       def eat(self, what):
           return '%s eats %s' % (self.name, what)

       def __str__(self):
           return self.name

   OBJECT = MyObject('Robot')
   DICTIONARY = {1: 'one', 2: 'two', 3: 'three'}

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       KW 1    ${OBJECT.name}
       KW 2    ${OBJECT.eat('Cucumber')}
       KW 3    ${DICTIONARY[2]}

当上面的测试执行时, 关键字获取到的参数解释如下:

- :name:`KW 1` 接收到字符串 `Robot`
- :name:`KW 2` 接收到字符串 `Robot eats Cucumber`
- :name:`KW 3` 接收到字符串 `two`

扩展的变量语法按照如下的顺序进行解析:
The extended variable syntax is evaluated in the following order:

1. 变量首先按照全名进行搜索(因为变量名可包含任意字符), 
   只有在没有匹配的情况下才会继续进行扩展语法的解析.

2. 创建基础变量名称. 从 `{` 后开始, 直到遇到空格或者非字母字符, 
   这之间的字符就是基础变量的名称. 例如, `${OBJECT.name}` 和 `${DICTIONARY[2]}` 基础变量分别是 `OBJECT` and `DICTIONARY`.

3. 搜索基础变量是否存在. 如果找不到匹配的变量, 则此处就会抛出异常, 当前测试用例失败.

4. 花括号内的表达式被作为Python表达式来运行. 
   如果因为语法非法或者属性不存在等情况造成运行失败, 此处就会抛出异常, 测试用例失败.

5. 整个扩展变量被表达式运行的结果替代.

如果对象是用Java实现的, 扩展的变量语法可以用来获取properties. 即假设有个对象 `${OBJ}` 有个方法 `getName`, 则 `${OBJ.name}` 等价于 `${OBJ.getName()}`. 

上例中的Python对象用Java实现的代码:

.. sourcecode:: java

 public class MyObject:

     private String name;

     public MyObject(String name) {
         name = name;
     }

     public String getName() {
         return name;
     }

     public String eat(String what) {
         return name + " eats " + what;
     }

     public String toString() {
         return name;
     }
 }

很多Python标准对象, 包括字符串和数字, 都提供了若干实例方法. 这些方法可以使用扩展变量语法(显式或隐式地)调用. 这样做有时候会很有用, 并减少临时变量的使用, 但是如果过度使用也可能会造成测试数据模糊难懂.

下面的例子展示了几个较好的用法:

.. sourcecode:: robotframework

   *** Test Cases ***
   String
       ${string} =    Set Variable    abc
       Log    ${string.upper()}      # Logs 'ABC'
       Log    ${string * 2}          # Logs 'abcabc'

   Number
       ${number} =    Set Variable    ${-2}
       Log    ${number * 10}         # Logs -20
       Log    ${number.__abs__()}    # Logs 2

虽然在Python代码中推荐使用 `abs(number)` 替代 `number.__abs__()` 的用法, 但是在Robot Framework中 `${abs(number)}` 不会生效. 这是因为在变量的扩展语法中, 变量名必须是紧跟着花括号的前端. 不过在测试数据中使用 `__xxx__` 方法也是值得商榷的事情, 最好还是将这些逻辑移到测试库中解决.

扩展变量语法对 `list variable`_ 也有效. 例如, 如果一个变量 `${EXTENDED}` 被赋值了一个对象, 其中包含属性 `attribute`, 该属性值是一个列表, 则可以使用 `@{EXTENDED.attribute}` 将该属性当列表变量使用.


.. Extended variable assignment

扩展的变量赋值
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robot Framework 2.7 版本开始, 可以将 `keyword return values`__ 通过 `extended variable syntax`_ 赋值给一个标量变量对象的某个属性. 

假设有变量 `${OBJECT}`, 它的属性值可以按下例中的方式设置:

__ `Return values from keywords`_

.. sourcecode:: robotframework

   *** Test Cases ***
   Example
       ${OBJECT.name} =    Set Variable    New name
       ${OBJECT.new_attr} =    Set Variable    New attribute

扩展的变量赋值语法按下面的规则解析处理:
The extended variable assignment syntax is evaluated using the
following rules:

1. 被赋值的变量必须是个标量, 至少包含一个点(`.`). 否则不会触发扩展赋值语法.
2. 如果存在一个全名匹配的变量(例如 `${OBJECT.name}`), 则该变量被赋值, 
   不会使用扩展语法.
3. 创建基础变量名称. 从 `${` 后开始, 直到最后一个点, 
   这之间的字符就是基础变量的名称. 例如, `${OBJECT.name}` 和 `${foo.bar.zap}` 基础变量分别是 `OBJECT` and `foo.bar`. 在第二个例子中, 基础名称也包含了普通的扩展变量语法.

4. 属性名取自最后一个点号直到结尾括号 `}` 之间的所有字符. 例如, `${OBJECT.name}` 
   属性名是 `name`. 如果属性名不是字母或下划线开始的, 并且只包含字母,数字和下划线, 则属性名被认为是非法的, 扩展语法不会生效. 整个变量名称被当作一个名字创建新的变量.

5. 属性名合法则开始匹配基础变量名称. 如果没有找到匹配的变量, 扩展语法不会生效. 
   整个变量名称被当作一个名字创建新的变量.

6. 如果找到的变量是一个字符串或者数字, 则扩展语法不会生效,
   整个变量名称被当作一个名字创建新的变量. 这是因为在Python中不能给字符串或数字增加新的属性.

.. This is
   done because you cannot add new attributes to Python strings or
   numbers, and this way the new syntax is also less
   backwards-incompatible. 

7. 如果上述所有规则都满足了, 基础变量的属性值才会被设置. 
   如果由于其它任何原因导致属性设置失败, 将会抛出异常, 测试失败. 

.. note:: 不同于普通的使用 `return values from keywords`_ 赋值给局部变量, 
          扩展的赋值语法不限制变量的作用域. 因为这其中没有新变量被创建, 改变的只有已有变量的状态, 该变量可用的作用域内的所有测试用例和关键字都能查看到这个变化.


.. Variables inside variables

嵌套变量
~~~~~~~~~~~~~~~~~~~~~~~~~~

变量名可以嵌套使用. 这种情况下, 变量的解析从内往外进行.

例如, 有一个变量 `${var${x}}`, `${x}` 首先被解析. 如果值为 `name`, 则最终的变量名变为 `${varname}`. 可以有多层嵌套, 不过如果任何一层变量不存在, 整个变量的解析失败.

如下例所示, :name:`Do X` 取值 `${JOHN HOME}` 或 `${JANE HOME}`, 取决于 :name:`Get Name` 是返回 `john` 还是 `jane`. 如果返回的是其它值, 则 `${${name} HOME}` 解析失败.

.. sourcecode:: robotframework

   *** Variables ***
   ${JOHN HOME}    /home/john
   ${JANE HOME}    /home/jane

   *** Test Cases ***
   Example
       ${name} =    Get Name
       Do X    ${${name} HOME}
