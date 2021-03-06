## OpenGL中的Texture

Texture有助于表现物体的真实性，同时与光照、材质等都有很大的联系，这里主要介绍纹理贴图的内容

### 纹理的概念
纹理是物体的表面颜色的组合，现实世界中的物体都是有着复杂、渐变的颜色的，而仅仅用之前的知识，在一个图案中，只能表示出非常简单的一些颜色。而想要表示更复杂的情况，最方便的做法就是像贴墙纸一样，在物体的表面贴一层图片，以实现模拟的效果，当然，在这个过程中也会出现很多的问题，例如纹理的尺寸、分辨率、材质等等，都会对真实的表现造成影响

### 纹理的加载

- 第一步，我们首先生成一个纹理对象并绑定：
    ```
    unsigned int texture1;
    glGenTextures(1, &texture1);
    glBindTexture(GL_TEXTURE_2D, texture1);
    ```
    与之前的VAO等对象类似，texture1是句柄，`glGentextures`声明了生成的纹理，其中第一个参数是纹理的数量，之后，`glBindTexture`将texture1绑定到了`GL_TEXTURE_2D`上，这样之后对`GL_TEXTURE_2D`的操作实际上都是在对texture1进行操作

- 加载纹理的第二步是加载图片，这里利用了C++里的一个第三方库`stb_image.h`来实现。使用该库需要新建一个.cpp文件，并在其中加入如下代码段
    ```
    #define STB_IMAGE_IMPLEMENTATION
    #include "stb_image.h"
    ```


- 加载图片的过程如下所示
    ```
    unsigned char* data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
    if (data)
    {
        glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
        glGenerateMipmap(GL_TEXTURE_2D);
    }
    else
    {
        std::cout << "Failed to load texture" << std::endl;
    }
    ```
    这里主要用到了两个函数，`glTexImage2D`指定了被绑定的纹理、要绑定的数据（也就是图片的字节流）、数据的格式等等 **如果发现纹理绑定不起作用，在确定其他设置无误的情况下，可以尝试调节数据读取格式`GL_RGB`为`GL_RGBA`等，因为有的图片可能会有灰度通道，按照RGB格式无法解析**
    
    其次，`glGenerateMipmap`可以根据加载的纹理生成mipmap，mipmap是一种有效的解决视角远近导致的纹理粗糙问题（**关于这个问题的解释可以在[有关GAMES101的笔记](https://github.com/Lee-Ft/Daily-Log/GAMES101)中找到**）mipmap也可以手动生成，OpenGL的该接口同样可用。

- 最后，如果想要在shader中渲染纹理，还需要指定纹理与图形的坐标对应关系，例如在之前的正方形中，加入每个点对应的纹理坐标：
    ```
    float vertices[] = {
        -0.5f, -0.5f, -0.5f,  0.0f, 0.0f,
         0.5f, -0.5f, -0.5f,  1.0f, 0.0f,
         0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
         0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
        -0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
        -0.5f, -0.5f, -0.5f,  0.0f, 0.0f
    };
    ```
    每一行的前三个坐标是空间中点的xyz值，后两个坐标是纹理的xy坐标（因为这个例子里的纹理是2维的图片）

    下面的这张图片形象地解释了数据在内存中的存储方式：(虽然例子有一些不同)
    ![avatar](../pictures/OpenGL/vertex_attribute_pointer_interleaved_textures.png)

    同时我们还需要改变VAO：
    ```
    glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
    glEnableVertexAttribArray(2);
    ```

- 需要修改vertex shader和fragment shader，以让其解析纹理坐标。在vertex shader中，主要的工作是接收输入，并将其传给fragment shader，在fragment shader中，我们会定义一个sampler对象来接收纹理对象的输入并显示：
    - Vertex Shader的代码
        ```
        layout (location = 2) in vec2 aTexCoord;
        out vec2 TexCoord;
        void main()
        {
            gl_Position = vec4(aPos, 1.0);
            ourColor = aColor;
            TexCoord = aTexCoord;
        }
        ```
    - Fragment Shader的代码
        ```
        out vec4 FragColor;
        in vec3 ourColor;
        in vec2 TexCoord;

        uniform sampler2D ourTexture;

        void main()
        {
            FragColor = texture(ourTexture, TexCoord);
        }
        ```

    在fragment shader中，我们定义了一个uniform的变量sampler2D，它可以接收纹理的输入，而我们并没有在程序中使用`glUniform`为其赋值。个人理解他的作用更像是一个句柄，指示了激活纹理单元的位置，在这个例子中，我们只使用了一个纹理，而该变量的默认值是0，因此无需额外设置；如果一个对象对应多个纹理的话，那就需要指定sampler2D的值：
    ```
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, texture1);

    glActiveTexture(GL_TEXTURE1);
    glBindTexture(GL_TEXTURE_2D, texture2);
    
    glUniform1i(glGetUniformLocation(shaderProgram, "texture1"), 0);

    glUniform1i(glGetUniformLocation(shaderProgram, "texture2"), 1);
    ```
    `glActiveTexture`激活了特定的纹理单元，而在其之后的`glBindTexture`就将纹理绑定到了当前激活的纹理单元上，
    后面的`glUniform1i`方法指定了sampler对应的纹理单元

- 总的来说，加载并显示纹理分为以下几步：声明纹理对象，加载图片到纹理对象中，指定纹理对象与被激活纹理单元的对应关系，指定sampler与纹理单元的对应关系



