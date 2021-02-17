## OpenGL的Shader

Shader是绘制过程中非常重要的一个环节，大部分特效都可以在这个过程中实现，在Learn OpenGL教程中，主要实现的是Vertex Shader 与 Fragment Shader
- Vertex Shader的作用是勾勒所有顶点的特征，类似于绘画的草稿
- Fragment Shader实现了每个像素的绘制，计算每个像素点的颜色

定义Shader用到的语言是GLSL，与C语言类似
### Vertex Shader
一个典型的Vertex Shader例子为:
```
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoord;
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
out vec3 Normal;
out vec3 FragPos;
out vec2 texCoord;
void main()
{
   gl_Position = projection * view * model * vec4(aPos.x, aPos.y, aPos.z, 1.0);
   FragPos = vec3(model * vec4(aPos, 1.0));
   Normal = mat3(transpose(inverse(model))) * aNormal;  
   texCoord = aTexCoord;

};
```
- `layout(location = 0)` 设定了变量的位置，以方便在main函数中定义该变量

- `uniform` 可以理解为Shader的参数，可以通过`glUniform`系列函数设置

- `out` 可以理解为该函数的返回值，在后续的Shader中，只要将该变量设置为`in`且名字对上，就可以在Shader之间传递变量

- `gl_Position` 是一个约定的变量，既没有声明输入也没有声明输出，但该变量的值会作为顶点的归一化坐标，在这个例子中，MVP操作也是在这里实现的



### Fragment Shader
一个Fragment Shader的例子为：
```
#version 330 core
out vec4 FragColor;
in vec3 Normal;
in vec3 FragPos;
in vec2 texCoord;
uniform vec3 viewPos;
uniform vec3 lightPos;
//uniform vec3 objectColor;
uniform sampler2D texture_diffuse1;
uniform vec3 lightColor;

void main()
{
    float ambientStrength = 0.1;
    vec3 ambient = ambientStrength * lightColor;

    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(lightPos - FragPos);
    //vec3 result = ambient * objectColor;
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = diff * lightColor;

    float specularStrength = 0.5;
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);
    vec3 specular = specularStrength * spec * lightColor;

    vec3 result = (ambient + diffuse + specular);
    
    FragColor = vec4(result, 1.0);
}
```

- Fragment Shader也需要声明in和out，其中唯一一个输出`FragColor`就是该像素的颜色，这个例子中还实现了光照，所以有些复杂

### Shader的编译与链接

对于简单的shader实现，可能并不会单独分出一个文件来写各种shader，而是以字符串硬编码的形式在`main.cpp`中存在，在这种情况下，想要使用自定义的shader，在创建shader对象的前提下，还需要在程序运行时编译shader。

1. 创建shader类似于：
    ```
    unsigned int vertexShader; // vertexShader句柄
    vertexShader = glCreateShader(GL_VERTEX_SHADER); // 创建Shader，类型为VertexShader
    ```

1. 而编译的过程类似于：
    ```
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL); // 指定Shader的源码位置
    glCompileShader(vertexShader); // 编译Shader
    ```
    `glShaderSource`实现了硬编码与Shader的绑定，`glCompileShader`实现了在运行时编译该代码

3. 最后，需要构建一个完整的shader程序将两个shader连接起来：
    ```
    unsigned int shaderProgram; // shader程序的句柄
    shaderProgram = glCreateProgram(); // 创建shaderProgram
    glAttachShader(shaderProgram, vertexShader); // 在程序中添加VertexShader
    glAttachShader(shaderProgram, fragmentShader); // 添加FragmentShader
    glLinkProgram(shaderProgram); // 将所有shader链接
    ```

    这样在使用的时候，就可以通过在循环中调用`glUseProgram(shaderProgram);`来激活相应的shader，想要使用其他的shader，就再次调用`glUseProgram`。
