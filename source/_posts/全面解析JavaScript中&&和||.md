---
title: 全面解析JavaScript中“&&”和“||”操作符
date: 2017-03-06 14:42:36
tags: JavaScript
---
逻辑 "||"
1、只要“||”前面为false,不管“||”后面是true还是false，都返回“||”后面的值。
2、只要“||”前面为true,不管“||”后面是true还是false，都返回“||”前面的值。
<!-- more --> 
知道了前面第一个的结果就知道最后的输出，如果为第一个为：true，则取第一个的值，如果第一个为false，则取第二个的值。

>在js逻辑运算中，0、”“、null、false、undefined、NaN都会判为false，其他都为true

逻辑 “&&”
1、只要“&&”前面是false，无论“&&”后面是true还是false，结果都将返“&&”前面的值;
2、只要“&&”前面是true，无论“&&”后面是true还是false，结果都将返“&&”后面的值;
typeof cb == "function" && cb(that.globalData.userInfo)
当前面为true的时候，执行后面

a() && b() :如果执行a()后返回true，则执行b()并返回b的值；如果执行a()后返回false，则整个表达式返回a()的值，b()不执行；

a() || b() :如果执行a()后返回true，则整个表达式返回a()的值，b()不执行；如果执行a()后返回false，则执行b()并返回b()的值；
&& 优先级高于 ||