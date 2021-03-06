﻿Title: WebGL 如何工作
Description: WebGL内部运作原理

本篇是 [WebGL基础原理](webgl-fundamentals.html)的后续教程。
在我们继续学习之前我们需要再讨论下WebGL和GPU在绘制图形时都做了些什么。
GPU的工作,可以分为两部分,第一部分将顶点坐标(或数据流)映射到剪辑空间,第二部分则是绘制第一部分处理的每个像素。


当执行如下代码时

    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    var count = 9;
    gl.drawArrays(primitiveType, offset, count);

`count = 9`意味着需要处理9个坐标点。

<img src="resources/vertex-shader-anim.gif" class="webgl_center" />

左边是我们提供的数据。顶点着色器(Vertex Shader)是一个使用 [GLSL](webgl-shaders-and-glsl.html)编写的函数,
每处理一个顶点数据时就会调用顶点着色器一次。你可以在顶点着色器通过一些计算并设置`gl_Position`为该顶点在剪辑空间的坐标值。
GPU会接收到这个值并在内部存储起来。

假设你在绘制 `TRIANGLES`(三角形), 当每处理三个坐标数据后,GPU将使用这三个顶点坐标来绘制一个三角形。
这三个坐标值确定了一个三角形包含了画布上的哪些像素, 之后栅格化这个三角形, 用更容易理解的话来说, 就是"绘制三角形的每个像素"。
绘制每个像素时, 会调用片元着色器来得到这个像素的颜色。
因此在片元着色器中我们使用了一个特殊的变量 `gl_FragColor`来接收颜色值。

这非常有意思, 在之前的示例中我们使用同一个颜色绘制每一个像素, 但我们可以通过设定 `varings`变量来从顶点着色器到片元着色器
传递每个顶点不同的颜色信息 。

像我们之前所展示的示例一样, 绘制一个简单的三角形

    // 将定义三角形顶点的数据放入缓冲区
    function setGeometry(gl) {
      gl.bufferData(
          gl.ARRAY_BUFFER,
          new Float32Array([
                 0, -100,
               150,  125,
              -175,  100]),
          gl.STATIC_DRAW);
    }

需要绘制三个顶点

    // 绘制图形
    function drawScene() {
      ...
      var primitiveType = gl.TRIANGLES;
      var offset = 0;
      var count = 3;
      gl.drawArrays(primitiveType, offset, count);
    }

这里我们需要在顶点着色器中声明一个 *varying* 变量, 用以将数据同步到片元着色器

    varying vec4 v_color;
    ...
    void main() {
      // 将位置信息与变换矩阵相乘(译注: 变换矩阵可以对图形进行平移,缩放,旋转等操作)
      gl_Position = vec4((u_matrix * vec3(a_position, 1)).xy, 0, 1);

      // 将剪辑空间映射到颜色空间
      // 剪辑空间的取值范围为 -1.0 到 +1.0
      // 颜色空间的取值范围为 0.0 到 1.0
    *  v_color = gl_Position * 0.5 + 0.5;
    }
接着在片元着色器中声明同样名称的 *varing* 变量

    precision mediump float;

    *varying vec4 v_color;

    void main() {
    *  gl_FragColor = v_color;
    }

WebGL会自动的将顶点着色器和片元着色器中相同名称的varying变量相连。

下面是代码运行的结果
{{{example url="../webgl-2d-triangle-with-position-for-color.html" }}}

使用右边的滑杆来移动,缩放或者旋转三角形, 注意这里三角形颜色的变化。因为这个例子中我们的颜色数据是从剪辑空间坐标转变而来,
是跟整个画布的背景相对应的。

现在想一想,我们仅仅处理了三个顶点。顶点着色器调用了三次,产生了三个颜色值。那么为什么这个三角形是由渐变色组成的呢?
这也是为什么传递颜色信息的变量类型为 *varying*(译注:英文中varying是指变化的,不同的意思)。

WebGL得到我们计算的三个顶点的颜色值, 栅格化三角形时会自动对三角形包含的每个像素进行插值计算。
(译注:插值计算是指通过已知点的数值来计算未知点的数据,在这个例子中通过三个顶点的颜色数据,利用线性插值得到三个顶点组成的三角形内部的每个像素点的色值)
在绘制每个像素时调用片元着色器并使用计算的插值作为当前像素的颜色。

