Title: WebGL 基础原理
Description: 你的第一堂WebG基础课

WebGL 通常被认为是浏览器3D渲染的API。大家认为"只要使用了WebGL我就能 *像变魔术似的* 做出酷炫的3D动画"。
事实上WebGL只是一种栅格化引擎。 它根据你提供的代码画出点,线和三角形。如果你想用WebGL做出更复杂的东西取决于你如何利用这些点,线以及三角形。

WebGL在你电脑的GPU上运行,因此你需要提供可以在GPU上运行的代码。你提供的代码为一对一对的着色器函数。这两个函数一个被称为顶点着色器(Vertex Shader)
,另一个被称为片元着色器(Fragment Shader)。他们都使用一种非常严谨的的类似C/C++的语言[GLSL](webgl-shaders-and-glsl.html)(GL Shader
Language)来编写。这两个函数搭配在一起被称为一段 *程序(program)*。

顶点着色器(Vertex Shader)的工作是计算顶点的位置。基于着色器输出的这些位置,WebGL可以栅格化各种基本几何图形比如说点,线和三角形。
当栅格化这些基本几何图形时它会调用片元着色器(Fragment Shader)。片元着色器(Fragment Shader)的作用是计算这些几何图形所包含的每个像素的颜色。

基本上WebGL的API就是通过不同的配置来控制这一对着色器函数运行。
你要为你要绘制的任何图形进行一系列配置然后使用`gl.drawArrays`或者`gl.drawElements`来让这两个着色器在GPU上运行。

任何你想要这两个着色器使用的数据都需要传给GPU。着色器接收数据有四种方法。

1. Attributes 和 Buffers

   缓冲区(Buffers)是你上传到GPU的二进制数组。通常缓冲区中包含了位置,标准,纹理坐标,顶点颜色等信息。你也可以把你想要传递的任何数据放在里面。

   Attributes用来指定用什么方式提取缓冲区中的数据并传给顶点着色器(Vertex Shader)。比如你可能会以3个(译注:分别对应x,y,z方向的值)32bit的浮点数来
   储存位置信息。你需要告诉一个特定的attribute变量从哪一个缓冲区中提取位置信息,提取什么类型的数据(3个32bit浮点数),数据在缓冲区中的开始位置,
   以及每个数据所占用的内存空间。

   Buffers并不是随机存取的。一个顶点着色器(Vertex Shader)执行指定的次数,每一次执行时提取缓冲区中相应的数据分配给一个attribute。

2. Uniforms

   Uniforms是你在执行你的着色器程序前设定的非常高效的全局变量。

3. Textures

   Textures是你在着色器程序中可以随机访问的数据数组。在Textures中最常见的是图片数据,但是Textures只是一种存储信息的数据形式, 并不是只能用来
   存储颜色信息。

4. Varyings

   Varyings是顶点着色器(Vertex Shader)传递到片元着色器(Fragment Shader)的一种数据形式。根据你渲染的图形,点,线或是三角形, varying的取值会在执行片元着色器时进行插值计算。

## WebGL Hello World

WebGL只关心两件事情: 剪辑空间坐标(clip space)和颜色。
你作为一个WebGL程序员所需要做的就是使用两个着色器来提供这两种信息。
顶点着色器(Vertex Shader)提供了剪辑空间坐标信息而片元着色器(Fragment Shader)提供颜色信息。

不管你的画布是多大,剪辑空间坐标信息的取值始终在-1和+1之间。这里有一个最简单的WebGL例子。

我们先来看下顶点着色器(Vertex Shader)

    // 一个attribute变量将从缓冲区中获取数据
    attribute vec4 a_position;

    // 每个着色器程序都有一个main函数
    void main() {
      // gl_Position是顶点着色器的一个特殊变量, 用来设置点的位置
      gl_Position = a_position;
    }

