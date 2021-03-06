---
title: 浏览器的JavaScript引擎
layout: page
category: bom
date: 2013-03-10
modifiedOn: 2013-09-26
---

浏览器通过内置的JavaScript引擎，读取网页中的代码，对其处理后运行。

## JavaScript代码嵌入网页的方法

### 静态嵌入

在网页中使用script标签，可以直接将JavaScript代码嵌入网页。

{% highlight html %}

<script>
// some JavaScript code
</script>

{% endhighlight %}

也可以指定外部的脚本文件，让script标签读取。

{% highlight html %}

<script src="example.js"></script>

{% endhighlight %}

下载和执行JavaScript代码时，浏览器会暂停页面渲染，等待执行完成，这是因为JavaScript代码可能会修改页面。由于这个原因，如果某段代码的下载或执行时间特别长，浏览器就会呈现“假死”状态，失去响应。为了避免这种情况，较好的做法是将script标签都放在页面底部，而不是头部。

如果有多个script标签，比如下面这样：

{% highlight html %}

<script src="1.js"></script>
<script src="2.js"></script>

{% endhighlight %}

浏览器会同时平行下载1.js和2.js，但是执行时会保证先执行1.js，然后再执行2.js，即使后者先下载完成，也是如此。也就是说，脚本的执行顺序由它们在页面中的出现顺序决定，这是为了保证脚本之间的依赖关系不受到破坏。

一个解决方法是加入defer属性。

{% highlight html %}

<script src="1.js" defer></script>
<script src="2.js" defer></script>

{% endhighlight %}

有了defer属性，浏览器下载脚本文件的时候，不会阻塞页面渲染。下载的脚本文件在DOMContentLoaded事件触发前执行，而且可以保证执行顺序就是它们在页面上出现的顺序。但是，浏览器对这个属性的支持不够理想，IE（<=9）还有一个bug，无法保证执行顺序（一旦1.js修改了页面的DOM结构，会引发2.js立即执行）。此外，对于没有src属性的script标签，以及动态生成的script标签，defer不起作用。

另一个解决方法是加入async属性。

{% highlight html %}

<script src="1.js" async></script>
<script src="2.js" async></script>

{% endhighlight %}

async属性可以保证脚本下载的同时，浏览器继续渲染。一旦渲染完成，再执行脚本文件，这就是“非同步执行”的意思。需要注意的是，一旦采用这个属性，就无法保证脚本的执行顺序。先下载完成的脚本，就会排在最前面执行。

对于来自同一个域名的资源，比如脚本文件、样式表文件、图片文件等，浏览器一般最多同时下载六个。如果是来自不同域名的资源，就没有这个限制。所以，通常把静态文件放在不同的域名之下，以加快下载速度。

另外，如果不指定协议，浏览器默认采用HTTP协议下载。

{% highlight html %}

<script src="example.js"></script>

{% endhighlight %}

上面的example.js默认就是采用http协议下载，如果要采用HTTPs协议下载，必需写明（假定服务器支持）。

{% highlight html %}

<script src="https://example.js"></script>

{% endhighlight %}

但是有时我们会希望，根据页面本身的协议来决定加载协议，这时可以采用下面的写法。

{% highlight html %}

<script src="//example.js"></script>

{% endhighlight %}

### 动态嵌入

除了用静态的script标签，将JavaScript代码插入网页，还可以动态生成script标签。

{% highlight javascript %}

['1.js', '2.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  document.head.appendChild(script);
});

{% endhighlight %}

这种方法的好处是，动态生成的script标签不会阻塞页面渲染，也就不会造成浏览器假死。但是问题在于，这种方法无法保证脚本的执行顺序，哪个脚本文件先下载完成，就先执行哪个。

如果想避免这个问题，可以设置async属性为false。

{% highlight javascript %}

['1.js', '2.js'].forEach(function(src) {
  var script = document.createElement('script');
  script.src = src;
  script.async = false;
  document.head.appendChild(script);
});

{% endhighlight %}

