.. All available settings in test data

可用配置项
========================

.. contents::
   :depth: 2
   :local:

.. Setting table

配置表
-------------

Setting表格在导入测试库, 资源文件和变量文件时有用, 同时也可为测试套件和测试用例定义元数据. 该表可存在于测试用例文件和资源文件中. 注意, 在资源文件中, Setting表只能包含导入(测试库, 资源文件和变量文件)相关的配置项.

The Setting table is used to import test libraries, resource files and
variable files and to define metadata for test suites and test
cases. It can be included in test case files and resource files. Note
that in a resource file, a Setting table can only include settings for
importing libraries, resources, and variables.

.. table:: Settings available in the Setting table
   :class: tabular

   +-----------------+--------------------------------------------------------+
   |       Name      |                         Description                    |
   +=================+========================================================+
   | Library         | Used for `importing libraries`_.                       |
   +-----------------+--------------------------------------------------------+
   | Resource        | Used for `taking resource files into use`_.            |
   +-----------------+--------------------------------------------------------+
   | Variables       | Used for `taking variable files into use`_.            |
   +-----------------+--------------------------------------------------------+
   | Documentation   | Used for specifying a `test suite`__ or                |
   |                 | `resource file`__ documentation.                       |
   +-----------------+--------------------------------------------------------+
   | Metadata        | Used for setting `free test suite metadata`_.          |
   +-----------------+--------------------------------------------------------+
   | Suite Setup     | Used for specifying the `suite setup`_.                |
   +-----------------+--------------------------------------------------------+
   | Suite Teardown  | Used for specifying the `suite teardown`_.             |
   +-----------------+--------------------------------------------------------+
   | Force Tags      | Used for specifying forced values for tags when        |
   |                 | `tagging test cases`_.                                 |
   +-----------------+--------------------------------------------------------+
   | Default Tags    | Used for specifying default values for tags when       |
   |                 | `tagging test cases`_.                                 |
   +-----------------+--------------------------------------------------------+
   | Test Setup      | Used for specifying a default `test setup`_.           |
   +-----------------+--------------------------------------------------------+
   | Test Teardown   | Used for specifying a default `test teardown`_.        |
   +-----------------+--------------------------------------------------------+
   | Test Template   | Used for specifying a default `template keyword`_      |
   |                 | for test cases.                                        |
   +-----------------+--------------------------------------------------------+
   | Test Timeout    | Used for specifying a default `test case timeout`_.    |
   +-----------------+--------------------------------------------------------+

.. note:: 所有的配置名字都可在结尾使用冒号, 例如: :setting:`Documentation:`.
          这个冒号是可选的, 主要是为了让配置更可读, 特别是在使用纯文本格式时.

__ `Test suite documentation`_
__ `Documenting resource files`_

.. Test Case table

测试用例表
---------------

测试用例表中的配置总是针对的当前特定的测试用例. 其中的某些配置将会覆盖在Settings表中定义的默认值.

.. table:: Settings available in the Test Case table
   :class: tabular

   +-----------------+--------------------------------------------------------+
   |      Name       |                         Description                    |
   +=================+========================================================+
   | [Documentation] | Used for specifying a `test case documentation`_.      |
   +-----------------+--------------------------------------------------------+
   | [Tags]          | Used for `tagging test cases`_.                        |
   +-----------------+--------------------------------------------------------+
   | [Setup]         | Used for specifying a `test setup`_.                   |
   +-----------------+--------------------------------------------------------+
   | [Teardown]      | Used for specifying a `test teardown`_.                |
   +-----------------+--------------------------------------------------------+
   | [Template]      | Used for specifying a `template keyword`_.             |
   +-----------------+--------------------------------------------------------+
   | [Timeout]       | Used for specifying a `test case timeout`_.            |
   +-----------------+--------------------------------------------------------+

.. Keyword table

关键字表
-------------

关键字表中的配置是针对的当前特定的用户关键字.

.. table:: Settings available in the Keyword table
   :class: tabular

   +-----------------+--------------------------------------------------------+
   |      Name       |                         Description                    |
   +=================+========================================================+
   | [Documentation] | Used for specifying a `user keyword documentation`_.   |
   +-----------------+--------------------------------------------------------+
   | [Tags]          | Used for specifying `user keyword tags`_.              |
   +-----------------+--------------------------------------------------------+
   | [Arguments]     | Used for specifying `user keyword arguments`_.         |
   +-----------------+--------------------------------------------------------+
   | [Return]        | Used for specifying `user keyword return values`_.     |
   +-----------------+--------------------------------------------------------+
   | [Teardown]      | Used for specifying `user keyword teardown`_.          |
   +-----------------+--------------------------------------------------------+
   | [Timeout]       | Used for specifying a `user keyword timeout`_.         |
   +-----------------+--------------------------------------------------------+
