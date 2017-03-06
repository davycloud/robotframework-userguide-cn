.. _test data template:

测试数据模板
===================

在为Robot Framework `创建测试数据`_ 的时候可以使用这些模板. 有可用于 `测试用例`_ 和 `资源文件`_ 的, 资源模板还可以用于创建 `测试套件初始化文件`_.

模板可使用 `HTML格式`_ 和 `TSV格式`_, 并可自由自定义. `纯文本格式`_ 没有模板, 因为纯文本没有太多样板, 所以模板没什么作用. 


`testcase_template.html`__
   HTML格式的测试用例文件模板

`testcase_template.tsv`__
   TSV格式的测试用例文件模板

`resource_template.html`__
   HTML格式的资源文件模板

`resource_template.tsv`__
   TSV格式的资源文件模板

`attd_template.html`__
   用于创建验收测试驱动开发(ATDD)风格的测试用例. 这种测试由高层不带参数的关键字创建, 
   相应的模板也做了简化.

可通过本手册获取到模板文件, 它们包含在了源代码发行包中, 也可在 `项目页面`__ 找到.

__ ../../templates/testcase_template.html
__ ../../templates/testcase_template.tsv
__ ../../templates/resource_template.html
__ ../../templates/resource_template.tsv
__ ../../templates/atdd_template.html
__ https://github.com/robotframework/robotframework/tree/master/templates