上面的代码依然不会阻塞页面渲染，而且可以保证2.js在1.js后面执行。不过需要注意的是，在这段代码后面加载的脚本文件，会因此都等待2.js执行完成后再执行。

## JavaScript虚拟机

JavaScript是一种解释型语言，也就是说，它不需要编译，可以由解释器实时运行。这样的好处是运行和修改都比较方便，刷新页面就可以重新解释；缺点是每次运行都要调用解释器，系统开销较大，运行速度慢于编译型语言。为了提高运行速度，目前的浏览器都将JavaScript进行一定程度的编译，生成类似字节码（bytecode）的中间代码，以提高运行速度。

早期，浏览器内部对JavaScript的处理过程如下：

1. 读取代码，进行词法分析（Lexical analysis），将代码分解成词元（token）。
2. 对词元进行语法分析（parsing），将代码整理成“语法树”（syntax tree）。
3. 使用“翻译器”（translator），将代码转为字节码（bytecode）。
4. 使用“字节码解释器”（bytecode interpreter），将字节码转为机器码。

逐行解释将字节码转为机器码，是很低效的。为了提高运行速度，现代浏览器改为采用“即时编译”（Just In Time compiler，缩写JIT），即字节码只在运行时编译，用到哪一行就编译哪一行，并且把编译结果缓存（inline cache）。通常，一个程序被经常用到的，只是其中一小部分代码，有了缓存的编译结果，整个程序的运行速度就会显著提升。

不同的浏览器有不同的编译策略。有的浏览器只编译最经常用到的部分，比如循环的部分；有的浏览器索性省略了字节码的翻译步骤，直接编译成机器码，比如chrome浏览器的V8引擎。

字节码不能直接运行，而是运行在一个虚拟机（Virtual Machine）之上，一般也把虚拟机称为JavaScript引擎。因为JavaScript运行时未必有字节码，所以JavaScript虚拟机并不完全基于字节码，而是部分基于源码，即只要有可能，就通过JIT（just in time）编译器直接把源码编译成机器码运行，省略字节码步骤。这一点与其他采用虚拟机（比如Java）的语言不尽相同。这样做的目的，是为了尽可能地优化代码、提高性能。下面是目前最常见的一些JavaScript虚拟机：

