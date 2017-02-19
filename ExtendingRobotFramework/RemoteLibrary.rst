远程测试库接口
========================

远程测试库接口使得测试库可以运行在Robot Framework所在之外的机器上, 并且也使测试库的实现语言不再局限于原生支持的Python和Java. 对于测试库的用户来说, 远程库和使用其它测试库基本雷同. 使用远程测试库接口来开发测试库也和创建 `普通测试库`__ 大同小异.

__ `Creating test libraries`_

.. contents::
   :depth: 2
   :local:

.. Introduction

介绍
------------

使用远程测试库API的两大原因:

* 是实际测试库和Robot Framework运行在不同的机器上, 可以实现分布式的测试.

* 远程测试库可以使用支持 `XML-RPC`_ 远程协议的任意编程语言来实现. 
  对于大部分主流编程语言如Python, Java, Ruby, .NET, Clojure, Perl 和 node.js, 当前已经存在了 `现成可用的远程服务`__ 实现.

远程库接口由 `standard libraries`_ 之一 `Remote` 库提供. Remote库自身不提供任何关键字, 只是作为一个代理存在于测试框架和在别处实现的关键字之间. Remote库通过远程服务器和实际的库进行交互, 它们之间的通信是基于XML-RPC的一种简单的 `remote protocol`_. 

整体的结构参见下图:

.. figure:: src/ExtendingRobotFramework/remote.png

   Robot Framework architecture with Remote library


.. note:: Remote库的远程客户端使用的是Python标准库 xmlrpclib__ . 不支持某些XML-RPC
          服务器实现的自定义扩展.

__ https://code.google.com/p/robotframework/wiki/RemoteLibrary#Available_remote_servers
__ http://docs.python.org/2/library/xmlrpclib.html

.. Taking Remote library into use

使用Remote库
------------------------------

.. Importing Remote library

导入Remote库
~~~~~~~~~~~~~~~~~~~~~~~~

Remote库需要指定远程服务器的地址, 其它使用关键字的方式和别的测试库并无二致. 如果在一个测试套件中用到多次Remote库, 或者只是想要取个描述性的名字, 都可以在import时使用 `WITH NAME syntax`_.

.. sourcecode:: robotframework

   *** Settings ***
   Library    Remote    http://127.0.0.1:8270       WITH NAME    Example1
   Library    Remote    http://example.com:8080/    WITH NAME    Example2
   Library    Remote    http://10.0.0.2/example    1 minute    WITH NAME    Example3

如果没有提供地址, 则上面第一个例子中的URL将是Remote库所用默认值. 类似地, 如果没有指定端口, 则默认使用端口号8270. (82和70在ASCII码中分别对应字母R和F)

.. note:: 当在本机使用时, 推荐使用地址 `127.0.0.1`, 避免使用 `localhost`.
          因为在Windows上, 地址解析可能会非常的慢, 见 `问题`__.
          在2.8.4版本之前的Robot Framework中Remote库默认地址用的是`localhost`

.. note:: 值得注意的是, 如果远程服务器地址的后面没有路径, 
          Remote库所使用的 `xmlrpclib module`__ 会默认使用 `/RPC2` 作为路径. 也就是说, 实际使用中 `http://127.0.0.1:8270` 等价于
          `http://127.0.0.1:8270/RPC2`. 这是否会带来问题取决于远程服务器的实现.
          如果指定了路径, 哪怕只有一个 `/`, 则后面不会再自动追加. 例如,
          `http://127.0.0.1:8270/` 和 `http://127.0.0.1:8270/my/path` 都将原封不动.

上面的最后一个例子展示了如何设置超时时间(timeout). timeout是Remote库的第2个可选参数. 超时时长用于初始连接服务器, 或者中途连接中断的情形. 超时时间的格式是Robot Framework 的 `time format`_ ,如 `60s` 或 `2 minutes 10 seconds`.

默认的超时值取决于操作系统和其配置, 一般是几分钟. 注意如果超时时长短于关键字的执行时间, 则关键字会被中断.

.. note:: Robot Framework 2.8.6版本开始支持timeout参数.
          Python/Jython 2.5 或 IronPython中timeout不生效.

__ http://stackoverflow.com/questions/14504450/pythons-xmlrpc-extremely-slow-one-second-per-call
__ https://docs.python.org/2/library/xmlrpclib.html

.. Starting and stopping remote servers

远程服务器的启停
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在导入Remote库之前, 提供关键字的远程服务器必须先启动. 如果服务器在测试执行之前启动, 则可以像上面的例子一样使用普通的 :setting:`Library` 来设置. 此外, 服务器还可以通过其它关键字(例如, Process_ or SSH__ )来启动, 这种情况下需要使用 `库导入关键字`__, 否则远程库将不可用.

