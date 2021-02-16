## OpenGL环境配置

### Requirement
- OpenGL >= 4.6
- Visual Studio 2019
- CMake 3.18.1
- GLFW 3.3.2
- [GLAD](https://glad.dav1d.de/)

### 构建GLFW

GLFW是一个可以构建OpenGL上下文，处理键盘等事件的库

1. 下载[源码压缩包](http://www.glfw.org/download.html)并解压，在其中新建`build`文件夹，打开CMake，分别选择`source code` 以及`build`路径为`/glfw-3.3.2` 以及`/glfw-3.3.2/build`

1. 直接configure, 保持默认配置，并在generate时选择目标为visual studio 2019.

1. 打开**GLFW.sln**并生成，在 **/src/Debug** 中找到`glfw3.lib`，并将其复制到 **/custom/lib**中。将GLFW的**include**也拷贝到 **/custom/include**中

### 构建GLAD

由于OpenGL中的函数都是由显卡生产商实现的，因此很多函数的位置都不同，GLAD帮助我们提供该位置，从而保证了函数的实现是相对透明的

1. 在[GLAD配置界面](http://glad.dav1d.de/)中选择自己的OpenGL版本，其余配置保持默认，点击`Generate`，将两个头文件目录**glad**和**KHR**复制到 **/custom/include**中，另一个文件`glad.c`会在后续加入到新创建的工程中

### 构建Visual Studio 项目

本文的实现均基于Visual Studio 2019，VSCode + 编译链接的形式应该也是可行的（在GAMES101中是这种方式）

1. 新建步骤略
1. 项目属性中需要修改链接器以及Include目录
    - include路径需要增加 **/custom/include/**
    - lib路径需要增加 **/custom/lib**
    - 附加依赖项需要添加 `opengl32.lib` 以及 `glfw3.lib`
    - `glad.c`需要加入到项目文件中
1. 额外的include路径
```
#include <glad/glad.h>
#include <GLFW/glfw3.h>
```


