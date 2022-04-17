# **重要**

## **边距塌陷/规则、BFC/规则/创建BFC/BFC应用、浮动/规则/清除浮动**

本节需要掌握，边距塌陷是什么？边距塌陷规则有哪些？如何解决边距塌陷？BFC是什么？BFC与普通元素的区别/BFC规则？触发元素为BFC的条件有哪些？BFC在开发中的应用？clear含义是什么？浮动是什么？如何将元素设置为浮动元素？浮动的规则？阻止普通块元素被浮动元素覆盖的方法？浮动的缺点有哪些？清除浮动的方法有哪些？浮动元素有哪些应用？

### **边距塌陷**

- **含义：**

  - margin-collapse 即边距塌陷/边距重叠，在同一BFC区域里的普通块元素垂直方向上的margin折叠（左右margin不会）

- **规则 ：**

  以下分3种情况讨论边距塌陷：相邻上下元素、相邻父子元素、空元素自身

  （千万注意以下讨论相邻的意思都是标签挨着标签，如`<div><div></div></div>`这样才会发生边距塌陷，如果代码是`<div>aaa<div></div></div>`这样是不会发生边距塌陷的，因为插入的aaa实际上是个文本节点造成了不相邻了）

  - **相邻上下元素**发生边距重叠，指上元素设置margin-bottom，下元素设置margin-top，两者之间的margin规则如下：
    - 两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值
    - 两个相邻的外边距都是负数时，折叠结果是两者的绝对值的较大值
    - 两个外边距一正一负时，折叠结果是两者的相加的和
  - **相邻父子**边距塌陷，子元素必须是挨着父元素的顶部或底部（注意文本会生成textNode因此不相邻）
    - 如果子设置了margin-top，子的margin-top会变成父的margin-top，此时子元素贴着父的顶部
    - 如果子设置margin-bottom，且父亲未设置高度或高度为auto，则子的margin-bottom变成父的margin-bottom，子元素贴着父的底部
    - 如果父子都有margin，那取值规则参照上面的相邻上下元素margin合并规则
  - **空元素自身**，且未设置height、border、padding等的情况下，同时设置了margin-bottom和margin-top也会边距塌陷（后面的元素与它的margin最终值的取值规则参照上面）

- **解决边距塌陷/阻止margin折叠：**
  
  - 触发元素为BFC（具体看下面的触发BFC的4种方法）
  - 设置padding使会发生边距塌陷的元素不相邻

### **BFC**

- **含义：**
  
  - BFC(Block Formatting Context)，块级格式化上下文，是一个独立的渲染区域（独立的容器）
- **规则：**
  
  - BFC在页面中是独立的容器，BFC内外的元素之间互相不影响
  - BFC不与外部的元素发生边距重叠（因此可将元素设置为BFC来解决边距重叠），但BFC内部的子元素仍遵守垂直方向边距重叠规则
  - BFC区域的元素不会被浮动元素覆盖（因此可用来清除浮动）
  - 计算BFC高度，浮动子元素也参与计算（是子元素）
- **触发条件：**（可为以下任意一条）
  
  - 根元素<html>
  
  - float !== none（left/right/inherit）
  
  - overflow !== visible（hidden/scroll/auto/inherit）
  
  - position != relative| static ( absolute / fixed / sticky) 
  
  - display = inline-block / table / flex
  
    待解决：table不生效，flex的子元素margin-bottom、margin-top有值时很奇怪
- **应用：**
  
  - 阻止外边距塌陷
  
  - 清除浮动，阻止普通元素被浮动元素覆盖
  
    待解决：清除浮动的原理是两个div都位于同一个BFC区域之中

### **浮动**

- **含义：**

  - 元素的float不为none，即float: left/right/inherit，使元素脱离正常文档流，按照指定的水平方向发生移动，直到碰到父元素的边界或另一个浮动元素边框为止
  - float对设置flex的子元素不起作用

