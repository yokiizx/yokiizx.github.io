# 经典三栏布局


学习 BFC 时，知道利用 BFC 特性，可以制作两栏自适应布局，这里复习一下经典的三栏布局吧。

### 圣杯布局

重点就是利用 container 的 padding 来给杯壁空间。

关键代码如下：

```html
<style>
    /* padding 留下空位 */
    .container {
        width: 100%;
        height: 100%;
        padding: 0 200px;
        box-sizing: border-box;
    }

    /* 中间设置100%宽度 */
    .m {
        width: 100%;
        height: 100%;
        background-color: red;
        word-break: break-all;
    }

    /* 左右设置固定宽度 */
    .l,
    .r {
        width: 200px;
        height: 100%;
        background-color: yellowgreen;
    }

    /* 脱离文档流 确定定位 */
    .l,
    .r,
    .m {
        position: relative;
        float: left;
    }

    /* 左右偏移到占位 */
    .l {
        left: -200px;
        margin-left: -100%;
    }

    .r {
        right: -200px;
        margin-left: -200px;
    }
</style>
<!-- dom 结构 -->
<div class="container">
    <div class="m"></div>
    <div class="l"></div>
    <div class="r"></div>
</div>
```

1. container 提供 padding 占位
2. float: left 脱离文档流
3. margin-left 偏移进 container 内，relative left 偏移进 padding 占位

### 双飞翼布局

重点就是利用 container 内部 m 的 margin 来给翅膀空间。

```html
<style>
    /* 中间 container 撑满占位 */
    .container {
        width: 100%;
        height: 100%;
    }

    /* 内部依靠 margin 来占位 (千万别设置宽度哦)*/
    .m {
        height: 100%;
        margin: 0 200px;
        background-color: red;
        word-break: break-all;
    }

    /* 左右设置固定宽度 */
    .l,
    .r {
        width: 200px;
        height: 100%;
        background-color: yellowgreen;
    }

    /* 脱离文档流 */
    .l,
    .r,
    .container {
        float: left;
    }

    /* 左右偏移到占位 */
    .l {
        margin-left: -100%;
    }
    .r {
        margin-left: -200px;
    }
</style>
<!-- dom 结构 -->
<div class="container">
    <div class="m"></div>
</div>
<div class="l"></div>
<div class="r"></div>
```

1. container 内部 m 使用 margin 来占位
2. float: left 脱离文档流
3. 左右通过 margin-left 来偏移到正确位置

相比下来，双飞翼不需要定位来做偏移，css 更加便捷一些。

### flex 布局

这个常用，就短说吧，中间 `flex: 1` 即可。

