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

   缓冲区并不是随机存取的。一个顶点着色器(Vertex Shader)执行指定的次数,每一次执行时提取缓冲区中相应的数据分配给一个attribute。

2. Uniforms

   Uniforms是你在执行你的着色器程序前设定的全局变量(译注:uniform意味着这个变量是一致不变的)。

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
    precision mediump float;

    void main() {
      // gl_FragColor 是片元着色器的一个特殊变量,用来设置颜色值
      // 译注: WebGL遵循严格的数据格式,默认数据精度为中等精度的浮点数
      //      因此我们需要始终使用浮点数来表示数据
      gl_FragColor = vec4(1.0, 0.0, 0.5, 1.0);

    }

这里我们设置`gl_FragColor`为`1.0, 0.0, 0.5, 1.0`,分别对应rgba中的红/绿/蓝/透明度通道,
即设定红色通道为1, 绿色通道为0.5, 蓝色通道为0.5, 透明度通道为1。需要注意的是,颜色在WebGL中是用0到1的取值表示的。

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
你可以用你所熟悉的任何JavaScript字符串格式来创建GLSL字符串, 比如通过字符串拼接,使用AJAX来从服务器获取或者通过模板字符串。
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
        gl_FragColor = vec4(1.0, 0.0, 0.5, 1.0);
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

WebGL可以让我们在全局的绑定点(bind points)上操作许多WebGL资源。
你可以将绑定点(bind points)看做WebGL内部的全局变量。
首先要将资源绑定到绑定点, 其他所有的函数都将通过这个绑定点读取数据。
接下来就将我们例子中创建的positionBuffer绑定到绑定点。

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
(译注:如果使用的是两倍屏需要相应的改变所设定的像素值)

本教程中的所有实例如果在独立的页面中展示会使用400x300像素的画布,但如果像在这个页面中的内嵌iframe中展示的时候,将会填满iframe所设定的空间。
使用CSS来设定画布的大小让我们可以非常简单的处理这两种不同的情况。

    webglUtils.resizeCanvasToDisplaySize(gl.canvas);

之前我们提到过WebGL使用剪辑空间坐标(clip space)来表示坐标位置。因此我们需要告诉WebGL如何将像素坐标信息和剪辑空间坐标信息进行映射。
通过调用`gl.viewport`并将画布的尺寸传入可以让WebGL完成这样的映射。

    gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);

上面这行代码告诉WebGL将-1到+1的剪辑空间映射到宽度范围为0->`gl.canvas.width`,高度范围从0->`gl.canvas.height`的画布中。

设定清空画布的颜色为`0.0, 0.0, 0.0, 0.0`(对应rgba), 因此画布显示为透明。

    // 清空画布
    gl.clearColor(0.0, 0.0, 0.0, 0.0);
    gl.clear(gl.COLOR_BUFFER_BIT);

告诉WebGL我们将执行的着色器程序

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

`gl.vertexAttribPointer`默认的将当前使用的`ARRAY_BUFFER`绑定到了attribute变量上。也就是说attribute变量和我们创建的`positionBuffer`
绑定在了一起。这意味着就算我们这时将`ARRAY_BUFFER`绑定到别的东西上, attribute变量仍然从`positionBuffer`中获取数据。

注意顶点着色器代码中的attribute变量`a_position`使用`vec4`数据格式

    attribute vec4 a_position;
`vec4`是一个四维浮点数值。你可以认为它等同于JavaScript中的`a_position = {x: 0, y: 0, z: 0, w: 0}`。
我们还设置了`size = 2`。attributes默认值为`0.0, 0.0, 0.0, 1.0`所以这里我们的attribute变量将从缓冲区中得到四维数据的前两个值(x和y),
z和w分量分别为默认值0和1。

最后我们终于可以让WebGL执行我们的GLSL程序了。

    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    var count = 3;
    gl.drawArrays(primitiveType, offset, count);

