.. Boolean arguments

布尔型参数
=================

Robot Framework的标准库中有很多关键字接受一个布尔型的值, true或false. 如果参数是字符串的形式提供, 则如果该字符串是空的, 或者转为小写后等于 `false` 或 `no`, 则表明该值是`false`. 其它的字符串一律视为`true`. 如果参数是其它类型, 则这些值按照 `Python语言的规则 <http://docs.python.org/2/library/stdtypes.html#truth-value-testing>`__ 来判断.

关键字还可以接受除了`false` 和 `no` 之外的其它特殊字符串为false. 例如, 内置_ 关键字 `Should Be True`, 如果它的 `values` 参数被传递字符串 `no values`, 则视作false.

.. sourcecode:: robotframework

   *** Keywords ***
   True examples
       Should Be Equal    ${x}    ${y}    Custom error    values=True         # Strings are generally true.
       Should Be Equal    ${x}    ${y}    Custom error    values=yes          # Same as the above.
       Should Be Equal    ${x}    ${y}    Custom error    values=${TRUE}      # Python `True` is true.
       Should Be Equal    ${x}    ${y}    Custom error    values=${42}        # Numbers other than 0 are true.

   False examples
       Should Be Equal    ${x}    ${y}    Custom error    values=False        # String `false` is false.
       Should Be Equal    ${x}    ${y}    Custom error    values=no           # Also string `no` is false.
       Should Be Equal    ${x}    ${y}    Custom error    values=${EMPTY}     # Empty string is false.
       Should Be Equal    ${x}    ${y}    Custom error    values=${FALSE}     # Python `False` is false.
       Should Be Equal    ${x}    ${y}    Custom error    values=no values    # Special false string in this context.

注意, 在Robot Framework 2.9 之前的版本中, 布尔值的判断并不统一. 有的关键字遵守上面的规则, 但是有的简单地把所有非空字符串(包括`false` 和 `no`)都视作true.