- **规则：**

  - 设置元素浮动后，该元素的display值自动变成block，但是行为是inline-block，可以设置高宽，未设置高宽便由内容物撑开，多个float水平排列
  - 浮动元素会遮住排在它后面块级元素，但不会遮住块级元素里的文本内容，此时文本内容围绕在浮动元素周围（非浮动元素会覆盖浮动元素的位置）
  - 浮动元素不会遮住行内元素或行内块元素，不管该元素里有无内容
  - 浮动元素参与BFC父级的高度计算，但不参与非BFC父级的高度计算（所以会导致父元素高度无法撑开，因此需要清除浮动）

- **清除浮动方法：**（空元素、伪元素、BFC）

  - 给父元素设置height

    - 弊端：父元素高度变成固定值，子元素超过父元素高度时会溢出，且父元素里若有其它子元素，这些子元素还是有可能被浮动元素覆盖

  - 在浮动元素后面添加空元素，且该空元素设置样式clear:both

    - 弊端：产生大量空元素

    ```html
    <div class="container">
        <div class="left"></div>
        <div class="right"></div>
        <p class='clearfix'></p>
    </div>
    .clearfix { clear: both }    // 弊端：产生大量空元素
    ```
    
  - 设置父元素的伪元素::after（推荐）

    ```html
    <div class="container clearfix">
        <div class="left"></div>
        <div class="right"></div>
    </div>
    .clearfix{
        zoom: 1;    //处理ie6兼容
    }
    .clearfix::after{
        content: "";   //需要有content
        clear: both;    //clear表示指定一个元素必须移动到在它前面的浮动元素下面
        display: block;
    }
    ```
    
  - 触发父元素为BFC（推荐）

    ```html
    <div class="container">
        <div class="left"></div>
        <div class="right"></div>
    </div>
    .container{
        overflow:auto;
        zoom:1;   //处理ie6兼容
    }
    ```

- **clear：**
  - clear属性指定一个元素必须移动到在它前面的浮动元素下面。clear属性适用于浮动和非浮动元素

## **布局（圣杯、双飞翼、响应式布局、flex）**

### **圣杯布局**

- 特点：三列，中间主体内容前置且宽度自适应，两边内容定宽

- 好处：重要的内容放在文档流前面可以优先渲染

- 原理：3个都浮动、最外层padding预留、左右负边距移动


```html
<div class="container">
	<div class="main"></div>   //主体内容优先渲染，padding预留左右位置
	<div class="left"></div>
	<div class="right"></div>
</div> 
.container {
    padding-left: 150px;   //外层必须设置padding
    padding-right: 190px;
}
.main {
    float: left;
    width: 100%;
}
.left {
    float: left;
    width: 150px;
    margin-left: -100%;     //left设置负左边距
    position: relative;
    left: -150px;
}
.right {
    float: left;
    width: 190px;
    margin-right: -190px;
}
```


圣杯布局是：外层盒子包裹三个内层盒子，分别是中左右，这样中间主体区最先渲染，外层的盒子设置padding-left和padding-right等于左右盒子的宽度，并且伪元素清除浮动，三个盒子都设置左浮动，中间盒子宽度100%，

左边盒子margin-left = - 100%移动父盒子的百分百宽度，在定位left= - 宽度出去，
右边盒子直接margin-right= -宽度 到右边去

### **双飞翼布局**

- 特点：对圣杯布局（使用相对定位，对以后布局有局限性）的改进，消除相对定位布局

- 原理：主体元素上设置左右边距，预留两翼位置。左右两栏使用浮动和负边距归位，消除相对定位


```html
<div class="main-wrap">
    <div class="main"></div>  //margin预留左右位置
</div>
<div class="left"></div>
<div class="right"></div>
.main-wrap {
    width: 100%;
    float: left;
}
.main {
    margin-left: 150px;
    margin-right: 190px;
}
.left {
    float: left;
    width: 150px;
    margin-left: -100%;
}
.right {
    float: left;
    width: 190px;
    margin-left: -190px;  //不是margin-right:-190px
}
```