`count`值为`3`意味着我们将执行顶点着色器三次。第一次执行时顶点着色器中的`a_position.x`和`a_position.y`将被设定为positionBuffer中的前两个值。
第二次执行时,这两个值分别被设定为接下来的两个值。最后一次执行则为positionBuffer中的最后两个值。

我们设定`primitiveType`为`gl.TRIANGLES`, 因此顶点着色器没执行三次WebGL会根据我们的顶点数据画三角形。不论画布多大,这些数据都会被映射到
剪辑空间, 每个方向的值从-1.0到+1.0。

这里我们仅仅是将positionBuffer中的数据复制到`gl_Position`中,所以三角形三个顶点对应的剪辑坐标值为

      0.0, 0.0,
      0.0, 0.5,
      0.7, 0.0,

WebGL将要把剪辑空间坐标转换为像素坐标。如果一块画布为400x300像素。我们可以得到相应的点的坐标值为

     剪辑空间           像素坐标
     0.0, 0.0    ->    200, 150
     0,0, 0.5    ->    200, 225
     0.7, 0.0    ->    340, 150

接下来WebGL就会渲染这个三角形。对于需要绘制的每一个像素,WebGL都会调用片元着色器。
我们示例中的片元着色器仅仅是将`gl_FragColor`设定为`1.0, 0.0, 0.5, 1.0`, canvas中颜色通道为8位, 因此WebGL将使用rgba值为
`[255, 0, 127, 255]`的颜色进行绘制。

这是执行的结果

{{{example url="../webgl-fundamentals.html" }}}

从这个示例你可以看到顶点着色器仅仅只是接收了我们传给它的顶点数据信息。*如果你想要绘制3D的图形, 你只需要为着色器提供剪辑空间的3D坐标值。*

你也许会疑惑为什么这个三角形是从中间开始向上和右绘制。剪辑空间的取值范围是从-1到+1,因此坐标的0值位于画布中心,而正值坐标指向画布右边。

至于为什么三角形在画布的上方, 在剪辑空间中-1是位于底部,+1在顶部。所以0位于垂直中心而垂直方向的正值指向画布上方。


对于2D图形的绘制你可能更习惯于使用像素坐标而不是剪辑空间坐标。因此我们这里对着色器代码稍作修改, 让它能将我们提供的像素坐标值自动映射到剪辑空间。
下面是修改过的顶点着色器代码:

    <script id="2d-vertex-shader" type="notjs">

    -  attribute vec4 a_position;
    *  attribute vec2 a_position;

    +  uniform vec2 u_resolution;

      void main() {
    +    // 将像素坐标映射到(0.0->1.0)取值范围内
    +    //(译注:u_resolution代表画布分辨率,这里为包含长宽信息的二维向量)
    +    vec2 zeroToOne = a_position / u_resolution;
    +
    +    // 将坐标从(0.0->1.0)映射到(0.0->2.0)
    +    vec2 zeroToTwo = zeroToOne * 2.0;
    +
    +    // 将坐标从(0.0->2.0)映射到(-1.0->1.0) (剪辑空间坐标)
    +    vec2 clipSpace = zeroToTwo - 1.0;
    +
    *    gl_Position = vec4(clipSpace, 0.0, 1.0);
      }

    </script>

在这些修改中需要注意的是,我们将`a_position`的数据类型从`vec4`变为了`vec2`,因为这个示例中我们仅仅使用了`x`和`y`方向的取值。
一个`vec2`数值类似`vec4`但是仅仅只有两个维度的分量。

接下来我们添加了一个`uniform`变量`u_resolution`。跟attribute变量类似,要通过JavaScript设定这个值我们需要找到它的位置

    var resolutionUniformLocation = gl.getUniformLocation(program, "u_resolution");

其他的部分应该注释中已经很清楚的说明了。通过将`u_resolution`设定为画布的分辨率,我们可以将像素坐标映射到剪辑空间中。