想象一下,如果使用JavaScript来代替GLSL执行任务,代码可能会是这样

    // *** PSUEDO CODE!! ***
    // *** 伪代码!! ***

    //(译注:顶点坐标数据)
    var positionBuffer = [
      0, 0, 0, 0,
      0, 0.5, 0, 0,
      0.7, 0, 0, 0,
    ];
    var attributes = {};
    var gl_Position;

    drawArrays(..., offset, count) {
      var stride = 4;
      var size = 4;
      for (var i = 0; i < count; ++i) {
         //将当前坐标所需要的位置数据信息取出并传给a_position参数
         attributes.a_position = positionBuffer.slice((offset + i) * stide, size);
         //译注:执行顶点着色器函数
         runVertexShader();
         ...
         doSomethingWith_gl_Position();
    }

实际的情况并不会这么简单,因为`positionBuffer`需要转变成二进制数据(后面会介绍)所以实际从缓冲区中提取数据的计算过程会不太一样。
但希望这个例子可以给你一个顶点着色器如何执行的概念。

接下来我们需要一个片元着色器(Fragment Shader)

    // 片元着色器需要我们设定一个默认数据精度。mediump是一个很好的默认值。
    // mediump表示"medium precision"(中等精度值)

    void main() {
      // gl_FragColor 是片元着色器的一个特殊变量,用来设置颜色值
      gl_FragColor = vec4(1, 0, 0.5, 1); // 返回棕紫色
    }

这里我们设置`gl_FragColor`为`1.0, 0.0, 0.5, 1.0`,分别对应rgba中的红/绿/蓝/透明度通道,
即设定红色通道为1, 绿色通道为0.5, 蓝色通道为0.5, 透明度通道为1。需要注意的是,颜色再WebGL中是用0到1的取值表示的。

现在我们已经准备好了这两个着色器代码, 接下来使用WebGL来运行它们

首先我们需要一个HTML canvas 对象

     <canvas id="c"></canvas>

接着我们在JavaScript中找到这个canvas对象

     var canvas = document.getElementById("c");

现在我们可以创建一个WebGLRenderingContext

     var gl = canvas.getContext("webgl");
     if (!gl) {
        // no webgl for you!
        ...

现在我们需要编译这些着色器,并把编译的程序发送到GPU, 首先我们需要以字符串的形式获取到着色器代码。
你可以用你所熟悉的任何JavaScript字符串格式来创建GLSL字符串, 比如通过字符串拼接,使用AJAX来从服务器获取或者通过多行字符串。
或者像我们示例中这样,将它们放到type为notjs(非JavaScript代码)的script标签中。

    <script id="2d-vertex-shader" type="notjs">

      // 一个attribute变量将从缓冲区中获取数据
      attribute vec4 a_position;

      // 每个着色器程序都有一个main函数
      void main() {

        // gl_Position是顶点着色器的一个特殊变量, 用来设置点的位置
        gl_Position = a_position;
      }

    </script>

    <script id="2d-fragment-shader" type="notjs">

      // 片元着色器需要我们设定一个默认数据精度。mediump是一个很好的默认值。
      // mediump表示"medium precision"(中等精度值)
      precision mediump float;

      void main() {
        // gl_FragColor 是片元着色器的一个特殊变量,用来设置颜色值
        gl_FragColor = vec4(1, 0, 0.5, 1);
      }

    </script>

大部分3D引擎使用各种模板或是字符串拼接等方式来动态生成GLSL着色器代码。
但在我们这个教程的例子中,并不需要动态生成GLSL,所以我们选择了这种比较简单且直观的方式来编写GLSL。

接下来我们需要一个函数,这个函数的功能是创建着色器, 读取GLSL代码并编译着色器。
这里我没有加入任何的注释因为通过调用的方法的名称大家可以很清楚的了解发生了什么。

    function createShader(gl, type, source) {
      var shader = gl.createShader(type);
      gl.shaderSource(shader, source);
      gl.compileShader(shader);
      var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
      if (success) {
        return shader;
      }

      console.log(gl.getShaderInfoLog(shader));
      gl.deleteShader(shader);
    }

现在我们可以使用这个函数来创建两个着色器。

    //译注: 获取顶点着色器的代码字符串
    var vertexShaderSource = document.getElementById("2d-vertex-shader").text;
    //译注: 获取片元着色器的代码字符串
    var fragmentShaderSource = document.getElementById("2d-fragment-shader").text;

    var vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
    var fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);

接着我们创建一个函数将这两个着色器*合并*为一个*程序(program)*

    function createProgram(gl, vertexShader, fragmentShader) {
      var program = gl.createProgram();
      gl.attachShader(program, vertexShader);
      gl.attachShader(program, fragmentShader);
      gl.linkProgram(program);
      var success = gl.getProgramParameter(program, gl.LINK_STATUS);
      if (success) {
        return program;
      }

      console.log(gl.getProgramInfoLog(program));
      gl.deleteProgram(program);
    }

并执行这个函数

    var program = createProgram(gl, vertexShader, fragmentShader);

现在我们就已经在GPU上创建了我们的GLSL程序,接下来我需要往里面传输数据
大部分WebGL的API都是用来设定如何向GLSL程序传输数据, 在我们这个例子中我们仅仅需要设定一个attribute变量`a_position`的值。
首先要在我们创建的GLSL程序中找到这个attribute变量的位置。

    var positionAttributeLocation = gl.getAttribLocation(program, "a_position");

注意获取attribute变量(或uniform变量)的位置是你需要在初始化时需要做的,而不是在渲染过程中去获取。

Attributes变量是从缓冲区中获取数据所以我们还需要创建一个缓冲区对象

    var positionBuffer = gl.createBuffer();

WebGL可以让我们在全局的绑定点(bind points)操作许多WebGL资源。
你可以将绑定点(bind points)看做WebGL内部的全局变量。
首先要将资源绑定到绑定点, 其他所有的函数都将通过这个绑定点读取数据。
接下来将我们例子中创建的positionBuffer进行绑定。

    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

现在我们可以将数据放入PositionBuffer中

    // 三个二维的点
    var positions = [
      0, 0,
      0, 0.5,
      0.7, 0,
    ];
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);

