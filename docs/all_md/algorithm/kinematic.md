## 全向轮正解算

!!! warning "注意"
    若公式未渲染出来，请刷新页面重试。
    
    公式渲染所需时间可能较长，请耐心等待。
    
$$\frac{1}{s}
    \begin{bmatrix}
        -\frac{\sqrt{2}}{2} & \frac{\sqrt{2}}{2} & r \\
        -\frac{\sqrt{2}}{2} & -\frac{\sqrt{2}}{2} & r \\
        \frac{\sqrt{2}}{2}  & -\frac{\sqrt{2}}{2} & r \\
        \frac{\sqrt{2}}{2}  & \frac{\sqrt{2}}{2} & r \\
    \end{bmatrix} 
    \begin{bmatrix}
        v_x \\ v_y \\ \omega
    \end{bmatrix} = 
    \begin{bmatrix}
        \omega_0 \\ \omega_1 \\ \omega_2 \\ \omega_3    \end{bmatrix}
$$


## 麦轮

$$
\frac{1}{s}
\begin{bmatrix}
    -1 &  1 & a+b \\
    -1 & -1 & a+b \\
     1 & -1 & a+b \\
     1 &  1 & a+b \\
\end{bmatrix}
\begin{bmatrix}
    v_x \\ v_y \\ \omega
\end{bmatrix} = 
\begin{bmatrix}
    \omega_0 \\ \omega_1 \\
    \omega_2 \\ \omega_3
\end{bmatrix}
$$

## 代码

=== "C"

```c
VX =  (float)(DBUS->REMOTE.CH0_int16)* 2.0f;
VR = -(float)(DBUS->REMOTE.DIR_int16) ;
VY =  (float)((DBUS->REMOTE.CH1_int16) * 16.0f);
    
// 运动学解算 COMPONENT 是可调节系数
MOTOR[MOTOR_D_CHASSIS_1].DATA.AIM = ( VX - VY - VR * COMPONENT[0]) * COMPONENT[1];
MOTOR[MOTOR_D_CHASSIS_2].DATA.AIM = (-VX - VY - VR * COMPONENT[0]) * COMPONENT[1];
MOTOR[MOTOR_D_CHASSIS_3].DATA.AIM = (-VX + VY - VR * COMPONENT[0]) * COMPONENT[1];
MOTOR[MOTOR_D_CHASSIS_4].DATA.AIM = ( VX + VY - VR * COMPONENT[0]) * COMPONENT[1];

```