上面的例子中我们用到了三个顶点
<style>
table.vertex_table {
  border: 1px solid black;
  border-collapse: collapse;
  font-family: monospace;
  font-size: small;
}

table.vertex_table th {
  background-color: #88ccff;
  padding-right: 1em;
  padding-left: 1em;
}

table.vertex_table td {
  border: 1px solid black;
  text-align: right;
  padding-right: 1em;
  padding-left: 1em;
}
</style>
<div class="hcenter">
<table class="vertex_table">
<tr><th colspan="2">Vertices</th></tr>
<tr><td>0</td><td>-100</td></tr>
<tr><td>150</td><td>125</td></tr>
<tr><td>-175</td><td>100</td></tr>
</table>
</div>

例子中顶点着色器应用变换矩阵来平移,旋转和缩放剪辑空间中的图形。默认的平移,旋转和缩放值分别是
translation =  (200, 150), rotation = 0, scale = (1,1)。通过计算将三个顶点坐标进行变换后
映射到剪辑空间得到的值为:

<div class="hcenter">
<table class="vertex_table">
<tr><th colspan="3">values written to gl_Position</th></tr>
<tr><td>0.000</td><td>0.660</td></tr>
<tr><td>0.750</td><td>-0.830</td></tr>
<tr><td>-0.875</td><td>-0.660</td></tr>
</table>
</div>

通过将顶点的位置映射到颜色空间并把值赋予我们声明的 *varying* 变量v_color,
可以得到每个顶点对应的v_color的取值:

<div class="hcenter">
<table class="vertex_table">
<tr><th colspan="3">values written to v_color</th></tr>
<tr><td>0.5000</td><td>0.750</td><td>0.5</td></tr>
<tr><td>0.8750</td><td>0.915</td><td>0.5</td></tr>
<tr><td>0.0625</td><td>0.170</td><td>0.5</td></tr>
</table>
</div>

v_color获取了这三个值后会自动进行插值计算,并在每次片元着色器执行时取得当前像素的插值颜色。

{{{diagram url="resources/fragment-shader-anim.html" caption="v_color通过v0,v1,v2的取值自动计算插值" }}}

我们也可以给顶点着色器传递更多的数据,这些数据也可以同时被片元着色器使用。
下面我们的例子将会绘制一个由两个不同颜色三角形组成的矩形。
为了绘制这个图形,需要在顶点着色器中加入另一个attribute变量, 用以设置颜色数据并同步到片元着色器。

    attribute vec2 a_position;
    +attribute vec4 a_color;
    ...
    varying vec4 v_color;

    void main() {
       ...
      // 从attribute变量中复制颜色数据到varing变量
      // 译注: 只有varying变量才能在片元着色器和片元着色器中同步数据
    *  v_color = a_color;
    }

接下来需要将颜色数据传递给WebGL使用
We now have to supply colors for WebGL to use.

      // 找到attribute变量的位置
      var positionLocation = gl.getAttribLocation(program, "a_position");
    +  var colorLocation = gl.getAttribLocation(program, "a_color");
      ...
    +  // 创建一个颜色缓冲区
    +  var colorBuffer = gl.createBuffer();
    +  gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer);
    +  // 设置颜色
    +  setColors(gl);
      ...

    +// 将组成矩形的两个三角形的颜色数据填入颜色缓冲区中
    +function setColors(gl) {
    +  // 取两个随机颜色
    +  var r1 = Math.random();
    +  var b1 = Math.random();
    +  var g1 = Math.random();
    +
    +  var r2 = Math.random();
    +  var b2 = Math.random();
    +  var g2 = Math.random();
    +
    +  gl.bufferData(
    +      gl.ARRAY_BUFFER,
    +      new Float32Array(
    +        [ r1, b1, g1, 1,
    +          r1, b1, g1, 1,
    +          r1, b1, g1, 1,
    +          r2, b2, g2, 1,
    +          r2, b2, g2, 1,
    +          r2, b2, g2, 1]),
    +      gl.STATIC_DRAW);
    +}