这里我们实际上进行了几项操作
首先我们创建了一个JavaScript数组变量`positions`。WebGL是一种强类型的语言,所以我们需要使用`new Float32Array(positions)`
来将JavaScript数组转换成WebGL可以处理的数据类型(32位浮点数)数组。在之前我们将positionBuffer绑定到了`ARRAY_BUFFER`绑定点上,所以
`gl.bufferData`会将我们提供的数据复制到GPU上的`positionBuffers`中。

`gl.bufferData`的最后一个参数提示WebGL我们将如何使用这些数据。WebGL将根据这个参数的设定来对程序的运行进行优化。
`gl.STATIC_DRAW`代表告诉WebGL这些数据是静态数据,我们不会改变它。

以上的这些代码就是我们程序的*初始化代码*。这些代码在我们加载页面完成后立即运行。
下面介绍*渲染代码*也就是每次我们的画布绘制/渲染时执行的代码

## 渲染

在我们绘制图像之前我们需要将设定画布的大小。画布跟图片一样有两个尺寸值,
它们实际的像素尺寸和显示尺寸。CSS用来设定画布的显示尺寸。请记住**始终使用CSS来设定画布的显示尺寸** ,因为它比其他任何方式都要简单和灵活。

设定画布的实际像素等于显示尺寸,我们使用了一个helper function[(具体定义看这里)](webgl-resizing-the-canvas.html)。
(译注:如果使用的是两倍屏需要相应的改版所设定的像素值)

本教程中的所有实例如果在独立的页面中会使用400x300像素的画布,但如果像在这个网站的内嵌iframe中展示的时候,将会填满iframe所设定的尺寸。
使用CSS来设定画布的大小我们可以非常简单的处理这两种不同的情况。

    webglUtils.resizeCanvasToDisplaySize(gl.canvas);

之前我们提到过WebGL使用剪辑空间坐标(clip space)来表示坐标位置。因此我们需要告诉WebGL如何将像素坐标信息和剪辑空间坐标信息进行映射。
通过调用`gl.viewport`并将画布的尺寸传入可以让WebGL完成这样的映射。

    gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);

上面这行代码告诉WebGL将-1到+1的剪辑空间映射到宽度范围为0->`gl.canvas.width`,高度范围从0->`gl.canvas.height`的画布中。

