# OpenGL Introduction

- [OpenGL Introduction](#opengl-introduction)
  - [Introduction](#introduction)
    - [Render Pipeline](#render-pipeline)
    - [Context](#context)
  - [First OpenGL Program](#first-opengl-program)
    - [GLFW](#glfw)
    - [Several Steps](#several-steps)
    - [Example Code](#example-code)
  - [GLSL Basics](#glsl-basics)
    - [Data Type](#data-type)
    - [Qualifier](#qualifier)
    - [Internal Variable](#internal-variable)
  - [References](#references)

## Introduction

OpenGL(Open Graphics Library)一般被认为是一个图形应用API，包含了一系列可以操作图像、图形的函数，进而利用GPU实现2D或3D图像渲染。然而，OpenGL本身并不是一个API，它仅仅是一个由Khronos组织制定并维护的规范。OpenGL规范严格规定了每个函数该如何执行，以及它们的输出值。至于内部具体每个函数是如何实现的，将由OpenGL库的开发者自行决定。实际上，OpenGL库的开发者通常是显卡的生产商。当产生一个bug时通常可以通过升级显卡驱动来解决。这些驱动会包括你的显卡能支持的最新版本的OpenGL。

**OpenCV & OpenCL & OpenMP**：

OpenCV(Open Source Computer Vision Library)是一个开源发行的跨平台计算机视觉库，可以运行在Linux、Windows和Mac OS操作系统上。它轻量级而且高效——由一系列 C 函数和少量 C++ 类构成，同时提供了Python、Ruby、MATLAB等语言的接口，实现了图像处理和计算机视觉方面的很多通用算法。它由C++语言编写，它的主要接口也是C++语言，但是依然保留了大量的C语言接口。该库也有大量的Python, Java and MATLAB/OCTAVE (版本2.5)的接口。这些语言的API接口函数可以通过在线文档获得。如今也提供对于C#,Ch, Ruby的支持。

OpenCL(Open Computing Language)是第一个面向异构系统通用目的并行编程的开放式、免费标准，也是一个统一的编程环境，便于软件开发人员为高性能计算服务器、桌面计算系统、手持设备编写高效轻便的代码，而且广泛适用于多核心处理器(CPU)、图形处理器(GPU)、Cell类型架构以及数字信号处理器(DSP)等其他并行处理器，在游戏、娱乐、科研、医疗等各种领域都有广阔的发展前景。

OpenMP(Open Multi-Processing)是用于共享内存并行系统的多处理器程序设计的一套指导性的编译处理方案(Compiler Directive)。OpenMP支持的编程语言包括C语言、C++和Fortran；而支持OpenMP的编译器包括Sun Compiler，GNU Compiler和Intel Compiler等。OpenMP提供了对并行算法的高层的抽象描述，程序员通过在源代码中加入专用的pragma来指明自己的意图，由此编译器可以自动将程序进行并行化，并在必要之处加入同步互斥以及通信。当选择忽略这些pragma，或者编译器不支持OpenMP时，程序又可退化为通常的程序(一般为串行)，代码仍然可以正常运作，只是不能利用多线程来加速程序执行。OpenMP和OpenCL都是用于高性能计算机，但前者主要是基于CPU的并行，后者主攻是异构系统中GPU并行。

**OpenGL模式**：

OpenGL的模式分为核心模式(Core Profile)和立即渲染模式(Immediate Mode)。其中，核心模式是OpenGL的一个现代版本，它移除了过时的特性，强制使用现代的、规范化的编程模型。核心模式专注于向后兼容性和性能，鼓励使用顶点着色器、片段着色器等现代渲染技术。立即渲染模式，也成为经典模式，是早期OpenGL版本中的一个编程模型。它允许开发者直接调用函数来绘制几何图形，如glBegin()和glEnd()。这种模式在现代OpenGL中已被弃用，因为它效率较低，且不支持并行计算。

### Render Pipeline

渲染管线（Render Pipeline），也被称为渲染流水线，主要包括以下流程：

- 应用程序阶段：该阶段由应用程序编写的代码组成，主要负责设定场景和对象的各种属性以及交互逻辑等。
- 几何顶点着色器阶段：该阶段将顶点数据转换为可用于图形渲染的数据类型，并对顶点进行变换、变形、裁剪等操作。
- 图元装配阶段：该阶段将原始几何形状（如点、线、三角形等）转换为图形原语，如多边形等。
- 光栅化阶段：该阶段将图形原语转换为一系列像素，用于显示在屏幕上。
- 片元着色器阶段：该阶段对每个像素执行光照、纹理映射、混合等操作，生成最终的像素颜色值。
- 帧缓冲阶段：该阶段将处理后的像素输出到帧缓冲区中，最终在屏幕上显示出来。

在《Real TimeRendering》一书中，渲染管线被划分为以下四个阶段：应用程序阶段（Application）、几何处理阶段（Geometry Processing）、光栅化（Rasterization）和像素处理阶段（Pixel Processing）。其中，应用阶段通常是在CPU端进行处理，包括碰撞检测、动画物理模拟以及视椎体剔除等任务，这个阶段会将数据送到渲染管线中；几何处理阶段主要执行顶点着色器、投影变换、裁剪和屏幕映射的功能；光栅化阶段和我们上面讨论的差不多，都是将图元离散化片段的过程；像素处理阶段包括像素着色和混合的功能。我们可以发现，虽然管线的划分粒度不一样，但是每个阶段的具体功能其实是差不多的，原理也是一样的，并没有太大的差异。

### Context

在应用程序调用任何OpenGL API之前，首先需要创建⼀个OpenGL的上下文。它是一个状态机保存了OpenGL中的各种信息，同时也是调用OpenGL函数的前提。事实上，OpenGL函数是类似C语言⼀样的面向过程的函数，使用这些函数本质上都是对OpenGL上下文这个庞大的状态机中的某个状态或者对象进行操作。当然在对其进行操作之前，得先把这个对象设置为当前对象。

此外，由于OpenGL上下文是⼀个巨大的状态机，切换上下文往会产生较大的开销。但是不同的绘制模块，可能需要使用完全独立的状态管理。因此，可以在应用程序中分别创建多个不同的上下文，在不同线程中使用不同的上下文，上下文之间共享纹理、缓冲区等资源。这样的方案，会比反复切换上下文，或者大量修改渲染状态，更加合理高效的。

## First OpenGL Program

### GLFW

正如前面所提到的，在编写GLSL之前，首先要做的就是创建一个OpenGL上下文和一个用于显示的窗口。这时，就需要使用GLFW。

GLFW是一个专门针对OpenGL的C语言库，提供了一组用于创建窗口、处理输入事件以及管理OpenGL上下文的函数。在它的[官网](https://www.glfw.org/download.html)上，可以直接下载Visual Studio（2012到2022都有）预编译好的二进制版本和相应的头文件，添加到项目配置中就可以直接使用了。

### Several Steps

**分配顶点缓存对象，传入顶点数据**：

`顶点缓存对象(Vertex Buffer Object VBO)`是OpenGL中向GPU传入顶点数据所依靠的对象。我们一般使用`glGenBuffers(GLsizei n, GLuint *buffers);`和`glBindBuffer(GLenum target, GLuint buffer);`函数创建和绑定该VBO对象。

``` c++
GLuint VBO;
// 创建一个gpu缓存对象
glGenBuffers(1, &VBO);  
GLuint VBOs[3];
// 创建多个gpu缓存对象
glGenBuffers(3, VBOs);  
// 绑定，即定义该缓存对象用途（由于时将顶点数据保存到缓存，因此使用GL_ARRAY_BUFFER类型）
// 其他的类型还包括：纹理缓存GL_TEXTURE_BUFFER、顶点索引缓存GL_ELEMENT_ARRAY_BUFFER等
glBindBuffer(GL_ARRAY_BUFFER, VBO)

// 写入数据/执行渲染操作等
// ...

// 解绑缓存对象
glBindBuffer(GL_ARRAY_BUFFER, 0);
// 删除缓存对象
glDeleteBuffers(1, &VBO);
// 删除多个缓存对象
glDeleteBuffers(3, VBOs);
```

值得注意的是，绑定缓存对象时，需要指定它所对应的类型。比如上述代码中的GL_ARRAY_BUFFER类型，就是用于存储顶点数据的。总体而言，常见的缓存类型包括：

- GL_ARRAY_BUFFER：顶点属性数据（如位置、颜色、法线）；
- GL_ELEMENT_ARRAY_BUFFER：索引数据（用于 EBO）；
- GL_UNIFORM_BUFFER：统一块数据（用于 UBO）；
- GL_TEXTURE_BUFFER：纹理缓冲（较少使用）。

创建并绑定好VBO对象之后，我们就可以使用`glBufferData(GLenum target, GLuint size, GLuint* buffer, GLenum type)`向指定的VBO对象传入顶点数据。其中，第四个参数指定了我们希望显卡如何管理给定的数据。它有三种形式：

- GL_STATIC_DRAW ：数据不会或几乎不会改变。
- GL_DYNAMIC_DRAW：数据会被改变很多。
- GL_STREAM_DRAW ：数据每次绘制时都会改变。

``` c++
// 传入顶点数据
GLfloat vertices[12] = {
    1.0f, 1.0f,  0.0f, -1.0f, 1.0f,  0.0f,
    1.0f, -1.0f, 0.0f, -1.0f, -1.0f, 0.0f,
};
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

具体来说，当调用`glBufferData`函数时，驱动程序实际上执行了两个沉重的步骤：

- 销毁与重建（Orphan Reallocation）：它会先释放（或标记为废弃）该缓冲对象之前的内存，然后根据你传入的大小，向显存重新申请一块新的连续空间。
- 数据拷贝：将CPU内存中的数据通过PCIe总线拉到GPU显存中。

可见，频繁的内存申请和释放（Alloc/Free）会造成显存碎片，并强制让CPU等待GPU清空之前的渲染指令，导致严重的流水线阻塞（Pipeline Stall）。

**使用顶点数组对象管理顶点缓存对象**：

在OpenGL的可编程渲染管线中，顶点着色器（Vertex Shader）负责处理每个顶点的数据，而片元着色器（Fragment Shader）负责计算每个像素的最终颜色。两者通过关键字`in`和`out`分别定义输入输出变量，并进行数据传递。

其中，顶点着色器的输入称为顶点属性（Vertex Attributes），由CPU通过顶点缓冲（VBO）提供，一般包括顶点位置、颜色、纹理坐标等，可以使用小写字母a开头表示；而其的输出则会在经过图元装配和光栅化后，作为片元着色器的输入传入。这类变量传统上被称为插值变量（Varying Variables），因而通常以小写字母v开头命名，以表明其为经插值传递的片段输入。

例如，在以下着色器代码中：

```glsl
// 顶点着色器
#version 330 core
layout(location = 0) in vec3 aVertexPos;  // 期望从 attribute 0 读取3个float
out vec3 vFragPos;

void main() {
    vFragPos = aVertexPos;
    gl_Position = vec4(aVertexPos, 1.0);
}
```

```glsl
// 片元着色器
layout(location = 0) in vec3 vFragPos;

void main() {
    gl_FragColor = vec4(vFragPos, 1.0);
}
```

为了让GPU正确理解VBO中的数据，OpenGL引入了顶点数组对象（Vertex Array Object, VAO）来描述VBO数据的布局与含义。

具体而言，VAO本身并不存储顶点数据，而是记录了顶点属性的配置状态。它能够记录以下操作：

- `glBindBuffer`函数所定义的缓存关联，包括GL_ARRAY_BUFFER、GL_ELEMENT_ARRAY_BUFFER等类型，也即具体使用哪个VBO和EBO。
- `glVertexAttribPointer`函数所定义的VBO布局（步长、偏移、格式）。
- `glEnableVertexAttribArray`函数所定义的属性的开启状态。

因此，一旦录制完成，正式渲染时，仅需绑定VAO，即可瞬间恢复复杂的顶点输入环境，极大减少了CPU向GPU发送状态指令的开销。

``` c++
// 创建VAO
GLuint VAO;
glGenVertexArrays(1, &VAO);  
// 绑定VAO
glBindVertexArray(VAO);

// 指定VBO或EBO等缓存对象
// ...

// 定义顶点属性0的大小、偏移量和步长
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
// 启用顶点属性0
glEnableVertexAttribArray(0);

// 解绑VAO
glBindVertexArray(0);
```

其中，`glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const void * pointer)`函数的参数含义如下：

- index：指定要配置的顶点属性索引，须与着色器中 layout(location = n) 的n一致。
- size：每个顶点属性的分量个数（如 vec3 对应 3）。
- type：数据类型（如 GL_FLOAT）。
- normalized：是否希望非浮点型数据被归一化到[0, 1]或[-1, 1]范围。
- stride：连续两个顶点属性组之间的步长，单位为字节。若数据紧密排列，可设为0让OpenGL自动计算。
- pointer：该属性在缓冲区中第一个组件相对于起始位置的字节偏移量。

此外，值得注意的是，片元着色器会自动将顶点着色器的输出作为其输入使用，而无需在代码中显式绑定或传递。这是因为OpenGL的着色器接口匹配机制会根据变量的名称和类型，自动将顶点着色器中声明为out的变量与片元着色器中同名且类型兼容的in变量进行连接。

**使用索引缓存对象减少顶点冗余，进而提高绘制效率**：

当多个图元共享相同顶点时（如两个相邻三角形共用一条边），直接重复上传顶点会造成内存浪费。`索引缓冲对象(Element Buffer Object, EBO)`通过存储顶点索引序列解决此问题。
EBO的创建与VBO类似，但绑定目标为GL_ELEMENT_ARRAY_BUFFER：

```c++
GLuint EBO;
// 创建EBO
glGenBuffers(1, &EBO);  
// 绑定EBO
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO)
```

不过，值得注意的是，EBO必须在VAO绑定时绑定！因为GL_ELEMENT_ARRAY_BUFFER的绑定状态是VAO的一部分。例如：

```c++
glBindVertexArray(VAO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glVertexAttribPointer(...);
glEnableVertexAttribArray(...);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO); // ← EBO 绑定也记录在 VAO 中
glBindVertexArray(0);
```

此外，与VBO类似，EBO数据写入也使用`glBufferData(GLenum target, GLuint size, GLuint* buffer, GLenum type)`函数。

**使用统一缓存对象替代普通Uniform变量传入相机矩阵以及光源参数**：

在OpenGL中，统一变量（Uniform）是一种在着色器之间共享数据的方式。它类似全局变量，但是只能在着色器中读取，不能在着色器中写入。常见的可能的通用变量包括：

- Camera信息（MVP矩阵）：ModelViewProjection矩阵，用于将顶点坐标转换为剪辑坐标。
- Light信息：包括光源位置、颜色、强度等。

当你有很多个着色器（比如 PBR、阴影、后期处理）都需要用到同一组数据（如 View 矩阵、Projection 矩阵、Light 信息）时，普通 Uniform 就会变得极其低效。此时，我们可以使用`统一缓冲对象(Uniform Buffer Object, UBO)`来替代普通的同一变量。 UBO 的做法是：在显存中开辟一块公共缓冲区，所有着色器直接去这块内存里“读”。

```glsl
// 所有着色器共享的 CameraBlock
layout(std140, binding = 0) uniform CameraBlock {
    mat4 view;
    mat4 projection;
    vec3 cameraPos;
};

layout(std140, binding = 1) uniform LightBlock {
    vec3 lightPos;
    vec3 lightColor;
    float intensity;
};
```

```c++
// 创建 UBO
GLuint cameraUBO, lightUBO;
glGenBuffers(1, &cameraUBO);
glGenBuffers(1, &lightUBO);

// 分配显存（std140 布局需注意对齐）
glBindBuffer(GL_UNIFORM_BUFFER, cameraUBO);
glBufferData(GL_UNIFORM_BUFFER, sizeof(CameraData), nullptr, GL_DYNAMIC_DRAW);
glBindBufferBase(GL_UNIFORM_BUFFER, 0, cameraUBO); // 绑定到绑定点 0

glBindBuffer(GL_UNIFORM_BUFFER, lightUBO);
glBufferData(GL_UNIFORM_BUFFER, sizeof(LightData), nullptr, GL_DYNAMIC_DRAW);
glBindBufferBase(GL_UNIFORM_BUFFER, 1, lightUBO); // 绑定到绑定点 1

// 每帧更新数据
glBindBuffer(GL_UNIFORM_BUFFER, cameraUBO);
glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(cameraMatrices), &cameraMatrices);
```

**管理着色器程序**：

OpenGL主要提供了两个函数`glLinkProgram(GLuint program)`和`glUseProgram(GLuint program)`来调用提前编译好的着色器程序。其中，一般在初始化时，调用`glLinkProgram(GLuint program)`将着色器程序链接；在实际渲染时，调用`glUseProgram(GLuint program)`。

**绘制**：

我们使用`glDrawArrays(GLenum mode, GLint first, GLsizei count)`或`glDrawElements(GLenum mode, GLsizei count, GLenum type, const void * indices)`绘制。前者根据顶点数组对象中定义的顶点属性指针，绘制顶点数据。后者根据顶点索引缓存对象中存储的索引数据，绘制顶点数据。

其中，模式和画法对应关系如下表：

| 枚举类型 | 显示模式 |
| --------- | -------- |
| GL_POINTS | 点 |
| GL_LINES | 线 |
| GL_LINE_STRIP | 条带线 |
| GL_LINE_LOOP | 循环线 |
| GL_TRIANGLES | 独立三角形 |
| GL_TRIANGLE_STRIP | 三角形条带 |
| GL_TRIANGLE_FAN | 三角形扇面 |

### Example Code

``` c++
// 1. 绑定顶点数组对象
glBindVertexArray(VAO);
// 2. 把我们的顶点数组复制到一个顶点缓冲中，供OpenGL使用
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 3. 复制我们的索引数组到一个索引缓冲中，供OpenGL使用
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
// 3. 设定顶点属性指针
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);
// 4. 解绑VAO
glBindVertexArray(0);

// 5. 创建并编译顶点着色器程序
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);

unsigned int shaderProgram;
shaderProgram = glCreateProgram();
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);

// . 绘图
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0)
glBindVertexArray(0);
```

最后，总结一下Vertex Array Object(VAO)和Vertex Buffer Object(VBO)的关系。在OpenGL中，VAO和VBO一起用于高效地在GPU上存储和管理顶点数据。其中，VAO是一个顶点数组的配置信息容器，它存储了如何解释和访问VBO中的顶点数据的规则。它包含了顶点属性指针(Vertex Attribute Pointers, VAPs)的设置，这些指针定义了顶点数据的布局、数据类型、步长等信息。而VBO是用于存储顶点数据的GPU内存对象。它可以包含顶点位置、颜色、纹理坐标、法线等信息。VBO将数据从CPU传输到GPU，使得GPU可以直接访问和处理这些数据，提高渲染性能。

因此，VAO和VBO一起工作时，VAO是配置信息，VBO是数据存储。VAO不直接存储顶点数据，而是存储了如何解释VBO中数据的规则。当VAO被绑定时，OpenGL知道如何从VBO中读取数据，并正确地渲染几何形状。

## GLSL Basics

### Data Type

**基础数据类型**：

- void
- bool
- int
- uint
- float

**纹理数据类型**：

- sampler1D
- sampler2D
- sampler3D

**聚合数据类型**：

- vec2/vec3/vec4
- mat2/mat3/mat4

### Qualifier

**变量存储限定符**：

- const：常量值必须在声明时初始化，它是只读的不可修改的。
- varying：顶点着色器的输出，主要负责在顶点着色器和片元着色器之间传递变量。
- uniform：一致变量。在着色器执行期间一致变量的值是不变的。与 const 常量不同的是，这个值在编译时期是未知的是由着色器外部初始化的，且一致变量在顶点着色器和片段着色器之间是共享的。
- attribute：表示只读的顶点数据，只用在顶点着色器中。

**函数参数限定符**：

- in：用在函数的参数中，表示该参数是输出参数，值是会改变的。
- out：用在函数的参数中，表示该参数是输出参数，值是会改变的。

### Internal Variable

- gl_Position：输出属性-变换后的顶点的位置，用于后面的固定的裁剪等操作。所有的顶点着色器都必须写这个值。

![GLSL语法基础](https://www.cnblogs.com/straywriter/articles/15889310.html)

## References

[learnopengl-cn](https://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/04%20Hello%20Triangle/)
