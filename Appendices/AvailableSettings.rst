.. All available settings in test data

可用配置项
==========

.. contents::
   :depth: 2
   :local:

.. _setting table:

配置表
------

Setting表格是用来为测试套件和测试用例导入测试库, 资源文件和变量文件, 同时也给它们定义元数据. 该表可存在于测试用例文件和资源文件中.

注意, 在资源文件中, Setting表只能包含导入(测试库, 资源文件和变量文件)相关的配置项.


.. table:: Settings available in the Setting table
   :class: tabular

   +-----------------+--------------------------------------------------------+
   |       Name      |                         Description                    |
   +=================+========================================================+
   | Library         | 用来 :ref:`importing libraries`.                       |
   +-----------------+--------------------------------------------------------+
   | Resource        | 用来 :ref:`taking resource files into use`.            |
   +-----------------+--------------------------------------------------------+
   | Variables       | 用来 :ref:`taking variable files into use`.            |
   +-----------------+--------------------------------------------------------+
   | Documentation   | 用来为 :ref:`测试套件 <test suite documentation>`      |
   |                 | :ref:`资源文件 <documenting resource files>` 设置文档. |
   +-----------------+--------------------------------------------------------+
   | Metadata        | 用来设置 :ref:`free test suite metadata`.              |
   +-----------------+--------------------------------------------------------+
   | Suite Setup     | 用来指定 :ref:`suite setup <suite setup>`.             |
   +-----------------+--------------------------------------------------------+
   | Suite Teardown  | 用来指定 :ref:`suite teardown <suite setup>`.          |
   +-----------------+--------------------------------------------------------+
   | Force Tags      | Used for specifying forced values for tags when        |
   |                 | :ref:`tagging test cases`.                             |
   +-----------------+--------------------------------------------------------+
   | Default Tags    | Used for specifying default values for tags when       |
   |                 | :ref:`tagging test cases`.                             |
   +-----------------+--------------------------------------------------------+
   | Test Setup      | 用来指定一个缺省的 :ref:`test setup <test setup>`.     |
   +-----------------+--------------------------------------------------------+
   | Test Teardown   | 用来指定一个缺省的 :ref:`test teardown <test setup>`.  |
   +-----------------+--------------------------------------------------------+
   | Test Template   | 用来为测试用例指定一个缺省的 :ref:`template keyword`   |
   +-----------------+--------------------------------------------------------+
   | Test Timeout    | 用来指定一个缺省的 :ref:`test case timeout`.           |
   +-----------------+--------------------------------------------------------+

.. note:: 所有的配置名字都可在结尾使用冒号, 例如: :setting:`Documentation:`.
          这个冒号是可选的, 主要是为了让配置更可读, 特别是在使用纯文本格式时.

.. Test Case table

测试用例表
----------

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
--------

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