双飞翼布局是，三个盒子都是浮动，main用父盒子包裹并设置宽度100%，里面main设置margin-left为左盒子宽度，margin-right为右盒子宽度，然后左盒子设置margin-left负100%正好移动到主体区的左侧，有盒子margin-left负自身宽度到主体区右侧

### **响应式布局**

- 特点：兼容多个终端，而不是为每个终端做一个特定的版本
- 原理：利用CSS3媒体查询，为不同尺寸的设备适配不同样式（如bootstrap的栅格结构），查询媒体并且设置最大和最小设备宽度,然后每级别设置font-size之后的rem会根据这个大小来响应式设置

### **flex布局**

- **特点：**

  - 采用Flex布局的元素称为"容器"（设置display:flex会触发BFC），它的所有子元素自动成为容器成员，称为"项目"。容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis），项目默认沿主轴排列
  - 设置display : flex，块级元素，占据整个宽度，设置display : inline-flex，行内元素，宽度由子元素撑开
  
  - 设为Flex布局以后，子元素的float、clear和vertical-align属性将失效
  
- **容器属性：**
    - `flex-direction`：row | row-reverse | column | column-reverse（主轴/水平排列方向）
    - `flex-wrap`：nowrap | wrap | wrap-reverse（是否换行，如果设为nowrap表示所有子元素挤占一行且平分宽度）
    - `flex-flow`：flex-direction、flex-wrap的简写
    - `justify-content`：flex-start | flex-end | center | space-between | space-around （主轴对齐方式）（space-between：两端贴着边框，项目之间的间隔都相等。space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍）
    
    - `align-items`：flex-start | flex-end | center | baseline | stretch（交叉轴对齐方式、单根主轴)
    - `align-content`：flex-start | flex-end | center | space-between | space-around | stretch（多根轴线的对齐方式）


- **项目属性：**
  
  - `order`：定义项目的排列顺序。数值越小，排列越靠前，默认为0
  - `flex-grow `：平分容器剩余空间的宽度比例（默认为0，表示如果存在剩余空间也不平分）
  - `flex-shrink`：定义项目的缩小比例，默认为 1，即如果空间不足，该项目将缩小
  - `flex-basis`：定义了在分配多余空间之前，占据的主轴空间，默认值auto(项目的本来大小)
  - `flex`：是flex-grow、flex-shrink、flex-basis的简写，默认值为0 1 auto。后两个属性可选，常设置flex:1来表示该元素占据该行的全部剩余空间
  - `align-self`：auto | flex-start | flex-end | center | baseline | stretch，align-self属性允许单个项目有与其他项目在交叉轴方向上不一样的对齐方式，默认值为auto，表示继承父元素的align-items属性，若设为其它值便会覆盖父的align-items属性
  
- **flex实现三栏布局**

  ```html
  <div class="main-wrap">
    <div class="main">main</div>  //使用项目属性order
    <div class="left">left</div>
    <div class="right">right</div>
  </div>
  .main-wrap {
    display: flex;
  }
  .main {
    order: 2;
    flex-grow: 1;   //占据所有剩余空间
    width: 100%;
  }
  .left {
    order: 1;
    width: 100px;
  }
  .right {
    order: 3;
    width: 200px;
  }
  ```

### **场景题**

以下的题目请想出尽可能多的解法并比较优缺点：

- 一个高度自适应的div，里面有两个div，一个高度100px，希望另一个填满剩下的高度(flex-flow:column  flex:1)

- 左边定宽，右边自适应方案( flex:1)
- 设高度默认100px，请写出三栏布局，其中左栏、右栏宽各位300px，中间自适应，需要3种以上的解法(圣杯 双飞翼 flex)


## **水平、垂直居中**

- **水平居中**
  
  - 行内元素：`text-align:center`（且text-align可继承）
  - 块级元素：`margin:auto`（只对块级元素生效， 行内元素只对确定值的左右margin生效）
  
