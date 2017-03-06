.. Resource and variable files

资源文件和变量文件
===========================

在 `test case files`_ 和  `test suite initialization files`_ 中的用户关键字和变量只能在创建它们的文件中使用. *资源文件* 提供了共享机制.

资源文件的结构和用例文件非常接近, 可以很容易地创建.

*变量文件* 则提供的强大的创建和共享变量的功能. 可以创建非字符串类型的复杂变量, 以及动态地创建过程. 因为变量文件使用的是Python代码, 所以非常灵活. 当然, 相对于 `Variable tables`_ 来说, 这也稍稍带来了一定的复杂度.

.. contents::
   :depth: 2
   :local:

.. _resource file:
.. _resource files:

资源文件
--------

.. _Taking resource files into use:

如何使用资源文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

资源文件通过在设置表格中设定 :setting:`Resource` 来引入. 跟在设置名称后面的值就是资源文件所在的路径. 

如果路径使用的是绝对路径格式, 则直接使用. 如果是相对路径的话, 首先在当前文件(即要引入资源文件的文件)所在路径下相对查找该路径. 如果没有找到, 则继续在Python的 `module search path`_ 下的目录中查找. 路径名称可以并且推荐使用变量, 以便做到路径和系统无关(例如 :file:`${RESOURCES}/login_resources.html` 或 :file:`${RESOURCE_PATH}`). 此外, 这里统一使用正斜杠(`/`), 在Windows系统中会自动转换为反斜杠(:codesc:`\\`).

.. sourcecode:: robotframework

   *** Settings ***
   Resource    myresources.html
   Resource    ../data/resources.html
   Resource    ${RESOURCES}/common.tsv

定义在资源文件中的用户关键字和变量在导入后即可使用. 同时, 该资源文件中从其它文件(测试库/资源文件/变量文件)导入的关键字和变量的关键字, 也变得可用.

.. The user keywords and variables defined in a resource file are
.. available in the file that takes that resource file into
.. use. Similarly available are also all keywords and variables from the
.. libraries, resource files and variable files imported by the said
.. resource file.

.. Resource file structure

资源文件的结构
~~~~~~~~~~~~~~~~~~~~~~~

资源文件的整体结构和测试用例文件一样, 只不过其中不能包含测试用例. 此外, 资源文件中的设置表格只能包含导入相关的设置(:setting:`Library`, :setting:`Resource`, :setting:`Variables`)和文档设置(:setting:`Documentation`). 变量表格和关键字表格则和用例文件中完全一样.

如果多个资源文件中包含了重名的用户关键字, 必须使用 `资源文件名作为前缀`__ 来区分. 例如,:name:`myresources.Some Keyword` and :name:`common.Some Keyword`. 并且, 如果多个资源文件包含了同名的变量, 则最终生效的是 *最先* 导入的那个.

__ `Handling keywords with same names`_

.. _Documenting resource files:

资源文件的文档
~~~~~~~~~~~~~~~~~~~~~~~~~~

资源文件中创建的关键字可以使用 :setting:`[Documentation]` 来设置文档. 和 `测试套件`__ 类似, 资源文件本身在设置表格中也可以通过 :setting:`Documentation` 设置文档.

Libdoc_ 和 RIDE_ 都会用到这些文档, 并且这些文档内容很自然的对打开资源文件的人来说也是可见的. 

当关键字运行时, 关键字文档的第一行将写入日志, 而资源文件的文档在测试执行过程中会被忽略.

__ `User keyword name and documentation`_
__ `Test suite name and documentation`_

.. _Example resource file:

资源文件示例
~~~~~~~~~~~~~~~~~~~~~

.. sourcecode:: robotframework

   *** Settings ***
   Documentation     An example resource file
   Library           Selenium2Library
   Resource          ${RESOURCES}/common.robot

   *** Variables ***
   ${HOST}           localhost:7272
   ${LOGIN URL}      http://${HOST}/
   ${WELCOME URL}    http://${HOST}/welcome.html
   ${BROWSER}        Firefox

   *** Keywords ***
   Open Login Page
       [Documentation]    Opens browser to login page
       Open Browser    ${LOGIN URL}    ${BROWSER}
       Title Should Be    Login Page

   Input Name
       [Arguments]    ${name}
       Input Text    username_field    ${name}

   Input Password
       [Arguments]    ${password}
       Input Text    password_field    ${password}

.. _variable file:
.. _variable files:

变量文件
--------------

