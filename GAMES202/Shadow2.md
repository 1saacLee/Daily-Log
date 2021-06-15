## 阴影

### 硬阴影

硬阴影的产生主要依靠Shadow Map实现，核心是two pass算法。two pass算法的流程可以简单描述为：将摄像机摆放在光源的位置（面光源也会简化为点光源），朝整个场景看去，利用深度测试记录各个像素上的最小深度，得到一张shadow map；之后再从真正的摄像机位置出发，以从摄像机角度看去的深度与shadowmap比较，如果某个的像素的深度大于shadowmap的值，那么意味着这个位置就应该被阴影覆盖，也就是这个像素的gl_Color应该为0

```
float useShadowMap(sampler2D shadowMap, vec4 shadowCoord){
  
  // return vec4(texture2D(shadowMap, vTextureCoord)).x;
  vec3 temp = shadowCoord.xyz;
  temp = temp * 0.5 + 0.5;
  vec4 depthpack = texture2D(shadowMap, temp.xy);
  float depthUnpack = unpack(depthpack);

  if(depthUnpack > temp.z - EPS)
      return 1.0;
  
  return 0.0;

}

```

硬阴影的问题有两个
- 第一是shadowmap的分辨率不可能是无限大，因此shadowmap上一个像素点在投影过后，可能在实际区域中是很大的一片。导致整个区域都是黑色的，而且像素化严重，这一点在长拖尾影子中表现非常明显；
- 第二个问题是，硬阴影的边界非常锐利，特别是较远处，这与现实生活不符，这很大程度上是因为现实生活中的光源不是点光源，因此较远处的影子实际上是很多个点光源互相生成阴影的叠加。
- 解决上述问题，有很多可行的方法，以作业中涉及到的PCF和PCSS为例说明。

### PCF
PCF的核心思想是利用随机采样+卷积，实现边界的柔化，这一点很好理解，因为柔化阴影边缘本质上就是对阴影边界做一个模糊，而因为阴影内部都是黑色的，因此全局模糊和局部模糊看起来也没区别。实现图像的模糊或者锐化，卷积是最常用的手段：

```
float PCF(sampler2D shadowMap, vec4 coords) {

  vec3 temp = coords.xyz;
  float res = 0.0;
  temp = temp * 0.5 + 0.5;
  for(int i = 0; i < NUM_SAMPLES; i++){
    vec3 random_coord = vec3(temp.x+ poissonDisk[i].x * KERNEL_SIZE, temp.y + poissonDisk[i].y * KERNEL_SIZE, temp.z);
    vec4 depthpack = texture2D(shadowMap, random_coord.xy);
    
    float depthUnpack = unpack(depthpack);

    if(depthUnpack > temp.z - EPS){
      res += 1.0;
    }
  }
  res = res / float(NUM_SAMPLES);
  return res;
}
```
这里的`poissonDisk`是一个随机数组，它可以保证我们在给定的`coords`周围采样`NUM_SAMPLES`次,`KERNEL_SIZE`用来控制卷积范围的大小，需要注意，这里最好能保证每个随机位置尽量落在不同的像素上，即`KERNEL_SIZE`最好是像素大小的整数倍

PCF解决了柔化边缘的问题，但它仍然无法完成阴影边缘的渐变模糊

### PCSS
首先思考，一个阴影之所以靠近的地方锐利，远离的地方模糊，本质上是因为

PCSS

