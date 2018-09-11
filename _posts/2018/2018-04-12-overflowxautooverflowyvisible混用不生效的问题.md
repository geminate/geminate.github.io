---
layout: post
title: overflow-x:auto;overflow-y:visible;混用不生效的问题
categories: Css
---

今天在实现表格中的功能按钮时遇到了一个问题

由于这个表格的宽度比较大，因此表格外部的wrapper div设置了 `overflow-x:auto` 如下图：

![ov001.png](https://geminate.github.io/assets/images/2018/ov001.png)

之后由于表格内部最后一列有个按钮，点击后会弹出一个下拉菜单，但是弹出后由于菜单较长会导致 wrapper 出现纵向滚动条，如下图：

![ov002.png](https://geminate.github.io/assets/images/2018/ov002.png)

期望效果：

![ov003.png](https://geminate.github.io/assets/images/2018/ov003.png)

因此我在wrapper div上又设置了 `overflow-y：visible` 发现没有出现应有的效果。代码：

{% highlight html %}
<div style="overflow-x: auto;overflow-y: visible">
    <table style=" width: 180%;">
        <tbody>
        <!-- 有多个如下的TR -->
        <tr>
            <td></td>
            <td></td>
            <td></td>
            <td>
                <div style="position: relative;">
                    <button>Button</button>
                    <ul style="position: absolute;left: 0px;top: 50%;">
                        <li>1</li>
                        <li>2</li>
                        <li>3</li>
                        <li>4</li>
                        <li>5</li>
                    </ul>
                </div>
            </td>
        </tr>
        </tbody>
    </table>
</div>
{% endhighlight %}

打开浏览器一看，发现overflow-y被计算成了auto

![ov004.png](https://geminate.github.io/assets/images/2018/ov004.png)

后到网上查询相关资料，W3C规定如果 `overflow-x` 和 `overflow-y` 中的一个被赋为 `'visible'` ，而另一个被赋值为 `'scroll'` 或 `'auto'` ，那么 `'visible'` 会被重置为 `'auto'` 。

因此要想实现类似图中的效果，需要使点击按钮的出现的菜单添加在外层元素上(例如body)，点击时获取按钮的位置，通过js计算按钮的绝对定位位置。bootstrap 的 Tooltips 组件就是用的这种方式。