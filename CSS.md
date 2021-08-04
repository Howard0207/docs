# CSS

## CSS mix-blend-mode滤色screen混合模式

### 滤色screen混合模式速览

`screen` 混合模式， 计算公式为： $C = 255 - (255 - A)*(255- B)/255$

![滤色模式的计算公式](CSS.assets/screen-math.png)

eg:

有一个红色，其RGB值是（255,0,0），还有一个蓝色，其RGB值是（0，0，255），则这两个颜色的混合模式之后的颜色为：

R: 255 - （255 - 255）* （255 - 0） / 255 = 255

G: 255 - （255 - 0） * （255 - 0） / 255 = 0

B: 255 - （255 - 0） * （255 - 255） / 255 = 255；

最终混合后的颜色如下：

<div style="width:150px;height:120px;background:RGB(255,0,0)">
<div style="height:100%;background:RGB(0,0,255);mix-blend-mode:screen"></div>
</div>

### 滤色模式的特性与web应用

1. 任何颜色与黑色执行滤色，还是原来的颜色；
2. 任何颜色和白色执行滤色得到的是白色；
3. 任何颜色和其他颜色执行滤色，混合后颜色会更浅，类似漂白效果。



## 伪类匹配列表数目实现微信群头像CSS布局的技巧

### 不同列表数量不同布局

聊天软件中的群头像，往往采用复合头像作为一个大的头像。

<img src="CSS.assets/2019-03-11_203856.png" alt="复合头像"  />



### 伪类实现技巧

在这个方法中，你不需要在父元素上设置当前列表个数，因此，HTML看起来平平无奇：

```html
<ul class="box">
    <li></li>
    <li></li>
    <li></li>
    ...
</ul>
```

关键在于CSS ， 我们可以借助伪类判断当前列表个数， 示意如下：

```css
li:only-child{ /* 1个 */}
li:first-child:nth-last-child(2) { /* 2个 */ }
li:first-child:nth-last-child(3) { /* 3个 */ }
```

在CSS中，伪类可以级联使用的，于是，如果列表可以匹配`:first-child:nth-last-child(2)`则表示当前`<li>`元素即是第一个子元素，优势从后往前的第2个元素，因此，我们就能判断当前总共两个 `<li>`子元素， 我们就能精准实现我们想要的布局了，只需要配合相邻兄弟选择符加号`+`以及兄弟选择符`~`即可，例如：

```css
/* 3个li项目的第1个列表项 */
li:first-child:nth-last-child(3) {}
/* 3个li项目的第1个列表项的后一个，也就是第2项的样式 */
li:first-child:nth-last-child(3) + li {}
/* 3个li项目的第一个列表项后面两个列表项，也就是第2项和第3项的样式 */
li:first-child:nth-last-child(3) ~ li {}
```