渲染的时候开启颜色attribute变量

    +gl.enableVertexAttribArray(colorLocation);
    +
    +// Bind the color buffer.
    +gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer);
    +
    +// Tell the color attribute how to get data out of colorBuffer (ARRAY_BUFFER)
    +var size = 4;          // 每次迭代处理四个数据
    +var type = gl.FLOAT;   // 每个数据为32位浮点数
    +var normalize = false; // 不对数据进行标准化
    +var stride = 0;        // 每次迭代的数据之间存储空间间隔为0
    +var offset = 0;        // 从缓冲区头部开始存储
    +gl.vertexAttribPointer(
    +    colorLocation, size, type, normalize, stride, offset)

设置`count = 6`(两个三角形总共6个顶点)
And adjust the count to compute 6 vertices for 2 triangles

    // Draw the geometry.
    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    *var count = 6;
    gl.drawArrays(primitiveType, offset, count);

运行的结果如下:

{{{example url="../webgl-2d-rectangle-with-2-colors.html" }}}

注意我们设定了两个固定的颜色值。我们将颜色值传给了 *varying*变量, 因此会对图形的每个像素进行自动的插值计算。
但每个三角形的三个顶点的颜色值都是相同的, 因此进行插值获得的值也是固定的。呈现出来就是一个单色的三角形。
如果对不同顶点应用不同的颜色值,我们将看到进行插值后的颜色的变化。

    function setColors(gl) {
      // 为每个顶点配置不同的颜色数据.
      gl.bufferData(
          gl.ARRAY_BUFFER,
          new Float32Array(
    *        [ Math.random(), Math.random(), Math.random(), 1,
    *          Math.random(), Math.random(), Math.random(), 1,
    *          Math.random(), Math.random(), Math.random(), 1,
    *          Math.random(), Math.random(), Math.random(), 1,
    *          Math.random(), Math.random(), Math.random(), 1,
    *          Math.random(), Math.random(), Math.random(), 1]),
          gl.STATIC_DRAW);
    }

进行插值之后的 *变化*。

{{{example url="../webgl-2d-rectangle-with-random-colors.html" }}}

虽然这个示例并不酷炫,但给大家演示了使用多个attribute变量以及将数据从顶点着色器向片元着色器传递。
如果你看了 [图片处理示例](webgl-image-processing.html), 会发现它们也使用了一个attribute变量来传递texture坐标

##那些缓冲区命令和attribute命令是干什么的?

GPU从缓冲区中获得顶点坐标和其他跟每个顶点相关数据。
`gl.createBuffer`命令创建一个缓冲区, `gl.bindBuffer`将我们创建的缓冲区设定为工作缓冲区,
`gl.bufferData`将数据复制到缓冲区中。这些命令通常在初始化程序时调用。

一旦将数据放入了缓冲区,我们需要告诉WebGL怎样从缓冲区中提取数据来传递给顶点着色器中的attribute变量。

为了实现这一目的,首先我们需要查询WebGL配置attribute变量的地址(内存中的位置)。
比如在上面的代码中:


    // 查询顶点坐标和颜色数据应该放入的位置.
    var positionLocation = gl.getAttribLocation(program, "a_position");
    var colorLocation = gl.getAttribLocation(program, "a_color");

这个步骤通常也是在初始化时完成。

当知道了attribute变量的地址后, 我们在绘制图形前还需要执行三个命令。

    gl.enableVertexAttribArray(location);

`gl.enableVertexAttribArray`这一命令告诉WebGL开启当前需要传递数据的attribute变量,为数据传输做准备。

    gl.bindBuffer(gl.ARRAY_BUFFER, someBuffer);

`gl.bindBuffer` 将某个缓冲区绑定到ARRAY_BUFFER绑定点。ARRAY_BUFFER是WebGL中的一个内部的全局变量。

    gl.vertexAttribPointer(
        location,
        size,
        type,
        normalize,
        stride,
        offset);

`gl.vertexAttribPointer`命令告诉WebGL怎样从当前绑定在绑定点的缓冲区中提取数据。
其中size为一个顶点的分量个数(1-4),type表示这些数据是什么类型的数据
(`BYTE`, `FLOAT`, `INT`, `UNSIGNED_SHORT`, etc...),normalize表明是否将非浮点型的数据归一化到[0,1]或[-1,1]区间,
stride指定相邻两个顶点间的字节间隔,默认为0,
offset指定缓冲区中的偏移量,即从缓冲区何处开始存储。

