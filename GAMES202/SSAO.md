## SSAO

- 假设1：间接光照是一个常数（类似于Phong模型的diffuse）
- 假设2：每个物体由于可见性，不可能都能接收到相同程度的光（这是与Phong模型不同的地方）
- 假设3：物体是diffuse的

- 思路：对每个物体，向四面八方看去（随机采样），如果有很多光线被遮蔽，那么这一点的可见性就差一些，因此应该更暗

- 考虑RE以及其近似：

$$
L_o(p, \omega_o) = \int_{\Omega+} L_i(p, \omega_i) f_r(p, \omega_i, \omega_o)V(p, \omega_i) d \omega_i
$$

$$
L_o(p, \omega_o) =\frac{\int_{\Omega_+}V(p,\omega_i)cos\theta_i d\omega_i} {\int_{\Omega+} cos\theta_i d \omega_i} \int_{\Omega+} L_i(p, \omega_i) f_r(p, \omega_i, \omega_o)cos\theta_i d \omega_i
$$

- 对于近似的前半部分记作$k_A$，半球上cos的积分是$\pi$,因此对于每一个fragment，前半部分的值就等于它采样可见性按照cos的加权平均值再除以$\pi$
- 对于近似的后半部分，由于假设1和3，$L_i$和$f_r$都是常数，因此后半部分的值等于$L_i * \frac {\rho}{\pi} * \pi  = L_i * \rho$

- 这个近似对SSAO是严格准确的


- 接下来的目标就是计算$k_A$, 在这里，我们对可见性的定义需要加以限制，即只要在fragment周围有限范围内没有被遮挡的光线，就认为它是可见的。因为如果不加以限制，最终每个光线都会在无限远的地方（墙壁，天空）被判定为遮挡