- **水平垂直居中**

  ```css
  //方法一：父元素设置flex布局（所有子元素都会水平垂直居中）
  - PC端无兼容性要求，推荐flex
  - 移动端推荐使用flex
  .father{
      display: flex;
      justify-content: center;
      align-items: center;
  }
  
  //方法二：父相子绝+子translate（可设置只有某一个子元素水平垂直居中）
  .father{
      position: relative;   //一定要设置父元素相对定位
      .son{
         position: absolute;
         top: 50%;
         left: 50%;
         transform: translate(-50%, -50%); 
      }
  }
  
  //方法三：父设置display:table-cell布局，子设置display:inline-block
  .father {
      display: table-cell;
      text-align: center;
      vertical-align: middle;
     .son {
       display: inline-block;
     }
  }
  
  //方法四：父相子绝+子margin:auto
  .father {
    width: 200px;
    height: 200px;
    position: relative;
    text-align: center;  //水平居中，子元素自动继承该属性
    .son {
      width: 100px;
      height: 100px;
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      margin: auto; //对块级元素有效
    }
  }
  
  //方法五：在已知子元素高度的情况下计算值设置父的padding、子的margin
  ```

## **元素（行内/块级）、属性（继承/不可继承、引发重排(回流)/引发重绘、伪类/伪元素）**

- **元素**
  - **行内元素：**
    - 不可以设置宽高（宽高默认由内容物撑开），不独占一行，上下margin不生效（但左右会生效），行内元素只能包含行内元素，不能包含块元素。常见：`a b span img input select strong i`
  - **块级元素：**
    - 可以设置宽高，独占一行，块级元素可以包含块级元素或行内元素（注：p是块元素）。常见：`p div ul ol li dl dt dd h1…h6 `
  - **空元素：**
    - 常见：`<br> <hr> <img> <input> <link> <meta>`
- **属性**
  - **可继承、不可继承：**
    - 可继承：与font相关的如color、cursor、visibility、line-height、text-align..
    - 不可继承的样式：width、height、padding、border、margin...（width/padding/margin值为百分比时是相对父的宽度的，height值为百分比时是相对父的高度的）
  - **引发重排(回流)reflow、引发重绘repaint：**
    -  引发重绘：背景或颜色相关的属性
- **伪类、伪元素**
  - **伪类：**
    - `:`单冒号，已有元素的不同状态，不产生新的元素（a标签的伪类有执行顺序）
  - **伪元素：**
    - `::`双冒号，创建dom中不存在的元素，并为其添加样式，虽然用户可以看见这些文本，但是实际上并不在dom文档中

## **css3新增**

- 新增选择器 `p:nth-child(n){color: rgba(255, 0, 0, 0.75)}`
- ==弹性盒模型 `display: flex;`==
- 多列布局 `column-count: 5;`
- ==媒体查询== `@media (max-width: 480px) {.box: {column-count: 1;}}`
- 个性化字体 `@font-face{font-family: BorderWeb; src:url(BORDERW0.eot);}`
- 颜色透明度 `color: rgba(255, 0, 0, 0.75);`
- 圆角 `border-radius: 5px;`
- 渐变 `background:linear-gradient(red, green, blue);`
- 阴影 `box-shadow:3px 3px 3px rgba(0, 64, 128, 0.3);`
- 倒影 `box-reflect: below 2px;`
- 文字装饰 `text-stroke-color: red;`
- ==文字溢出 `text-overflow:ellipsis;`==
- 背景效果 `background-size: 100px 100px;`
- 边框效果 `border-image:url(bt_blue.png) 0 10;`
- 转换
  - ==旋转 `transform: rotate(20deg);`==
  - ==倾斜 `transform: skew(150deg, -10deg);`==
  - ==位移 `transform: translate(20px, 20px);`==
  - ==缩放 `transform: scale(.5);`==
- 平滑==过渡== `transition: all .3s ease-in .1s;`
- ==动画== `@keyframes anim-1 {50% {border-radius: 50%;}} animation: anim-1 1s;`

