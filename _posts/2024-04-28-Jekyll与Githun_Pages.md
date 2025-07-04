---
layout: post
title: "Jekyll与Github Pages"
categories: Misc
---

> GitHub Pages 是通过 GitHub 托管和发布的公共网页。 启动和运行的最快方法是使用 Jekyll 主题选择器加载预置主题。 然后，您可以修改 GitHub Pages 的内容和样式。

# 1 简介
关于如何创建Github Pages并获取Jekyll目录：<br/>
[https://docs.github.com/en/pages](https://docs.github.com/en/pages)<br/>
关于如何使用Jekyll修改、预览网页：<br/>
[https://jekyllrb.com/docs/](https://jekyllrb.com/docs/)<br/>
Jekyll的主要目录与功能：

| 目录 | 主要文件格式 | 功能 |
| --- | --- | --- |
| _include | .html | 可以重复使用的内容 |
| _layout | .html | 确定页面的内容与布局 |
| _posts | .md或其他 | 博客主要内容 |
| _sass | .scss | SCSS文件 |
| assets | .css与.js | CSS与脚本 |
{% raw %}
基本原理：给博客指定_layout中的布局，布局文件中给出网页将呈现的元素，布局文件中的`{{ content }}`代表了博客内容，布局文件还可引用_include内的内容，虽然布局文件中可以直接指定布局（各元素大小字体等），但一般都会使用CSS文件实现，SCSS文件与CSS类似但功能更强大。

# 2 网页编辑
页面编辑需要熟悉HTML、CSS与Liquid的相关内容。<br/>
HTML: [https://www.w3school.com.cn/html/index.asp](https://www.w3school.com.cn/html/index.asp)<br/>
CSS: [https://www.w3school.com.cn/cssref/index.asp](https://www.w3school.com.cn/cssref/index.asp)<br/>
Liquid: [https://learn.microsoft.com/zh-cn/training/modules/power-pages-liquid/basics](https://learn.microsoft.com/zh-cn/training/modules/power-pages-liquid/basics)<br/>
遍历所有博客
```liquid
{%- for post in site.posts -%}
    {{ post.title }}
{%- endfor -%}
```
如果给博客分类，可以遍历所有分类
```liquid
{%- for categories in site.categories -%}
    {{ categories[0] }}
{%- endfor -%}
```
遍历分类下所有博客
```liquid
{%- for categories in site.categories -%}
    {%- for post in categories[1] -%}
        {{ post.title }}
    {%- endfor -%}
{%- endfor -%}
```
可以看出得到的`categories`是一个数组，第一个元素是其名称，第二个元素中包含了其下所有博客。<br/>
此外，`site.categories["Code"]`可以代表"Code"这一个类别，因此假如一个页面的title是"Code"，则可以用`site.categories[page.title]`，其他关键字也是类似，不过使用需要注意的是博客的`page.categories`返回的是一个数组，故需要使用`page.categories[0]`，直接使用会报错，故需要赋值到另一个变量：<br/>
`{%- assign page_cate = page.categories[0] -%}`<br/>
我们可以通过这一特性遍历与当前博客同一类别下的所有博客。
{% endraw %}
# 3 博客导航与公式支持
博客导航需要脚本支持
```html
<head>
  <link rel="stylesheet" href="{{ '/assets/css/style.css?v=' }}">
  <script src="https://code.jquery.com/jquery-3.3.0.min.js" integrity="sha256-RTQy8VOmNlT6b2PIRur37p6JEBZUE7o8wPgMvu18MC4=" crossorigin="anonymous"></script>
  <script src="{{ '/assets/js/main.js' | relative_url }}"></script>
</head>
```
来源于Leap Day主题，脚本文件可以下载于[https://github.com/pages-themes/leap-day/blob/master/assets/js/main.js](https://github.com/pages-themes/leap-day/blob/master/assets/js/main.js)，博客元素要放下`<section>`元素内。 <br/>
公式支持：
```html
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML" type="text/javascript"></script>
```
网络上的给出的脚本有多个，它们显示的公式字体有可能有差异。
# 4 问题解决
Github Pages的页面编译失败，未更新，本地运行以下命令：
```
bundle update --bundler
bundle install
```
Gemfile.lock会更新，提交至Github仓库即可，具体机制未研究。
