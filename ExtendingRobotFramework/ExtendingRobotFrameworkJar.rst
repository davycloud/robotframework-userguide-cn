.. Extending the Robot Framework Jar

扩展Robot Framework的Jar包
==========================

为Robot Framework的jar包添加额外的测试库或支撑代码很简单, 直接使用标准JDK安装包含的 ``jar`` 命令即可. Python代码必须位于jar文件内的 :file:`Lib` 路径内, Java代码可以按包结构直接置于jar的根路径.

例如, 要添加Python包 `mytestlib`, 首先将文件夹 :file:`mytestlib` 拷贝到 :file:`Lib` 文件夹之下, 然后在包含 :file:`Lib` 的文件夹中执行::

  jar uf /path/to/robotframework-2.7.1.jar Lib

要将编译后的java类添加到jar中, 必须将按Java包结构的整个目录添加到jar包内.

例如, 要添加 包 `org.test` 内的类 `MyLib.class`, 类文件路径必须是 :file:`org/test/MyLib.class`, 然后执行::

  jar uf /path/to/robotframework-2.7.1.jar org
