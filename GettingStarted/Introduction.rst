介绍
====

Robot Framework 是一个基于Python的、可扩展的、关键字驱动的测试自动化框架，用于端到端的验收测试或者验收驱动测试开发（ATDD）中。


为什么选择Robot Framework
-------------------------

* 表格式的语法简单易用，以统一的方式 :ref:`创建测试用例 <creating test cases>`
* 可以通过现有关键字创建可复用的 :ref:`高层关键字 <creating user keywords>`
* 提供了直观的HTML格式的 :ref:`测试报告 <report file>` 和 :ref:`日志文件 <log file>`
* 作为一个测试平台，是应用无关的
* 提供了 :ref:`测试库API <creating test libraries>`，可以轻易地使用Python或者Java创建自定义的测试库
* 提供了 :ref:`命令行接口 <executing test cases>` 和基于XML的 :ref:`输出文件 <output file>`，可以与现有框架集成（如持续集成系统）
* 提供了多种测试库支持，如用于web测试的Selenium，Java GUI测试，启动进程，Telnet，SSH等
* 可以创建 :ref:`数据驱动的测试用例 <data-driven style>`
* 内置支持 :ref:`变量 <variables>`，在不同的环境中特别实用
* 提供 :ref:`标签 <tagging test cases>` 来分类和 :ref:`选择测试用例 <selecting test cases>`
* 非常容易与源码控制系统集成，因为 :ref:`测试套件 <creating test suites>` 就是文件夹和文本文件
* 提供了 :ref:`用例级别 <test setup and teardown>` 和 :ref:`测试套件级别 <suite setup and teardown>` 的setup和teardown
* 模块化的架构，支持针对不同接口的应用程序创建测试
  

整体架构
--------

Robot Framework是一个通用的，应用和技术无关的框架。它的高度模块化的架构如下图所示：

.. image:: architecture.png


测试数据(:ref:`test data <test data>`)使用非常简单、易于编辑的表格格式. Robot Framework会解析测试数据, :ref:`执行测试用例 <executing test cases>`, 并生成日志和报告. 框架本身对测试对象一无所知, 而是通过 :ref:`测试库 <creating test libraries>` 与其交互. 测试库可能是直接使用被测应用程序的接口, 也可以使用其它底层的测试工具作为驱动.


示例截图
--------

以下是 :ref:`测试数据 <test data>` 和 :ref:`测试报告 <report file>` 和 :ref:`测试日志 <log file>` 的截图:

.. figure:: testdata_screenshots.png
   :alt:	testdata_screenshots

   测试用例文件


.. figure:: screenshots.png
   :alt:	screenshots

   执行报告和日志



如何获取更多信息
------------------------

项目页面
~~~~~~~

获取Robot Framework更多权威资讯的首要地方当然是其官网, http://robotframework.org. 项目源码是托管在  `GitHub`_ 

.. _GitHub: https://github.com/robotframework/robotframework

.. _mailing lists:

邮件列表
~~~~~~~~

.. note:: 一般邮件列表都是使用英文交流, 能加入的同学肯定不需要翻译了, 这段偷懒略过.


There are several Robot Framework mailing lists where to ask and
search for more information. The mailing list archives are open for
everyone (including the search engines) and everyone can also join
these lists freely. Only list members can send mails, though, and to
prevent spam new users are moderated which means that it might take a
little time before your first message goes through.  Do not be afraid
to send question to mailing lists but remember `How To Ask Questions
The Smart Way`__.

robotframework-users__
   General discussion about all Robot Framework related
   issues. Questions and problems can be sent to this list. Used also
   for information sharing for all users.

robotframework-announce__
    An announcements-only mailing list where only moderators can send
    messages. All announcements are sent also to the
    robotframework-users mailing list so there is no need to join both
    lists.

robotframework-devel__
   Discussion about Robot Framework development.

__ http://www.catb.org/~esr/faqs/smart-questions.html
__ http://groups.google.com/group/robotframework-users
__ http://groups.google.com/group/robotframework-announce
__ http://groups.google.com/group/robotframework-devel