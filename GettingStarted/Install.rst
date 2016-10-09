安装指导
========

本文介绍如何在不同的操作系统安装和卸载Robot Frameworks.

如果你已经安装了 `pip <http://pip-installer.org>`_ ,那么只需如此::

   pip install robotframework


概述
------------

`Robot Framework <http://robotframework.org>`_ 是使用 `Python <http://python.org>`_ 开发实现, 
同时支持 `Jython <http://jython.org>`_ (JVM) 和 `IronPython <http://ironpython.net>`_ (.NET). 
显然, 这些python解释器至少需要安装其中之一.

安装Robot Framework的各种方法都会在下面列出并在随后的章节中详细说明.

`使用pip安装`_
   推荐方式. 作为Python标准包管理器, 最新的Python, Jython和IronPython版本里都自带了pip.
   如果你已经有了pip, 只需运行::

      pip install robotframework

`从源码安装`_ 
   从源码安装不关心是何种操作系统和使用何种Python解释器. 从 `PyPI <https://pypi.python.org/pypi/robotframework>`_
   下载源码, 或者从 `GitHub 代码仓库 <https://github.com/robotframework/robotframework>`_ clone源码.

`独立的JAR包安装`_
   如果使用Jython执行测试就可以, 则最简单的方式是下载独立的 ``robotframework-<version>.jar`` 文件,
   下载地址是 `Maven <http://search.maven.org/#search%7Cga%7C1%7Ca%3Arobotframework>`_.
   JAR包中包含了Jython和Robot Framework, 所以只需要系统安装了  `Java <http://java.com>`_  即可.

`手动安装`_
   如果有特殊需求其它方式无法满足的, 可以手动自定义安装.


前提条件
-------------

Robot Framework 除了可以在 Python_ (both Python 2 and Python 3), Jython_
(JVM) 和 IronPython_ (.NET) 上运行外, 还可以运行在 `PyPy <http://pypy.org>`_.
要使用的解释器必须在安装Robot Framework之前安装.

使用何种解释器一般取决于测试库和测试环境. 有些库使用的工具或模块只能在Python上运行.
而其它的库可能使用了一些Java相关的工具, 则需要Jython; 或者使用了.Net库则需要IronPython.
还有很多工具和库和这些解释器都兼容.

如果你没有特殊需求而只是想试试这个框架, 推荐使用Python, 毕竟这是最成熟的解释器, 而且一般
会比Jython或IronPython更快(特别是启动时间). 而且在大多数的类UNIX操作系统里都内置了.

Python 2 vs Python 3
^^^^^^^^^^^^^^^^^^^^

Python 2和Python 3虽然是同一种语言, 但是有诸多不兼容的地方. 
例如, Python 3中所有字符串都是Unicode, 而Python 2中缺省是bytes.
最新的Python 2发布版本是2010年发布的Python 2.7, 将支持到2020年.

关于两者之间更多的差别,以及如何编写兼容两个版本的代码,请参考  `Should I use Python 2 or 3?`__ 

Robot Framework 3.0 是首个支持Python 3的版本. 同时也继续支持Python 2, 并且计划在Python 2的
官方支持期内一直保持支持. 我们希望广大的测试库和工具开发者们也开始关注支持Python 3.

__ https://wiki.python.org/moin/Python2orPython3

Python的安装
^^^^^^^^^^^^

大多数的类UNIX系统如 ``Linux`` 和 ``OS X`` 默认都安装了Python.
Windows用户需要自行安装, 推荐访问Python的官网 http://python.org, 选择适合自己系统的安装文件.

Robot Framework 3.0 支持 Python 2.6, 2.7, 3.3 以及更新的版本, 不过计划 `在RF3.1版本中放弃支持 Python 2.6`__. 
如果你需要使用更老的系统, Robot Framework 2.5-2.8 支持 Python 2.5, Robot Framework 2.0-2.1 支持 Python 2.3 和 2.4.

在Windows下安装Python推荐使用管理员(administrator)安装并且选择安装给所有用户.
此外, 不能设置环境变量 ``PYTHONCASEOK`` 

.. note:: ``PYTHONCASEOK`` 如果被设置, Python在import module时不区分大小写.

安装完Python, 需要配置环境变量 ``PATH``, 

