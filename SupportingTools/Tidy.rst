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

Alternative execution
~~~~~~~~~~~~~~~~~~~~~

Although Tidy is used only with Python in the synopsis above, it works
also with Jython and IronPython. In the synopsis Tidy is executed as
an installed module (`python -m robot.tidy`), but it can be run also as
a script::

    python path/robot/tidy.py [options] arguments

Executing as a script can be useful if you have done `manual installation`_
or otherwise just have the :file:`robot` directory with the source code
somewhere in your system.

Output encoding
~~~~~~~~~~~~~~~

All output files are written using UTF-8 encoding. Outputs written to the
console use the current console encoding.

Cleaning up test data
---------------------

Test case files created with HTML editors or written by hand can be normalized
using Tidy. Tidy always writes consistent headers, consistent order for
settings, and consistent amount of whitespace between cells and tables.

Examples::

    python -m robot.tidy messed_up_tests.html cleaned_tests.html
    python -m robot.tidy --inplace tests.txt

Changing test data format
-------------------------

Robot Framework supports test data in HTML, TSV and TXT formats and Tidy
makes changing between the formats trivial. Input format is always determined
based on the extension of the input file. Output format can be set using
the :option:`--format` option, and the default value is got from the extension
of the possible output file.

Examples::

    python -m robot.tidy tests.html tests.txt
    python -m robot.tidy --format txt --inplace tests.html
    python -m robot.tidy --format tsv --recursive mytests
