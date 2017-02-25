.. _tidy:

.. Test data clean-up tool (Tidy)

测试数据整理工具(Tidy)
==============================

.. contents::
   :depth: 1
   :local:

Tidy是Robot Framework内置的用于整理和转换测试数据文件的工具.

工具执行结果的默认输出是标准输出流, 不过自从Robot Framework 2.7.5版本后可以将输出文件作为可选参数给出. 

如果使用了选项 :option:`--inplace` 或 :option:`--recursive`, 文件就会就地修改(也就是原文件会被整理后的文件就地覆盖).

.. General usage

用法
-------------

.. Synopsis

总览
~~~~~~~~

::

    python -m robot.tidy [options] inputfile
    python -m robot.tidy [options] inputfile [outputfile]
    python -m robot.tidy --inplace [options] inputfile [more input files]
    python -m robot.tidy --recursive [options] directory

.. Options

选项
~~~~~~~

 -i, --inplace    Tidy given file(s) so that original file(s) are overwritten
                  (or removed, if the format is changed). When this option is
                  used, it is possible to give multiple input files. Examples::

                      python -m robot.tidy --inplace tests.html
                      python -m robot.tidy --inplace --format txt *.html

 -r, --recursive  Process given directory recursively. Files in the directory
                  are processed in place similarly as when :option:`--inplace`
                  option is used.
 -f, --format <robot|txt|html|tsv>
                  Output file format. If the output file is given explicitly,
                  the default value is got from its extension. Otherwise
                  the format is not changed.
 -p, --use-pipes  Use a pipe character (|) as a cell separator in the txt format.
 -s, --spacecount <number>
                  The number of spaces between cells in the txt format.
                  New in Robot Framework 2.7.3.
 -l, --lineseparator <native|windows|unix>
                  Line separator to use in outputs. The default is 'native'.

                  - *native*: use operating system's native line separators
                  - *windows*: use Windows line separators (CRLF)
                  - *unix*: use Unix line separators (LF)

                  New in Robot Framework 2.7.6.
 -h, --help       Show this help.

.. Alternative execution

其它的执行方式
~~~~~~~~~~~~~~~~~~~~~

虽然Tidy在上面总览只用了Python来执行, 但是它还支持Jython和IronPython. 而且上面Tide是作为已安装的模块来执行的(`python -m robot.tidy`), 实际还可以当作脚本来执行::


    python path/robot/tidy.py [options] arguments


如果你是 `手动安装`_, 或者只是把 :file:`robot` 目录拷贝到系统某个目录时,  按脚本执行就会非常有用.

.. Output encoding

输出的编码
~~~~~~~~~~~~~~~

所有的输出文件都使用UTF-8编码写入, 控制台输出则使用当前控制台的编码.


.. Cleaning up test data

整理测试数据
---------------------

使用HTML编辑器或者手工编写的测试用例文件都可以使用Tidy来标准化. Tidy总是使用一致的题头, 一致顺序的settings, 以及在表格和单元格中间使用一致的空格.


一些例子::

    python -m robot.tidy messed_up_tests.html cleaned_tests.html
    python -m robot.tidy --inplace tests.txt

.. Changing test data format

转换测试数据格式
-------------------------

Robot Framework支持多种格式的测试数据, 包括HTML, TSV和TXT. Tidy可以轻松地在这些格式之间转换. 输入格式总是基于输入文件的扩展名来判断, 输出的格式则使用 :option:`--format` 选项来指定, 如果不指定该选项, 则默认的格式将从可能的输出文件扩展名来获取.


示例::

    python -m robot.tidy tests.html tests.txt
    python -m robot.tidy --format txt --inplace tests.html
    python -m robot.tidy --format tsv --recursive mytests