.. tip:: 最新的Python Windows安装文件在安装过程中,可以选择 `Add python.exe to Path`. 该选项默认没有勾选.



__ https://github.com/robotframework/robotframework/issues/2276

Jython的安装
^^^^^^^^^^^^

使用了Java实现的测试库或者内部使用了Java工具, 则需要在Jython上运行Robot Framework, 当然, 
这就需要Java运行环境(JRE)或者Java开发工具集(JDK). 这两者的安装超出了本文档的范围, 请参考官网
http://java.com

安装Jython则比较简单, 从 http://jython.org 下载安装包, 一个可执行的JAR包, 然后通过命令行运行
`java -jar jython_installer-<version>.jar`. 如果系统有所配置, 也可以双击安装.

Robot Framework 3.0 支持 Jython 2.7,需要 Java 7 或更新版本.
如果需要使用更老的 Jython 或 Java 版本, Robot Framework 2.5-2.8 支持
Jython 2.5 (需要 Java 5 或更新), Robot Framework 2.0-2.1 支持 Jython 2.2.

安装完Jython, 可能还需要配置环境变量PATH


IronPython的安装
^^^^^^^^^^^^^^^^

IronPython_ 使得Robot Framework可以运行在 `.NET平台 <http://www.microsoft.com/net>`_ .
与 C# 和其它 .NET 语言或APIs交互. 只支持 IronPython 2.7.

当使用IronPython, 需要安装一个依赖模块 `elementtree <http://effbot.org/downloads/#elementtree>`_ 
(1.2.7 preview release). 这是因为 ``elementtree`` 在IronPython中发布版无法工作(
`详见 <https://github.com/IronLanguages/main/issues/968>`__).
可以下载该模块的源码, 解压, 并在当前路径的命令行中运行 ``ipy setup.py install``


配置环境变量 ``PATH``
^^^^^^^^^^^^^^^^^^^^^^^^

``PATH`` 环境变量定义了一系列路径, 系统运行命令时默认从这些路径中搜索.
为了在命令行中更方便的使用 Robot Framework, 推荐将 运行脚本_ 所在的路径添加到 ``PATH``.

大多数的类UNIX系统中默认都自带了Python并且配置好了 ``PATH``, 无需额外操作.
对于Windows, 或其它解释器, ``PATH`` 必须单独配置.

.. tip:: 最新的Python Windows安装文件在安装过程中,可以选择 `Add python.exe to Path`. 
         该选项默认没有勾选. 但是如果勾选了, 将把 Python 安装路径和 ``Python脚本`` 路径
         都加入到 ``PATH``.

哪些路径要加入到 ``PATH``
"""""""""""""""""""""""""

到底要将哪些路径加入到 ``PATH`` 中取决于解释器和操作系统.
第一个路径是python解释器的安装路径(如: ``C:\\Python27``), 另一个则是该解释器的脚本安装路径.
Python和IronPython将脚本安装至安装路径的子目录 ``Scripts`` 中 (如: :file:`C:\\Python27\\Scripts`),
Jython则使用 ``bin`` 路径, (如: :file:`C:\\jython2.7.0\\bin`)

注意, ``Scripts`` 或 ``bin`` 路径在解释器安装完后不一定就立即创建, 会在随后安装Robot Framework
或者其他第三方模块时才创建.

在Windows系统中配置 ``PATH``
"""""""""""""""""""""""""""""

假设Python的安装路径是 ``C:\Python27``, 则脚本的路径是 ``C:\Python27\Scripts``

1. 打开设置环境变量的对话框(系统属性-高级-环境变量), 一般分为 `用户变量` 和 `系统变量`,
   两者的区别在于用户变量只影响当前用户, 而系统变量影响所有用户.
2. 找到 ``PATH`` 变量(注意,Windows系统中环境变量的名称不区分大小写, 也可能是 `Path`),
   选择 `编辑..`, 将 `;C:\Python27;C:\Python27\Scripts` 添加到值的末尾.
3. 保存
4. 开启命令行, 检查是否生效
   
注意, 如果安装有多个Python版本, 当执行 ``robot`` 或者 ``rebot`` 时, 将总是使用 *第一个* 出现在
``PATH`` 中的Python解释器, 而不管该脚本是安装在哪个版本下的. 为了避免这种情况, 可以直接执行
robot模块, 如 `C:\\Python27\\python.exe -m robot`.

在类UNIX系统中配置 ``PATH``
"""""""""""""""""""""""""""""