​																				[查看Demo](https://www.zhangxinxu.com/study/201903/css-nth-last-child-group-avatar-demo.php)

### 延伸与拓展：文字多字号自动变小

和上面类似的原理，我们可以实现文字内容多，字号自动变小的效果，不过我们需要把所有的字符使用一个标签包裹起来，剩下的事情就全部交给CSS就好了。

​																				[查看Demo](https://www.zhangxinxu.com/study/201903/css-font-size-auto-demo.php)

```css
.box span {
    font-size: 40px;    
}
/* 字符个数 大于13个 */
.box span:first-child:nth-last-child(n+13),
.box span:first-child:nth-last-child(n+13) ~ span {
    font-size: 30px;    
}
/* 字符个数大于17个 */
.box span:first-child:nth-last-child(n+17),
.box span:first-child:nth-last-child(n+17) ~ span {
    font-size: 20px;    
}
/* 字符个数大于25个 */
.box span:first-child:nth-last-child(n+25),
.box span:first-child:nth-last-child(n+25) ~ span {
    font-size: 14px;    
}


```

​															**本方法IE9+都支持，放心使用。**



## CSS :placeholder-shown伪类实现Material Design占位符交互效果

### Material Design规范中占位符交互效果

![Material Design占位符交互截屏示意](CSS.assets/placeholder-shown.gif)



这种设计可以借助 CSS `:placeholder-shown`伪类，纯CSS ，无任何JS ，实现这样的占位符交互效果。

> :placeholder-shown表示，当输入框的placeholder内容显示的时候，输入框如何如何。

兼容性：

![image-20210804111628572](CSS.assets/image-20210804111628572.png)

​																				[查看Demo](https://www.zhangxinxu.com/study/201812/placeholder-shown-label-transition-demo.php)



#### 实现原理

```html
<div class="input-fill-x">
    <input class="input-fill" placeholder="邮箱">
    <label class="input-label">邮箱</label>
</div>
```

<b>首先</b>，让浏览器默认的placeholder效果不可见， 我们可以让颜色透明即可，如下CSS

```CSS
/* 默认placeholder颜色透明不可见 */
.input-fill:placeholder-shown::placeholder {
    color: transparent;
}
```

<b>然后</b>，后面的`.input-label`这个label元素代替成为我们肉眼看到的占位符。我们可以采用绝对定位：

```css
.input-fill-x {
    position: relative;
}
.input-label {
    position: absolute;
    left: 16px; top: 14px;
    pointer-events: none;
}
```

<b>最后</b>，对这个label元素在输入框focus的时候，以及非placeholder显示的时候进行重定位（缩小，位移到上方）

```css
.input-fill:not(:placeholder-shown) ~ .input-label,
.input-fill:focus ~ .input-label {
    transform: scale(0.75) translate(0, -32px);
}
```

​											[Input Material Design占位符交互效果 (codepen.io)](https://codepen.io/xxsn/project/editor/ZdJxWz)



## CSS 实现抛物线运动效果

先上效果： [Demo](https://www.zhangxinxu.com/study/201808/css3-parabola-shopping.php)

抛物线效果的核心就是CSS代码实现的。原理如下：

抛物线运动元素使用至少内外两层标签，例如，demo中的抛物线运动物体是CSS世界这本书的缩略图，外层是`<div>`，里面是`<img>`图片：

```html
<div class="fly-item">
    <img src="xx.jpg" />
</div>
```

然后内外两次标签一个负责水平方向的translate移动， 一个负责垂直方向的translate移动， 然后使用不同的缓动函数，也就是使用不同的`timing-function`， 在CSS3`animation`动画效果中是`animation-timing-function`属性，在CSS3`transition`过渡效果中是`transition-timing-function`属性，demo使用的是`transition`过渡，因此，CSS代码如下：

```css
.fly-item {
    /* 水平移动，线性匀速 */
    transition-timing-function: linear;
}
.fly-item > img {
    /* 垂直移动，先慢后快 */
    transition-timing-function: cubic-bezier(.55,0,.85,.36);
}
```

然后同时执行`translate`移动，抛物线效果就出现了。




## 渐变虚框及边框滚动动画的纯CSS实现

### 渐变虚线边框的实现

如果对边框的样式细节不是很在意，可以借助反向镂空的方法实现， 也就是虚线原本实色的地方和周围颜色融为一体，看上去透明，而原来虚框透明的部分透出渐变背景色，于是看上去是渐变色。

如：

```html
<div class="box">
    <div class="content"></div>
</div>
```

```css
.box {
    width: 150px;
    border: 2px dashed #fff;
    background: linear-gradient(to bottom, #34538b, #cd0000);
    background-origin: border-box;
}
.content {
    height: 100px;
    background-color: #fff;
}
```

这种实现，兼容性很好，但是虚实比例反过来了虚线太稀疏，边角无法形成直角。



### 借助CSS遮罩实现精致的渐变虚框

这个方法HTTML只需要一层标签即可，而且没有荣誉的唇色覆盖，适用于各种背景场合， HTML代码如下：

```html
<div class="box">
</div>
```

CSS代码如下，渐进增强，不支持遮罩的浏览器还是纯色虚框：

```css
.box {
    width: 200px;
    height: 150px;
    border: 2px dashed #cd0000;
    box-sizing: border-box;
}
@supports (-webkit-mask: none) or (mask: none) {
  .box {
    border: none;
    background: linear-gradient(to bottom, #34538b, #cd0000) no-repeat;
    mask: linear-gradient(to right, #000 6px, transparent 6px) repeat-x,
          linear-gradient(to bottom, #000 6px, transparent 6px) repeat-y,
          linear-gradient(to right, #000 6px, transparent 6px) repeat-x 0 100%,
          linear-gradient(to bottom, #000 6px, transparent 6px) repeat-y 100% 0;
    mask-size: 8px 2px, 2px 8px, 8px 2px, 2px 8px;
  }    
}
```

​																				[查看Demo](https://www.zhangxinxu.com/study/201808/border-dashed-gradient.php)

#### 关于CSS遮罩

默认情况下，CSS遮罩可以让元素只显示遮罩图片有颜色部分区域，于是，这里，我们只要使用`mask`属性绘制一个黑色虚框，就能实现真正意义上的渐变虚框效果了。



### 虚线边框滚动动画

```html
<div class="box">
    <div class="content">
        占位内容
    </div>
</div>
```

```css
.box {
    width: 200px;
    background: repeating-linear-gradient(135deg, transparent, transparent 3px, #000 3px, #000 8px);
    animation: shine 1s infinite linear;
    overflow: hidden;
}
.content {
    height: 128px;
    margin: 1px; padding: 10px;
    background-color: #fff;    
}
@keyframes shine {
    0% { background-position: -1px -1px;}
    100% { background-position: -12px -12px;}
}
```

​																					[查看Demo](https://www.zhangxinxu.com/study/201808/border-dashed-around-animation.php)

实现原理：

这种边框跑马灯一样的效果的实现原理，可以参见 **使用CSS实现Photoshop选取效果**一文。[link](https://www.zhangxinxu.com/wordpress/?p=895)

边框区域镂空，然后背景图片设为下面这个GIF平铺背景即可：

<img src="CSS.assets/selection-big.gif" alt="10倍大小的选区动画图片" style="zoom:25%" />

例如下面这个水果的选取背景效果：

![image-20210804175307955](CSS.assets/image-20210804175307955.png)



### 一个实线边框loading动画

先看效果吧，[查看Demo](https://www.zhangxinxu.com/study/201808/border-solid-loading-animation.php)

实现的效果是一条边框实线，像一个贪吃蛇一样，一直围着这个图片元素跑跑跑。

实现原理：

使用CSS `clip`属性对边框进行裁剪而已， 使用`clip`属性是因为兼容性较好，如果项目不兼容IE，则可以使用`clip-path`属性来进行裁剪。

具体如下：

```html
<div class="box">
    <img src="mm.jpg" width='123' height="90" />
</div>
```

```css
.box {
    display: inline-block;
    padding: 10px;
    position: relative;
}
.box::before {
    content: '';
    position: absolute;
    left: 0; top: 0; right: 0; bottom: 0;
    border: 2px solid #cd0000;
    animation: borderAround 1.5s infinite linear;    
}
@keyframes borderAround {
    0%, 100% { clip: rect(0 148px 2px 0); }
    25% { clip: rect(0 148px 116px 146px); }
    50% { clip: rect(114px 148px 116px 0); }
    75% { clip: rect(0 2px 116px 0); }
}
```



















































