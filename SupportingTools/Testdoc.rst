.. _testdoc:

.. Test data documentation tool (Testdoc)

测试数据文档工具(Testdoc)
======================================

.. contents::
   :depth: 1
   :local:

Testdoc是Robot Framework内置的工具, 用于从测试用例生成HTML文档. 生成的文档包含名称, 每个测试套件和测试用例的文档和元数据, 以及顶层关键字和它们的参数.

Testdoc is Robot Framework's built-in tool for generating high level
documentation based on test cases. The created documentation is in HTML
format and it includes name, documentation and other metadata of each
test suite and test case, as well as the top-level keywords and their
arguments.

.. General usage

用法
-------------

.. Synopsis

总览
~~~~~~~~

::

    python -m robot.testdoc [options] data_sources output_file

.. Options

选项
~~~~~~~

 -T, --title <title>           Set the title of the generated documentation.
                               Underscores in the title are converted to spaces.
                               The default title is the name of the top level suite.
 -N, --name <name>             Override the name of the top level test suite.
 -D, --doc <doc>               Override the documentation of the top level test suite.
 -M, --metadata <name:value>   Set/override free metadata of the top level test suite.
 -G, --settag <tag>            Set given tag(s) to all test cases.
 -t, --test <name>             Include tests by name.
 -s, --suite <name>            Include suites by name.
 -i, --include <tag>           Include tests by tags.
 -e, --exclude <tag>           Exclude tests by tags.
 -h, --help                    Print this help in the console.

除了 :option:`--title` 以外的所有选项的语义都和在 `执行测试用例`__ 时使用完全一样. 

All options except :option:`--title` have exactly the same semantics as same
options have when `executing test cases`__.

__ `Configuring execution`_

.. Generating documentation

生成文档
------------------------

生成文档的数据源可以是单个文件, 单个目录, 也可以是多个文件和目录. 所有这些情况, 最后那个参数都必须是最终文档输出要写入的文件.

Data can be given as a single file, directory, or as multiple files and
directories. In all these cases, the last argument must be the file where
to write the output.

Testdoc适用于Robot Framework支持的所有解释器(Python, Jython and IronPython). 它可以以python模块的方式执行, 如 `python -m robot.testdoc`, 也可以当作一个脚本来执行, 例如: `python path/robot/testdoc.py`.

一些例子::

  python -m robot.testdoc my_test.html testdoc.html
  jython -m robot.testdoc --name smoke_tests --include smoke path/to/my_tests smoke.html
  ipy path/to/robot/testdoc.py first_suite.txt second_suite.txt output.html