设定清空画布的颜色为`0, 0, 0, 0`(对应rgba), 因此画布显示为透明。

    // 清空画布
    gl.clearColor(0, 0, 0, 0);
    gl.clear(gl.COLOR_BUFFER_BIT);

我们需要告诉WebGL我们将执行的着色器程序

    // 告诉WebGL使用哪一对着色器程序
    gl.useProgram(program);

接着我们需要告诉WebGL怎样从我们创建的buffer中取出数据并应用到着色器中的attribute变量上。
首先我们要通过下面的代码开启需要的attribute变量

    gl.enableVertexAttribArray(positionAttributeLocation);

再指定如何取出数据

    // 绑定数据
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

    // 告诉attribute变量怎样从positionuffer（ARRAY_BUFFER)中取数据
    var size = 2;          // 指定缓冲区中每个顶点的分量个数
    var type = gl.FLOAT;   // 数据格式为32位浮点数
    var normalize = false; // 不用标准化数据
    var stride = 0;        // 相邻的两个顶点间的字节数,默认为0
    var offset = 0;        // 指定缓冲区对象的偏移量，即attribute变量从缓冲区中的何处开始储存
    gl.vertexAttribPointer(
        positionAttributeLocation, size, type, normalize, stride, offset)


A hidden part of `gl.vertexAttribPointer` is that it binds the current `ARRAY_BUFFER`
to the attribute. In other words now this attribute is bound to
`positionBuffer`. That means we're free to bind something else to the `ARRAY_BUFFER` bind point.
The attribute will continue to use `positionBuffer`.

note that from the point of view of our GLSL vertex shader the `a_position` attribute was a `vec4`

    attribute vec4 a_position;

`vec4` is a 4 float value. In JavaScript you could think of it something like
`a_position = {x: 0, y: 0, z: 0, w: 0}`. Above we set `size = 2`. Attributes
default to `0, 0, 0, 1` so this attribute will get its first 2 values (x and y)
from our buffer. The z, and w will be the default 0 and 1 respectively.

After all that we can finally ask WebGL to execute our GLSL program.

    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    var count = 3;
    gl.drawArrays(primitiveType, offset, count);

Because the count is 3 this will execute our vertex shader 3 times. The first time `a_position.x` and `a_position.y`
in our vertex shader attribute will be set to the first 2 values from the positionBuffer.
The 2nd time `a_position.xy` will be set to the 2nd two values. The last time it will be
set to the last 2 values.

Because we set `primitiveType` to `gl.TRIANGLES`, each time our vertex shader is run 3 times
WebGL will draw a triangle based on the 3 values we set `gl_Position` to. No matter what size
our canvas is those values are in clip space coordinates that go from -1 to 1 in each direction.

Because our vertex shader is simply copying our positionBuffer values to `gl_Position` the
triangle will be drawn at clip space coordinates

      0, 0,
      0, 0.5,
      0.7, 0,

Converting from clip space to screen space WebGL is going to draw a triangle at. If the canvas size
happned to be 400x300 we'd get something like this

     clip space      screen space
       0, 0       ->   200, 150
       0, 0.5     ->   200, 225
     0.7, 0       ->   340, 150

WebGL will now render that triangle. For every pixel it is about to draw WebGL will call our fragment shader.
Our fragment shader just sets `gl_FragColor` to `1, 0, 0.5, 1`. Since the Canvas is an 8bit
per channel canvas that means WebGL is going to write the values `[255, 0, 127, 255]` into the canvas.

Here's a live version

{{{example url="../webgl-fundamentals.html" }}}

In the case above you can see our vertex shader is doing nothing
but passing on our position data directly. Since the position data is
already in clipspace there is no work to do. *If you want 3D it's up to you
to supply shaders that convert from 3D to clipspace because WebGL is only
a rasterization API*.

You might be wondering why does the triangle start in the middle and go to toward the top right.
Clip space in `x` goes from -1 to +1. That means 0 is in the center and positive values will
be to the right of that.

As for why it's on the top, in clip space -1 is at the bottom and +1 is at the top. That means
0 is in the center and so positive numbers will be above the center.