`transition`：==定义了元素在变化过程中是怎么样的==，包含`transition-property`、`transition-duration`、`transition-timing-function`、`transition-delay`。

`transform`：==定义元素的变化结果==，包含`rotate`、`scale`、`skew`、`translate`。

`animation`：==动画定义了动作的每一帧（`@keyframes`）有什么效果==，包括`animation-name`，`animation-duration`、`animation-timing-function`、`animation-delay`、`animation-iteration-count`、`animation-direction`

## **css优化**

- 多个css合并，尽量减少http请求
- css雪碧图来减少http网络请求次数
- 将css文件放在页面最上面（script放在body）
- 选择器优化嵌套，尽量避免层级过深
- 充分利用css继承属性，减少代码量
- 抽象提取公共样式，减少代码量
- 属性值为0时，不加单位
- 属性值为小于1的小数时，省略小数点前面的0

# **基础**

## **盒模型**

本节需要掌握，css的盒模型分类及分别包含哪些？浏览器默认是哪一种盒模型？如何设置不同的盒模型？

- **分类：**
  
  - 标准盒子模型：
  
    - width和height指的是content的宽度和高度
    - 增加padding、border、margin不会影响content的尺寸，但还是会增加元素的总尺寸
    - 计算盒子宽度 = `width + padding + border + margin`
  
  - IE盒子模型：
  
    - width和height指的是content+padding+border的宽度和高度
    - 增加padding、border会挤占原本的宽高
    - 计算盒子宽度 = `width + margin`
  
    注：margin不管是哪个盒模型，都不会挤占原本的宽高
  
- **设置：**（可通过box-sizing属性设置盒模型，默认为标准盒模型）
  
  - `box-sizing: content-box`   // 标准盒模型，默认值  （边界是content）
  - `box-sizing: border-box`     // IE盒模型                      （边界是border）

## **优先级**

本节需要掌握，css样式优先级、各个选择器权重、样式穿透

- 优先级：! important（最大） > 内联样式/行内样式（1000） > id选择器（100）> 类选择器 = 伪类选择器 = 属性选择器（10） > 元素选择器 = 伪元素选择器（1） > 通配选择器 = 后代选择器 = 兄弟选择器

  注：可简记为`!important > 行内 > id > class > tag`

  ​       !important优先级最大、第一

  ​       如果权值一样，按照样式规则的先后顺序，顺序靠后的覆盖靠前的规则
  
  ​       修改第三方样式时用样式穿透

## **position、display**

- **position：**
  
  - `static`：默认值，没有定位
  
  - `relative`：相对定位
  
  - `absolute`：绝对定位，相对于距离最近的已定位父元素进行定位（即position !== 'static'）
  
  - `fixed`：固定定位，相对于浏览器窗口进行定位
  
  - `sticky` :   在某个位置前是relative定位，超过阈值后是fixed定位（可以设置距离视口的top、left等为阈值）
  
  - `inherit`：规定从父元素继承position属性的值
  
    注：absolute、fixed、sticky的区别是：absolute是相对于最近的已定位的父元素进行定位，fixed一开始时就固定在视图的某个位置，sticky是超过阈值后才固定住
  
- **display：**
  
  - `none`：设置元素不可见
    
  - `inline`：行内元素，水平排列，宽高由内容撑开
    
  - `block`：块元素，独占一行，可设置宽高或上下margin、paddiig
  
  - `inline-block`：行内块元素
  
  - `list-item`：像块类型元素一样显示，并添加样式列表标记
  
  - `table`：此元素会作为块级表格来显示
  
  - `inherit`：规定应该从父元素继承display属性的值
  
    待解决：设置为行内元素才可以text-align=center，设置table才可以vertical-align=center

## **margin负值**

|     负值      |                     行为（讨论的是负值）                     |
| :-----------: | :----------------------------------------------------------: |
|  margin-top   |                         元素向上拖拽                         |
|  margin-left  |                         元素向左拖拽                         |
| margin-bottom | 元素自身位置不变、下边元素上移（自身高度减少了margin的负值绝对值） |
| margin-right  | 元素自身位置不变、右边元素左移（自身宽度减少了margin的负值绝对值） |

