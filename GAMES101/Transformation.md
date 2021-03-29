## 图形学中的变换
- 这篇是GAMES101的基础内容，也是入门内容，主要回顾了线性代数的内容，介绍了平移、缩放、旋转用矩阵的表示形式，一些基础内容可能会跳过

- 变换的核心思想是构建三维模型描述物体，并将其映射至二维空间（屏幕）中 

##点和向量的表示
假设三维空间中一个点，那么它可以表示为 
$$
point = \left[ \begin{matrix} x \\ y \\ z \\ 1  \end{matrix} \right]
$$
这里的第四维可以将平移操作统一至线性变换中，同时也可以区分点和向量（点的第四维是1，而向量的第四维是0）即如果有一个向量形如：
$$
vec = \left[ \begin{matrix} x \\ y \\ z \\ 0  \end{matrix} \right]
$$
那么$vec$就表示了一个从原点出发，指向（x,y,z）的向量
### 缩放
缩放可以表示成：

$$
\left[ \begin{matrix} x' \\ y' \\ z' \\ 1  \end{matrix} \right] = \left[ \begin{matrix} S_x & 0 & 0 & 0 \\ 0 & S_y & 0 & 0 \\ 0&0&S_z &0 \\ 0&0&0&1  \end{matrix} \right]* \left[ \begin{matrix} x \\ y \\ z \\ 1  \end{matrix} \right]
$$

其中$S_x, S_y,S_z$是各自维度上的缩放比例，该式很好理解，只要展开成矩阵乘法就不难看出。

### 旋转

旋转以矩阵形式表示为：
$$
\left[ \begin{matrix} x' \\ y' \\ z' \\ 1  \end{matrix} \right] = R_x(\theta)* \left[ \begin{matrix} x \\ y \\ z \\ 1  \end{matrix} \right]
$$


$$
R_x(\theta) = \left[ \begin{matrix} 1 & 0 & 0 & 0 \\ 0 & \cos\theta & -\sin\theta &0 \\ 0 & \sin\theta & \cos\theta & 0 \\ 0&0&0&1  \end{matrix} \right]
$$


$$
R_y(\theta) = \left[ \begin{matrix} \cos\theta & 0 & \sin\theta & 0 \\ 0 & 1 & 0 & 0 \\ -\sin\theta & 0 & \cos\theta &0 \\ 0&0&0&1  \end{matrix} \right]
$$

$$
R_z(\theta) = \left[ \begin{matrix} \cos\theta & -\sin\theta & 0 & 0 \\ \sin\theta & \cos\theta & 0 & 0 \\ 0& 0 & 1 &0 \\ 0&0&0&1  \end{matrix} \right]
$$


注意旋转分为沿某一轴旋转，这里只考虑了沿着xyz轴旋转，如果是普通旋转，可以将旋转分解，然而，因为复合分解时并不会移动坐标新，因此这里会出现一个万向节死锁的问题，例如沿着Z轴旋转90°，就会导致原先的X轴与现在的Y轴重合，从而使得旋转X与旋转Y起到了相同的作用。(一个生动的例子是脸朝下转动脑袋与肩膀，以及脸朝前转动脑袋与肩膀，可以发现脸朝前的时候失去了一个方向的转动能力)

想要具体解决这个问题需要引入四元数。略

### 平移


$$
T_{t_x, t_y, t_z} = \left[ \begin{matrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0& 0 & 1 &0 \\ t_x& t_y&t_z&1  \end{matrix} \right]
$$


### 一般的仿射变换

一般的变换可以以矩阵的形式表示为：
$$
\left[ \begin{matrix} a & b & c & t_x \\ d & e & f & t_y \\ g& h & i &t_z \\ 0& 0&0&1  \end{matrix} \right]
$$



