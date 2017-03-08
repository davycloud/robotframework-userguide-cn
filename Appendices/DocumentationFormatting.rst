.. _documentation syntax:

.. Documentation formatting

文档格式
========================

在所有文档出现的地方, 包括 `测试套件`__, `测试用例`__, `用户关键字`__, `测试套件的元数据`__, 以及 `测试库文档`__ 中, 都可以使用简单的HTML格式.
格式和大多数wiki用到的样式类似, 被设计成不管是纯文本还是转换成HTML都很易懂.

__ `test suite documentation`_
__ `test case documentation`_
__ `user keyword documentation`_
__ `free test suite metadata`_
__ `Documenting libraries`_

.. contents::
   :depth: 2
   :local:

.. Representing newlines

如何表示换行
---------------------

.. _newlines in test data:

测试数据中的换行
~~~~~~~~~~~~~~~~

对所有的文档, 换行都可以通过手动添加 `字面换行符`__` (`\n`)来实现.


__ `Handling whitespace`_

.. sourcecode:: robotframework

  *** Settings ***
  Documentation    First line.\n\nSecond paragraph, this time\nwith multiple lines.
  Metadata         Example    Value\nin two lines

手动添加换行符对于长篇文档来说是个负担, 并且额外的字符也使得文档内容难以阅读. 从Robot Framework 2.7版本开始, `连续的文档和元数据行`__ 中间会自动插入换行. 也就是说, 上面的例子还可以写成下面这种情况:

.. sourcecode:: robotframework

  *** Settings ***
  Documentation
  ...    First line.
  ...
  ...    Second paragraph, this time
  ...    with multiple lines.
  Metadata
  ...    Example
  ...    Value
  ...    in two lines

如果一行已经以字面换行符结尾了, 或者以一个 `转义的反斜杠`__ 结尾, 则不会自动添加换行. 同一行中多列文字则使用空格拼接起来. 这种方式的拆分对 `HTML格式`_ 比较有用, 因为HTML表格的列一般比较窄. 

下面的例子使用不同的方法来分割文档, 所有3个用例的文档最终效果都是一样的, 都包含了两行文档.

__ `Dividing test data to several rows`_
__ Escaping_

.. sourcecode:: robotframework

  *** Test Cases ***
   Example 1
       [Documentation]    First line\n    Second line in    multiple parts
       No Operation

   Example 2
       [Documentation]   First line
       ...               Second line in    multiple parts
       No Operation

   Example 3
       [Documentation]    First line\n
       ...                Second line in\
       ...                multiple parts
       No Operation

.. Documentation in test libraries

测试库的文档
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

测试库的文档使用正常的换行就可以了.

下面的例子中, 关键字的文档最终效果和上面的例子是一样的.

.. sourcecode:: python

  def example_keyword():
      """First line.

      Second paragraph, this time
      with multiple lines.
      """
      pass


.. Paragraphs

段落
----------

从Robot Framework 2.7.2版本开始, 格式化后的HTML文档里, 所有的普通文本都表示为段落. 实际上, 不管是手动还是自动换行都会造成分段. 多个段落之间可以是空行, 也可以是其它的文本块, 比如表格, 列表等.

例如, 下面的测试套件或资源文件文档:

.. sourcecode:: robotframework

  *** Settings ***
  Documentation
  ...    First paragraph has only one line.
  ...
  ...    Second paragraph, this time created
  ...    with multiple lines.

转为HTML格式后:

.. raw:: html

  <div class="doc">
  <p>First paragraph has only one line.</p>
  <p>Second paragraph, this time created with multiple lines.</p>
  </div>

.. note:: 2.7.2版本之前的段落处理并不一致. Libdoc_ 生成的文档是段落组成的, 但是
          日志和报告里面的文档不是.


.. Inline styles

行内样式
-------------

文档语法支持的行内样式包括: **粗体**, *斜体* and `代码`. 粗体文字是用星号把一个或多个单词包住, 例如 `*this is bold*`. 类似地, 斜体是使用下划线, 例如 `_italic_`. 两者组合可以生成粗斜体 `_*bold italic*_`.

行内的代码使用双反引号 :codesc:`\`\`code\`\``. 其效果是生成淡灰色背景加等宽字体. 代码样式是在2.8.6版本新加的功能.