远程服务器如何停止取决于它的实现方式. 典型的服务器支持以下方法:

* 不管使用什么库, 远程服务器应该提供 :name:`Stop Remote Server` 关键字, 
  这样可以很容易地在测试执行中停止服务.
* 远程服务器应该在XML-RPC接口中提供 `stop_remote_server` 方法.
* 在运行服务器程序的终端(console)按下 `Ctrl-C` 可以停止服务.
* 可以通过终止操作系统的进程(例如, ``kill``)来停止服务.

* Regardless of the library used, remote servers should provide :name:`Stop
  Remote Server` keyword that can be easily used by executed tests.
* Remote servers should have `stop_remote_server` method in their
  XML-RPC interface.
* Hitting `Ctrl-C` on the console where the server is running should
  stop the server.
* The server process can be terminated using tools provided by the
  operating system (e.g. ``kill``).

.. note:: :name:`Stop Remote Server` 关键字或 `stop_remote_server` 方法
          都不是必需的.

__ https://github.com/robotframework/SSHLibrary
__ `Using Import Library keyword`_

.. Supported argument and return value types

支持的参数和返回值类型
-----------------------------------------

由于XML-RPC协议并不支持所有可能的对象类型, 所以在Remote库和远程服务器之间传递的值必须要转换为某种兼容的类型. 这种转换适用于关键字的参数传递(Remote库->远程服务器)以及返回值(远程服务器->Remote库).

Because the XML-RPC protocol does not support all possible object
types, the values transferred between the Remote library and remote
servers must be converted to compatible types. This applies to the
keyword arguments the Remote library passes to remote servers and to
the return values servers give back to the Remote library.

Remote库和Python的远程服务器遵从下面的规则来处理Python值. 其它远程服务器的处理方式也是类似的.


* 字符串, 数字, 布尔值都无需修改, 直接传递.

* Python `None` 转换为空字符串.

* 所有的列表, 元组和其它可迭代对象(除了字符串和字典)都以列表(list)来传递, 其中的内容都是递归处理. 

* 字典和其它映射(mappings)作为字典传递, 其中的键转换为字符串, 值按支持类型转换, 同样递归处理.

* 返回的字典转换为一种所谓的 *可通过点号访问的字典*, 这种字典使用 `extended variable syntax`_ 来访问键值, 如 `${result.key}`.  嵌套的字典 `${root.child.leaf}`.

* 如果字符串包含了XML中不能表示的ASCII字节(例如, 空字节), 则将以 `Binary objects`__  传递, 内部使用的是XML-RPC base64数据类型. 接收到的Binary objects会自动转换回字符串. 

* 其它类型转换为字符串.

.. note:: 在Robot Framework 2.8.3版本之前, 只有列表, 元组和字典可以按上述规则处理.
          其它迭代器和映射对象并不支持. 此外, 支持binary是Robot Framework 2.8.4版本功能, 返回点号返回的字典是Robot Framework 2.9新增功能.

__ http://docs.python.org/2/library/xmlrpclib.html#binary-objects

.. Remote protocol

远程协议
---------------

本节介绍Remote库和远程服务器之间的通信协议. 这部分信息主要针对的是要创建远程服务的人. 现成的Python和Ruby服务器都可以用作示例.

该远程协议是基于 `XML-RPC`_ 实现的, XML-RPC是通过HTTP传递XML来实现的一个简单的远程过程调用(remote procedure call)协议.
大部分主流的编程语言(Python, Java, C, Ruby, Perl, Javascript, PHP,...)都内置或者可通过扩展来支持XML-RPC.


.. Required methods

必需的方法
~~~~~~~~~~~~~~~~

一个远程服务器就是一个XML-RPC服务器, 其公共的接口提供了 `动态库API`_ 所要求的方法. 其中`get_keyword_names` 和 `run_keyword` 是必须的,  `get_keyword_arguments` 和 `get_keyword_documentation` 为推荐实现. 注意这些方法名现在还不支持使用驼峰命名法. 

实际的关键字是如何实现的和Remote库并无关联. 远程服务器既可以仅充当一个真正测试库的包装器(wrapper), 就像提供的Python和Ruby服务器那样, 也可以自己实现关键字.

远程服务器可以在公共接口中额外提供 `stop_remote_server` 方法, 以便停止服务. 最好还可以将此方法自动暴露为关键字 :name:`Stop Remote Server`,以便在测试用例中使用. 允许用户停止服务并不总是合适的, 所以服务器最好还要提供某种方法来控制此功能. 例如, 提供一个方法(并暴露为关键字), 返回 `True` 或者 `False` 来标示该服务器是否允许被终止. 这样外部的工具也有办法知道停止服务器是否成功.

