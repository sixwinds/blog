---
layout: default
title: 图片加载掩饰导致 jquery domready 延时响应
---
# 1 问题
项目中碰到一个奇怪的问题：

我们的页面上有一个Alexa Certify 的script，这个script会用一个 amazon 的图片( x.png )作为请求发送统计信息。但是这个图片的地址会被 GFW 墙掉导致超时。我们的页面 js 有的时候会因为这个超时导致加载部分功能很慢。

我测试了下 $() 中的回调函数要等到 x.png 加载完或者超时之后才会执行，慢的这部分代码其实都是放在 $() 中的，也就是说需要 dom ready 后执行的代码。但是照道理 img 的加载和 dom ready 并不应该有冲突，而且为啥有的页面没有这个问题而主页有呢？

# 2 分析

看下主页，是用 requirejs 来加载 script 的，而其他页面是直接把 script 贴在 html 最底下的。导致的区别就是 requirejs 异步加载模块的并且执行到 $() 的时候，dom 已经 ready 了。而直接贴 script 的 js 在运行到 $()时候 dom 还没有完全 ready。

但是为啥 requirejs 异步加载的代码在 dom ready 之后如果运行 $() 依然会被执行( 或者说回调 )？

这个时候就要看我们坑爹的 jq 是如何实现了。jq 的 $() 会把回调函数注册到 document 的 DOMContentLoaded 事件上和 window 的 load 事件上去( 对，你没看错是同时注册到两个事件上去 )。这两个事件的区别不详尽了，大家自行google。但是这两个事件触发的时间是不同的，当我们的 dom 加载完而 x.png 还在 loading 的时候 DOMContentLoaded 就会触发了，而 window 的 load 事件是会在 x.png 加载完或者超时之后才会触发。

那么我们用 requirejs 加载 js 和直接在 html 贴 script 的差别就来了。html 直接贴 script 的 js 响应的是 document 的 DOMContentLoaded 事件，自然不会受到 x.png 的影响，而 requirejs 加载的 js 就不同了，当他被动态加到 html中并且运行到 $() 的时候 document 的 DOMContentLoaded 事件已经触发过了，$() 中的回调函数必须等待 window 的 load 事件触发才会被调用到，所以用requirejs加载的代码会受到 x.png 加载是否完成的影响，这就是我们主页慢的原因。而 jq 的 ready 对页面加载的影响也是臭名昭著了。

详细的东西大家可以自行 google。( 这里我是测试了chrome，firefox 得出的结论，而更具体的原理待我研究后在下一篇里面讲 )

# 3 解决方案

虽然requirejs 是动态加载 js 的，但是 requirejs 本身并不是，而我们的 cdn 又支持合并 js 的请求

譬如：
```sh
http://static-web.b5m.com/public/js/??jquery-1.9.1.min.js,jquery-window.js,imglazyload.min.js?_a=1&v=2015412015043014435154
```

那么我们页面上的 requirejs 可以这么贴：

```sh
<script type="text/javascript" id="requirejs" data-main="/home/js/test.js" src="http://CDN/public/js/??require-min.js,require-domready.js?_a=1"></script>`
```
在require-domready.js中我们主动注册 document 的 DOMContentLoaded ，并且在此事件触发时标记 DOMContentLoaded 已触发，而其他业务代码在用到 $() 的时候先去判断这个标记是否存在，如果存在则可以直接写要执行的代码，如果没有的话再去调用 $()，我们可以把这个逻辑判断包装成我们自己的ready函数，这样就能解决该死的 x.png 和 $() 的问题了。