星号, 下划线, 双反引号这些字符如果单独出现, 或者出现在文字中间, 则不会起作用, 不过如果前后出现的是标点符号, 则不受影响. 当 段落__ 中有多行, 行内样式可以跨越多行.


__ paragraphs_

.. raw:: html

   <table class="tabular docutils">
     <caption>Inline style examples</caption>
     <tr>
       <th>Unformatted</th>
       <th>Formatted</th>
     </tr>
     <tr>
       <td>*bold*</td>
       <td><b>bold</b></td>
     </tr>
     <tr>
       <td>_italic_</td>
       <td><i>italic</i></td>
     </tr>
     <tr>
       <td>_*bold italic*_</td>
       <td><i><b>bold italic</b></i></td>
     </tr>
     <tr>
       <td>``code``</td>
       <td><code>code</code></td>
     </tr>
     <tr>
       <td>*bold*, then _italic_ and finally ``some code``</td>
       <td><b>bold</b>, then <i>italic</i> and finally <code>some code</code></td>
     </tr>
     <tr>
       <td>This is *bold\n<br>on multiple\n<br>lines*.</td>
       <td>This is <b>bold</b><br><b>on multiple</b><br><b>lines</b>.</td>
     </tr>
   </table>

URLs
----

所有看起来像URL的字符串都会自动转换为可点击的链接. 此外, 如果URL以图片类扩展名如 :file:`.jpg`, :file:`.jpeg`, :file:`.png`, :file:`.gif` 或 :file:`.bmp` (大小写无关) 结尾, 则将自动创建图片. 

例如, 网址 `http://example.com` 转为链接, `http:///host/image.jpg` 和 `file:///path/chart.png` 则转为图片链接.

URL的自动转换对日志和报告内的所有数据都启用, 但是创建图片只对测试套件文档, 测试用例和关键字文档, 以及测试套件的元数据起作用.

.. Custom links and images

自定义链接和图片
-----------------------

从Robot Framework 2.7版本开始, 可以通过一个特殊语法来创建自定义的链接和嵌入图片, 语法格式为 `[link|content]`. 最终生成的效果取决于  `link` 和 `content`.

是否是图片同样是通过文件扩展名来判断, 和 URLs_ 中一样. 不管什么情况, 该语法中的方括号和中间的管道符都是必需的.

.. Link with text content

带文本内容的链接
~~~~~~~~~~~~~~~~~~~~~~