.. 不太明白这个方法的必要性


提供的Python远程服务器可作为一个实现的参考.

.. Getting remote keyword names and other information

获取远程关键字名称和其它信息
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remote库通过调用 `get_keyword_names` 方法来从远程服务器上获取其提供的关键字列表. 该方法必须将关键字名称以字符串的列表形式返回.

远程服务器可以, 也应该, 实现 `get_keyword_arguments` 和 `get_keyword_documentation` 方法来提供关于关键字更多的信息. 这两个方法都接受关键字的名称作为参数. 关键字的参数必须以字符串的列表返回, 其中格式和 `动态库的定义`__ 一样, 而关键字的文档则必须以 `字符串`__ 返回.

远程服务器还可以提供 `general library documentation`__ , 供文档生成工具 Libdoc_ 使用.

__ `Getting keyword arguments`_
__ `Getting keyword documentation`_
__ `Getting general library documentation`_

.. Executing remote keywords

远程关键字的执行
~~~~~~~~~~~~~~~~~~~~~~~~~

当Remote库想要远程服务器执行某个关键字时, 调用远程服务器的的 `run_keyword` 方法, 并传入关键字的名字, 一系列参数, 还可能有字典表示的 `自由命名参数`__. 基础类型参数可以直接使用, 复杂的类型 `被转换为支持的类型`__.

远程服务器的执行结果必须以字典形式返回, 其中包含的项参见下表. 注意, 其中只有 `status` 字段是必须的, 其它如果用不上的都可忽略.

.. table:: Entries in the remote result dictionary
   :class: tabular

   +------------+-------------------------------------------------------------+
   |     Name   |                         Explanation                         |
   +============+=============================================================+
   | status     | Mandatory execution status. Either PASS or FAIL.            |
   +------------+-------------------------------------------------------------+
   | output     | Possible output to write into the log file. Must be given   |
   |            | as a single string but can contain multiple messages and    |
   |            | different `log levels`__ in format `*INFO* First            |
   |            | message\n*HTML* <b>2nd</b>\n*WARN* Another message`. It     |
   |            | is also possible to embed timestamps_ to the log messages   |
   |            | like `*INFO:1308435758660* Message with timestamp`.         |
   +------------+-------------------------------------------------------------+
   | return     | Possible return value. Must be one of the `supported        |
   |            | types`__.                                                   |
   +------------+-------------------------------------------------------------+
   | error      | Possible error message. Used only when the execution fails. |
   +------------+-------------------------------------------------------------+
   | traceback  | Possible stack trace to `write into the log file`__ using   |
   |            | DEBUG level when the execution fails.                       |
   +------------+-------------------------------------------------------------+
   | continuable| When set to `True`, or any value considered                 |
   |            | `True` in Python, the occurred failure is considered        |
   |            | continuable__. New in Robot Framework 2.8.4.                |
   +------------+-------------------------------------------------------------+
   | fatal      | Like `continuable`, but denotes that the occurred           |
   |            | failure is fatal__. Also new in Robot Framework 2.8.4.      |
   +------------+-------------------------------------------------------------+

__ `Different argument syntaxes`_
__ `Supported argument and return value types`_
__ `Logging information`_
__ `Supported argument and return value types`_
__ `Reporting keyword status`_
__ `Continue on failure`_
__ `Stopping test execution gracefully`_

.. Different argument syntaxes

参数语法
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remote库是一个 `动态库`_, 因此它和其它动态库一样, `遵从相同的规则`__, 处理不同的参数语法. 包括必填参数, 默认值, varargs, 以及 `命名参数语法`__.

自由命名参数(`**kwargs`)主要也和`其它动态库一样`__. 首先, `get_keyword_arguments` 需要返回参数列表的规范, 其中必须包含 `**kwargs`, 这点和其它动态库是一样的. 主要的不同在于远程服务器的 `run_keyword` 方法必须包含一个可选的第3个参数, 用来接收用户指定的kwargs. 为了向后兼容, 这个参数必须设置为可选的(optional), 因为Remote库只在测试数据用到时才会传递kwargs到  `run_keyword` 方法.

实际上 `run_keyword` 和下面的Python和Java示例看上去差不多, 差别在于程序语言是怎么处理可选参数的.

.. sourcecode:: python

    def run_keyword(name, args, kwargs=None):
        # ...


.. sourcecode:: java

    public Map run_keyword(String name, List args) {
        // ...
    }

    public Map run_keyword(String name, List args, Map kwargs) {
        // ...
    }

.. note:: Remote library supports `**kwargs` starting from
          Robot Framework 2.8.3.

__ `Getting keyword arguments`_
__ `Named argument syntax with dynamic libraries`_
__ `Free keyword arguments with dynamic libraries`_
