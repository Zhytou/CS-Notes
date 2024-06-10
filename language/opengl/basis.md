# OpenGL Introduction

- [OpenGL Introduction](#opengl-introduction)
  - [Introduction](#introduction)
  - [First OpenGL Program](#first-opengl-program)
    - [Several Steps](#several-steps)
    - [Example Code](#example-code)
  - [References](#references)

## Introduction

OpenGL(Open Graphics Library)是个定义了一个跨编程语言、跨平台的编程接口规格的专业的图形程序接口。它用于三维图像（二维的亦可），是一个功能强大，调用方便的底层图形库。

**OpenCV & OpenCL & OpenMP**：

OpenCV(Open Source Computer Vision Library)是一个开源发行的跨平台计算机视觉库，可以运行在Linux、Windows和Mac OS操作系统上。它轻量级而且高效——由一系列 C 函数和少量 C++ 类构成，同时提供了Python、Ruby、MATLAB等语言的接口，实现了图像处理和计算机视觉方面的很多通用算法。

它由C++语言编写，它的主要接口也是C++语言，但是依然保留了大量的C语言接口。该库也有大量的Python, Java and MATLAB/OCTAVE (版本2.5)的接口。这些语言的API接口函数可以通过在线文档获得。如今也提供对于C#,Ch, Ruby的支持。

OpenCL(Open Computing Language)是第一个面向异构系统通用目的并行编程的开放式、免费标准，也是一个统一的编程环境，便于软件开发人员为高性能计算服务器、桌面计算系统、手持设备编写高效轻便的代码，而且广泛适用于多核心处理器(CPU)、图形处理器(GPU)、Cell类型架构以及数字信号处理器(DSP)等其他并行处理器，在游戏、娱乐、科研、医疗等各种领域都有广阔的发展前景。

OpenMP(Open Multi-Processing)是用于共享内存并行系统的多处理器程序设计的一套指导性的编译处理方案(Compiler Directive)。OpenMP支持的编程语言包括C语言、C++和Fortran；而支持OpenMP的编译器包括Sun Compiler，GNU Compiler和Intel Compiler等。OpenMP提供了对并行算法的高层的抽象描述，程序员通过在源代码中加入专用的pragma来指明自己的意图，由此编译器可以自动将程序进行并行化，并在必要之处加入同步互斥以及通信。当选择忽略这些pragma，或者编译器不支持OpenMP时，程序又可退化为通常的程序(一般为串行)，代码仍然可以正常运作，只是不能利用多线程来加速程序执行。

OpenMP和OpenCL都是用于高性能计算机，但是关键点不一样，前者主要是基于CPU的并行，后者主攻是异构系统中GPU并行。

**OpenGL Pipeline**:

- 应用程序阶段：该阶段由应用程序编写的代码组成，主要负责设定场景和对象的各种属性以及交互逻辑等。
- 几何顶点着色器阶段：该阶段将顶点数据转换为可用于图形渲染的数据类型，并对顶点进行变换、变形、裁剪等操作。
- 图元装配阶段：该阶段将原始几何形状（如点、线、三角形等）转换为图形原语，如多边形等。
- 光栅化阶段：该阶段将图形原语转换为一系列像素，用于显示在屏幕上。
- 片元着色器阶段：该阶段对每个像素执行光照、纹理映射、混合等操作，生成最终的像素颜色值。
- 帧缓冲阶段：该阶段将处理后的像素输出到帧缓冲区中，最终在屏幕上显示出来。

## First OpenGL Program

### Several Steps

分配顶点缓存对象，传入顶点数据：

`顶点缓存对象Vertex Buffer Object VBO`是OpenGL中向GPU传入顶点数据的对象。我们一般使用`glGenBuffers(GLsizei n, GLuint *buffers);`和`glBindBuffer(GLenum target, GLuint buffer);`函数创建和绑定该VBO对象。其中，绑定一个缓存时，需要指定它所对应的类型。缓存类型有很多种，它们用于实现不同OpenGL功能。

``` c++
GLuint VBO;
// 创建vbo对象（1是序号，第1个vbo）
glGenBuffers(1, &VBO);  
// 绑定（由于时将顶点保存到缓存，因此使用GL_ARRAY_BUFFER类型）
glBindBuffer(GL_ARRAY_BUFFER, VBO)
```

创建并绑定好VBO对象之后，我们就可以使用`glBufferData(GLenum target, GLuint size, GLuint* buffer, GLenum type)`当前绑定的VBO对象传入顶点数据。其中，第四个参数指定了我们希望显卡如何管理给定的数据。它有三种形式：

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

使用顶点数组对象管理顶点缓存对象：

我们需要使用`顶点数组对象(Vertex Array Object, VAO)`来管理VBO并解释VBO中顶点数据的含义。其创建和绑定和VBO类似。至于，向OpenGL解释传入顶点数据的含义，具体可以使用`glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const void * pointer)`指明传入数据的含义。

例如：

``` c++
// 创建VAO
GLuint VAO;
glGenVertexArrays(1, &VAO);  
// 绑定VAO
glBindVertexArray(VAO);
// 定义顶点数据含义
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);
// 解绑VAO
glBindVertexArray(0);
```

编译链接着色器程序：

OpenGL主要提供了两个函数`glLinkProgram(GLuint program)`和`glUseProgram(GLuint program)`。其中，一般在初始化时，调用`glLinkProgram(GLuint program)`将着色器程序链接；在实际渲染时，调用`glUseProgram(GLuint program)`。

绘制：

我们使用`glDrawArrays(GLenum mode, GLint first, GLsizei count)`绘制。其中，模式和画法对应关系如下表：

|  枚举类型  | 显示模式 |
| --------- | -------- |
| GL_POINTS | 点 |
| GL_LINES  | 线 |
| GL_LINE_STRIP | 条带线 |
| GL_LINE_LOOP | 循环线 |
| GL_TRIANGLES | 独立三角形|
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

// 5. 绘图
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0)
glBindVertexArray(0);
```

## References

[learnopengl-cn](https://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/04%20Hello%20Triangle/)
