# 前言

多行文本超出高度限制出现省略号 , 移动端多为webkit内核的 , 有扩展属性-webkit-line-clamp , 但并不是CSS规范中的属性 , PC端往往要借助js去实现这一效果，但麻烦且不是很靠谱，今天就用纯CSS来实现一个完全兼容的多行省略。

# 正文

## 一、webkit内核的实现

`-webkit-line-clamp`设置块元素包含的文本行数；
`display: -webkit-box`设置块元素的布局为伸缩布局；
`-webkit-box-orient`设置伸缩项的布局方向；
`text-overflow: ellipsis;`则表示超出盒子的部分使用省略号表示。

```html
<div style = 'width:400px;
              height:70px;
              border:1px solid red;'>
           <p style='display:
                     -webkit-box;//对象作为弹性伸缩盒子模型显示 
                     -webkit-box-orient: vertical;//设置或检索伸缩盒对象的子元素的排列方式 
                     -webkit-line-clamp: 2;//溢出省略的界限
                     overflow:hidden;//设置隐藏溢出元素'>
                 这是一些文本这是一些文本这是一些文本这是一些文本这是一些文本这是一些文本这是一些文本这是一些文本这是一些文本这是一些文本
         </p>
</div>
```



## 二、非webkit内核实现

> 不过本文将要介绍的方法是`采用CSS规范中的属性`，并结合特殊的实现技巧完成的。这意味着在实现CSS2.1规范的浏览器中都是可以兼容的，不将仅仅是纯粹的移动端领域，在传统的PC浏览器（你们懂得我指的是哪些浏览器）中仍是可行的。好吧，让我们一起见识下。

一共7个步骤，最简单的就是截断文本，最难的部分则是`让一个元素出在其父块的溢出时的右下方`，并且当父元素未溢出时该元素消失不可见。（`代码可直接粘贴看效果`）

### 1、基本结构布局

最简单的开始：当父包含框比较小时，将子元素布局到父包含框的右下角；
HTML

```html
    <div class="wrap">
        <div class="prop">
            1.prop
        </div>
        <div class="main">
            2.main 这里是主题内容，多行省略
        </div>
        <div class="end">
            3.end
        </div>
    </div>
```

CSS

```css
.wrap {
    width: 400px;
    height: 50px;
    margin: 20px 20px 50px 300px;
    border: 5px solid #AAA;
    line-height: 25px;
}
.prop {
    float: left;
    width: 50px;
    height: 50px;
    background: #FAF;
}
.main {
    float: right;
    width: 350px;
    background: #AFF;
    word-break: break-all;
}
.end {
    position: relative;
    float: right;
    width: 50px;
    background: #FFA;
}
```

**效果如下面几张图（`注意黄色end的位置`）, 其实这个实现完全利用了元素浮动的基本规则。**

