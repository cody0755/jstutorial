---
title: Express框架
category: nodejs
layout: page
date: 2013-09-13
modifiedOn: 2013-09-13
---

Express是目前最流行的基于Node.js的Web开发框架。它可以快速地搭建网站原型。

## 安装和新建项目

首先，将Express安装为全局模块。

{% highlight bash %}

npm install -g express

{% endhighlight %}

然后，建立一个项目目录（假定这个目录叫做demo）。进入该目录，新建一个package.json文件，写入项目的配置信息。

{% highlight javascript %}

{
   "name": "demo",
   "description": "My First Express App",
   "version": "0.0.1",
   "dependencies": {
      "express": "3.x"
   }
}

{% endhighlight %}

在项目目录中，新建文件app.js。项目的代码就放在这个文件里面。

{% highlight javascript %}

var express = require('express');
var app = express();
 
app.listen(3000);

{% endhighlight %}

这时，运行下面的命令，就可以在浏览器访问http://127.0.0.1:3000。

{% highlight bash %}

node app.js

{% endhighlight %}

这时，网页提示“Cannot GET /”，表示没有为网站的根路径指定可以显示的内容。所以，下一步就是配置路由。

## 配置路由

所谓“路由”，就是指为不同的访问路径，指定不同的处理方法。

在app.js之中，先指定根路径的处理方法。

{% highlight javascript %}

app.get('/', function(request, response) {
   response.send("此处放HTML代码");
});

{% endhighlight %}

上面代码的get方法，表示处理客户端发出的GET请求。它的第一个参数是访问路径，正斜杠（/）就代表根路径；第二个参数是回调函数，它的request参数表示客户端发来的HTTP请求对象，response代表发向客户端的HTTP回应对象。

再运行下面的命令，然后在浏览器中访问http://127.0.0.1:3000。

{% highlight bash %}

node app.js

{% endhighlight %}

网页就会显示“此处放HTML代码”。

然后，再定义路径/api的处理方法。

{% highlight javascript %}

app.get('/api', function(request, response) {
   response.send({name:"张三",age:40});
});

{% endhighlight %}

重新启动node以后，再访问路径/api，浏览器就会显示一个JSON对象。

{% highlight javascript %}

{
  "name": "张三",
  "age": 40
}

{% endhighlight %}

如果只向浏览器发送简单的文本信息，上面的方法已经够用；但是如果要向浏览器发送复杂的内容，还是应该使用预先生成的网页模板。

## 静态网页模板

在项目目录之中，建立一个子目录views，用于存放网页模板。

假定这个项目有三个路径：根路径（/）、自我介绍（/about）和文章（/article）。那么，app.js可以这样写：

{% highlight javascript %}

var express = require('express');
var app = express();
 
app.get('/', function(req, res) {
   res.sendfile('./views/index.html');
});
 
app.get('/about', function(req, res) {
   res.sendfile('./views/about.html');
});
 
app.get('/article', function(req, res) {
   res.sendfile('./views/article.html');
});
 
app.listen(3000);

{% endhighlight %}

上面代码表示，三个路径分别对应views目录中的三个模板：index.html、about.html和article.html。另外，向服务器发送信息的方法，从send变成了sendfile，后者专门用于发送文件。

假定index.html的内容如下：

{% highlight html %}

<html>
<head>
   <title>首页</title>
</head>
 
<body>
<h1>Express Demo</h1>
 
<footer>
<p>
   <a href="/">首页</a> - <a href="/about">自我介绍</a> - <a href="/article">文章</a>
</p>
</footer>
 
</body>
</html>

{% endhighlight %}

上面代码是一个静态网页。如果想要展示动态内容，就必须使用动态网页模板。

## 动态网页模板

### 安装模板引擎