顾名思义, 变量文件中包含了测试数据中的 variables_. 虽然变量可以通过变量表格中创建, 或者通过命令行设置, 不过这些方法有所局限, 而变量文件可以动态地创建任意类型的变量.

变量文件一般就是由Python模块实现, 有两种不同的方法来创建变量:

 `直接创建`_
   变量就是模块的属性. 最简单的情形下, 这种语法几乎不需要真正的编程. 例如, `MY_VAR = 'my value'` 就创建了变量 `${MY_VAR}`, 后面是变量的值.

 `通过特殊函数获取变量`_
   变量文件中可以包含一个特殊的函数 `get_variables` (或者 `getVariables`),  该函数 将变量按字典的形式返回. 该函数还可以接受参数, 所以这种方法非常灵活.
   Because the method can take arguments this approach is very flexible.

此外变量文件还可以由 `Python or Java classes`__ 来实现. 具体的方法类似.

__ `Implementing variable file as Python or Java class`_

.. _Taking variable files into use:

如何使用变量文件
~~~~~~~~~~~~~~~~

.. _Setting table:

通过Setting
'''''''''''''

所有的测试数据文件都可以在设置表中通过 :setting:`Variables` 来导入变量, 如同使用 :setting:`Resource` 来 `导入资源文件`__ 一样. 和资源文件的查找顺序类似, 待导入的变量文件路径最开始在相对于当前要导入变量的文件所在路径上寻找, 如果找不到, 则继续在 `模块搜索路径`_ 上搜寻. 路径名称可以使用变量, 并且在Windows中也可以使用正斜杠.

如果 `变量文件可以接受参数`_, 这些参数跟在路径后面的单元格中, 并且这些参数同样可以使用变量.

__ `Taking resource files into use`_
__ `Getting variables from a special function`_

.. sourcecode:: robotframework

   *** Settings ***
   Variables    myvariables.py
   Variables    ../data/variables.py
   Variables    ${RESOURCES}/common.py
   Variables    taking_arguments.py    arg1    ${ARG2}

变量文件中定义的所有变量在导入它的测试文件中都是可见的. 如果同时导入了多个变量文件并且存在名称冲突, 则最先导入的生效. 此外, 通过变量表格和命令行方式设置的变量会覆盖变量文件中的同名变量.

.. _Command line:

通过命令行
''''''''''''

还可以通过命令行选项 :option:`--variablefile` 来指定变量文件. 选项后面跟着文件的路径, 如果要传递参数的话, 使用冒号 (`:`) 来分隔::

   --variablefile myvariables.py
   --variablefile path/variables.py
   --variablefile /absolute/path/common.py
   --variablefile taking_arguments.py:arg1:arg2

从Robot Framework 2.8.2版本开始, 通过命令行设置的变量文件同样支持在 `模块搜索路径`_ 上搜寻.

如果文件路径使用了Windows的绝对路径格式, 驱动器号后面的冒号不会被视作分隔符::

   --variablefile C:\path\variables.py

从Robot Framework 2.8.7版本开始, 还可以使用分号(`;`)作为参数的分隔符. 这种情况对参数本身也包含冒号时特别有用. 不过需要注意, 在UNIX-like操作系统中, 要使用双引号将整个选项值括起来::

   --variablefile "myvariables.py;argument:with:colons"
   --variablefile C:\path\variables.py;D:\data.xls

这些变量文件中的变量在所有测试文件中全局可见, 这点和通过选项 :option:`--variable` 来设置 `individual variables`__ 类似.

如果同时使用了 :option:`--variablefile` 和 :option:`--variable` 选项, 并且发生变量名冲突, 则使用 :option:`--variable` 选项设置的变量胜出.

__ `Setting variables in command line`_

.. _Creating variables directly:

直接创建变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Basic syntax

基础语法
''''''''''''

当使用变量文件时, 它们像Python的模块一样被导入, 其中的非下划线(`_`)开头的全局属性均被视作变量. 因为变量的名字是不区分大小写的, 所以不管小写还是大写字母都是可以的, 通常推荐大写字母用作全局变量和属性.

.. sourcecode:: python

   VARIABLE = "An example string"
   ANOTHER_VARIABLE = "This is pretty easy!"
   INTEGER = 42
   STRINGS = ["one", "two", "kolme", "four"]
   NUMBERS = [1, INTEGER, 3.14]
   MAPPING = {"one": 1, "two": 2, "three": 3}

