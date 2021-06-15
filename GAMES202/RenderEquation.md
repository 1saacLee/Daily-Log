## Rendering Equation
单独记录一下rendering equation（RE）的内容，因为后面很多东西都与他直接相关

### 辐射度量学
- Radiant Flux

- Radiance

- Irradiance 
### RE的形式：
只考虑反射，没有自发光的情况下，RE为：
$$
L_o(p, \omega_o) = \int_{\Omega+} L_i(p, \omega_i) f_r(p, \omega_i, \omega_o)V(p, \omega_i) d \omega_i,
$$
其中 $p$是世界空间中的某点，$\omega_o$是观察方向， $\omega_i$是入射方向，$f_r()$是p点处的BRDF值，$L_i$是p点处接收到的光照强度，$V$是可见性。围绕着整个半球对$\omega_i$进行积分，就能够获得该点向外出射的光照强度

- $f_r()$项的实际形式应该为：$f_r(p, \omega_i, \omega_o)cos(\theta_i)$，即有一个cos项控制权重
