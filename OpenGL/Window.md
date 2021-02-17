## 绘制窗口和图案

### 创建窗口

创建窗口前需要先初始化GLFW：
```
glfwInit();
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
```
通过`glfwWindowHint`函数指定了OpenGL的版本号、模式等，之后创建窗口对象：
```
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
if (window == NULL)
{
    std::cout << "Failed to create GLFW window" << std::endl;
    glfwTerminate();
    return -1;
}
glfwMakeContextCurrent(window);
```

之后初始化GLAD：

```
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
    std::cout << "Failed to initialize GLAD" << std::endl;
    return -1;
}
```

此外还需要指定渲染的视口大小（这是OpenGL计算坐标的基准，而窗口大小是我们实际看到的尺寸，这两个如果不同，可能会出现黑框或者显示超出窗口的现象）
```
glViewPort(0,0, 800, 600);
```

最后在一个将窗口放在一个循环中以持续显示：
```
while(!glfwWindowShouldClose(window))
{
    glfwSwapBuffers(window);
    glfwPollEvents();    
}
```
`glfwSwapBuffers`函数会交换OpenGL的前后缓冲，这里涉及到OpenGL的双缓冲机制，后缓冲是绘制所有图案的位置，而前缓冲是实际的显示，双缓冲可以使得每帧的每个像素同时出现，而不是逐像素点绘制
`glfwPollEvents`函数会调用一些用户事件。


### VBO

绘制图案的顶点可以由很多属性组成，包括3D坐标，纹理、法线向量等，在OpenGL教程中，所有顶点的所有属性都存储在一个一维数组中，例如
```
// 每行为一个顶点，每列依次为该点的xyz坐标以及法向量的xyz坐标
float vertices[] = {
        -0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
         0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
         0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
         0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
        -0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
        -0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
```
绘制图案的第一步是将所有顶点批量传到GPU上（因为CPU与GPU的通信相对较慢），这里就用到VBO（Vertex Buffer Object）来存储以及发送顶点。例如：

```
unsigned int VBO; // 声明VBO的句柄

glGenBuffers(1, &VBO); // 以VBO为句柄创建缓冲对象

glBindBuffer(GL_ARRAY_BUFFER, VBO); // 将VBO绑定到GL_ARRAY_BUFFER缓冲上
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); // 设定缓冲的数据为vertices
```
可以看到创建一个VBO分为几步，首先是创建一个缓冲对象，但此时的缓冲尚未指定其类型，通过`glBindBuffer`函数，可以将该对象指定为特定的类型，VBO即为`GL_ARRAY_BUFFER`类型。最后，`glBufferData`将`vertices`的数据复制到`GL_ARRAY_BUFFER`中。

在循环中绘制的代码类似于：
```
while(true){
    glBindVertexArray(lightVAO);
    glDrawArrays(GL_TRIANGLES, 0, 3);
}

```

### VAO
有了顶点数组，下一步是对数组进行正确的切分解析以获得坐标等属性，VAO（Vertex Array Object）控制了我们如何去处理属性。例如：
```
unsigned int VAO; // 声明句柄

glGenVertexArrays(1, &VAO); // 创建VertexArrays对象
glBindVertexArray(VAO); // 绑定VAO
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0); // 设置Shader属性指针，将属性与Shader中的变量对应
glEnableVertexAttribArray(0); // 启用顶点属性
```

`glVertexAttribPointer`和`glEnableVertexAttribArray`是设置Shader所需的函数，可以理解为将当前绑定的VBO中的数据以某种格式解析并在Shader中启用，具体参数为：
```
0： 变量在Shader中的location
3：变量长度
GL_FLOAT：数据格式
GL_FALSE：是否正则化数据
3 * sizeof（float）：步长，也就是每隔多少字节取到下一个数据，通过这个设置可以跳过某些不需要的属性
(void*)0：数据起始位置的偏移量，设为0就是从头开始，可以跳过前面的某些顶点
```
个人认为这一部分的流程类似于声明式语句，因为VAO所解析的缓冲取决于当前的Buffer绑定到哪个VBO上（如果有多个VBO，需要在设置shader之前正确地将对应的VBO与buffer绑定）因此语句的顺序会对结果造成很大影响

之后在渲染循环中，绘制某物体时绑定对应的VAO：
```
while(true){
    glBindVertexArray(lightVAO);
    Draw...
}
```

VAO在绘制不同物体时非常有用，例如在后面出现的光照例子中，光源与箱子会共享一套坐标点，但是他们的颜色不同，而且光源不需要法向量等属性，这时候就可以创建两个VAO，分别设置不同的属性指针，并在循环中分别绑定两个VAO绘制即可。

### EBO
EBO的最大作用是引入顶点索引以减小存储公共顶点的开销，这在绘制多个相邻三角形的时候非常实用，例如一个立方体，实际只有8个顶点，但如果按照三角形的形式去画，需要3*2*6 = 36个顶点，EBO的创建方式为：
```
unsigned int EBO; // 生成EBO句柄
glGenBuffers(1, &EBO); // 创建缓冲对象
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO); // 绑定该对象到GL_ELEMENT_ARRAY_BUFFER上
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW); // 设置该缓冲区的数据
```

与VBO相比，EBO的区别在于它绑定的缓冲区不同，其余过程类似

在循环中使用EBO绘制的函数略有区别：
```
while(true){
    glBindVertexArray(lightVAO);
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
}
```