In the example above, variables `${VARIABLE}`, `${ANOTHER VARIABLE}`, and
so on, are created. The first two variables are strings, the third one is
an integer, then there are two lists, and the final value is a dictionary.
All these variables can be used as a `scalar variable`_, lists and the
dictionary also a `list variable`_ like `@{STRINGS}` (in the dictionary's case
that variable would only contain keys), and the dictionary also as a
`dictionary variable`_ like `&{MAPPING}`.

在上面的例子中, 创建了 `${VARIABLE}`, `${ANOTHER VARIABLE}` 等变量. 前面2个是字符串, 第3个是整数, 接下来是两个列表, 最后一个是字典. 这些变量都可以用作 `scalar variable`_, 列表和字典还可以当作 `list variable`_ 如 `@{STRINGS}` (注字典当列表变量使用时只包含字典的键), 而字典显然可以被当作 `dictionary variable`_ 如 `&{MAPPING}`_.

如果想让列表和字典类型的变量显得更明确, 可以分别使用前缀 `LIST__` 和 `DICT__`来区分(注意后面是两个下划线):

.. sourcecode:: python

   from collections import OrderedDict

   LIST__ANIMALS = ["cat", "dog"]
   DICT__FINNISH = OrderedDict([("cat", "kissa"), ("dog", "koira")])

这些前缀最终不会被视作变量名称的一部分, 只是使得Robot Framework校验变量的值的类型是否符合. 对字典来说, 变量值还将转换为特殊的字典类型, 就像 `creating dictionary variables`_ 中使用的一样. 这样这些字典之中的值就可以像访问属性一样获取, 如 `${FINNISH.cat}`. 同时这些字典还是排序的, 不过保持源顺序要求初始的字典是排序的.

上面例子中的变量同样可以使用下面的方式在变量表中创建. 

These prefixes will not be part of the final variable name, but they cause
Robot Framework to validate that the value actually is list-like or
dictionary-like. With dictionaries the actual stored value is also turned
into a special dictionary that is used also when `creating dictionary
variables`_ in the Variable table. Values of these dictionaries are accessible
as attributes like `${FINNISH.cat}`. These dictionaries are also ordered, but
preserving the source order requires also the original dictionary to be
ordered.

The variables in both the examples above could be created also using the
Variable table below.

.. sourcecode:: robotframework

   *** Variables ***
   ${VARIABLE}            An example string
   ${ANOTHER VARIABLE}    This is pretty easy!
   ${INTEGER}             ${42}
   @{STRINGS}             one          two           kolme         four
   @{NUMBERS}             ${1}         ${INTEGER}    ${3.14}
   &{MAPPING}             one=${1}     two=${2}      three=${3}
   @{ANIMALS}             cat          dog
   &{FINNISH}             cat=kissa    dog=koira

.. note:: 变量文件中的字符串中的变量格式是不会当变量替换的. 例如, 
          `VAR = "an ${example}"` 将创建变量 `${VAR}`, 其值为 `an ${example}`.
          是否存在变量 `${example}` 都不会影响.


.. _Using objects as values:

