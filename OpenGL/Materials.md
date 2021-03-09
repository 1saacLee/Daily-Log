## OpenGL中的材质
材质的内容相对较少，而且实现的方法千奇百怪，因此这篇笔记只做简单的介绍

### 材质
在计算机图像中，一切生成的图像的材质都反映在光照上（因为只能看，不能摸）。而正如上一篇光照笔记所说，Phong模型使用了环境光照、漫反射和镜面反射来构成光照，因此，一个模型的材质也可以由这三个量的参数来定义：

```
struct Material {
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
}; 

uniform Material material;
```
这里额外定义了一个`shiness`作为镜面反射的半径。
有这样的定义以后，不同的材质只需要设置他们对应material的这四个参数即可：

```
lightingShader.setVec3("material.ambient",  1.0f, 0.5f, 0.31f);
lightingShader.setVec3("material.diffuse",  1.0f, 0.5f, 0.31f);
lightingShader.setVec3("material.specular", 0.5f, 0.5f, 0.5f);
lightingShader.setFloat("material.shininess", 32.0f);
```