- margin的值设为百分比时是相对于父元素content的宽度
- margin:auto，只对块级元素生效。行内元素只对确定值的左右margin生效，上下margin不生效
- 元素的margin、padding设置为百分比时都是相对于其父元素的宽度，不是自身宽度！
- left、right、top、bottom生效的条件有：
  - 本身的position不能是static
  - 相对最近的position值不为static的父元素进行定位，且left、right、top、bottom值为百分比时是相对该父元素的宽高的百分比

- translate设置百分比时是相对自身的宽高

## **场景题**

### **如何实现小于12px的字体效果**

浏览器默认支持最小字体为12px，小于12px的都按12px显示，若要实现显示小于12px的字体，可通过scale缩放：

```css
display:inline-block;   //若为行内元素，还需设置此行
transform: scale(0.7);
```

### **用纯CSS创建一个三角形的原理是什么**

```css
/* 设置该div长和宽都为0，及border-color的上、左、右三条边隐藏掉（颜色设为 transparent）*/
#demo {
  width: 0;
  height: 0;
  border-width: 20px;
  border-style: solid;
  border-color: transparent transparent red transparent;
}
```

### **图片不超出块区域的设置方法？**

### **超出div的文字省略？**

```css
//单行溢出，超出省略
.desc{
  overflow: hidden; 
  text-overflow: ellipsis; 
  white-space: nowrap; 
}

//多行，超出行数后省略
.desc {
  display: -webkit-box; //将元素设为盒子伸缩模型显示
  -webkit-box-orient: vertical; //伸缩方向设为垂直方向
  -webkit-line-clamp: 3; //超出3行隐藏，并显示省略号
  overflow: hidden;
}
```

### **一个高度自适应的div，里面有两个div，一个高度100px，希望另一个填满剩下的高度**

```css
//方案1：
  .sub { 
      height:calc(100% - 100px);
   }
//方案2：
  .container {
      position:relative; 
  }
  .sub { 
      position: absolute; 
      top: 100px; 
      bottom: 0; 
  }
//方案3：
  .container { 
      display:flex; 
      flex-direction:column;
  }
  .sub { 
      flex:1;
  }
```

### **左边定宽，右边自适应方案**

```css
/* 方案1 */ 
.left {
  width: 120px;
  float: left;
}
.right {
  margin-left: 120px;
}
/* 方案2 */ 
.left {
  width: 120px;
  float: left;
}
.right {	
  width: calc(100% - 120px);
  float: left;
}
```

注：以上的代码待解决

# **else**

canvas、svg的区别？

px、em、rem的区别？（42、85）

em相对于父，rem相对于root

img的alt、title的区别？

strong、em、i、b的区别？

display:none、visibility:hidden的区别？

href、src的区别？

link、@import的区别？

stylus、sass、less的区别？

rgba()和opacity()设置透明效果的区别？

元素竖向的百分比设定是相对于容器的宽度，而不是高度

全屏滚动的原理是什么？ 用到了CSS的那些属性？

什么是视差滚动效果，如何给每页做不同的动画？

li与li之间有看不见的空白间隔是什么原因引起的？有什么解决办法？

v-show的dom元素为它设置为bfc元素(脱离文档流)提高性能,因为v-show设置display:none有无的时候会影起重排重绘,这是消耗性能的

`<p>`是块级元素

div1、div2、div3都设置float:right，此时在右侧排列顺序应该是3、2、1

align-self、align-items、align-content的区别？

innerWidth、outerWidth、clientWidth区别？

避免引起重排重绘，都有哪些措施？添加多个子节点的时候，不要多次调用appendChild，而是使用documentFragment、或使用innerHtml=‘’/<tamplate>。v-show、v-if。浮动元素

title 和 alt 区别：title通常当鼠标滑动到元素上的时候显示。alt是<img>的特有属性，用于图片无法加载时显示s

