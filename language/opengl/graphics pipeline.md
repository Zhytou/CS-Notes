# Graphics Pipeline

- [Graphics Pipeline](#graphics-pipeline)
  - [Introduction](#introduction)
  - [Vertex Shader](#vertex-shader)
    - [Vertex Data](#vertex-data)

## Introduction

`着色器Shader`是OpenGL中一个最重要的概念，它是图形硬件设备所执行的一类特殊函数。简单来说，可以将其理解成一种GPU能够编译和执行的一种小型程序。OpenGL中最常用的着色器包括`顶点着色器Vertex Shader`和`片元着色器Frame Shader`。而`OpenGL Shading Language GLSL`则是一款用于编写着色器的编程语言。

以下是两个最简单的着色器程序：

``` GLSL
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

```glsl
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
} 
```

## Vertex Shader

### Vertex Data

顶点着色器的输入可以包括括顶点坐标、顶点颜色、顶点法线、纹理坐标等数据，利用这些输入数据我们可以在片段着色器计算片段的光照信息，最终输出到颜色缓冲器。
