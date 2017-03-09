关于本手册
==========

本手册是 `Robot Framework <http://robotframework.org>`_ 用户手册的中文翻译版.

想直接阅读英文的请点击 `这里 <http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html>`_

获取源码
--------

你可以在 `GitHub <https://github.com>`_ 上获取到本手册的源码文件

- `UserGuide <https://github.com/robotframework/robotframework/tree/master/doc/userguide>`_

- `用户手册 <https://github.com/davyyy/robotframework-userguide-cn>`_

如果你觉得这份手册对你有帮助, 请Star一个以示鼓励. 觉得有翻译不当之处也敬请指正.

源文件格式
----------

原英文文档使用的是reStructureText标准格式, 中文文档使用了Sphinx文档工具, 虽然文件的格式是一样的, 但是其中某些标记和原文有所不同, 特别是文档的内部链接形式. 原文整个文档是一个页面, 中文文档分章节显示.

翻译说明
--------

尽量做到按原文翻译, 有些地方感觉原文实在太啰嗦了, 在不影响理解的情况下有所省略.

术语的翻译
^^^^^^^^^^

术语的翻译尽量使用通用的译法, 一般确定不会影响歧义的都直接翻译, 有的会在第一次出现时备注原英文, 有的不方便直译的保留原文或者其它更熟悉的缩写替代.

例如: free keyword arguments 如果后面翻译为"关键字参数"难免会和测试关键字产生混淆, 除了少数地方直译为"任意命名参数", 后面一般都使用大家都熟悉的 ``**kwargs`` 写法指代.


表格和代码示例
^^^^^^^^^^^^^^

文档中出现的大量代码示例和表格, 绝大部分都没有做任何翻译, 因为这些内容都很简单, 且大多数都是无法翻译的代码, 如果只翻译部分, 混杂一起反而很难看. 
表格部分的情况和代码类似, 而且reST中编辑表格是一件比较痛苦的事情, 翻成中文原表格会出现无法对齐的问题, 需要额外的工作量. 
