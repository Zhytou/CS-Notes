# OpenGL Introduction

- [OpenGL Introduction](#opengl-introduction)
  - [First OpenGL Program](#first-opengl-program)
    - [Several Steps](#several-steps)
    - [Example Code](#example-code)
  - [Grammar Basis](#grammar-basis)
    - [Data Type](#data-type)
    - [Qualifier](#qualifier)
    - [Internal Variable \& Function](#internal-variable--function)
  - [Window Controlling](#window-controlling)
    - [Window Creation](#window-creation)
    - [Event Handling](#event-handling)
  - [Buffer Object](#buffer-object)
    - [VAO / VBO](#vao--vbo)
    - [UBO / SSBO](#ubo--ssbo)
    - [FBO](#fbo)
  - [Texture \& Sampler](#texture--sampler)
  - [References](#references)

OpenGL（Open Graphics Library） 在工程上常被误认为是一个单纯的图形库，但它本质上并非一套具体的软件代码，而是由 Khronos Group 制定并维护的跨平台图形渲染 API 规范（Specification）。OpenGL 规范严格定义了每一个接口的函数签名、状态机行为及预期输出，而其底层的具体实现（Implementation）则由显卡硬件厂商（如 NVIDIA、AMD、Intel）在其驱动程序中提供。因此，更新显卡驱动本质上就是更新显卡厂商对 OpenGL 规范的最新硬件级实现；当遇到图形渲染的硬件兼容性 Bug 时，也常通过升级驱动来解决。

为了兼顾现代 GPU 硬件架构的高效并行演进与老旧项目的历史包袱，OpenGL 从 3.2 版本开始在规范中明确划分了两套不同的上下文模式（Profiles）：

- Core Profile（核心模式）：删除了所有传统的固定管线 API（如 glBegin/glEnd 即时模式、内置矩阵变换及传统灯光系统），强制要求开发者采用现代的着色器（GLSL）、顶点缓冲区对象（VBO）与顶点数组对象（VAO）进行显示列表的显存化管理。这是现代图形渲染管线与跨平台开发的唯一标准模式。
- Compatibility Profile（兼容模式）：完整保留了早期 OpenGL 的所有固定管线函数与历史特性，允许开发者在同一个上下文中混合调用传统 API 与现代 GLSL 着色器。该模式主要用于维护工业级老旧代码库（如早期 CAD 或建模软件），在现代图形引擎与游戏开发中已被淘汰。

## First OpenGL Program

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

## Grammar Basis

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

**布局限定符**：

以 layout(...) 的形式出现，用于告诉 GPU 数据在内存中的排列方式、接口绑定位置或线程组大小，主要包括：

- std140：规范化的标准内存对齐规则。强制所有变量按照固定规则对齐（例如 vec3 强制按照 vec4 的 16 字节对齐）。它的作用是保证 CPU（C++ 结构体内存）和 GPU（Shader 内存）的内存布局完全一致，跨平台无缝传递。
- std430：(SSBO 专享) 比 std140 更紧凑的内存对齐规则（如 vec3 变为 12 字节对齐），能节省显存，但仅支持 SSBO。

- layout(location = 0)：指定顶点属性位置或 Fragment 输出缓存索引。
- layout(binding = 1)：指定 UBO/SSBO/Sampler 绑定的纹理/缓冲单元槽位。

### Internal Variable & Function

**顶点着色器的内置输入/输出变量**：

- gl_Position：输出属性-变换后的顶点的位置，用于后面的固定的裁剪等操作。所有的顶点着色器都必须写这个值。

**片元着色器的内置输入/输出变量**：

- gl_FragCoord (vec4)：当前片元在屏幕窗口空间中的相对坐标 $(x, y, z, 1/w)$。其中，gl_FragCoord.xy 为当前屏幕像素坐标（以像素为单位，如 $(1920, 1080)$）；gl_FragCoord.z 为映射到 $[0, 1]$ 深度缓冲区的深度值（$z_{win}$）。

- gl_FragDepth：手动覆盖片元的 Z 深度值（默认由硬件自动插值写入gl_FragCoord.z）。值得注意的是，在 Fragment Shader 中写入 gl_FragDepth 会破坏硬件的 Early-Z 深度测试优化，导致性能下降。
- gl_FragColor：输出属性-片元的颜色，早期用于直接输出片元颜色，3.3版本后核心模式废弃，改为统一采用用户自定义的out vec4 FragColor输出。

**内置函数**：

- min/max/clamp
- abs/sign
- pow/sqrt/exp/log
- sin/cos/tan
- dot/cross/normalize

除了常见的数学函数外，还有以下重要的函数：

- refract/reflect：折射/反射函数，用于计算折射/反射的向量。
- texture/textureLod：纹理采样函数，用于采样纹理。
- dFdx/dFdy：计算纹理坐标在x轴和y轴方向的导数。

## Window Controlling

在 OpenGL 开发中，OpenGL 本身是一个纯粹的跨平台图形渲染 API，它并不包含任何创建窗口、处理系统消息或接收键盘鼠标输入的代码。

因此，使用 OpenGL 开发应用时，要么就需要直接调用操作系统底层接口（如 Windows HWND/Win32 API、X11、Wayland、AppKit），要么就使用第三方跨平台库（如 GLFW、GLUT、SDL、SFML）来实现。

以 GLFW 为例, 使用它开发基于 OpenGL 的窗口应用时，主要需要完成以下几个步骤：

- 初始化 GLFW
- 创建窗口
- 配置窗口属性
- 设置回调函数
- 进入主循环
- 渲染场景
- 清理资源

### Window Creation

### Event Handling

## Buffer Object

### VAO / VBO

### UBO / SSBO

### FBO

## Texture & Sampler

```c++
struct SamplerDesc {
    // Texture Filtering
    // | Macro                     | Behavior                  | Available For       | Typical Usage                     |
    // |---------------------------|---------------------------|---------------------|-----------------------------------|
    // | GL_NEAREST                | Nearest pixel (no blend)  | Min / Mag           | Shadow map, pixel art, UI         |
    // | GL_LINEAR                 | Bilinear blend (smooth)   | Min / Mag           | Common PBR texture, general image |
    // | GL_NEAREST_MIPMAP_NEAREST | Nearest + nearest mip     | Min only            | Fast low-quality mipmapping       |
    // | GL_LINEAR_MIPMAP_NEAREST  | Linear + nearest mip      | Min only            | Balanced performance & quality    |
    // | GL_NEAREST_MIPMAP_LINEAR  | Nearest + linear mip blend | Min only           | Smooth mip transition             |
    // | GL_LINEAR_MIPMAP_LINEAR   | Trilinear filtering       | Min only            | Highest quality, terrain/landscape|
    // Note: magFilter only supports GL_NEAREST / GL_LINEAR (no mipmap variants)
    GLenum minFilter = GL_LINEAR_MIPMAP_LINEAR; // minification filter (texture scaled down)
    GLenum magFilter = GL_LINEAR;               // magnification filter (texture scaled up), NO mipmap modes allowed

    // Texture Wrapping (UV addressing mode)
    // | Macro               | Behavior when UV out of [0, 1] | Extra Config Required | Typical Usage                     |
    // |---------------------|--------------------------------|-----------------------|-----------------------------------|
    // | GL_REPEAT           | Tile texture repeatedly        | No                    | Floor, wall, seamless tile texture|
    // | GL_MIRRORED_REPEAT  | Mirror then tile texture       | No                    | Natural texture, reduce tile seam |
    // | GL_CLAMP_TO_EDGE    | Extend texture edge pixels     | No                    | Model texture, UI, skybox         |
    // | GL_CLAMP_TO_BORDER  | Use custom border color        | Yes (borderColor)     | Shadow depth map, special effect  |
    GLenum wrapS = GL_REPEAT; // wrap mode for S coordinate (U / horizontal axis)
    GLenum wrapT = GL_REPEAT; // wrap mode for T coordinate (V / vertical axis)
    GLenum wrapR = GL_REPEAT; // wrap mode for R coordinate (3D texture / cubemap only)

    // Border Color
    float borderColor[4] = { 0.0f, 0.0f, 0.0f, 1.0f }; // border color for GL_CLAMP_TO_BORDER mode

    // Level of Detail (LOD)
    GLfloat lodBias = 0.0f; // LOD bias for mipmap sampling
    GLfloat minLod  = -1000.0f;
    GLfloat maxLod  = 1000.0f;

    /// Depth texture
    GLenum compareMode = GL_NONE; // depth compare mode (for shadow depth texture)
    GLenum compareFunc = GL_LESS; // depth compare function (valid when compareMode != GL_NONE)
};
```

## References

- [learnopengl-cn](https://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/04%20Hello%20Triangle/)
- [glsl grammar basis](https://www.cnblogs.com/straywriter/articles/15889310.html)
