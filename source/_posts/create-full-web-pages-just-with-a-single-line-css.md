title: 只用一行CSS打造大背景全屏网页！  
date: 2015-02-09 22:01  
modified: 2016-05-14 17:40  
tag:
 - 前端开发
 - CSS

photos:
 - http://dn-doting.qbox.me/20141028-1.jpg
---

现代网站很多都在其主页放上一个全屏区域`section`。就算不是全屏幕，也至少占了全屏的80%~90%，通常完成这个工作这需要大量的开发时间和javascript代码。

<!--more-->
下面我举一些正在使用此布局的网站例子。

<u><strong><a href="http://www.nimber.com" target="_blank">www.nimber.com</a></strong></u>

他们使用`data`属性来保存高度比例比如:  `data-autosize="0.6" `  然后用JS来设置每一个区域的高度

![1](http://dn-doting.qbox.me/20141028-2.jpg)

<a href="http://www.exposure.co" target="_blank"><u><strong>www.exposure.co</strong></u></a>

Exposure 为他们的头部使用了一个固定的`90%`的高度，在缩放时使用JS来改变高度

![1](http://dn-doting.qbox.me/20141028-3.jpg)

<u><strong><a href="http://www.nimber.com" target="_blank">www.nimber.com</a></strong></u>

Nimber 使用的技术和<a href="http://www.spotify.com" target="_blank">spotify</a>类似，高度用JS设置为90%并加上`min-height`来确保这块区域高于视图区域，这样通常在手机端也能正确显示。

![1](http://dn-doting.qbox.me/20141028-4.jpg)

<a href="http://www.flickr.com" target="_blank"><u><strong>www.flickr.com</strong></u></a>

雅虎在今年早些时候推出了一个全屏版本的Flickr，设置每一个区域(section)的高度为100%，并且替换默认的滚动为JS控制的滚动高度，这里有一篇文章介绍它是<a href="http://code.flickr.net/2014/04/23/building-flickrs-new-hybrid-signed-out-homepage/" target="_blank">如何实现</a>的。

![1](http://dn-doting.qbox.me/20141028-5.jpg)

前面所有的例子都是用`JavaScript`实现的，而下面就是见证奇迹的时刻:


----------


我们使用__纯CSS__来实现前面的说的布局，并且支持大多数的现代浏览器

要是我们只用一行CSS来实现……

```css
.section { height: 100vh; }
```

是的，__视窗高度（viewport height）， 1vh = 1%的浏览器高度__

无需任何辅助，视窗高度知道你的浏览器每一时刻的高度，并且可以由此设置你要的区域(section)高度，我用此方法做了一个Demo，我试着改变窗口大小它也能完美适应哦！

![1](http://dn-doting.qbox.me/20141028-6.jpg)

<p style="text-align: center;">
	<span style="font-size:16px;"><a href="http://codepen.io/ckor/pen/lBnxh/" target="_blank">查看代码</a>&nbsp;&nbsp;| &nbsp;<a href="http://codepen.io/ckor/full/cf2134280cd25e8ac7e57f1b05bb0b49/" target="_blank">查看<span style="line-height: 20.7999992370605px;">Demo</span></a></span>
</p>

这个方法是如此强大，因为你可以无限的组合布局，如果你说，你想要所有的区域都是100%的高度，除了第一屏是90%，同时要保持页面是连续的。好的，你需要的是另加一行CSS:

{% codeblock lang:javascript %}
.section { height: 100vh;}
.section—first { height: 90vh; }
{% endcodeblock %}

至于浏览器的支持情况也是非常好的，它在测试网站<a href="http://caniuse.com" target="_blank"><strong><u>caniuse.com</u></strong></a>得到了78.38% 的结果，同时支持IE9+

![1](http://dn-doting.qbox.me/20141028-7.jpg)

这个方法看起来是非常不错的，虽然没有得到大范围的使用测试，我也说不出这种技术有什么缺陷的地方，或者你该用与不该用。但是你可以视自身情况尝试一下。你可以先备份好你为浏览器写的一些JavaScript，或者找其他的替代。

最后不得不再提一个切图很棒例子，它也是只使用了CSS，但是用了一点点不同的技巧。

<a href="http://www.6wunderkinder.com" target="_blank"><u><strong><span style="font-size:14px;">www.6wunderkinder.com</span></strong></u></a>

来自[wunderlist](http://www.wunderlist.com/)的这货，在头部使用了一个` position:fixed `并给它设置了一个完美的`height:100%`，这是一个稍稍不同的例子，各有利弊，但确实是一个不用JS的例子！

![1](http://dn-doting.qbox.me/20141028-8.jpg)


翻译自：https://medium.com/@ckor/make-full-screen-sections-with-1-line-of-css-b82227c75cbd  

(转载请注明出处，本文完)