现在我们可以使用像素坐标来绘制图形了,这次我们将绘制由两个三角形组成的矩形。每个三角形包含三个坐标点。

    var positions = [
    *  10, 20,
    *  80, 20,
    *  10, 30,
    *  10, 30,
    *  80, 20,
    *  80, 30,
    ];
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);

当决定了使用哪个program(译注:一对顶点着色器和片元着色器可以组成一个program)后,我们可以给创建的uniform变量传数据。
我们使用`gl.useProgram`来设定当前使用的program, 之后调用`gl.uniformXXX`(译注:XXX代表不同的数据格式)方法来设定当前program的uniform参数。

    gl.useProgram(program);

    ...

    // 设定uniform参数为画布分辨率(2f表示两个浮点数)
    gl.uniform2f(resolutionUniformLocation, gl.canvas.width, gl.canvas.height);

为了画出两个三角形,我们需要WebGL调用顶点着色器6次,所以我们需要将`count`值变为`6`。

    // 绘制
    var primitiveType = gl.TRIANGLES;
    var offset = 0;
    *var count = 6;
    gl.drawArrays(primitiveType, offset, count);

注意: 本教程中的所有实例都使用了[`webgl-utils.js`](/webgl/resources/webgl-utils.js)脚本, 它包含了编译和组合着色器的工具函数。
将这些工具函数[boilerplate](webgl-boilerplate.html)提取到独立的文件是为了让我们的代码简洁易懂。

{{{example url="../webgl-2d-rectangle.html" }}}

这次你看到矩形处于画布的底部。WebGL以左下角为原点(0, 0)。如果要将它转变为传统2D canvas的以左上角为原点, 我们需要对剪辑空间的y轴进行翻转。

    *   gl_Position = vec4(clipSpace * vec2(1.0, -1.0), 0.0, 1.0);

现在这个矩形出现在了我们需要的位置上。

{{{example url="../webgl-2d-rectangle-top-left.html" }}}

我们还可以将定义矩形的代码封装成函数, 通过传参设定它为不同的大小, 并将颜色作为一个变量通过传参设定。

首先我们需要在片元着色器中设定一个接收颜色信息的uniform变量

    <script id="2d-fragment-shader" type="notjs">
      precision mediump float;

    +  uniform vec4 u_color;

      void main() {
    *    gl_FragColor = u_color;
      }
    </script>

下面是绘制50个随机位置随机颜色矩形的代码

      // 获取uniform变量u_color的位置
      var colorUniformLocation = gl.getUniformLocation(program, "u_color");
      ...

      // 绘制50个随机位置随机颜色矩形
      for (var ii = 0; ii < 50; ++ii) {
        // 设置随机矩形坐标位置
        // 因为我们将positionBuffer绑定到了ARRAY_BUFFER绑定点上
        // 这里会将矩形的位置颜色数据写入positionBuffer
        setRectangle(
            gl, randomInt(300), randomInt(300), randomInt(300), randomInt(300));

        // 设置随机颜色
        gl.uniform4f(colorUniformLocation, Math.random(), Math.random(), Math.random(), 1);

        // 绘制矩形
        gl.drawArrays(gl.TRIANGLES, 0, 6);
      }
    }

    // 返回一个取值范围从 0 到 -1 的整数
    function randomInt(range) {
      return Math.floor(Math.random() * range);
    }

    // 将矩形的坐标数据填充到缓冲区

    function setRectangle(gl, x, y, width, height) {
      var x1 = x;
      var x2 = x + width;
      var y1 = y;
      var y2 = y + height;

      // 注意: gl.bufferData(gl.ARRAY_BUFFER, ...)
      // 会影响`ARRAY_BUFFER`绑定点上绑定缓冲区
      // 但是在我们这一节的例子中,只有一个缓冲区对象
      // 如果我们有多个缓冲区对象,我们需要首先将当前用到的缓冲区绑定到`ARRAY_BUFFER`

      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
         x1, y1,
         x2, y1,
         x1, y2,
         x1, y2,
         x2, y1,
         x2, y2]), gl.STATIC_DRAW);
    }