使用对象
'''''''''''''''''''''''

变量文件中变量定义突破了变量表格中只能定义字符串和基础类型的限制, 现在变量可以包含任意类型的对象. 在下面的例子中, 变量 `${MAPPING}` 包含了一个Java哈希表, 其中包含两个值(该例子只适用于Jython上运行).

.. sourcecode:: python

    from java.util import Hashtable

    MAPPING = Hashtable()
    MAPPING.put("one", 1)
    MAPPING.put("two", 2)

第二个例子创建了Python的字典 `${MAPPING}`, 同样包含两个值, 且这两个值是该文件中自定义类的实例.

.. sourcecode:: python

    MAPPING = {'one': 1, 'two': 2}

    class MyObject:
        def __init__(self, name):
            self.name = name

    OBJ1 = MyObject('John')
    OBJ2 = MyObject('Jane')

.. _Creating variables dynamically:

动态创建变量
''''''''''''''''''''''''''''''

因为变量文件就是真正的编程语言, 其中几乎可以包含任意的代码逻辑来设置变量.

.. sourcecode:: python

   import os
   import random
   import time

   USER = os.getlogin()                # current login name
   RANDOM_INT = random.randint(0, 10)  # random integer in range [0,10]
   CURRENT_TIME = time.asctime()       # timestamp like 'Thu Apr  6 12:45:21 2006'
   if time.localtime()[3] > 12:
       AFTERNOON = True
   else:
       AFTERNOON = False

The example above uses standard Python libraries to set different
variables, but you can use your own code to construct the values. The
example below illustrates the concept, but similarly, your code could
read the data from a database, from an external file or even ask it from
the user.

上面的例子中使用了Python标准库来设置不同的变量, 你也可以使用自己的代码来构造这些值.

下面的例子展示了概念, 类似地, 真实的代码中的数据可以是来自数据库, 或者外部文件, 甚至是要求用户输入.

.. sourcecode:: python

    import math

    def get_area(diameter):
        radius = diameter / 2
        area = math.pi * radius * radius
        return area

    AREA1 = get_area(1)
    AREA2 = get_area(2)

.. _Selecting which variables to include:

选择性的包含变量
''''''''''''''''''''''''''''''''''''

当 Robot Framework 处理变量文件时, 这些文件(模块)中所有的属性只要不是以下划线开头, 都会被视作变量, 这其中甚至包括函数或类, 不管是在文件中创建的还是从其它模块导入的. 例如, 上面最后一个例子中除了 `${AREA1}` 和 `${AREA2}` 这两个我们预期的变量外, 最终还包含了 `${math}` 和 `${get_area}` 这两个变量.

虽然通常情况下这些额外的变量不会造成什么问题, 但是它们有可能会无意覆盖其它的变量名, 由此引发的错误将难以定位. 一个可行的解决办法是通过加下划线作为前缀来忽略这些属性:

.. sourcecode:: python

    import math as _math

    def _get_area(diameter):
        radius = diameter / 2.0
        area = _math.pi * radius * radius
        return area

    AREA1 = _get_area(1)
    AREA2 = _get_area(2)

但是如果属性的数量非常多, 这样做就很不方便(同时, 这种做法也不符合Python的编码风格). 推荐的做法是使用特殊属性 `__all__`, 将要作为变量暴露的属性名放在列表中赋值给它.

.. sourcecode:: python

    import math

    __all__ = ['AREA1', 'AREA2']

    def get_area(diameter):
        radius = diameter / 2.0
        area = math.pi * radius * radius
        return area

    AREA1 = get_area(1)
    AREA2 = get_area(2)

.. note:: `__all__` 属性在Python中最初就是用来设置哪些属性可以在
          `from modulename import *` 的语法中被导入.


.. _Getting variables from a special function:

通过特殊函数获取变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在变量文件中获取变量的另一种方法是通过特殊的函数 `get_variables`(或 `getVariables`). 如果这个函数存在, Robot Framework将调用该函数, 并且预期返回的结果是Python的字典类型或者Java中的 `Map` 类型, 其中变量的名称是键, 而值就是变量的值. 

创建的变量可以用作标量, 列表和字典, 就和 `creating variables directly`_ 完全一样, 同样可以使用前缀 `LIST__` 和 `DICT__` 来明确表示创建的是列表和字典. 

下面的例子和 `creating variables directly`_ 中的第一个例子在功能上完全相同.

.. sourcecode:: python

    def get_variables():
        variables = {"VARIABLE ": "An example string",
                     "ANOTHER VARIABLE": "This is pretty easy!",
                     "INTEGER": 42,
                     "STRINGS": ["one", "two", "kolme", "four"],
                     "NUMBERS": [1, 42, 3.14],
                     "MAPPING": {"one": 1, "two": 2, "three": 3}}
        return variables

`get_variables` 可以接受参数, 这样可以很方便的改变实际要创建什么样的变量. 参数的数量和类型和普通的Python函数并无二致. 当在测试数据中 `taking variable files into use`_ 时, 调用参数跟在变量文件后面的表格里, 而在命令行中则通过冒号或分号和文件路径分开.


下面这个傻傻的例子展示了变量文件如何使用参数. 在更真实的场景中, 这些参数可能是一个用来读取参数的外部文件的路径, 或者是数据库的地址.

.. sourcecode:: python

    variables1 = {'scalar': 'Scalar variable',
                  'LIST__list': ['List','variable']}
    variables2 = {'scalar' : 'Some other value',
                  'LIST__list': ['Some','other','value'],
                  'extra': 'variables1 does not have this at all'}

    def get_variables(arg):
        if arg == 'one':
            return variables1
        else:
            return variables2

.. _Implementing variable file as Python or Java class:

用类实现变量文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

从Robot Framework 2.7版本开始, 还可以使用Python或Java之中的类来实现变量文件.

.. Implementation

具体实现
''''''''''''''

因为变量导入时使用的文件路径, 所有使用类实现的时候有一些限制:

  - Python的类名必须和所在的模块名相同.
  - Java类必须在默认包中.
  - 指向Java类的路径必须以 :file:`.java` 或 :file:`.class` 结尾, class文件必须存在.

不管以何种语言实现, 框架都将不带参数的构造一个实例, 通过该实例获取变量. 和使用模块类似, 变量可以直接定义为实例的属性, 也可以使用特殊的 `get_variables`(或 `getVariables`) 方法.

当直接定义变量时, 会忽略所有可调用的(callable)的属性以避免调用实例的方法. 如果需要可调用的变量, 需要使用其它的方法来创建变量文件.

.. Examples

示例
''''''''

第一个例子通过属性直接创建变量, 同时以Python和Java两种语言实现. 两个例子的效果相同, 都通过类的属性创建了变量 `${VARIABLE}` and `@{LIST}`, 并通过实例的属性创建变量 `${ANOTHER VARIABLE}`.

.. sourcecode:: python

    class StaticPythonExample(object):
        variable = 'value'
        LIST__list = [1, 2, 3]
        _not_variable = 'starts with an underscore'

        def __init__(self):
            self.another_variable = 'another value'

.. sourcecode:: java

    public class StaticJavaExample {
        public static String variable = "value";
        public static String[] LIST__list = {1, 2, 3};
        private String notVariable = "is private";
        public String anotherVariable;

        public StaticJavaExample() {
            anotherVariable = "another value";
        }
    }

第二个例子通过动态的方法来获取变量. 同样, 两种语言的效果一样, 都创建了唯一的变量 `${DYNAMIC VARIABLE}`.

.. sourcecode:: python

    class DynamicPythonExample(object):

        def get_variables(self, *args):
            return {'dynamic variable': ' '.join(args)}

.. sourcecode:: java

    import java.util.Map;
    import java.util.HashMap;

    public class DynamicJavaExample {

        public Map<String, String> getVariables(String arg1, String arg2) {
            HashMap<String, String> variables = new HashMap<String, String>();
            variables.put("dynamic variable", arg1 + " " + arg2);
            return variables;
        }
    }

.. _Variable file as YAML:

YAML格式的变量文件
~~~~~~~~~~~~~~~~~~~~~

变量文件还可以使用  `YAML <http://yaml.org>`_ 文件. YAML是一种数据序列化的标记语言, 拥有简单的语法和友好的可读性. 下面的例子展示了一个简单的YAML文件:

.. sourcecode:: yaml

    string:   Hello, world!
    integer:  42
    list:
      - one
      - two
    dict:
      one: yksi
      two: kaksi
      with spaces: kolme

.. note:: 在Robot Framework中使用YAML文件要求安装 `PyYAML
          <http://pyyaml.org>`_ 模块. 如果已经有了 pip_, 则使用下面的命令即可安装
          `pip install pyyaml`.

          Robot Framework从2.9版本开始支持YAML. 从2.9.2版本开始, `standalone JAR distribution`_ 已经默认包含了PyYAML.

YAML 变量文件的使用和其它变量文件完全一样, 既可以使用命令行选项 :option:`--variablefile`, 也可以使用配置 :setting:`Variables`, 或者使用关键字 :name:`Import Variables` 动态导入. 唯一需要记住的是, 导入YAML文件的路径名必须以 :file:`.yaml` 扩展名结尾.

上例中的YAML文件创建的变量和下面的变量表格创建的变量完全一样.

.. sourcecode:: robotframework

   *** Variables ***
   ${STRING}     Hello, world!
   ${INTEGER}    ${42}
   @{LIST}       one         two
   &{DICT}       one=yksi    two=kaksi

使用YAML文件作为变量文件必须总是使用顶层的映射(mappings). 如上例所示, 映射中的键和值分别是变量的名称和值. 变量的值可以是YAML语法支持的任意数据类型. 如果名称或值中包含了non-ASCII的字符, 则YAML文件必须使用UTF-8编码格式.

如果值是mapping类型, 则最终将转换为特殊的字典, 这一点等同于在变量表格中 `creating dictionary variables`_. 这样就可以使用 `${DICT.one}` 这样的属性访问方法来获取到字典的值. 当然, 这里要求键的名字必须是合法的Python属性名称, 如果其中包含了空格或者其他非法的名称, 则还是可以使用 `&{DICT}[with spaces]` 语法来获取字典的值. 这个生成的字典也是有序的, 不过遗憾的是, 原始的YAML文件中的顺序没法保留下来.