如果不管  `link` 或 `content` 都不是图片, 则结果会生成一个普通的链接, 其中 `link`  是链接目标, 而 `content` 是显示文本::


    [file.html|this file] -> <a href="file.html">this file</a>
    [http://host|that host] -> <a href="http://host">that host</a>

.. Link with image content

带图片的链接
~~~~~~~~~~~~~~~~~~~~~~~

如果 `content` 是图片, 则生成的链接显示的内容是图片. 而链接的目标由 `link` 决定, 可能是普通的文本链接, 也可能是图片::

    [robot.html|robot.png] -> <a href="robot.html"><img src="robot.png"></a>
    [image.jpg|thumb.jpg] -> <a href="image.jpg"><img src="thumb.jpg"></a>

.. Image with title text

带title文本的图片
~~~~~~~~~~~~~~~~~~~~~

如果 `link` 是图片, 而 `content` 不是, 则生成的结果是一幅图片, 而 `content` 作为图片的title属性, 也就是当鼠标停在图片上面时显示:: 

If `link` is an image but `content` is not, the syntax creates an
image where the `content` is the title text shown when mouse is over
the image::

    [robot.jpeg|Robot rocks!] -> <img src="robot.jpeg" title="Robot rocks!">

.. Section titles

章节标题
--------------

如果文档内容较长, 则通常会分为几个章节. 从Robot Framework 2.7.5 版本开始, 可以使用语法格式 `= My Title =` 设置章节标题. 其中, 等号(`=`)的数量表示标题的级别::

    = First section =

    == Subsection ==

    Some text.

    == Second subsection ==

    More text.

    = Second section =

    You probably got the idea.

注意, 最多支持三级标题, 并且标题文本和前后的等号之间 *必须* 要留有空格.


.. Tables

表格
------

表格通过两边留有空格的管道符(即竖线)来作为列的分隔, 用换行表示新的一行(row). 在单元内的文字两边加上等号来标记表头, 如 `= Header =` 或 `=Header=`. 

表格单元格内的文字同样支持行内样式以及链接格式. 例如::

   | =A= |  =B=  | = C =  |
   | _1_ | Hello | world! |
   | _2_ | Hi    |

生成的表格总是带有窄边框, 正常文字是左对齐, 而表头的字体是粗体且居中. 自动添加空单元格以保证表格每行的长度一致. 例如, 上例转为HTML后的格式如下:

.. raw:: html

  <div class="doc">
    <table>
      <tr><th>A</th><th>B</th><th>C</th></tr>
      <tr><td><i>1</i></td><td>Hello</td><td>world</td></tr>
      <tr><td><i>2</i></td><td>Hi</td><td></td></tr>
    </table>
  </div>

.. note:: 支持表头是 Robot Framework 2.8.2 的新特性.

.. Lists

列表
-----

在行首用连字符(即减号`-`)开始, 后面跟空格, 然后是列表项. 列表项可以分为多行, 多行情况下, 后续行要缩进至少一个空格. 一旦遇到没有不是以 `- `开始的行且没有缩进, 则标志着列表的结束::

  Example:
  - a list item
  - second list item
    is continued

  This is outside the list.

上面的文档转为HTML:

.. raw:: html

  <div class="doc">
  <p>Example:</p>
  <ul>
    <li>a list item</li>
    <li>second list item is continued</li>
  </ul>
  <p>This is outside the list.</p>
  </div>

.. note:: 多列表的支持在2.7.2版本增加. 在这之前, 该语法阻止 Libdoc_ 把这些行拼成段落,
          所以最终结果也差不多. 列表项可以分为多行是在2.7.4版本增加的功能.

.. Preformatted text

预格式的文本
-----------------

Robot Framework 2.7 版本开始, 可以在文档中嵌入预格式的(preformatted)文本. 以 '| ' 作为一行的开始, 其中管道符后面必须要有至少一个空格(空行是个例外). 最终转换到HTML时, 行首的 '| ' 被去掉, 但是其它所有的空格都会被保留. 

在下面的文档中, 中间的两行就是预格式的文本块::

  Doc before block:
  | inside block
  |    some   additional whitespace
  After block.

转为HTML:

.. raw:: html

  <div class="doc">
  <p>Doc before block:</p>
  <pre>inside block
    some   additional whitespace</pre>
  <p>After block.</p>
  </div>

当在Robot Framework的测试数据中编写这样包含多个空格的文档, 需要对空格进行转义, 以 `prevent ignoring spaces`_. 所以上面的例子实际会写作::

  Doc before block:
  | inside block
  | \ \ \ some \ \ additional whitespace
  After block.

.. Horizontal ruler

分割线
----------------

水平分割线(即`<hr>`)常用来分隔大的章节, 在单独一行内使用3个或以上的连字符即可::

   Some text here.

   ---

   More text...

上面的文档转为HTML:

.. raw:: html

  <div class="doc">
  <p>Some text here.</p>
  <hr>
  <p>More text...</p>
  </div>
