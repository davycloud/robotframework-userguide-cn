.. Time format

时间格式
===========

Robot Framework有一套自己的时间格式, 既灵活且易懂. 这个格式被用于好几个关键字中(例如, BuiltIn_ 关键字 :name:`Sleep` 和 :name:`Wait Until Keyword Succeeds`), DateTime_ 库, 以及 `timeouts`_.

 has its own time format that is both flexible to use and easy
to understand. It is used by several keywords (for example, BuiltIn_ keywords
:name:`Sleep` and :name:`Wait Until Keyword Succeeds`), DateTime_ library, and
`timeouts`_.

.. Time as number

作为数字的时间
--------------

时间总是可以用数字来表示, 此时解释为秒数. 整数和浮点数都可以, 并且可以是真正的数字, 也可以是包含数字值的字符串.

.. Time as time string

时间字符串
-------------------

以时间字符串来表示时间意味着使用类似于 `2 minutes 42 seconds` 这种格式, 这种情况通常比纯的秒数更容易懂. 比如说, `4200` 秒有多长恐怕很难理解, 但是 `1 hour 10 minutes` 就清楚的多了.

这种格式的基本思想就是前面有一个数字, 然后跟一个表示时间单位的文本. 数字部分可以是整数或浮点数, 整个格式不区分大小写, 也忽略空格, 同时还可以添加 `-` 前缀来表示负数. 

可用的表示时间的单位有:

* days, day, d
* hours, hour, h
* minutes, minute, mins, min, m
* seconds, second, secs, sec, s
* milliseconds, millisecond, millis, ms

示例::

   1 min 30 secs
   1.5 minutes
   90 s
   1 day 2 hours 3 minutes 4 seconds 5 milliseconds
   1d 2h 3m 4s 5ms
   - 10 seconds

"timer" 字符串
----------------------

Robot Framework 2.8.5 版本开始, 时间还可以用时钟格式 `hh:mm:ss.mil` 来表示. 这种格式里的小时部分和微秒部分是可选的, 开头部分的0在没有意义的时候可以省略. 可通过前缀 `-` 表示负数. 

下表中左边timer格式和右边时间字符串格式的值是等价的: 

.. table:: Timer and time string examples
   :class: tabular

   ============  ======================================
      Timer                   Time string
   ============  ======================================
   00:00:01      1 second
   01:02:03      1 hour 2 minutes 3 seconds
   1:00:00       1 hour
   100:00:00     100 hours
   00:02         2 seconds
   42:00         42 minutes
   00:01:02.003  1 minute 2 seconds 3 milliseconds
   00:01.5       1.5 seconds
   -01:02.345    \- 1 minute 2 seconds 345 milliseconds
   ============  ======================================
