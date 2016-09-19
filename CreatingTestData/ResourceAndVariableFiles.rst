.. Resource and variable files

资源文件和变量文件
===========================

在 `test case files`_ 和  `test suite initialization files`_ 中的用户关键字和变量只能在创建它们的文件中使用. *资源文件* 提供了共享机制.

资源文件的结构和用例文件非常接近, 可以很容易地创建.

*变量文件* 则提供的强大的创建和共享变量的功能. 可以创建非字符串类型的复杂变量, 以及动态地创建过程. 因为变量文件使用的是Python代码, 所以非常灵活. 当然, 相对于 `Variable tables`_ 来说, 这也稍稍带来了一定的复杂度.

.. contents::
   :depth: 2
   :local:

.. Resource files

资源文件
--------------

.. Taking resource files into use

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

.. Documenting resource files

资源文件的文档
~~~~~~~~~~~~~~~~~~~~~~~~~~

资源文件中创建的关键字可以使用 :setting:`[Documentation]` 来设置文档. 和 `测试套件`__ 类似, 资源文件本身在设置表格中也可以通过 :setting:`Documentation` 设置文档.

Libdoc_ 和 RIDE_ 都会用到这些文档, 并且这些文档内容很自然的对打开资源文件的人来说也是可见的. 

当关键字运行时, 关键字文档的第一行将写入日志, 而资源文件的文档在测试执行过程中会被忽略.

__ `User keyword name and documentation`_
__ `Test suite name and documentation`_

.. Example resource file

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

.. Variable files

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

.. Taking variable files into use

如何使用变量文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Setting table

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

.. Command line

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

.. Creating variables directly

直接创建变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Basic syntax

基础语法
''''''''''''

When variable files are taken into use, they are imported as Python
modules and all their global attributes that do not start with an
underscore (`_`) are considered to be variables. Because variable
names are case-insensitive, both lower- and upper-case names are
possible, but in general, capital letters are recommended for global
variables and attributes.

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

To make creating a list variable or a dictionary variable more explicit,
it is possible to prefix the variable name with `LIST__` or `DICT__`,
respectively:

.. sourcecode:: python

   from collections import OrderedDict

   LIST__ANIMALS = ["cat", "dog"]
   DICT__FINNISH = OrderedDict([("cat", "kissa"), ("dog", "koira")])

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

.. note:: Variables are not replaced in strings got from variable files.
          For example, `VAR = "an ${example}"` would create
          variable `${VAR}` with a literal string value
          `an ${example}` regardless would variable `${example}`
          exist or not.

