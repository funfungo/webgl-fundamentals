﻿Title: WebGL 2D 旋转
Description: 怎么对2D图形进行旋转

本篇是一系列关于WebGL的教程中的续篇, 如果你是第一次看到这个教程,建议你先学习[WebGL基础原理](webgl-fundamentals.html)。

我必须承认要讲清楚这个主题并不是那么容易,但是我想尽力尝试一下。

首先我想先介绍一下什么是"单位圆"。如果你还记得初中数学(别睡着了), 那么你肯定知道一个圆有一个唯一的半径值。
半径是指从圆心到圆周的距离, 而一个单位圆的半径为1.0。

这里是一个单位圆

{{{diagram url="../unit-circle.html" width="300" height="300" }}}

当你拖动圆周上的蓝色圆柄时, 圆周上相应点X和Y轴的坐标值发生改变。 当圆柄在圆周顶部时, Y值为1, X值为0,
在圆周最右边时X值为1, Y值为0。

你一定知道当你用一个数字与1相乘时,结果不会发生任何改变。比如 123 x 1 = 123,
非常直观。对于一个单位圆来说, 一个半径为1.0的圆也是某种形式的1(1个单位)。
所以理论上你可以使用一个坐标值和单位圆上的任一点相乘, 得到的点与圆心的距离保持不变,但是围绕圆心进行了一定角度的旋转。

我们从单位圆上选取一个点和之前的例子 [WebGL 2D位移](webgl-2d-translation.html)中的几何图形相乘。

下面是更新后的顶点着色器代码

    <script id="2d-vertex-shader" type="x-shader/x-vertex">
    attribute vec2 a_position;

    uniform vec2 u_resolution;
    uniform vec2 u_translation;
    +uniform vec2 u_rotation;

    void main() {
    +  // 旋转坐标点
    +  vec2 rotatedPosition = vec2(
    +     a_position.x * u_rotation.y + a_position.y * u_rotation.x,
    +     a_position.y * u_rotation.y - a_position.x * u_rotation.x);

      // 添加位移
    *  vec2 position = rotatedPosition + u_translation;

还需要修改JavaScript的代码来传入这两个参数

      ...

    +  var rotationLocation = gl.getUniformLocation(program, "u_rotation");

      ...

    +  var rotation = [0, 1];

      ...

      // 绘制
      function drawScene() {

        ...

        // 设置位移
        gl.uniform2fv(translationLocation, translation);

    +    // 设置旋转
    +    gl.uniform2fv(rotationLocation, rotation);

        // 绘制图形
        var primitiveType = gl.TRIANGLES;
        var offset = 0;
        var count = 18;
        gl.drawArrays(primitiveType, offset, count);
      }

运行结果如下。拖动圆柄或者滑块来进行旋转或位移。

{{{example url="../webgl-2d-geometry-rotation.html" }}}

这里面的工作原理是什么? 下面我们来看看其中的数学公式

    rotatedX = a_position.x * u_rotation.y + a_position.y * u_rotation.x;
    rotatedY = a_position.y * u_rotation.y - a_position.x * u_rotation.x;

假如你想要旋转一个矩形,在旋转之前右上角的坐标是(3.0, 9.0)。选择单位圆上顺时针方向30度的一个点与这个右上角的点相乘。

<img src="../resources/rotate-30.png" class="webgl_center" />

单位圆上这个点的坐标值是(0.50, 0.87)

<pre class="webgl_center">
   3.0 * 0.87 + 9.0 * 0.50 = 7.1
   9.0 * 0.87 - 3.0 * 0.50 = 6.3
</pre>

得到的结果等于将矩形顺时针方向旋转30度后右上角的坐标值

<img src="../resources/rotation-drawing.svg" width="500" class="webgl_center"/>

如果旋转60度我们也会得到类似的结果

<img src="../resources/rotate-60.png" class="webgl_center" />

单位圆上相应的点的坐标为(0.87, 0.50)

<pre class="webgl_center">
   3.0 * 0.50 + 9.0 * 0.87 = 9.3
   9.0 * 0.50 - 3.0 * 0.87 = 1.9
</pre>

可以看到当我们顺时针转动这个点是, 点的X坐标逐渐变大, Y坐标逐渐变小。 继续旋转超过90度后,
X开始变小, Y值变大。这个公式达到了让图像旋转的目的。

对于单位圆上的点,还有另一种表示方式, 被称为sine和cosine。 对于任意一个角度,我们可以通过这样的方式获取它的sine值和cosine值。

    function printSineAndCosineForAnAngle(angleInDegrees) {
      var angleInRadians = angleInDegrees * Math.PI / 180;
      var s = Math.sin(angleInRadians);
      var c = Math.cos(angleInRadians);
      console.log("s = " + s + " c = " + c);
    }

如果你将上面的代码复制到你的浏览器控制台中,然后执行 `printSineAndCosignForAngle(30)`,
可以得到 `s = 0.49 c = 0.87`(注意这里只取了小数点后两位的数据)

将上面的代码整合在一起, 你就可以将你的几何图形旋转任意的角度。仅仅只需通过设置你所需要的旋转角度的sine和cosine值。

      ...
      var angleInRadians = angleInDegrees * Math.PI / 180;
      rotation[0] = Math.sin(angleInRadians);
      rotation[1] = Math.cos(angleInRadians);

下面是一个添加了角度旋转的示例, 拖动滑块来进行旋转和位移。

{{{example url="../webgl-2d-geometry-rotation-angle.html" }}}

希望通过我的讲解能让你理解2D旋转的原理。 在之后你会学到旋转图形更加简单和通用的方法。
那在这之前我们先来看看比较简单的 [WebGL 2D缩放](webgl-2d-scale.html)。

<div class="webgl_bottombar"><h3>什么是弧度?</h3>
<p>
弧度是与圆,旋转和角度相关的一个衡量单位, 就像我们使用英寸,码, 米等单位来衡量距离, 同样的,我们可以使用弧度或是角度来表示一个角的大小。
</p>
<p>
你可能知道使用公制单位来进行数学计算要比使用英制单位要简单。将英寸转网尺, 需要除以12, 如果要转化为码,则要除以36。
我不知道你数学怎样,但对于我来说是不太可能对除以36进行心算。 但是如果是用公制单位就简单很多了, 从毫米转化为厘米,仅仅需要除以10, 而从毫米转化为米
也只是需要除以1000。
对这种形式的转化进行心算是大家都能做到的。
</p>
<p>
弧度和角度之间的区别也类似。 使用角度为单位, 使相应的计算变得困难, 使用弧度可以更加简单的完成计算。
一个圆的角度是360度, 弧度为2π。 因此转动一圈就是转动了2π的弧度值, 转半圈的话是转动了1π弧度。
同样的转动1/4圈为1/2π弧度。
如果你想将某个物体转动90角度, 只需要使用<code>Math.PI * 0.5</code>, 如果转动45角度, 则可以使用<code>Math.PI * 0.25</code>。
</p>
<p>
如果你转变为弧度制的思维方式来看待角,圆以及旋转的问题时, 其中的数学计算就变得非常简单了。所以可以试试使用弧度制来代替角度制。
</p>
</div>