Express支持多种模板引擎，这里采用Handlebars模板引擎的服务器端版本[hbs](https://github.com/donpark/hbs)模板引擎。

先安装hbs。

{% highlight html %}

npm install hbs --save-dev

{% endhighlight %}

上面代码将hbs模块，安装在项目目录的子目录node_modules之中。save-dev参数表示，将依赖关系写入package.json文件。安装以后的package.json文件变成下面这样：

{% highlight javascript %}

// package.json文件

{
  "name": "demo",
  "description": "My First Express App",
  "version": "0.0.1",
  "dependencies": {
    "express": "3.x"
  },
  "devDependencies": {
    "hbs": "~2.3.1"
  }
}

{% endhighlight %}

上面代码的devDependencies属性表示，列入其中的模板都是本地安装的模块，而不是全局模板。

安装模板引擎之后，就要改写app.js。

{% highlight javascript %}

// app.js文件

var express = require('express');
var app = express();

// 加载hbs模块
var hbs = require('hbs');

// 指定模板文件的后缀名为html
app.set('view engine', 'html');

// 运行hbs模块
app.engine('html', hbs.__express);

app.get('/', function(req, res) {
	res.render('index');
});

app.get('/about', function(req, res) {
	res.render('about');
});

app.get('/article', function(req, res) {
	res.render('article');
});

{% endhighlight %}

上面代码改用render方法，对网页模板进行渲染。render方法的参数就是模板的文件名，默认放在子目录views之中，后缀名已经在前面指定为html，这里可以省略。所以，res.render('index') 就是指，把子目录views下面的index.html文件，交给模板引擎hbs渲染。

### 新建数据脚本

渲染是指将数据代入模板的过程。实际运用中，数据都是保存在数据库之中的，这里为了简化问题，假定数据保存在一个脚本文件中。

在项目目录中，新建一个文件blog.js，用于存放数据。blog.js的写法符合CommonJS规范，使得它可以被require语句加载。

{% highlight javascript %}

// blog.js文件

var entries = [
	{"id":1, "title":"第一篇", "body":"正文", "published":"6/2/2013"},
	{"id":2, "title":"第二篇", "body":"正文", "published":"6/3/2013"},
	{"id":3, "title":"第三篇", "body":"正文", "published":"6/4/2013"},
	{"id":4, "title":"第四篇", "body":"正文", "published":"6/5/2013"},
	{"id":5, "title":"第五篇", "body":"正文", "published":"6/10/2013"},
	{"id":6, "title":"第六篇", "body":"正文", "published":"6/12/2013"}
];

exports.getBlogEntries = function() {
   return entries;
}
 
exports.getBlogEntry = function(id) {
   for(var i=0; i < entries.length; i++) {
      if(entries[i].id == id) return entries[i];
   }
}

{% endhighlight %}

### 新建网页模板

接着，新建模板文件index.html。

{% highlight html %}

<!-- views/index.html文件 -->

<h1>文章列表</h1>
 
{{"{{"}}#each entries}}
   <p>
      <a href="/article/{{"{{"}}id}}">{{"{{"}}title}}</a><br/>
      Published: {{"{{"}}published}}
   </p>
{{"{{"}}/each}}

{% endhighlight %}

模板文件about.html。

{% highlight html %}

<!-- views/about.html文件 -->

<h1>自我介绍</h1>
 
<p>正文</p>

{% endhighlight %}

模板文件article.html。

{% highlight html %}

<!-- views/article.html文件 -->

<h1>{{"{{"}}blog.title}}</h1>
Published: {{"{{"}}blog.published}}
 
<p/>
 
{{"{{"}}blog.body}}

{% endhighlight %}

可以看到，上面三个模板文件都只有网页主体。因为网页布局是共享的，所以布局的部分可以单独新建一个文件layout.html。

{% highlight html %}

<!-- views/layout.html文件 -->

<html>
 
<head>
   <title>{{"{{"}}title}}</title>
</head>
 
<body>
 
	{{"{{{"}}body}}}
 
   <footer>
      <p>
         <a href="/">首页</a> - <a href="/about">自我介绍</a>
      </p>
   </footer>
    
</body>
</html>

{% endhighlight %}

### 渲染模板

最后，改写app.js文件。

{% highlight javascript %}

// app.js文件

var express = require('express');
var app = express();
 
var hbs = require('hbs');

// 加载数据模块
var blogEngine = require('./blog');
 
app.set('view engine', 'html');
app.engine('html', hbs.__express);
app.use(express.bodyParser());
 
app.get('/', function(req, res) {
   res.render('index',{title:"最近文章", entries:blogEngine.getBlogEntries()});
});
 
app.get('/about', function(req, res) {
   res.render('about', {title:"自我介绍"});
});
 
app.get('/article/:id', function(req, res) {
   var entry = blogEngine.getBlogEntry(req.params.id);
   res.render('article',{title:entry.title, blog:entry});
});
 
app.listen(3000);

{% endhighlight %}

上面代码中的render方法，现在加入了第二个参数，表示模板变量绑定的数据。

现在重启node服务器，然后访问http://127.0.0.1:3000。

{% highlight bash %}

node app.js

{% endhighlight %}

可以看得，模板已经使用加载的数据渲染成功了。

### 指定静态文件目录

模板文件默认存放在views子目录。这时，如果要在网页中加载静态文件（比如样式表、图片等），就需要另外指定一个存放静态文件的目录。

{% highlight javascript %}

app.use(express.static('public'));

{% endhighlight %}

上面代码在文件app.js之中，指定静态文件存放的目录是public。于是，当浏览器发出非HTML文件请求时，服务器端就到public目录寻找这个文件。比如，浏览器发出如下的样式表请求：

{% highlight javascript %}

<link href="/bootstrap/css/bootstrap.css" rel="stylesheet">

{% endhighlight %}

服务器端就到public/bootstrap/css/目录中寻找bootstrap.css文件。

## 参考链接

- Raymond Camden, [Introduction to Express](http://net.tutsplus.com/tutorials/javascript-ajax/introduction-to-express/)
