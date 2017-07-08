---
title: 基于canvas实现图像处理
date: 2017-07-05 15:57:05
mathjax: true
tags:
- 图像处理
- HTML5
- canvas
categories: 
- js
- HTML5
---
HTML5新增加的最重要的一个元素，我认为非`canvas`莫属了。canvas就相当于一个画布，在这上面可以随意进行绘图，还可以显示3D场景和模型。通过`canvas`可以获取`img`的像素信息，用来做图像处理自然也不在话下了。
<!-- more -->
## canvas元素
`<canvas>`看起来和`<img>`元素很相像，唯一的不同就是它并没有`src`和`alt`属性。实际上，`<canvas>`标签只有两个属性——`width`和`height`。可以通过`DOM`或`CSS`设置。`<canvas>`本质上和其他元素也没有什么不同，依然可以使用`CSS`设置其他样式。要注意的是，如果不想内容被拉伸变形，应该尽量设置同比例的宽高。
> 当没有设置宽度和高度的时候，`canvas`会初始化宽度为300像素和高度为150像素。该元素可以使用`CSS`来定义大小，但在绘制时图像会伸缩以适应它的框架尺寸：如果`CSS`的尺寸与初始画布的比例不一致，它会出现扭曲。

### 替换内容
`canvas`只在现代浏览器中支持，如IE8以下不兼容，常用备用的替换内容作为提示或备选方案。原理是支持`canvas`的浏览器会忽略`canvas`标签内的内容，并且只正常渲染`canvas`，而不支持的浏览器会渲染`canvas`的内容，如下面所示，提示文字只有在不支持`canvas`的浏览器中才会显示。
```` html
<canvas>你的浏览器不支持canvas<canvas>
````
### 渲染上下文
每个`canvas`元素都有一个对应的`context`对象（上下文对象），Canvas的API定义在这个`context`对象上面，所以需要获取这个对象，方法是使用`getContext`方法。
```` javascript
var canvas = document.getElementById('canvasID');
var ctx = canvas.getContext('2d');
````
`getContext`只有一个参数，就是标示上下文的格式，如2D图像使用`“2d”`。如果参数是`“3d”`，就表示用于生成3D图像，这部分实际上单独叫做`WebGL API`。通过检查`getContext`方法存在性，也可以检查`canvas`的特性支持
```` javascript
var canvas = document.getElementById('canvasID');

if (canvas.getContext){
    var ctx = canvas.getContext('2d');
    // 渲染canvas 代码
} else {
    // 不兼容的代码
}
````
### 与图像处理相关的API
本文只着重介绍与图像处理的API，其他API参考MDN的[Canvas教程](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial "Canvas教程")。`canvas`拥有强大的图像操作能力，常用的有以下方法：
#### drawImage()
使用`drawImage`，`canvas`可以将图像源插入画布，在`canvas`上进行重绘。`drawImage`语法
```` javascript
void ctx.drawImage(image, dx, dy);
void ctx.drawImage(image, dx, dy, dWidth, dHeight);
void ctx.drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight);
````
- image
`image`是图像源，即要被绘制到`canvas`上的对象。可以是`HTMLImageElement`、`HTMLVideoElement`、`HTMLCanvasElement`、`ImageBitmap`中的任何一种，意味着页面是页面上的`<img>`、`<video>`以及`<canvas>`元素，也可以使用脚本创建的，如`Image()`构造函数创建的`HTMLImageElement`对象。
- dx
图像源的左上角在目标`canvas`上X轴的位置
- dy
图像源的左上角在目标`canvas`上Y轴的位置
- dWidth
图像源在`canvas`上的宽度，会对图像进行缩放。如果不说明则绘制的图片源宽度不会缩放
- dHeight
图像源在`canvas`上的高度，会对图像进行缩放。如果不说明则绘制的图片源高度不会缩放
- sx
图像源矩阵选择框左上角x坐标
- sy
图像源矩阵选择框左上角y坐标
- sWidth
图像源矩阵选择框宽度，默认是sx到图像右下角的X轴距离
- sHeight
图像源矩阵选择框高度，默认是sy到图像右下角的Y轴距离