![clipboard.png](https://segmentfault.com/img/bVLA53?w=417&h=69)

![clipboard.png](https://segmentfault.com/img/bVLA6t?w=426&h=122)

### 2、定位模拟'...'

我们通过创建一个子元素来替代将要显示的省略号，当本文溢出的情形下该元素显示在正确的位置上。在接下来的实现中，我们创建一个realend元素，利用上一步骤中end元素浮动后的位置来实现realEnd元素的定位。
HTML

```html
    <div class="wrap">
        <div class="prop">1.prop</div>
        <div class="main">
            2.main 这里是要多行文本溢出省略的，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点
        </div>
        <div class="end">
            3.end
            <div class="realEnd">realEnd</div>
        </div>
    </div>
```

CSS

```css
// 新增下面样式即可
.realEnd {
    position: absolute;
    width: 100%;
    height: 25px;
    top: -25px; /*等于高度的负值，就是文字的行高*/
    left: 350px;
    background: #FAA;
    font-size: 13px;
}
```

**效果如图下图**

![clipboard.png](https://segmentfault.com/img/bVLBbo?w=431&h=119)

若父元素并没有溢出，那么realend元素会出现在其右侧(设置wrap overflow：hidden即可，下面会解决)

![clipboard.png](https://segmentfault.com/img/bVLBbO?w=792&h=133)

### 3、优化定位

第二步中我们针对end元素设置了相对定位，对realEnd元素设置了绝对定位。但是我们可以采用更简单的代码来实现，即只使用相对定位。熟悉定位模型的同学应该知道，相对定位的元素仍然占据文本流，同时针对元素设置偏移。这样， 就可以去掉end元素了，仅针对realEnd元素设置相对定位。
HTML

```html
    <div class="wrap">
        <div class="prop">1.prop</div>
        <div class="main">
            2.main 这里是要多行文本溢出省略的，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点
        </div>
        <div class="realEnd">realEnd</div>
    </div>
```

CSS

```css
// 删除end样式，realEnd样式更改如下
.realEnd {
    float: right;
    position: relative;
    width: 50px;
    height: 25px;
    top: -25px; /*等于高度的负值，就是文字的行高*/
    left: 350px;
    background: #FAA;
    font-size: 13px;
}
```

**效果如图（`黄色的end已经没了`）**

![clipboard.png](https://segmentfault.com/img/bVLBee?w=457&h=130)

### 4、削窄prop元素

目前，最左侧的prop元素的作用在于让realend元素在文本溢出时处在其正下方，在前几节的示例代码中为了直观的演示，设置prop元素的宽度为50px，那么现在为了更好的模拟实际的效果，我们缩小prop元素的宽度。

CSS

```css
// 更改部分样式如下
.prop {
    float: left;
    width: 5px;/*缩小宽度为5px，其余属性不变*/
    height: 50px;
    background: #FAF;
}
.main {
    float: right;
    width: 400px;
    margin-left: -5px;/*让main元素左移5px，这样main元素在宽度上就完全占满了父元素；*/
    background: #AFF;
    word-break: break-all;
}
.realEnd {
    float: right;
    position: relative;
    width: 50px;
    height: 25px;
    top: -25px; /*等于高度的负值，就是文字的行高*/
    left: 400px;
    /*而设置margin-left: -50px、padding-right: 5px则是为了让realend元素的盒模型的最终宽度计算为5px。*/
    margin-left: -50px;
    padding-right: 5px;
    background: #FAA;
    font-size: 13px;
}
```

> 由于CSS规范规定padding的值不可以为负数，因此只有设置margind-left为负值，`且等于其宽度`。这样做的最终目的就是保证realend元素处在prop元素的下方，保证在文本溢出的情况下定位准确性:

**效果图如下**

![clipboard.png](https://segmentfault.com/img/bVLBlq?w=423&h=122)

### 5、流式布局 + 伪元素

目前，realend元素的相关属性仍采用px度量，为了更好的扩展性，可以改用%替代。

同时，prop元素和realend元素可以采用伪元素来实现，减少额外标签的使用。

CSS

```css
.mutil-line-ellipsis {
    width: 400px;
    height: 50px;
    margin: 20px 20px 50px 300px;
    border: 5px solid #AAA;
    line-height: 25px;
    /*overflow: hidden;*/
}
/*相当于之前的prop*/
.mutil-line-ellipsis:before {
    content: '';
    float: left;
    width: 5px;/*缩小宽度为5px，其余属性不变*/
    height: 50px;
    background: #FAF;
}
/*相当于之前的main*/
.mutil-line-ellipsis > :first-child {
    float: right;
    width: 100%;
    margin-left: -5px;/*让main元素左移5px，这样main元素在宽度上就完全占满了父元素；*/
    background: #AFF;
    word-break: break-all;
}
/*相当于之前的realEnd*/
.mutil-line-ellipsis:after {
    content: 'realEnd';
    float: right;
    position: relative;
    width: 50px;
    height: 25px;
    top: -25px; /*等于高度的负值，就是文字的行高*/
    left: 100%;
    /*而设置margin-left: -50px、padding-right: 5px则是为了让realend元素的盒模型的最终宽度计算为5px。*/
    margin-left: -50px;
    padding-right: 5px;
    background: #FAA;
    font-size: 13px;
}
```

HTML

```html
<div class="mutil-line-ellipsis">
    <div>
        2.main 这里是要多行文本溢出省略的，内容多一点，内容多一点，内容多一点，内容多一点
        内容多一点，内容多一点，内容多一点，内容多一点，内容多一点，内容多一点
    </div>
</div>
```

效果和上一步一样

### 6、隐藏

之前的实现中在文本未溢出的情况下，realEnd元素会出现在父元素的右侧，正如
![clipboard.png](https://segmentfault.com/img/bVLHXc?w=882&h=105)
设置
CSS

```css
.mutil-line-ellipsis {
     overflow: hidden;   
}
```

### 7、美化

现在我们离完结就差一步了，即去掉各元素的背景色，并且用“...”替换文本。最后为了优化体验，采用渐变来隐藏“...”覆盖的文本，（省略了一些兼容性的属性）。
CSS

```css
.mutil-line-ellipsis {
    width: 400px;
    height: 50px;
    line-height: 25px;
    margin: 20px 20px 50px 300px;
    border: 5px solid #AAA;
    line-height: 25px;
    overflow: hidden;
}
/*相当于之前的prop*/
.mutil-line-ellipsis:before {
    content: '';
    float: left;
    width: 5px;/*缩小宽度为5px，其余属性不变*/
    height: 50px;
}
/*相当于之前的main*/
.mutil-line-ellipsis > :first-child {
    float: right;
    width: 100%;
    margin-left: -5px;/*让main元素左移5px，这样main元素在宽度上就完全占满了父元素；*/
    word-break: break-all;
}
/*相当于之前的realEnd*/
.mutil-line-ellipsis:after {
    content: '...';
    box-sizing: content-box;
    float: right;
    position: relative;
    width: 50px;
    height: 25px;
    top: -25px; /*等于高度的负值，就是文字的行高*/
    left: 100%;
    /*而设置margin-left: -50px、padding-right: 5px则是为了让realend元素的盒模型的最终宽度计算为5px。*/
    margin-left: -50px;
    padding-right: 5px;
    font-size: 13px;
    text-align: right;
    background: linear-gradient(to right, rgba(255, 255, 255, 0), #ffffff 40px);
}
```

效果（自己可调整省略样式）

![clipboard.png](https://segmentfault.com/img/bVLHZu?w=434&h=79)

# 总结之兼容性

从上文的实现细节来看，我们利用的技巧完全是CSS规范中的浮动+定位+盒模型宽度计算，唯一存在兼容性问题的在于无关痛痒的渐变实现，因此可以在大多数浏览器下进行尝试。