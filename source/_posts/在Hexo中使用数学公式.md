---
title: 在Hexo中使用数学公式
date: 2017-10-12 12:17:15
tags:
	- 乱讲
	- Hexo
---

之前在写LeetCode解题报告的时候总是不可避免地要通过一些比较数学的方式来描述题解，原本认为Hexo自带的Markdown Renderer不支持公式渲染于是就用代码块将就了。后来越写越不能忍受，才下定决定一定要把公式功能加上。

>  $A_{i,j}$比`A[i][j]`酷多了啊！

---

在查解决方案的时候发现很幸运地，现在我在用的主题[Next](theme-next.iissnan.com)其实已经集成了*MathJax*，只需要在`themes/next/_config.yml`里面把对应设置项打开就行了：

```yaml
# MathJax Support
mathjax:
  enable: true
  per_page: false
  cdn: //cdn.bootcss.com/mathjax/2.7.1/latest.js?config=TeX-AMS-MML_HTMLorMML
```

刷新之后发现的确已经能显示数学公式了，但在需要用到下标的时候（例如打算使用`$A_i$`得到$A_i$），里面的下划线会和Markdown语法中的`_斜体_`语法冲突，也就是说，一旦有`$A_i$一些其他的文本$B_i$`这样的情况出现，里面的两个`_`会匹配成为Markdown的斜体标记，公式就渲染出错了。

> 一个Naive的解决方案就是在每一个下划线前面加转义符，也就是用`\_`替代`_`，这样能防止被解析成斜体

但这实在是太麻烦了，谁想每次在写公式下标的时候都加个斜杠呢。

---

后来找到了一个几乎完美的解决方案：修改Hexo渲染Markdown时用的渲染引擎！

在`package.json`里能看到Hexo原本用的是`hexo-renderer-marked`，把它换成`hexo-renderer-pandoc`即可。

> [Pandoc](https://pandoc.org)是一个十分强大的开源格式转换工具。
>
> `hexo-renderer-pandoc`通过调用系统中的Pandoc来渲染Markdown，因此需要先在系统中安装Pandoc才能正常使用

```shell
npm rm hexo-renderer-marked
npm i hexo-renderer-pandoc
```

完美解决公式渲染问题，能在博客里面爽用MathJax写公式了（唯一的缺点是需要额外安装Pandoc，不能直接`npm i`解决所有依赖问题，但这又何妨呢）。



