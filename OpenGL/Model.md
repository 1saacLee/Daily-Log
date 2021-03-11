# TODO
- 目前对源码的理解尚且不深，因此先留一个尾巴
## OpenGL的模型


- 本篇笔记预计为OpenGL入门的最后一篇笔记，后续的内容，例如[OenGL官方教程](https://learnopengl-cn.github.io/)中的高级光照等，会在复习完GAMES101并学习完虚幻以后再补充吧.

- 模型是摆脱立方体盒子的最后一步，从加载模型开始，基本就难以一人独自完成一个项目了，因为绘制模型的工作量相当大


- 导入模型可以使用`assimp`库。在示例中，需要导入
```
#include <assimp/Importer.hpp>
#include <assimp/scene.h>
#include <assimp/postprocess.h>
```