下面是绘制的结果

{{{example url="../webgl-2d-rectangles.html" }}}

希望你能通过这里的例子了解WebGL其实是非常简单的API。
也许,"简单"并不准确,但它所做的事情是很简单的。
它仅仅是执行了开发者提供的顶点着色器代码和片元着色器代码来绘制三角形,线和点。
当涉及到3D应用的时候,它可能会变得比较复杂,但就WebGL本身来说,它的概念非常简单,仅仅是一个栅格化功能器。

这一节中的示例讲解了怎样将数据传入attribute变量和uniform变量。
对WebGL程序来说, 通常会有多个attribute和uniform变量。上面我们还提到过 *varying* 和 *texture* 变量。
我们会在接下来的章节中涉及。

这里需要提一下,我们上面`setRectangle`的示例中所做的操作并不适合大多数需要在缓冲区更新数据的应用。
我使用了这个例子是因为我认为使用像素坐标会让大家更清晰的理解, 还可以在GLSL中进行一些算数计算来让大家对它有进一步了解。
它本身并没有问题,  但是大家需要[继续学习WebGL中更通用的定位, 定向, 缩放图形的方法](webgl-2d-translation.html)。

如果你是一个网页开发的新手(也可能不是), [安装及初始化(Setup and Installation)](webgl-setup-and-installation)
提供了一些怎样进行WebGL开发的建议。

如果你对WebGL没有任何了解, 完全不知道什么是GLSL,shader,GPU,你可以看看 [WebGL如何工作](webgl-how-it-works.html)。

建议你了解一下示例中用到的组合两个着色器的工具函数( [the boilerplate code](webgl-boilerplate.html)),
也可以通过阅读[如何绘制多个图形](webgl-drawing-multiple-things.html)来了解更多的更典型的WebGL程序是怎样构建的,因为在我们的示例中通常
只绘制一个图形而缺乏对这种典型结构的介绍。

接下来你可以从两个方向继续学习。如果你对WebGL中的图片处理感兴趣,可以看看 [WebGL中如何绘制2D图片](webgl-image-processing.html)。
如果你对图形平移, 旋转以及缩放感兴趣,你可以阅读 [WebGL 2D变换](webgl-2d-translation.html)

<div class="webgl_bottombar">
<h3>type="notjs" 是什么?</h3>
<p>
 <code>&lt;script&gt;</code> 标签内部默认包含的是JavaScript代码。
你既可以不设定type标签,也可以可以设定 <code>type="javascript"</code> 或者
<code>type="text/javascript"</code>,浏览器会自动将标签内的代码作为JavaScript来解析。

如果你在<code>type</code>中设定其他的值,浏览器会自动忽略这个标签内的内容。
也就是说,<code>type="notjs"</code>或是<code>type="foobar"</code>对于浏览器来说都是一样的没有任何意义。

<p>我们将着色器代码放入<code>type="notjs"</code>的<code>script</code>标签中, 可以让我们的着色器代码直观并且易于编辑。
当然我们也可以使用字符串拼接的方式来编写我们的着色器代码</p>


<pre class="prettyprint">
  var shaderSource =
    "void main() {\n" +
    "  gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);\n" +
    "}";
</pre>

<p>我们也可以使用ajax请求来获取着色器代码,但它是异步进行并且速度会相对较慢</p>
<p>还有一个更加新的方法是使用ES6标准中的模板字符串</p>
<pre class="prettyprint">
  var shaderSource = `
    void main() {
      gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
    }
  `;
</pre>
<p>模板字符串在所有支持WebGL的现代浏览器中都可以使用。但是对于比较古老的浏览器,你需要使用<a href="https://babeljs.io/">Babel</a></p>
来让你的代码被这些浏览器支持。
</div>