Using objects as values
'''''''''''''''''''''''

Variables in variable files are not limited to having only strings or
other base types as values like variable tables. Instead, their
variables can contain any objects. In the example below, the variable
`${MAPPING}` contains a Java Hashtable with two values (this
example works only when running tests on Jython).

.. sourcecode:: python

    from java.util import Hashtable

    MAPPING = Hashtable()
    MAPPING.put("one", 1)
    MAPPING.put("two", 2)

The second example creates `${MAPPING}` as a Python dictionary
and also has two variables created from a custom object implemented in
the same file.

.. sourcecode:: python

    MAPPING = {'one': 1, 'two': 2}

    class MyObject:
        def __init__(self, name):
            self.name = name

    OBJ1 = MyObject('John')
    OBJ2 = MyObject('Jane')

Creating variables dynamically
''''''''''''''''''''''''''''''

Because variable files are created using a real programming language,
they can have dynamic logic for setting variables.

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

.. sourcecode:: python

    import math

    def get_area(diameter):
        radius = diameter / 2
        area = math.pi * radius * radius
        return area

    AREA1 = get_area(1)
    AREA2 = get_area(2)

Selecting which variables to include
''''''''''''''''''''''''''''''''''''

When Robot Framework processes variable files, all their attributes
that do not start with an underscore are expected to be
variables. This means that even functions or classes created in the
variable file or imported from elsewhere are considered variables. For
example, the last example would contain the variables `${math}`
and `${get_area}` in addition to `${AREA1}` and
`${AREA2}`.

Normally the extra variables do not cause problems, but they
could override some other variables and cause hard-to-debug
errors. One possibility to ignore other attributes is prefixing them
with an underscore:

.. sourcecode:: python

    import math as _math

    def _get_area(diameter):
        radius = diameter / 2.0
        area = _math.pi * radius * radius
        return area

    AREA1 = _get_area(1)
    AREA2 = _get_area(2)

If there is a large number of other attributes, instead of prefixing
them all, it is often easier to use a special attribute
`__all__` and give it a list of attribute names to be processed
as variables.

.. sourcecode:: python

    import math

    __all__ = ['AREA1', 'AREA2']

    def get_area(diameter):
        radius = diameter / 2.0
        area = math.pi * radius * radius
        return area

    AREA1 = get_area(1)
    AREA2 = get_area(2)

.. Note:: The `__all__` attribute is also, and originally, used
          by Python to decide which attributes to import
          when using the syntax `from modulename import *`.

Getting variables from a special function
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An alternative approach for getting variables is having a special
`get_variables` function (also camelCase syntax
`getVariables` is possible) in a variable file. If such a function
exists, Robot Framework calls it and expects to receive variables as
a Python dictionary or a Java `Map` with variable names as keys
and variable values as values. Created variables can be used as scalars,
lists, and dictionaries exactly like when `creating variables directly`_,
and it is possible to use `LIST__` and `DICT__` prefixes to make creating
list and dictionary variables more explicit. The example below is functionally
identical to the first `creating variables directly`_ example.

.. sourcecode:: python

    def get_variables():
        variables = {"VARIABLE ": "An example string",
                     "ANOTHER VARIABLE": "This is pretty easy!",
                     "INTEGER": 42,
                     "STRINGS": ["one", "two", "kolme", "four"],
                     "NUMBERS": [1, 42, 3.14],
                     "MAPPING": {"one": 1, "two": 2, "three": 3}}
        return variables

`get_variables` can also take arguments, which facilitates changing
what variables actually are created. Arguments to the function are set just
as any other arguments for a Python function. When `taking variable files
into use`_ in the test data, arguments are specified in cells after the path
to the variable file, and in the command line they are separated from the
path with a colon or a semicolon.

The dummy example below shows how to use arguments with variable files. In a
more realistic example, the argument could be a path to an external text file
or database where to read variables from.

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

Implementing variable file as Python or Java class
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting from Robot Framework 2.7, it is possible to implement variables files
as Python or Java classes.

Implementation
''''''''''''''

Because variable files are always imported using a file system path, creating
them as classes has some restrictions:

  - Python classes must have the same name as the module they are located.
  - Java classes must live in the default package.
  - Paths to Java classes must end with either :file:`.java` or :file:`.class`.
    The class file must exists in both cases.

Regardless the implementation language, the framework will create an instance
of the class using no arguments and variables will be gotten from the instance.
Similarly as with modules, variables can be defined as attributes directly
in the instance or gotten from a special `get_variables`
(or `getVariables`) method.

When variables are defined directly in an instance, all attributes containing
callable values are ignored to avoid creating variables from possible methods
the instance has. If you would actually need callable variables, you need
to use other approaches to create variable files.

Examples
''''''''

The first examples create variables from attributes using both Python and Java.
Both of them create variables `${VARIABLE}` and `@{LIST}` from class
attributes and `${ANOTHER VARIABLE}` from an instance attribute.

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

The second examples utilizes dynamic approach for getting variables. Both of
them create only one variable `${DYNAMIC VARIABLE}`.

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

Variable file as YAML
~~~~~~~~~~~~~~~~~~~~~

Variable files can also be implemented as `YAML <http://yaml.org>`_ files.
YAML is a data serialization language with a simple and human-friendly syntax.
The following example demonstrates a simple YAML file:

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

.. note:: Using YAML files with Robot Framework requires `PyYAML
          <http://pyyaml.org>`_ module to be installed. If you have
          pip_ installed, you can install it simply by running
          `pip install pyyaml`.

          YAML support is new in Robot Framework 2.9. Starting from
          version 2.9.2, the `standalone JAR distribution`_ has
          PyYAML included by default.

YAML variable files can be used exactly like normal variable files
from the command line using :option:`--variablefile` option, in the settings
table using :setting:`Variables` setting, and dynamically using the
:name:`Import Variables` keyword. The only thing to remember is that paths to
YAML files must always end with :file:`.yaml` extension.

If the above YAML file is imported, it will create exactly the same
variables as the following variable table:

.. sourcecode:: robotframework

   *** Variables ***
   ${STRING}     Hello, world!
   ${INTEGER}    ${42}
   @{LIST}       one         two
   &{DICT}       one=yksi    two=kaksi

YAML files used as variable files must always be mappings in the top level.
As the above example demonstrates, keys and values in the mapping become
variable names and values, respectively. Variable values can be any data
types supported by YAML syntax. If names or values contain non-ASCII
characters, YAML variables files must be UTF-8 encoded.

Mappings used as values are automatically converted to special dictionaries
that are used also when `creating dictionary variables`_ in the variable table.
Most importantly, values of these dictionaries are accessible as attributes
like `${DICT.one}`, assuming their names are valid as Python attribute names.
If the name contains spaces or is otherwise not a valid attribute name, it is
always possible to access dictionary values using syntax like
`&{DICT}[with spaces]` syntax. The created dictionaries are also ordered, but
unfortunately the original source order of in the YAML file is not preserved.
