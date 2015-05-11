---
layout: default
title: letter-spacing和text-align:center的矛盾
---
letter-spacing和text-align:center的矛盾
---------------------------------------

# 1 问题
有的时候ui给的设计搞中会出现2个字之间有比较宽的间距，特别是button。
![button示例图]()
我们可以用在文字之间加&nbsp;，但是这种做法一来没有办法精确的控制文字间的间距，二来如果文字很多那么加空格的加死了。我们的css中有letter-spacing可以指定文字间的间距。

```html
<style>
	button {
		letter-spacing:4px;
		text-align:center;
	}
</style>
<button>搜索</button>
```
我们看到文字间的间距是增加，但是为什么文字看上去没有居中呢，有了一定的偏差呢？

#2分析
我们用chrome的developtool看下点击下文字，看到文字的选中框，原来letter-spacing不但在各个文字间添加了间距，在最后一个字的后面还添加了一个间距，所以在text-align时候自然产生了偏差。

我们来看下letter-spacing的[规范](http://dev.w3.org/csswg/css-text-3/#letter-spacing-property)：
> Because letter-spacing is not applied at the beginning or end of a line, text always fits flush with the edge of the block

规范中指出letter-spacing不允许在文字的头尾添加间隔。所以浏览器的实现其实并不符合规范。

# 3 解决方案
那么我们可以通过添加padding-left或者text-indent来在文字的头可以营造一个间隔来达到使整个文字居中的效果
```html
<style>
	button {
		letter-spacing:4px;
		text-align:center;
		padding-left:4px;
		/* text-indent:4px; */
	}
</style>
<button>搜索</button>
```
![button示例图]()

参考：
http://stackoverflow.com/questions/6315491/conflict-between-letter-spacing-and-text-aligncenter
http://dev.w3.org/csswg/css-text-3/#letter-spacing-property