如果你只在缓冲区中存储同一类信息的数据(译注:比如只存储坐标信息或只存储颜色信息), stride和offset的值始终未0。
stride为0意味着相邻数据之间的间隔与数据的大小相同。offset为0表示数据从缓冲区的起始位置开始存储。
使用缓冲区存储多种数据(译注:这种情况下stride和offset将不为0)会让程序变得复杂,但会带来性能上的优化。

希望通过上面的内容让你更加的了解buffers和attributes。

接下来我们接着看 [shaders and GLSL](webgl-shaders-and-glsl.html)。

<div class="webgl_bottombar"><h3>vertexAttribPointer中的normalize参数是什么?</h3>
<p>
normalize参数通常是用于标准化非浮点数的数据类型。当它为false时, 数据将保持其原本的数据类型。
比如BTYE类型数据取值范围为 -128 到 127, UNSIGNED_BYTE类型范围为 0 到255, SHORT为 -32768 到 32767。
(译注: JavaScript是一种弱类型语言, 其本身并没有上面提到的数据类型的区分。但我们在WebGL基础原理那一章节中也提到过,
GLSL是一种非常严谨的的类似C/C++的语言,它使用严格定义的数据类型,因此使用JavaScript将数据传递到GPU时,必须将数据转换为所需要的数据类型。
我们可以使用JavaScript类型数据来完成转换。)
</p>
<p>
如果normalize参数设置为true, 那么对于一个BYTE类型的数据, 其取值将从 -128 到 127 映射到 -1.0 到 1.0。
UNSIGNED_BYTE 从 0 到 255 映射为 0.0 到 +1.0, SHORT数据也会被映射到 -1.0 到 1.0范围内,相比于BYTE数据,
其取值会更加的精确(译注:小数点后具有更多的位数)。
</p>
<p>
一个常见的使用normalize参数的例子是颜色数据的标准化。 着色器中颜色的取值范围是从 0.0 到 1.0。
使用浮点数()来存储颜色各通道(红,绿,蓝,透明度)的信息,每个顶点的每个颜色信息将会占用16字节的空间(4byte * 4)。
绘制复杂的图形时,因为有非常多的顶点和颜色信息,这些数据加起来将占用非常多的空间。
这种情况下使用UNSIGNED_BYTE类型(0-255)来存储颜色信息, 再通过normalize标准化将其转换到0.0到1.0范围, 你仅仅需要4位的空间来存储颜色
节省了75%的空间。
</p>
<p>通过修改示例中的代码来帮助大家理解。 当WebGL获取颜色数据时, 我们执行 </p>
<pre class="prettyprint showlinemods">
  // Tell the color attribute how to get data out of colorBuffer (ARRAY_BUFFER)
  var size = 4;                  // 每次迭代处理四个数据
*  var type = gl.UNSIGNED_BYTE;   // 数据为8位unsigned byte类型
*  var normalize = true;          // 对数据进行标准化
  var stride = 0;                // 每次迭代的数据之间存储空间间隔为0
  var offset = 0;                // 从缓冲区头部开始存储
  gl.vertexAttribPointer(
      colorLocation, size, type, normalize, stride, offset)
</pre>
<p>当我们往缓冲区中填充颜色数据时, 执行</p>
<pre class="prettyprint showlinemods">
// 往缓冲区中填充组成矩形的两个三角形的颜色数据
function setColors(gl) {
  // 随机选取两个颜色
  var r1 = Math.random() * 256; // 当使用类型数组Uint8Array
  var b1 = Math.random() * 256; // 对这些随机数值转换时,
  var g1 = Math.random() * 256; // 随机数值小数点后的部分
  var r2 = Math.random() * 256; // 会被自动截断
  var b2 = Math.random() * 256;
  var g2 = Math.random() * 256;

  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Uint8Array(   // Uint8Array
        [ r1, b1, g1, 255,
          r1, b1, g1, 255,
          r1, b1, g1, 255,
          r2, b2, g2, 255,
          r2, b2, g2, 255,
          r2, b2, g2, 255]),
      gl.STATIC_DRAW);
}
</pre>
<p>
运行的结果:
</p>

{{{example url="../webgl-2d-rectangle-with-2-byte-colors.html" }}}
</div>
