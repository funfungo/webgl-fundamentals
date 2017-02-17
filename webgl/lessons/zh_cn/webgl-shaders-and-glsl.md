Title: WebGL 着色器(Shaders) 和 GLSL
Description: 什么是shader?什么是GLSL?

本篇是 [WebGL基础原理](webgl-fundamentals.html)的后续教程。
在开始本节内容前你可能需要先了解[WebGL 如何工作](webgl-how-it-works.html)。

在之前的讲解中我们多次提到了着色器和GLSL,但并没有对这两个概念深入讲解。
我认为在之前的示例中已经能让大家初步认识它们, 现在会让大家更加深入的去理解这两个概念。

在 [WebGL 如何工作](webgl-how-it-works.html)提到过WebGL在绘制图形时需要两个着色器,一个顶点着色器和一个片元着色器。
每个着色器都是一个函数, 顶点着色器和片元着色器可以连接在一起编译为一个着色器程序。
一个典型的WebGL应用会包含多个着色器程序。

## 顶点着色器(Vertex Shader)

顶点着色器的工作是生成一个剪辑空间, 它始终使用这样一种函数形式:

    void main() {
       gl_Position = doMathToMakeClipspaceCoordinates
    }

顶点着色器会在处理每个顶点时调用一次, 每次调用时你需要对一个特殊的全局变量`gl_Position`传递某个剪辑空间坐标值。

顶点着色器需要传入数据, 它主要通过三种形式获得数据。

