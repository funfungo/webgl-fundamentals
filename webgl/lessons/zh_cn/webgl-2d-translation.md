Title: WebGL 2D 位移
Description: 如何对2图形进行位移

在我们开始3D图形相关内容前, 再多关注一些2D图形的内容。
可能这部分内容对于有些人来说是非常简单的。
but I'll build up to a point in a few articles.

本篇是 [WebGL基础原理](webgl-fundamentals.html)的后续教程。
建议你先学习WebGL基础原理的内容再来继续这里的学习。

位移(Translation)是一类数学计算的代名词, 主要是指移动某个物体,
在这里主要是指移动几何图形。
在我们第一篇 [WebGL基础原理](webgl-fundamentals.html)中最后的示例里,
可以看到我们可以通过改变传递给`setRectangle`(译注: `setRectangle`的具体代码请参考
 [WebGL基础原理](webgl-fundamentals.html)中最后一个示例)的顶点参数值来非常简单的移动所绘制的矩形,
这里有一个基于这个示例的演示

一开始我们定义一些变量来存储矩形的位移值, 宽度, 高度以及颜色。

```
  var translation = [0, 0];
  var width = 100;
  var height = 30;
  var color = [Math.random(), Math.random(), Math.random(), 1];
```

接着定义一个绘制图形的函数, 在我们完成位移更新图形顶点坐标后调用这个函数来进行绘制。

```
  function drawScene() {
    webglUtils.resizeCanvasToDisplaySize(gl.canvas);

    // 设置WebGL的viewpoint大小
    gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);

    // 清除画布
    gl.clear(gl.COLOR_BUFFER_BIT);

    gl.useProgram(program);

    // 开启attribute变量
    gl.enableVertexAttribArray(positionLocation);

    // 绑定顶点坐标缓冲区
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

    // 设置矩形
    setRectangle(gl, translation[0], translation[1], width, height);

    // attribute变量从缓冲区中提取数据
    var size = 2;
    var type = gl.FLOAT;
    var normalize = false;
    var stride = 0;
    var offset = 0;
    gl.vertexAttribPointer(
        positionLocation, size, type, normalize, stride, offset)

    // 设置画布分辨率
    gl.uniform2f(resolutionLocation, gl.canvas.width, gl.canvas.height);

    // 设置颜色
    gl.uniform4fv(colorLocation, color);

    // 绘制图形
    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    var count = 6;
    gl.drawArrays(primitiveType, offset, count);
  }
```

在下面的演示中设置了两个滑杆, 这两个滑杆分别对应了x,y方向的位移值, 当改变的时候会更新相应的值到
`translation[0]` 和 `translation[1]` ,并调用 `drawScene`。你可以试着改变一下滑杆的值来观察矩形的位移变化。

{{{example url="../webgl-2d-rectangle-translate.html" }}}

看起来效果还不错,但如果我们想要移动更复杂的形状呢?

我们来试着绘制一个包含了6个三角形的 'F'的形状。(译注: 一个矩形由两个三角形组成)

<img src="../resources/polygon-f.svg" width="200" height="270" class="webgl_center">

为了达到对这个图形进行位移的目的,需要修改 `setRectangle` 函数。

```
// 将定义'F'字母图形的顶点坐标数据填入缓冲区
function setGeometry(gl, x, y) {
  var width = 100;
  var height = 150;
  var thickness = 30;
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
          // 左边一竖
          x, y,
          x + thickness, y,
          x, y + height,
          x, y + height,
          x + thickness, y,
          x + thickness, y + height,

          // 上面一横
          x + thickness, y,
          x + width, y,
          x + thickness, y + thickness,
          x + thickness, y + thickness,
          x + width, y,
          x + width, y + thickness,

          // 中间一横
          x + thickness, y + thickness * 2,
          x + width * 2 / 3, y + thickness * 2,
          x + thickness, y + thickness * 3,
          x + thickness, y + thickness * 3,
          x + width * 2 / 3, y + thickness * 2,
          x + width * 2 / 3, y + thickness * 3,
      ]),
      gl.STATIC_DRAW);
}
```

从上面的代码可以看出按这种方法我们的代码并没有很好的扩展性。
如果绘制包含上百上千个部件组成的复杂图形时, 代码会变得非常复杂并且难以维护。
并且每次绘制时都需要在JavaScript中更新相应的坐标值。

更简单的方法是在着色器中进行几何图案的更新和位移计算。

这里是优化后的着色器代码

```
&lt;script id="2d-vertex-shader" type="x-shader/x-vertex"&gt;
attribute vec2 a_position;

uniform vec2 u_resolution;
+uniform vec2 u_translation;

void main() {
*   // 将顶点坐标与位移值相加
*   vec2 position = a_position + u_translation;

   // 将矩形坐标从像素映射为剪切空间坐标
*   vec2 zeroToOne = position / u_resolution;
   ...
```

我们还需要重新组织一下代码,重构后图形坐标只需要设置一次。

```
// 将定义'F'字母图形的顶点坐标数据填入缓冲区
function setGeometry(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
          // 左边一竖
          0, 0,
          30, 0,
          0, 150,
          0, 150,
          30, 0,
          30, 150,

          // 上面一横
          30, 0,
          100, 0,
          30, 30,
          30, 30,
          100, 0,
          100, 30,

          // 中间一横
          30, 60,
          67, 60,
          30, 90,
          30, 90,
          67, 60,
          67, 90,
      ]),
      gl.STATIC_DRAW);
}
```

这样在每次更新位移值时, 只需要在绘制前更新 `u_translation`。

```
  ...

  // 在内存中找到WebGL中uniform变量'u_translation'的地址
  var translationLocation = gl.getUniformLocation(
             program, "u_translation");
  ...

  var positionBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

+  // 将图形坐标填入缓冲区
+  setGeometry(gl);

  ...

  // 绘制
  function drawScene() {

    ...

+    // 设置位移值
+    gl.uniform2fv(translationLocation, translation);

    var primitiveType = gl.TRIANGLES;
    var offset = 0;
*    var count = 18;
    gl.drawArrays(primitiveType, offset, count);
  }
```

注意 `setGeometry`只调用一次, 从 `drawScene` 中提取了出来。

以下是执行的结果。

{{{example url="../webgl-2d-geometry-translate-better.html" }}}

在这个示例中, WebGL完成了大部分计算的工作。我们仅仅需要设置一个位移值, 然后让它进行绘制。
即便是我们的图形包含成千上万的坐标点, 代码也能保持这样的简洁。

如果你感兴趣,可以对比使用JavaScript完成顶点坐标更新的 [复杂版本](../webgl-2d-geometry-translate.html" target="_blank)

希望通过这节的示例可以让你有所收获, 在之后的教程中我们会介绍更加优化的方法来完成这个效果。
接下来我们来看看 [WebGL 2D旋转](webgl-2d-rotation.html)。