类UNIX系统一般通过编辑系统或者用户配置文件来配置环境变量, 具体的文件因系统而异.

配置 ``https_proxy``
^^^^^^^^^^^^^^^^^^^^^^

如果要 `使用pip安装`_ , 但是当前系统是通过代理上网, 则需要设置 ``https_proxy`` 环境变量.
具体的设置方式因系统而异, Windows系统一般是在Internet选项中设置, 其它类UNIX系统的设置
类似 `配置环境变量 PATH`_ . 代理服务器的地址需要咨询网络管理员, 一般是一个URL地址, 
如 `http://10.0.0.42:8080`.

使用pip安装
-------------------

Python标准包管理器是 pip_, 但还有其它的选择, 例如 `Buildout <http://buildout.org>`__ 
和 `easy_install <http://peak.telecommunity.com/DevCenter/EasyInstall>`__.
本章节只覆盖使用pip的情况, 其它包管理器应该也可以用来安装Robot Framework.

最新的Python, Jython和IronPython版本已经捆绑安装了pip.

Python下安装pip
^^^^^^^^^^^^^^^^^^^^

从2.7.9版本开始, Python的Windows安装包已经默认会安装并激活pip. 如果你已经设置完了环境变量,
那么就直接运行 `pip install robotframework`

Windows之外的系统, 或者比较老的Python版本, 需要自己安装pip. Linux系统可能使用系统的包管理器
比如 Apt 或者 Yum 安装, 当然也可以参考 pip_ 主页上的指导手册进行操作.

如果你安装了多个Python版本和pip, 当运行 ``pip`` 命令时, 实际执行的是最先出现在 PATH_ 定义中
的. 可以直接指定Python版本, 调用 ``pip`` 模块:

.. sourcecode:: bash

    python -m pip install robotframework
    python3 -m pip install robotframework

Jython下安装pip
^^^^^^^^^^^^^^^^^^^^

Jython 2.7 包含了 pip, 但是在使用前需要激活, 使用下面的命令:

.. sourcecode:: bash

    jython -m ensurepip

Jython的pip安装至 :file:`<JythonInstallation>/bin` 路径. 执行 `pip install robotframework` 
命令实际运行的是否是该路径下的, 同样取决于最先出现在 PATH_ 定义中的pip.
另一种方式是直接指定Jython版本, 调用 ``pip`` 模块:

.. sourcecode:: bash

    jython -m pip install robotframework

IronPython下安装pip
^^^^^^^^^^^^^^^^^^^^

IronPython从 `版本 2.7.5`__ 开始包含pip. 与Jython类似, 需要先激活:

.. sourcecode:: bash

    ipy -X:Frames -m ensurepip

注意 `-X:Frames` 命令行选项在激活和使用pip时都要带上.

IronPython将pip安装至 :file:`<IronPythonInstallation>/Scripts` 路径.
运行 `pip install robotframework` 实际调用的pip版本同样取决于最先出现
在 PATH_ 定义中的pip. 可以直接运行IronPython调用 ``pip`` 模块:

.. sourcecode:: bash

    ipy -X:Frames -m pip install robotframework

IronPython 早于 2.7.5 的版本官方并不支持 pip.

__ http://blog.ironpython.net/2014/12/pip-in-ironpython-275.html

使用pip
^^^^^^^^^^^^^^^^^^^^

安装完pip, 并设置完 https代理_ (如果需要的话), 使用pip是很简单的事情. 最
简单的办法就是直接使用pip, 让它自动从 `Python Package Index (PyPI)`__ 下载
并安装包.

__ PyPI_

.. sourcecode:: bash

    # Install the latest version
    pip install robotframework

    # Upgrade to the latest version
    pip install --upgrade robotframework

    # Install a specific version
    pip install robotframework==2.9.2

    # Install separately downloaded package (no network connection needed)
    pip install robotframework-3.0.tar.gz

    # Uninstall
    pip uninstall robotframework