1.  [Attributes](#attributes) (从缓冲区中获得的数据)
2.  [Uniforms](#uniforms) (每次绘制时所有顶点共享的数据)
3.  [Textures](#textures-in-vertex-shaders) (纹理/纹素数据)

### Attributes

顶点着色器最普遍的获取数据的方式是通过缓冲区和attribute变量。
在 [WebGL 如何工作](webgl-how-it-works.html)中介绍了缓冲区和attribute变量。
你需要创建缓冲区,

    var buf = gl.createBuffer();

将数据放入缓冲中

    gl.bindBuffer(gl.ARRAY_BUFFER, buf);
    gl.bufferData(gl.ARRAY_BUFFER, someData, gl.STATIC_DRAW);

你需要在着色器程序中找到你需要的attribute变量的位置。

    var positionLoc = gl.getAttribLocation(someShaderProgram, "a_position");

在绘制时, 告诉WebGL如何从缓冲区中提取数据到attribute变量。

    // 开启某个attribute变量
    gl.enableVertexAttribArray(positionLoc);

    var numComponents = 3;  // (x, y, z)
    var type = gl.FLOAT;    // 32位浮点数
    var normalize = false;  // 不对数据进行标准化
    var offset = 0;         // 数据从缓冲区头部开始存储
    var stride = 0;         // 每次迭代的数据之间存储空间间隔为0// 0 = use the correct stride for type and numComponents

    gl.vertexAttribPointer(positionLoc, numComponents, type, false, stride, offset);

在 [WebGL 如何工作](webgl-how-it-works.html)中我们没有对传递的数据进行任何的操作,直接将其传给了`gl_Position`。

    attribute vec4 a_position;

    void main() {
       gl_Position = a_position;
    }

Attribute变量的类型可以是`float`, `vec2`, `vec3`, `vec4`, `mat2`, `mat3`, and `mat4`。

### Uniforms
对于着色器来说,uniform变量是传递给着色器, 在每次绘制中对每个顶点保持一致的数据,是所有顶点共用的数据。
举一个简单的例子, 我们在顶点着色器中添加一个代表偏移量的uniform变量。

    attribute vec4 a_position;
    +uniform vec4 u_offset;

    void main() {
       gl_Position = a_position + u_offset;
    }

这样我们就可以对每个顶点设置相同数值的偏移。首先我们需要在初始化时找到这个uniform变量的地址。

    var offsetLoc = gl.getUniformLocation(someProgram, "u_offset");

在绘制之前设置这个uniform变量

    gl.uniform4fv(offsetLoc, [1, 0, 0, 0]);  // 向右移动半个画布宽度
                                             // 译注: 这里uniform变量代表偏移量,
                                             // [1, 0, 0, 0]分别代表x,y,z,w分量上的偏移值,
                                             // 而剪辑空间的取值范围为(-1.0, 1.0),
                                             // 当x = 1时代表在x方向向正方形移动半个画布宽度

注意uniform变量属于定义它的着色器程序。如果有多个着色器程序, 它们中相同名称的uniform变量都会有自己独立的内存地址和取值。
当调用`gl.uniformXXX`(XXX代表数值类型)时, 你只是为当前的着色器程序设置uniform变量。
(当前的着色器是你在`gl.useProgram`中传递的着色器)

Uniform变量可以存储很多数据类型, 设置特定类型的uniform变量只需调用相应的方法
(译注: vec2/3/4分别表示具有相应分量个数的向量数据)

    gl.uniform1f (floatUniformLoc, v);                 // 设置float数据
    gl.uniform1fv(floatUniformLoc, [v]);               // 设置float或float数组
    gl.uniform2f (vec2UniformLoc,  v0, v1);            // 设置vec2数据
    gl.uniform2fv(vec2UniformLoc,  [v0, v1]);          // 设置vec2或vec2数组
    gl.uniform3f (vec3UniformLoc,  v0, v1, v2);        // 设置vec3数据
    gl.uniform3fv(vec3UniformLoc,  [v0, v1, v2]);      // 设置vec3或vec3数组
    gl.uniform4f (vec4UniformLoc,  v0, v1, v2, v4);    // 设置vec4数据
    gl.uniform4fv(vec4UniformLoc,  [v0, v1, v2, v4]);  // 设置vec4或vec4数组

    gl.uniformMatrix2fv(mat2UniformLoc, false, [  4x element array ])  // 设置mat2或mat2数组
    gl.uniformMatrix3fv(mat3UniformLoc, false, [  9x element array ])  // 设置mat3或mat3数组
    gl.uniformMatrix4fv(mat4UniformLoc, false, [ 16x element array ])  // 设置mat4或mat4数组

    gl.uniform1i (intUniformLoc,   v);                 // 设置int数据
    gl.uniform1iv(intUniformLoc, [v]);                 // 设置int或int数组
    gl.uniform2i (ivec2UniformLoc, v0, v1);            // 设置ivec2数据
    gl.uniform2iv(ivec2UniformLoc, [v0, v1]);          // 设置ivec2或ivec2数组
    gl.uniform3i (ivec3UniformLoc, v0, v1, v2);        // 设置ivec3数据
    gl.uniform3iv(ivec3UniformLoc, [v0, v1, v2]);      // 设置ivec3或ivec3数组
    gl.uniform4i (ivec4UniformLoc, v0, v1, v2, v4);    // 设置ivec4数据
    gl.uniform4iv(ivec4UniformLoc, [v0, v1, v2, v4]);  // 设置ivec4或ivec4数组

    gl.uniform1i (sampler2DUniformLoc,   v);           // 设置sampler2D (textures)数据
    gl.uniform1iv(sampler2DUniformLoc, [v]);           // 设置sampler2D或sampler2D数组

    gl.uniform1i (samplerCubeUniformLoc,   v);         // 设置samplerCube (textures)数据
    gl.uniform1iv(samplerCubeUniformLoc, [v]);         // 设置samplerCube或samplerCube数组

还有诸如 `bool`, `bvec2`, `bvec3`等类型的数据,它们都使用 `gl.uniform?f?` 或 `gl.uniform?i?` 这样的方法设置uniform变量的数据。

对于uniform数组数据,你可以一次性的设置数组中所有的值,比如:

    // 着色器中定义uniform变量为具有三个vec2类型数据的数组
    uniform vec2 u_someVec2[3];

    // JavaScript中找到数组位置
    var someVec2Loc = gl.getUniformLocation(someProgram, "u_someVec2");

    // 绘制时设置数组的取值
    gl.uniform2fv(someVec2Loc, [1, 2, 3, 4, 5, 6]);  // 设置u_someVec3数组的数值
                                                     // 译注: someVec2代表每个值为具有两个分量的向量值
                                                     // 因此总共设置时参数具有6个数据

不过如果你想要设定数组中的单个数值, 你需要找到相应的uniform变量的地址

    // JavaScript中初始化时找到相应变量的地址
    var someVec2Element0Loc = gl.getUniformLocation(someProgram, "u_someVec2[0]");
    var someVec2Element1Loc = gl.getUniformLocation(someProgram, "u_someVec2[1]");
    var someVec2Element2Loc = gl.getUniformLocation(someProgram, "u_someVec2[2]");

    // 绘制时设置变量的取值
    gl.uniform2fv(someVec2Element0Loc, [1, 2]);  // set element 0
    gl.uniform2fv(someVec2Element1Loc, [3, 4]);  // set element 1
    gl.uniform2fv(someVec2Element2Loc, [5, 6]);  // set element 2

类似的你还可以创建一个结构

    struct SomeStruct {
      bool active;
      vec2 someVec2;
    };
    uniform SomeStruct u_someThing;

找到这个结构中相应变量的位置

    var someThingActiveLoc = gl.getUniformLocation(someProgram, "u_someThing.active");
    var someThingSomeVec2Loc = gl.getUniformLocation(someProgram, "u_someThing.someVec2");

### Textures in Vertex Shaders

请看 [片元着色器中的纹理(Texture)](#textures-in-fragment-shaders)。

## 片元着色器(Fragment Shader)
片元着色器为每个需要渲染的像素提供颜色信息,它始终使用这样的函数形式:

    precision mediump float;

    void main() {
       gl_FragColor = doMathToMakeAColor;
    }

片元着色器在每个像素渲染后都会调用一次, 每次调用时你需要设置一个特殊的全局变量 `gl_FragColor`为某一颜色值。

片元着色器同样需要传入数据, 它们也有三种方式来获取数据。

1.  [Uniforms](#uniforms) (每次绘制时所有像素共享的数据)
2.  [Textures](#textures-in-fragment-shaders) (纹理/纹素数据)
3.  [Varyings](#varyings) (从顶点着色器中插值计算后同步的数据)

### 片元着色器中的uniform变量

请看 [着色器中的uniform变量](#uniforms)。
See [Uniforms in Shaders](#uniforms).

### 片元着色器中的纹理(texture)对象

从纹理对象中获取数据时, 需要创建一个 `sampler2D`类型的uniform变量, 使用GLSL中一个方法 `Texture2D` 从中提取数值。

    precision mediump float;

    uniform sampler2D u_texture;

    void main() {
       vec2 texcoord = vec2(0.5, 0.5)  // get a value from the middle of the texture
       gl_FragColor = texture2D(u_texture, texcoord);
    }

从纹理对象中获得的数据是什么取决于很多配置,请看 [WebGL 3D纹理](webgl-3d-textures.html)。
最基本的是我们要在JavaScript中将数据传到纹理对象中, 例如:

    var tex = gl.createTexture();       // 创建一个texture元素
    gl.bindTexture(gl.TEXTURE_2D, tex); // 绑定texture
    var level = 0;
    var width = 2;
    var height = 1;
    var data = new Uint8Array([
       255, 0, 0, 255,   // 一个红色像素
       0, 255, 0, 255,   // 一个绿色像素
    ]);
    gl.texImage2D(gl.TEXTURE_2D, level, gl.RGBA, width, height, 0, gl.RGBA, gl.UNSIGNED_BYTE, data);

初始化时需要查询着色器程序中uniform变量的地址。

    var someSamplerLoc = gl.getUniformLocation(someProgram, "u_texture");

激活一个纹理单元,并将创建的纹理对象进行绑定

    var unit = 5;  // 选择某个单元
    gl.activeTexture(gl.TEXTURE + unit); // 激活所选的纹理单元
    gl.bindTexture(gl.TEXTURE_2D, tex);  // 译注: gl.TEXTURE_2D是指当前纹理对象为二维纹理

告诉着色器你将纹理对象绑定到了哪个纹理单元

    gl.uniform1i(someSamplerLoc, unit);

### Varyings

varying变量可以将一个参数从顶点着色器中传到片元着色器中,
在 [WebGL 如何工作](webgl-how-it-works.html)中我们有涉及varying变量的示例。

要使用varying变量需要在顶点着色器和片元着色器中声明同样名称的varying变量。
顶点着色器每次执行时给varying变量设置数据, 当WebGL绘制每个像素时, 将使用所有顶点的varying数据通过插值计算得到的当前像素的值
传入片元着色器中相应的varying变量中。

顶点着色器代码

    attribute vec4 a_position;

    uniform vec4 u_offset;

    +varying vec4 v_positionWithOffset;

    void main() {
      gl_Position = a_position + u_offset;
    +  v_positionWithOffset = a_position + u_offset;
    }

片元着色器代码

    precision mediump float;

    +varying vec4 v_positionWithOffset;

    void main() {
    +  // 将剪辑空间坐标数据(范围从 -1.0 到 1.0)映射到颜色数据空间(从 0.0 到 1.0).
    +  vec4 color = v_positionWithOffset * 0.5 + 0.5
    +  gl_FragColor = color;
    }

这个例子中每个顶点的颜色数据和剪辑空间坐标信息连接在了一起, 因此对于画布上的每个像素, 根据其坐标的不同会获得不同的颜色数据。

## GLSL
GLSL是Graphics Library Shader Language的缩写,是编写着色器使用的语言。
它拥有一些在JavaScript中不常见的特性, 是为了栅格化图形时所必须进行的一系列数学计算所设计的。
在之前的例子中, 会出现一些数据类型比如`vec2`, `vec3`, and `vec4`分别代表拥有2/3/4个分量的向量数据。
类似的 `mat2`, `mat3`和 `mat4`分别代表了2x2, 3x3, 4x4的矩阵数据。
你可以使用一个常量去与一个向量相乘

    vec4 a = vec4(1, 2, 3, 4);
    vec4 b = a * 2.0;
    // b is now vec4(2, 4, 6, 8);

同样的它还可以完成矩阵与矩阵相乘或是向量与举证相乘的计算

    mat4 a = ???
    mat4 b = ???
    mat4 c = a * b;

    vec4 v = ???
    vec4 y = c * v;

此外,还可以使用几种特殊的属性来使用向量中某一具体分量, 比如在vec4向量中:

    vec4 v;

*   `v.x` 或 `v.s` 或 `v.r` 或 `v[0]` 都代表向量第一个分量.
*   `v.y` 或 `v.t` 或 `v.g` 或 `v[1]` 都代表向量的第二个分量.
*   `v.z` 或 `v.p` 或 `v.b` 或 `v[2]` 都代表向量的第三个分量.
*   `v.w` 或 `v.q` 或 `v.a` 或 `v[3]` 都代表向量的第四个分量.

(译注: 使用xyzw还是rgba来表示向量分量取决于向量表示什么数据,比如坐标数据使用xyzw表示会更加清晰,颜色数据则是使用rgba更好)

甚至你可以在使用时任意搭配向量的各个分量

    v.yyyy

等同于

    vec4(v.y, v.y, v.y, v.y)

表示使用一个四个分量都为v向量第二个分量值的向量

同样的

    v.bgra

等同于

    vec4(v.b, v.g, v.r, v.a)

当构建一个向量或者矩阵数据时, 你可以同时传入不同部分的数据,比如:

    vec4(v.rgb, 1)

等同于

    vec4(v.r, v.g, v.b, 1)

同样的

    vec4(1)

等同于

    vec4(1, 1, 1, 1)

你需要注意的一件事是GLSL是一种强类型的语言

    float f = 1;  // 报错,因为1是一个int的类型,不能将它赋值给float类型的变量

正确的做法是:

    float f = 1.0;      // use float
    float f = float(1)  // cast the integer to a float

上面例子中 `vec4(v.rgb, 1)`没有报错是因为 `vec4` 会自动将内部的数值转化就像执行了 `float(1)`。

GLSL有很多内置函数。 其中很多函数会同时处理多个数据(译注:因为某一变量类型可能包含多个分量),例如:

    T sin(T angle)

T可能为 `float`, `vec2`, `vec3` or `vec4`。如果为 `vec4` 你将最终得到一个 `vec4` 类型的数据, 这个数据是通过
对数据类型同样为 `vec4`的参数的每一部分分别执行sin操作获得的。 用另一种方式说, 如果v为一个 `vec4` 数据,那么

    vec4 s = sin(v);

等同于

    vec4 s = vec4(sin(v.x), sin(v.y), sin(v.z), sin(v.w));

有时一个参数为float类型, 其他数据为向量或矩阵数据。在执行时会将这个float类型分别应用到每个分量中,比如
如果 `v1` 和 `v2`为 `vec4`类型, `f`为float类型,则
Sometimes one argument is a float and the rest is `T`. That means that float will be applied
to all the components. For example if `v1` and `v2` are `vec4` and `f` is a float then

    vec4 m = mix(v1, v2, f);

等同于
    vec4 m = vec4(
      mix(v1.x, v2.x, f),
      mix(v1.y, v2.y, f),
      mix(v1.z, v2.z, f),
      mix(v1.w, v2.w, f));

mix函数为一个内置方法,你可以在 [WebGL 参考索引](https://www.khronos.org/files/webgl/webgl-reference-card-1_0.pdf)
中查看。如果你想要更多干活可以查看 [The OpenGL ES Shading Language](https://www.khronos.org/files/opengles_shading_language.pdf)。

## 总结

WebGL创建了各种着色器,并将数据传入这些着色器, 之后调用 `gl.drawArrays` 或者 `gl.drawElements`来绘制图形。
绘制时WebGL调用当前使用的顶点着色器处理每个顶点包含的信息, 之后使用当前设置的片元着色器来渲染每个像素。

创建着色器程序会执行一些代码, 对于任何WebGL程序来说这些代码都是一样的,所以在讲解中我们忽略了这一部分,如果你想了解怎样编译GLSL着色器
(片元着色器和顶点着色器)并且将它们合并编译成着色器程序的话可以查看 [WebGL boilerplate](webgl-boilerplate.html)。

接下来你可以从两个方向继续学习。如果你对WebGL中的图片处理感兴趣,可以看看 [WebGL中如何绘制2D图片](webgl-image-processing.html)。
如果你对图形平移, 旋转以及缩放感兴趣,你可以阅读 [WebGL 2D变换](webgl-2d-translation.html)