For 2D stuff you would probably rather work in pixels than clipspace so
let's change the shader so we can supply the position in pixels and have
it convert to clipspace for us. Here's the new vertex shader

    <script id="2d-vertex-shader" type="notjs">

    -  attribute vec4 a_position;
    *  attribute vec2 a_position;

    +  uniform vec2 u_resolution;

      void main() {
    +    // convert the position from pixels to 0.0 to 1.0
    +    vec2 zeroToOne = a_position / u_resolution;
    +
    +    // convert from 0->1 to 0->2
    +    vec2 zeroToTwo = zeroToOne * 2.0;
    +
    +    // convert from 0->2 to -1->+1 (clipspace)
    +    vec2 clipSpace = zeroToTwo - 1.0;
    +
    *    gl_Position = vec4(clipSpace, 0, 1);
      }

    </script>

Some things to notice about the changes. We changed `a_position` to a `vec2` since we're
only using `x` and `y` anyway. A `vec2` is similar to a `vec4` but only has `x` and `y`.

Next we added a `uniform` called `u_resolution`. To set that we need to look up its location.

    var resolutionUniformLocation = gl.getUniformLocation(program, "u_resolution");

The rest should be clear from the comments. By setting `u_resolution` to the resolution
of our canvas the shader will now take the positions we put in `positionBuffer` supplied
in pixels coordinates and convert them to clip space.

Now we can change our position values from clip space to pixels. This time we're going to draw a rectangle
made from 2 triangles, 3 points each.

    var positions = [
    *  10, 20,
    *  80, 20,
    *  10, 30,
    *  10, 30,
    *  80, 20,
    *  80, 30,
    ];
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);

And after we set which program to use we can set the value for the uniform we created.
Use program is like `gl.bindBuffer` above in that it sets the current program. After
that all the `gl.uniformXXX` functions set uniforms on the current program.

    gl.useProgram(program);

    ...

    // set the resolution
    gl.uniform2f(resolutionUniformLocation, gl.canvas.width, gl.canvas.height);

And of course to draw 2 triangles we need to have WebGL call our vertex shader 6 times
so we need to change the `count` to `6`.

    // draw
    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    *var count = 6;
    gl.drawArrays(primitiveType, offset, count);

And here it is

Note: This example and all following examples use [`webgl-utils.js`](/webgl/resources/webgl-utils.js)
which contains functions to compile and link the shaders. No reason to clutter the examples
with that [boilerplate](webgl-boilerplate.html) code.

{{{example url="../webgl-2d-rectangle.html" }}}

Again you might notice the rectangle is near the bottom of that area. WebGL considers the bottom left
corner to be 0,0. To get it to be the more traditional top left corner used for 2d graphics APIs
we can just flip the clip space y coordinate.

    *   gl_Position = vec4(clipSpace * vec2(1, -1), 0, 1);

And now our rectangle is where we expect it.

{{{example url="../webgl-2d-rectangle-top-left.html" }}}

Let's make the code that defines a rectangle into a function so
we can call it for different sized rectangles. While we're at it
we'll make the color settable.

First we make the fragment shader take a color uniform input.

    <script id="2d-fragment-shader" type="notjs">
      precision mediump float;

    +  uniform vec4 u_color;

      void main() {
    *    gl_FragColor = u_color;
      }
    </script>

And here's the new code that draws 50 rectangles in random places and random colors.

      var colorUniformLocation = gl.getUniformLocation(program, "u_color");
      ...

      // draw 50 random rectangles in random colors
      for (var ii = 0; ii < 50; ++ii) {
        // Setup a random rectangle
        // This will write to positionBuffer because
        // its the last thing we bound on the ARRAY_BUFFER
        // bind point
        setRectangle(
            gl, randomInt(300), randomInt(300), randomInt(300), randomInt(300));

        // Set a random color.
        gl.uniform4f(colorUniformLocation, Math.random(), Math.random(), Math.random(), 1);

        // Draw the rectangle.
        gl.drawArrays(gl.TRIANGLES, 0, 6);
      }
    }

    // Returns a random integer from 0 to range - 1.
    function randomInt(range) {
      return Math.floor(Math.random() * range);
    }

    // Fills the buffer with the values that define a rectangle.

    function setRectangle(gl, x, y, width, height) {
      var x1 = x;
      var x2 = x + width;
      var y1 = y;
      var y2 = y + height;

      // NOTE: gl.bufferData(gl.ARRAY_BUFFER, ...) will affect
      // whatever buffer is bound to the `ARRAY_BUFFER` bind point
      // but so far we only have one buffer. If we had more than one
      // buffer we'd want to bind that buffer to `ARRAY_BUFFER` first.

      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
         x1, y1,
         x2, y1,
         x1, y2,
         x1, y2,
         x2, y1,
         x2, y2]), gl.STATIC_DRAW);
    }

