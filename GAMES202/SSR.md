## Screen Space Reflection (SSR)
- SSR的第一个假设是被反射的物体应该大部分在场景中
- 镜面反射SSR的大致流程：对每个fragment，按照观察方向计算镜面反射方向，计算从fragment出发到场景的交点，以反射到点的颜色作为光照颜色
- 高光的SSR需要在高光的方向上进行若干次采样，而不是仅在镜面反射方向。
- 在fragment的上半球随机采样，并在一个小范围内对光线求交，如果能够求到交点，那么就将交点处的BRDF值与fragment的BRDF相乘，即以交点作为光源，向fragment进行照射。