注意, pip 1.4 及随后的版本默认只安装稳定版本. 如果你想要安装 alpha 或者 beta 或者
发布候选版本, 需要显示指定版本号, 或者使用 :option:`--pre` 选项:

.. sourcecode:: bash

    # Install 3.0 beta 1
    pip install robotframework==3.0b1

    # 更新到最新版本, 即使该版本是pre-release版本
    pip install --pre --upgrade robotframework

从源码安装
-------------------

从源码安装适用于任何操作系统上的任何Python解释器. 看上去从源码安装比较恐怖,
实际过程很简单直接.

获取源码
^^^^^^^^^^^^^^^^^^^^

源码一般就是下载 `.tar.gz` 格式的源代码打包文件. 最新的版本可以在  PyPI_ 下载.
2.8.1和以前的版本可以在 `Google Code <https://code.google.com/p/robotframework/downloads/list?can=1>`_ 下载.
将包下载到本地后, 先解压, 得到文件夹, 名称为 `robotframework-<version>`, 其中
包含所有源代码和安装所需要的脚本.

此外, 还可以通过 `GitHub 代码仓库`_ 克隆项目源码. 这样可以获取到最新的代码,
也可以切换到不同的发布版本或者标签.


安装
^^^^^^^^^^^^^^^^^^^^

Robot Framework 使用 Python 标准的 ``setup.py`` 脚本来执行安装. 该脚本就在源码
路径内, 使用命令行调用对应的python解释器执行:

.. sourcecode:: bash

   python setup.py install
   jython setup.py install
   ipy setup.py install

``setup.py`` 可以接受若干参数, 例如, 安装到其它不需要管理员权限的路径.
具体帮助请运行 `python setup.py --help` .

独立的JAR包安装
-------------------

Robot Framework 还提供了一个独立的Java包, 包含了 Jython_ 和 Robot Framework 版本.
这样就只需要 Java_ 环境即可. 这样做的好处是无需安装, 但是不足之处就是这样无法再使用
普通的 Python_ 解释器运行.

JAR包的名字是  ``robotframework-<version>.jar``, 可在 `Maven`_ 获取.
下载后, 可以直接按下面的方式运行测试:

.. sourcecode:: bash

  java -jar robotframework-3.0.jar mytests.robot
  java -jar robotframework-3.0.jar --variable name:value mytests.robot

如果想要使用 ``Rebot`` 运行 `输出结果处理`_, 或者其它内置的工具, 则需要将这些
命令名称, ``rebot``, ``libdoc``, ``testdoc`` 或 ``tidy`` 作为第一个参数传递:

.. sourcecode:: bash

  java -jar robotframework-3.0.jar rebot output.xml
  java -jar robotframework-3.0.jar libdoc MyLibrary list

获取帮助提示, 不带参数的执行该JAR包.

除了Python标准库和Robot Framework模块, JAR包版本从 2.9.2 开始还包含了 PyYAML
依赖包, 用来处理 ``yaml`` 格式的变量文件.

手动安装
-------------------

如果不想使用上述的自动安装方式, 可以手动安装将Robot Framework安装到自己指定的路径.
按照如下步骤:

1. 获取源码
2. 拷贝源码到需要的地方
3. 决定 怎样运行测试

检查安装结果
----------------------

成功安装后, 执行 `运行脚本`_ 并带上 :option:`--version` 选项, 如果如下所示可以获取到
Robot Framework的版本和Python解释器的版本, 则表示安装成功:

.. sourcecode:: bash

   $ robot --version
   Robot Framework 3.0 (Python 2.7.10 on linux2)

   $ rebot --version
   Rebot 3.0 (Python 2.7.10 on linux2)

如果运行失败并报"命令找不到"类似错误提示, 首先要去检查 PATH_ 的设置.

文件被安装到了哪里
^^^^^^^^^^^^^^^^^^^

当使用自动安装包时, Robot Framework源码会被拷贝到存放Python外部模块的路径下.
在类UNIX系统中, 如果是系统自带的Python, 这个路径的具体位置在不同操作系统中也
有所不同. 如果是自己安装的, 则这个路径一般是安装路径下面的 :file:`Lib/site-packages`.
例如,  :file:`C:\\Python27\\Lib\\site-packages`. 实际的Robot Framework
代码所在的文件夹名为 :file:`robot`.