And here's the rectangles.

{{{example url="../webgl-2d-rectangles.html" }}}

I hope you can see that WebGL is actually a pretty simple API.
Okay, simple might be the wrong word. What it does is simple. It just
executes 2 user supplied functions, a vertex shader and fragment shader and
draws triangles, lines, or points.
While it can get more complicated to do 3D that complication is
added by you, the programmer, in the form of more complex shaders.
The WebGL API itself is just a rasterizer and conceptually fairly simple.

We covered a small example that showed how to supply data in an attribute and 2 uniforms.
It's common to have multiple attributes and many uniforms. Near the top of this article
we also mentioned *varyings* and *textures*. Those will show up in subsequent lessons.

Before we move on I want to mention that for *most* applications updating
the data in a buffer like we did in `setRectangle` is not common. I used that
example because I thought it was easiest to explain since it shows pixel coordinates
as input and demonstrates doing a small amount of math in GLSL. It's not wrong, there
are plenty of cases where it's the right thing to do, but you should [keep reading to find out
the more common way to position, orient and scale things in WebGL](webgl-2d-translation.html).

If you're new to web development or even if you're not please check out [Setup and Installation](webgl-setup-and-installation)
for some tips on how to do WebGL development.

If you're 100% new to WebGL and have no idea what GLSL is or shaders or what the GPU does
then checkout [the basics of how WebGL really works](webgl-how-it-works.html).

You should also, at least briefly read about [the boilerplate code used here](webgl-boilerplate.html)
that is used in most of the examples. You should also at least skim
[how to draw mulitple things](webgl-drawing-multiple-things.html) to give you some idea
of how more typical WebGL apps are structured because unfortunately nearly all the examples
only draw one thing and so do not show that structure.

Otherwise from here you can go in 2 directions. If you are interested in image procesing
I'll show you [how to do some 2D image processing](webgl-image-processing.html).
If you are interested in learning about translation,
rotation and scale and eventually 3D then [start here](webgl-2d-translation.html).

<div class="webgl_bottombar">
<h3>What does type="notjs" mean?</h3>
<p>
<code>&lt;script&gt;</code> tags default to having JavaScript in them.
You can put no type or you can put <code>type="javascript"</code> or
<code>type="text/javascript"</code> and the browser will interpret the
contents as JavaScript. If you put anything for else for <code>type</code> the browser ignores the
contents of the script tag. In other words <code>type="notjs"</code>
or <code>type="foobar"</code> have no meaning as far as the browser
is concerned.</p>
<p>This makes the shaders easy to edit.
Other alterntives include string concatenations like</p>
<pre class="prettyprint">
  var shaderSource =
    "void main() {\n" +
    "  gl_FragColor = vec4(1,0,0,1);\n" +
    "}";
</pre>
<p>or we'd could load shaders with ajax requests but that is slow and asynchronous.</p>
<p>A more modern alternative would be to use multiline template literals.</p>
<pre class="prettyprint">
  var shaderSource = `
    void main() {
      gl_FragColor = vec4(1,0,0,1);
    }
  `;
</pre>
<p>Multiline template literals work in all browsers that support WebGL.
Unfortunately they don't work in really old browsers so if you care
about supporting a fallback for those browsers you might not want to
use mutliline template literals or you might want to use <a href="https://babeljs.io/">a transpiler</a>.
</p>
</div>