- [Chakra](http://en.wikipedia.org/wiki/Chakra_(JScript_engine\))(Microsoft Internet Explorer)
- [Nitro/JavaScript Core](http://en.wikipedia.org/wiki/WebKit#JavaScriptCore) (Safari)
- [Carakan](http://dev.opera.com/articles/view/labs-carakan/) (Opera)
- [SpiderMonkey](https://developer.mozilla.org/en-US/docs/SpiderMonkey) (Firefox)
- [V8](http://en.wikipedia.org/wiki/V8_(JavaScript_engine\)) (Chrome, Chromium)

## 单线程模型

### 含义

JavaScript采用单线程模型，也就是说，所有的任务都在一个线程里运行。这意味着，一次只能运行一个任务，其他任务都必须在后面排队等待。这又被称为“事件循环”模型（event loop）。

新的任务被加在队列的尾部，只有前面的所有任务运行结束，才会轮到它执行。如果有一个任务特别耗时，后面的任务都会停在那里等待，造成浏览器失去响应，又称“假死”。为了避免“假死”，当某个操作在一定时间后仍无法结束，浏览器就会跳出提示框，询问用户是否要强行停止脚本运行。

JavaScript本身也提供了一些“假死”的解决方法。最常见的就是用 setTimeout 和 setInterval 方法，将耗时的任务移到任务队列的尾部，在较晚的时间运行。另一个例子是，XMLHttpRequest对象将Ajax操作中的HTTP通信（非常耗时的任务），移到另一个线程，从而不会堵塞主线程。

### setTimeout方法

setTimeout方法的作用是推迟某个任务的运行时间，从而改变JavaScript的正常执行顺序。

{% highlight javascript %}

console.log(1);
console.log(2);
console.log(3);

{% endhighlight %}

正常情况下，上面三行语句按照顺序执行，输出1--2--3。现在，用setTimeout改变执行顺序。

{% highlight javascript %}

console.log(1);
setTimeout(function(){console.log(2);},1000);
console.log(3);

{% endhighlight %}

上面代码的输出结果就是1--3--2，因为setTimeout方法指定第二行语句，在所有任务结束后，等待1000毫秒再执行。

setTimeout方法的前两个参数是必需的。第一个参数是回调函数，第二个参数是推迟执行的时间，单位为毫秒。除了这两个参数以外，其他参数都是可选的，将在回调函数运行时传入回调函数。

{% highlight javascript %}

setTimeout(function(a,b){console.log(a+b);},1000,1,1);

{% endhighlight %}

上面代码表示，将在1000毫秒之后执行回调函数，输出1加1的和。

IE小于9.0的版本，只允许setTimeout有两个参数，不支持更多的参数。这时有三种解决方法，第一种是自定义setTimeout，使用apply方法将参数输入回调函数；第二种是在一个匿名函数里面，让回调函数带参数运行，再把匿名函数输入setTimeout；第三种使用bind方法，把多余的参数绑定在回调函数上面，生成一个新的函数输入setTimeout。

除了参数问题，setTimeout还有一个需要注意的地方：被setTimeout推迟执行的回调函数是在全局环境执行，这有可能不同于函数定义时的上下文环境。

{% highlight javascript %}

var n = 1;

var o = {
	n: 2,
	m: function(){console.log(this.n);}
};

var timeoutID = setTimeout(o.m,1000);

{% endhighlight %}

上面代码输出的是1，而不是2，这表示回调函数的运行环境已经变成了全局环境。

### setInterval方法

setInterval方法的使用格式和需要注意的地方，与setTimeout完全一致，两者的区别仅仅在于setInterval指定某个任务每隔一段时间就执行一次。

{% highlight javascript %}

setInterval(function(){console.log(2);},1000);

{% endhighlight %}

上面代码表示每隔1000毫秒就输出一个2。

除了前两个参数，setInterval 方法还可以接受更多的参数，它们会传入回调函数。

{% highlight javascript %}

function f(){
	for (var i=0;i<arguments.length;i++){
		console.log(arguments[i]);
	}
}

setInterval(f, 1000, "Hello World");

{% endhighlight %}

上面代码的运行结果如下：

{% highlight bash %}

Hello World
Hello World
Hello World
...

{% endhighlight %}

如果网页不在浏览器的当前窗口（或tab），许多浏览器限制setInteral指定的反复运行的任务最多每秒执行一次。

### clearTimeout 和 clearInterval 方法，

setTimeout 和 setInterval 方法都返回一个表示计数器编号的整数值，将该整数传入clearTimeout 和 clearInterval 方法，就可以取消指定的操作。

{% highlight javascript %}

var id1 = setTimeout(f,1000);
var id2 = setInterval(f,1000);

clearTimeout(id1);
clearInterval(id2);

{% endhighlight %}

### 运行队列

本质上，setTimeout和setInterval都是把任务添加到“运行队列”的尾部，等到前面的任务都执行完，再开始执行。由于前面的任务到底需要多少时间执行完，是不确定的，所以没有办法保证，被推迟的任务一定会按照预定时间执行。

{% highlight javascript %}

setTimeout(someTask,100);
veryLongTask();

{% endhighlight %}

上面代码的setTimeout，指定100毫秒以后运行一个任务。但是，如果后面立即运行的任务非常耗时，过了100毫秒还无法结束，那么被推迟运行的someTask就只有等着，等到前面的veryLongTask运行结束，才轮到它执行。

这一点对于setInterval影响尤其大。

{% highlight javascript %}

setInterval(function(){console.log(2);},1000);

(function (){ sleeping(3000);})();

{% endhighlight %}

上面的第一行语句要求每隔1000毫秒，就输出一个2。但是，第二行语句需要3000毫秒才能完成，请问会发生什么结果？

结果就是等到第二行语句运行完成以后，立刻连续输出三个2，然后开始每隔1000毫秒，输出一个2。为了进一步理解JavaScript的单线程模型，请看下面这段伪代码。

{% highlight javascript %}

    function init(){
        { 耗时5ms的某个操作 } 
        触发mouseClickEvent事件
        { 耗时5ms的某个操作 }
        setInterval(timerTask,10);
        { 耗时5ms的某个操作 }
    }

    function handleMouseClick(){
          耗时8ms的某个操作 
    }

    function timerTask(){
          耗时2ms的某个操作 
    }

{% endhighlight %}

请问调用init函数后，这段代码的运行顺序是怎样的？

- **0-15ms**：运行init函数。

- **15-23ms**：运行handleMouseClick函数。请注意，这个函数是在5ms时触发的，应该在那个时候就立即运行，但是由于单线程的关系，必须等到init函数完成之后再运行。

- **23-25ms**：运行timerTask函数。这个函数是在10ms时触发的，规定每10ms运行一次，即在20ms、30ms、40ms等时候运行。由于20ms时，JavaScript线程还有任务在运行，因此必须延迟到前面任务完成时再运行。

- **30-32ms**：运行timerTask函数。

- **40-42ms**：运行timerTask函数。

由于setInterval无法保证每次操作之间的间隔，为了避免这种情况，可以反复调用setTimeout，替代setInterval。

{% highlight javascript %}

var recursive = function () {
    console.log("It has been one second!");
    setTimeout(recursive,1000);
}

recursive();

{% endhighlight %}

上面这样的写法，就能保证两次recursive之间的运行间隔，一定是1000毫秒。

### 推迟时间的极限和setTimeout(f,0)的作用

跟据[HTML 5标准](http://www.whatwg.org/specs/web-apps/current-work/multipage/timers.html#timers)，setTimeOut推迟执行的时间，最少是4毫秒。但是，实际应用中，可以看到有推迟0秒的。

{% highlight javascript %}

setTimeout(function (){console.log("你好！"), 0);

{% endhighlight %}

这种写法的含义是，当前任务队列一结束就运行setTimeout指定的回调函数。

{% highlight javascript %}

console.log("任务队列开始");

setTimeout(function() { console.log("任务队列结束后运行");}, 0);

function a(x) { 
	console.log("a() 开始运行");
	b(x);
	console.log("a() 结束运行");
}

function b(y) { 
	console.log("b() 开始运行");
	console.log("传入的值为" + y);
	console.log("b() 结束运行");
}

console.log("当前任务开始");
a(42);
console.log("任务队列结束");

{% endhighlight %}

上面代码的运行结果如下：

{% highlight bash %}

任务队列开始
当前任务开始
a() 开始运行
b() 开始运行
传入的值为42
b() 结束运行
a() 结束运行
任务队列结束
任务队列结束后运行

{% endhighlight %}

可以看到，setTimeout(function, 0)将任务移到当前任务队列结束后运行。

浏览器内部使用32位带符号的整数，来储存推迟执行的时间。这意味着setTimeout最多只能推迟执行2147483647毫秒，超过这个时间会发生溢出，导致回调函数将在当前任务队列结束后立即执行，即等同于setTimeout(f,0)的效果。

本质上，setTimeout(f, 0)这种写法反映了JavaScript单线程运行的特点。但是，现在的计算机CPU普遍是多核的，单线程就意味着只使用一核。这当然没有充分利用资源，所以HTML5提供了Web Worker，允许通过这个API实现多线程操作。

## 参考链接

- John Dalziel, [The race for speed part 2: How JavaScript compilers work](http://creativejs.com/2013/06/the-race-for-speed-part-2-how-javascript-compilers-work/)
- Jake Archibald，[Deep dive into the murky waters of script loading](http://www.html5rocks.com/en/tutorials/speed/script-loading/)
- Mozilla Developer Network, [window.setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/window.setTimeout)