Robot Framework `运行脚本`_ 则创建并被拷贝到另一个平台相关的位置. 在类UNIX
系统中, 一般是 :file:`/usr/bin` 或 :file:`/usr/local/bin`.在Windows中,
包括使用Jython 和 IronPython, 这个目录在解释器安装目录下的 :file:`Scripts`
或 :file:`bin` 目录.

卸载
--------------

最简单的卸载Robot Framework的方式也是使用 pip_:

.. sourcecode:: bash

   pip uninstall robotframework

即使是使用源码安装的包也可以使用pip删除. 如果没有pip或者是手动安装的,
则找到文件安装路径, 将其手动删除即可

升级
---------

如果使用 pip_, 升级到某个新版本可以带上  `--upgrade` 选项, 或者直接指定
这个版本号:

.. sourcecode:: bash

   pip install --upgrade robotframework
   pip install robotframework==2.9.2

使用pip会自动卸载旧版本, 然后再安装新版本. 如果使用源码安装方式, 则直接覆盖安装即可.
如果遇到问题, 先 卸载_, 再安装.

当升级 Robot Framework时, 有时新版本包含了向后不兼容的改动, 这些改动会影响现有的测试.
一般这种情况在小版本变化是很少发生,如 2.8.7 或 2.9.2, 但是在大版本变化时比较普遍, 如
2.9 和 3.0. 向后不兼容的改动和说明, 以及废弃的特性功能, 都在发布说明中会有解释. 所以,
在升级一个大版本前, 最好先仔细学习下发布说明.

运行 Robot Framework
-------------------------

使用 ``robot`` 和 ``rebot`` 脚本
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

从Robot Framework 3.0版本开始, 测试执行使用 ``robot`` 脚本, 运行结果处理使用
``rebot`` 脚本:

.. sourcecode:: bash

    robot tests.robot
    rebot output.xml

如果 PATH_ 设置正确, 这两个脚本都可以直接在命令行中运行. 它们除了在Windows中是
批处理文件, 其它系统都是使用的Python脚本实现.

老的Robot Framework版本不包含 ``robot`` 脚本, 同时 ``rebot`` 脚本也只在Python
解释器下安装. 对应于不同的解释器, 老版本中使用  ``pybot``, ``jybot`` 和 ``ipybot`` 
执行测试, 使用 ``jyrebot`` 和 ``ipyrebot`` 处理测试输出. 这些脚本现在仍能工作,
不过将在未来的版本中废弃并删除.

执行安装的 ``robot`` 模块
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

执行测试的另一种方式是使用Python的 `-m 命令行选项`__ 直接调用 ``robot`` 模块, 
或者子模块 ``robot.run``. 这种方法在同时使用多Python版本时非常有用.

.. sourcecode:: bash

   python -m robot tests.robot
   python3 -m robot.run tests.robot
   jython -m robot tests.robot
   /opt/jython/jython -m robot tests.robot

直接使用 ``python -m robot`` 是 Robot Framework 3.0 版本新增特性, 在老版本
中, 只支持  ``python -m robot.run``. 现在Python 2.6版本中仍然必须使用后者.

处理测试输出也是相同的办法, 只是模块是 ``robot.rebot``:

.. sourcecode:: bash

   python -m robot.rebot output.xml

__ https://docs.python.org/2/using/cmdline.html#cmdoption-m


执行安装的 ``robot`` 目录
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你知道Robot Framework安装到了哪里, 还可以直接运行 :file:`robot` 路径
或者其中的文件 :file:`run.py`, 执行方法是:

.. sourcecode:: bash

   python path/to/robot/ tests.robot
   jython path/to/robot/run.py tests.robot

直接运行路径是 Robot Framework 3.0 版本新增特性, 在老版本中, 只支持运行
:file:`robot/run.py` 文件.

理测试输出也是相同的办法, 只是文件变为 :file:`robot/rebot.py`:

.. sourcecode:: bash

   python path/to/robot/rebot.py output.xml

这种方式在 `手动安装`_ 时特别有用.

.. _PATH: `配置环境变量 PATH`_
.. _https代理: `配置 https_proxy`_
.. _运行脚本: `使用 robot 和 rebot 脚本`_
.. _输出结果处理: `使用 robot 和 rebot 脚本`_