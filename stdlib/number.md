---
title: Number对象
layout: page
category: stdlib
date: 2013-03-16
modifiedOn: 2013-09-03
---

## 属性

Number拥有一些特别的属性。

- Number.POSITIVE_INFINITY 表示正的无限，指向关键字Infinity。
- Number.NEGATIVE_INFINITY 表示负的无限，指向-Infinity。
- Number.NaN 表示非数值，指向NaN。
- Number.MAX_VALUE 表示最大的正数。最大的负数为-Number.MAX_VALUE。
- Number.MIN_VALUE 表示最小的正数（即最接近0的正数）。最接近0的负数为-Number.MIN_VALUE。

{% highlight javascript %}

Number.POSITIVE_INFINITY
// Infinity

Number.NEGATIVE_INFINITY
// -Infinity

Number.NaN
// NaN

Number.MAX_VALUE
// 1.7976931348623157e+308

Number.MIN_VALUE
// 5e-324

{% endhighlight %}

## Number实例对象的方法

### toString方法

Number对象部署了单独的toString方法，可以接受一个参数，表示将一个数字转化成某个进制的字符串。

{% highlight javascript %}

(10).toString() // "10"

(10).toString(2) // "1010"

(10).toString(8) // "12"

(10).toString(16) // "a"

{% endhighlight %}

之所以要把10放在括号里，是为了表明10是一个单独的数值，后面的点表示调用对象属性。如果不加括号，这个点会被JavaScript引擎解释成小数点，从而报错。

{% highlight javascript %}

10.toString(2) 
// SyntaxError: Unexpected token ILLEGAL

{% endhighlight %}

但是，在10后面加两个点，JavaScript会把第一个点理解成小数点（即10.0），把第二个点理解成调用对象属性，从而得到正确结果。

{% highlight javascript %}

10..toString(2) 
// "1010"

{% endhighlight %}

这实际上意味着，可以直接对一个小数使用toString方法。

{% highlight javascript %}

10.5.toString() // "10.5"

10.5.toString(2) // "1010.1"

10.5.toString(8) // "12.4"

10.5.toString(16) // "a.8"

{% endhighlight %}

### toFixed方法

toFixed方法用于将一个数转为指定位数的小数。

{% highlight javascript %}

(10).toFixed(2)
// "10.00"
// 10必须放在括号里，否则后面的点运算符会被处理小数点，而不是表示调用对象的方法。

(10.005).toFixed(2)
// "10.01" 

{% endhighlight %}

toFixed的输出支持0到20位小数。

### toExponential方法

toExponential方法用于将一个数转为科学计数法形式。

{% highlight javascript %}

(10).toExponential(1)
// "1.0e+1"

(1234).toExponential(1)
// "1.2e+3"

{% endhighlight %}

toExponential方法的参数表示小数点后有效数字的位数。

### toPrecision方法

toPrecision方法用于将一个数转为指定位数的有效数字。

{% highlight javascript %}

(12.34).toPrecision(1)
// "1e+1"

(12.34).toPrecision(2)
// "12"

(12.34).toPrecision(3)
// "12.3"

(12.34).toPrecision(4)
// "12.34"

(12.34).toPrecision(5)
// "12.340"

{% endhighlight %}

toPrecision方法提供的有效数字位数的范围是1到21。