是不是有点绕，其实就是在图像源上通过一个矩阵框选择部分放置到`canvas`上的一个矩阵框中，两个矩形框都是通过左上角的坐标(x,y)和矩形宽高决定。参考下面的示意图，需要注意的是设置`dWidth`和`dHeight`会使图像缩放而导致变形。
![drawImage示意图](https://mdn.mozillademos.org/files/225/Canvas_drawimage.jpg)
下面是一个9个参数的demo，黑色框是`canvas`的边框。
<iframe width="100%" height="500" src="//jsrun.net/PzYKp/embedded/result/dark/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

#### getImageData()
`getImageData()`方法可以获得`canvas`的内容，返回一个`imageData`对象。
````javascript
ImageData ctx.getImageData(sx, sy, sw, sh);
````
`imageData`对象，只包含`width`、`height`和`data`属性。`data`它的值是一个一维数组。该数组的值，依次是每个像素的（R）、绿（G）、蓝（B）、不透明度（alpha通道 A）的值，每个值的范围是0–255。因此该数组的长度等于图像的像素宽度 x 图像的像素高度 x 4。这个数组不仅可读，而且可写，因此通过操作这个数组的值，就可以达到操作图像的目的。
> 同`drawImage()`一样，`getImageData()`方法的四个参数分别表示要获取的矩阵框大小的内容，`sx`，`sy`表示矩阵框的起始位置，`sw`, `sh`分别表示矩形框的宽高。

#### putImageData()
`putImageData()`是与`getImageData()`相反的操作，`putImageData()`把一个`imageData`对象绘制到`canvas`中
````javascript
void ctx.putImageData(imagedata, dx, dy);
void ctx.putImageData(imagedata, dx, dy, sx, sy, sWidth, sHeight);
````
> `dx`，`dy`表示`imageData`对象在`canvas`中绘制的起始点，`sx`, `sy`, `sWidth`, `sHeight`确定要绘制在`canvas`上的`imageData`对象的矩形大小

## 图像处理实现
网页图片为RGBA模式，即每一像素分别由红（R）、绿（G）、蓝（B）、不透明度（alpha通道 A）构成，每个值有256种（0-255）。根据一定的算法改变每个像素的值，生成的新图像就有了相应的变化。本文使用两种类型的算法，一种是像素处理是独立的，另一种是每个像素点处理结果与周围像素点相关的滤波处理。
本文的算法主要参考以下两篇博文
[Image Filters with Canvas](https://www.html5rocks.com/en/tutorials/canvas/imagefilters/ "Image Filters with Canvas")
[图像卷积与滤波的一些知识点](http://blog.csdn.net/zouxy09/article/details/49080029 "图像卷积与滤波的一些知识点")
### 简单图像处理
我们先看看简单的图像处理，一般使用一个公式遍历处理每个像素点就可以了，如**灰度**

灰度就是去色。不考虑A值，一般的处理方法是将图片颜色值的RGB三个通道值设为一样，这样原本的256\*256\*256种颜色就只有256种了，即只剩下亮度值。一般有三种算法：最大值、平均值、加权平均。这里我们用加权法：一般由于人眼对不同颜色的敏感度不一样，所以三种颜色值的权重不一样，一般来说绿色最高，红色其次，蓝色最低，最合理的取值分别为30％，59％，11％。
````javascript
//灰度
greyScale(imgData) {
    //图片像素数据
    imgData = imgData || this.getImageData();
    let data = imgData.data,
        r, g, b, v;
    for(let i = 0, len = data.length; i < len; i += 4){
        r = data[i]; g = data[i + 1]; b = data[i + 2];
        //加权取值
        v = .299 * r + .587 * g + .114 * b;
        data[i] = data[i + 1] = data[i + 2] = v;
    }
    return imgData;
}
````
原图
![原图](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/origin.png)
应用灰度去色后效果
![灰度](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/grey.png)


### 线性滤波与矩阵卷积
前面介绍的处理都是每个像素独立的处理，实际上我们还可以把每个像素周围的像素信息也用起来做线性叠加操作，这就是线性滤波。线性滤波可以说是图像处理最基本的方法，它可以允许我们对图像进行处理，产生很多不同的效果。中间像素和它的领域像素的线性叠加可以用一个矩阵（卷积核）来表示，如图是应用一个二维3\*3滤波器对二维图像进行处理的示意图。如图，对源图像中的每一个像素点和该点周围8个像素共9个点，与卷积核对应相乘并累加，最终的值作为中心点的新值，这就是卷积。
![矩阵计算](http://img.blog.csdn.net/20151012211045222?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

> 矩阵卷积过程中，每次计算新的像素值，使用的都是源图像的像素值，而不是滤波后的值。
> 一般来说为了保证有中心点，矩阵应该是奇数阶。
> 需要保证原像素的亮度滤波前后亮度一致时，则矩阵核和应该为0

#### 边界处理
因为矩阵核每次计算只能得到中心的值，如果遇到在边界的像素，怎么办？一般的简单处理方法有4种：
- 填充0
不够的像素，用0来填充
- 用边界像素值拓展
这种方案是认为图像是无限大的，我们使用的只是一部分，边界外的像素值与边界的值是近似的。出于这种思想，我们自然可以用边界值代替
- 周期拓展
周期拓展是认为图像是像平铺一样周期性重复的，左边界与右边界相接，上边界与下边界相邻。
- 不处理

#### 矩阵核应用
为了应用矩阵核，我们编写一个函数处理
````javascript
/**
 * 卷积核应用
 *
 * @param    {array}   mat          卷积矩阵，一维数组表示
 * @param    {object}  imgData      要处理的imageData对象
 * @param    {number}  divisor      可选，对卷积后数值归一化系数，默认为1，
 * @param    {number}  order        可选，卷积核的阶数，默认为3
 * @returns  {object}  outputImg    处理后的imageData对象
 *
 */
conv(mat, imgData, order = 3, divisor = 1){

    imgData = imgData || this.getImageData();
    let data = imgData.data,
        w = this.width,
        h = this.height,
        outputImg = this.ctx.createImageData(w, h),
        outData = outputImg.data,
        radius = Math.floor(order  / 2);

    //先遍历图片像素(x, y)
    for(let y = 0; y < h; y++){
        for(let x = 0; x < w; x++){
            //遍历r,g,b三通道，做一样的处理
            for(let z = 0; z < 3; z++){
                //中心点像素(x, y)在data中的索引
                let i = (y * w + x) * 4 + z;

                //边界处理使用最简单的方法，即不做处理
                if (x < radius || y < radius || x >= w - radius || y >= h - radius){
                    outData[i] = data[i];
                }
                //非边界处矩阵卷积
                else{
                    //卷积和
                    let convSum = 0,
                        matIndex = 0;
                    //遍历矩阵行
                    for (let m = -radius; m <= radius; m++ ){
                        //矩阵列 (x-m,y)
                        let rowIndex = i + w*4*m;

                        for (let n = -radius; n <= radius; n++){
                            //(x-m, y-n)
                            let colIndex = rowIndex + n*4;
                            convSum += mat[matIndex] * data[colIndex];
                            matIndex++;

                        }
                    }
                    outData[i] = convSum / divisor;
                }
            }
            // 设置透明度
            outData[(y * w + x) * 4 + 3] = 255;
        }
    }
    return outputImg;
}
````
现在来看看应用一个锐化矩阵的效果
````javascript
conv([0, -1, 0, -1, 5, -1, 0, -1, 0], imgData);
````
原图
![原图](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/origin2.png)
锐化效果
![锐化](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/sharpen.png)
矩阵核多种多样，有常见的浮雕、锐化、模糊等，当然也可以自己定义。比如这个我自己随便设置如下加亮矩阵
![自定义矩阵](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/custom_mat.png)
效果如下
![自定义效果](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/custom.png)

#### 综合应用
是不是越来越有意思啦？现在把之前的结合一起应用做个稍微复杂点的。来先把源图像变成灰度图
````javascript
let imgGrey = this.greyScale(imgData);  //灰度去色
````
灰度图我们后面需要继续用，所以复制一份保留。可以拷贝数组，但是还记得之前说的`putImageData()`和`getImageData()`吗？用它来得到一份新的imgData副本效率更高
````javascript
//img图像数据通过canvas复制一份
copyImageData(imgData){
    imgData = imgData || this.getImageData();
    let canvas =  document.createElement('canvas'),
        ctx = canvas.getContext('2d'),
        w = imgData.width,
        h = imgData.height;
    canvas.width = w;
    canvas.height = h;
    ctx.putImageData(imgData, 0, 0);
    return ctx.getImageData(0, 0, w, h);
}
````
有了副本，那么可以随意做其他处理了。对副本反相。
````javascript
let imgGrey = this.greyScale(imgData),  //灰度去色
    imgInvert = this.invert(this.copyImageData(imgGrey));  //复制一份反向imgInvert
````
反相又是什么？反相又叫反色，就是我们常见的底片效果 ，实现也相当简单。用255-原来的像素值并代替原来像素的值就可以了。
````javascript
//底片
invert(imgData){
    imgData = imgData || this.getImageData();
    let data = imgData.data
    for(let i = 0, len = data.length; i < len; i += 4){
        data[i]     = 255 - data[i];
        data[i + 1] = 255 - data[i + 1];
        data[i + 2] = 255 - data[i + 2];
    }
    return imgData;
}
````
反相后模糊处理。模糊滤波器就是对周围像素进行加权平均处理，均值模糊很简单，周围像素的权值都相同，所以不是很平滑。高斯模糊就有这个优点，所以被广泛用在图像降噪上。我们就选用高斯模糊。高斯模糊的计算可以参考[高斯模糊的算法](http://blog.csdn.net/jiandanjinxin/article/details/51281828 "高斯模糊的算法")
````javascript
/**
 * 高斯模糊
 * 理论值遵循3σ原则，一般σ = radius/3，这里根据效果适当调大，在较小的阶数下也能有比较好的效果
 *
 * @param    {object}  imgData      要处理的imageData对象
 * @param    {number}  radius       可选，卷积核半径，默认为1，
 * @param    {number}  sigma        可选，卷积核的阶数，默认为radius
 * @returns  {object}  outputImg    处理后的imageData对象
 *
 */
gaussBlur(imgData, radius = 1, sigma = radius){
    imgData = imgData || this.getImageData();
    let order = radius*2 + 1,
        a = -1 / (2 * sigma * sigma),
        b = -a / Math.PI,
        gaussMat = new Array(order * order),
        gaussSum = 0;
    for (let x = -radius, i = 0; x <= radius; x++){
        for (let y = -radius; y <= radius; y++, i++){
            gaussMat[i] = b * Math.exp(a * (x*x + y*y));
            gaussSum += gaussMat[i];
        }
    }
    return this.conv(gaussMat, imgData, order, gaussSum);
}
````
嗯，只差最后一步了。把灰度图A和高斯模糊后的图像B进行图像混合，这一步是颜色减淡。千万别弄错顺序哦，上面的步骤也别弄混了（我按照网上某篇博文的算法死活弄不出来，结果是它的步骤顺序是错的），公式为
$$C =MIN( A + \frac{A×B}{255-B},255)$$
````javascript
let imgGrey = this.greyScale(imgData),  //灰度去色
    imgInvert = this.invert(this.copyImageData(imgGrey)),  //复制一份反向imgInvert
    imgBlur = this.gaussBlur(imgInvert), //对imgInvert进行高斯模糊
    data = imgGrey.data,
    dataBlur = imgBlur.data,
    v1, v2, v, k;
//颜色减淡
for(let i = 0, len = data.length; i < len; i += 4){
    for (let j = 0; j < 3; j++){
        k = i + j;
        v1 = data[k]; v2 = dataBlur[k];
        v = v1 + v1 * v2 / (255 - v2);
        v = v > 255 ? 255 : v;
        data[k] = v;
    }
}
return imgGrey;
````
终于完成了，我们看看效果。
原图
![原图](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/origin3.jpg)
处理后
![素描](http://ogvm9vr0q.bkt.clouddn.com/blog/20170705/sketch.png)


嗯，没错这就是素描效果。完整的应用请看[imgFilter](imgFilter.ytime.me; "imgFilter")
在搜集算法的时候看到一个效果很惊艳铅笔画算法，感兴趣的可以看看这篇介绍的博文
[图像铅笔画算法](http://blog.csdn.net/bluecol/article/details/45422763 "图像铅笔画算法")