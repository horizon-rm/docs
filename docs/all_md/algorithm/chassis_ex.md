
# RoboMaster 电控进阶培训：底盘与云台协同控制


## 1. 前言
在RM赛场上，机器人的灵活性至关重要。本教程将围绕 **“底盘跟随云台”** 、 **“小陀螺行进”** 展开。

学习目标：
1. 理解底盘跟随中的PID解耦控制。
2. 掌握小陀螺模式下的坐标系转换与速度合成。

---

## 2. 底盘跟随 (Chassis Follow)
**场景**：云台瞄准目标时，底盘可以独立运动，但始终保证车头（底盘x轴）指向云台朝向，或者保持相对位置。

### 2.1 原理
底盘跟随的核心是**角度闭环**。我们需要控制底盘旋转，使得底盘的朝向（通常由陀螺仪yaw或底盘电机编码器推算）与云台的朝向之间的误差为0。

设：

- \( \theta_{gimbal} \)：**云台当前朝向**（相对于世界坐标系/惯性系）。
  
- \( \theta_{chassis} \)：**底盘正方向**朝向。
  
- 误差 \( \theta_{Angle\_relative} = \theta_{gimbal} - \theta_{chassis} \)。

通过PID控制器（通常是位置式PID）计算出一个**角速度** \( \omega_{out} \)，叠加到底盘的运动学解算中。

### 2.2 实现逻辑

根据公式

$$
\begin{cases}
\text{PID}_{\text{error}} = \theta_{\text{gimbal}} - \theta_{\text{chassis}} \\
\text{PID}_{\text{aim}} = 0 \\
\omega_{out} = \text{PID}_{\text{out}}
\end{cases}
$$

=== "C"

```c
ANGLE_Relative = (float)MOTOR_V_GIMBAL[MOTOR_D_GIMBAL_YAW].DATA.ANGLE_NOW - (float)MOTOR_V_GIMBAL[MOTOR_D_GIMBAL_YAW].DATA.ANGLE_INIT;  // if add 4096
spinLittleRound(&ANGLE_Relative);   // 跟随最小距离

// 底盘跟随
if (DBUS->REMOTE.S2_u8 != 1)
{
    (!((DBUS->REMOTE.DIR_int16)||(DBUS->KEY_BOARD.SHIFT)))?(PRIDICT = DBUS->REMOTE.CH2_int16 * 3.0f,VR = PID_F_Cal(&FOLLOW_PID, 0, -ANGLE_Relative)):(PRIDICT = 0.0f);     // 分离 滚轮影响小陀螺
}
```


## 3. 小陀螺行进 (Spin Move)

### 3.1 场景与坐标系定义
小陀螺模式下，底盘以恒定角速度旋转，同时需要进行平移运动。此时，我们希望遥控器的推杆方向与 **云台的瞄准方向** 解耦，即无论车头朝向何处，推杆向前始终让机器人朝着云台指向的目标方向移动。

为了方便控制，我们做如下假设：

- **云台坐标系 = 世界坐标系**。这是因为在正常作战中，云台通过 PID 控制稳定指向目标，其 Yaw 轴相对于世界坐标系基本保持静止（或变化缓慢）。因此，我们可以将遥控器指令直接定义为 **世界坐标系下的期望速度** \((v_{x\_world}, v_{y\_world})\)。

- **底盘坐标系** 随车体旋转，其 Yaw 角 \(\theta_{chassis}\) 由 IMU 或电机编码器实时获取。

### 3.2 坐标转换原理
由于遥控器指令已经表达在世界坐标系下，我们只需要将世界系速度 **转换到底盘坐标系**，即可代入麦轮运动学公式计算各轮转速。

设：

- 世界系下的期望平移速度：\((v_{x\_world}, v_{y\_world})\)（由遥控器直接给定）

- 底盘当前朝向角：\(\theta_{chassis}\)（相对于世界系）

- \( \theta_{gimbal} \)：**云台当前朝向**（相对于世界坐标系/惯性系）

- 小陀螺旋转角速度：\(\omega_{spin}\)（开环给定）

则底盘坐标系下的期望速度 \((v_{x\_chassis}, v_{y\_chassis})\) 为：

\[
\begin{bmatrix} 
v_{x\_chassis} \\ 
v_{y\_chassis} 
\end{bmatrix} = 
\begin{bmatrix} 
\cos\theta_{Angle\_relative}& -\sin\theta_{Angle\_relative} \\ 
\sin\theta_{Angle\_relative} & \cos\theta_{Angle\_relative} 
\end{bmatrix}
\begin{bmatrix} 
v_{x\_world} \\ 
v_{y\_world} 
\end{bmatrix}
\]

此矩阵表示将世界系下的向量旋转 \(\theta_{Angle\_relative}\) 角度，得到其在底盘系下的坐标。

### 3.3 完整控制流程
1. 读取遥控器数据，直接作为世界系期望速度 \((v_{x\_world}, v_{y\_world})\)。
2. 读取底盘当前朝向 \(\theta_{chassis}\)（由 IMU 或编码器积分获得）。
3. 通过旋转矩阵将世界系速度转换到底盘系。
4. 叠加小陀螺旋转速度 \(\omega_{spin}\)（恒定值）。
5. 利用麦轮运动学公式计算四个电机的目标转速，并通过 CAN 总线发送。

### 3.4 代码实现

=== "C"

```c
ANGLE_Relative = (float)MOTOR_V_GIMBAL[MOTOR_D_GIMBAL_YAW].DATA.ANGLE_NOW - (float)MOTOR_V_GIMBAL[MOTOR_D_GIMBAL_YAW].DATA.ANGLE_INIT;  // if add 4096
spinLittleRound(&ANGLE_Relative);
ANGLE_Rad = ANGLE_Relative * MATH_D_RELATIVE_PARAM; // 单位转换成弧度

// rotate matrix
double COS = cos(ANGLE_Rad);
double SIN = sin(ANGLE_Rad);

ROTATE_VX = -VY * SIN + VX * COS;
ROTATE_VY =  VY * COS + VX * SIN;

// 运动学解算 COMPOENT是可调节系数
(DBUS->IS_OFF) ? (MOTOR[MOTOR_D_CHASSIS_1].DATA.AIM = 0) : (MOTOR[MOTOR_D_CHASSIS_1].DATA.AIM = ( ROTATE_VX - ROTATE_VY - VR * COMPONENT[0]) * COMPONENT[1] + PRIDICT);
(DBUS->IS_OFF) ? (MOTOR[MOTOR_D_CHASSIS_2].DATA.AIM = 0) : (MOTOR[MOTOR_D_CHASSIS_2].DATA.AIM = (-ROTATE_VX - ROTATE_VY - VR * COMPONENT[0]) * COMPONENT[1] + PRIDICT);
(DBUS->IS_OFF) ? (MOTOR[MOTOR_D_CHASSIS_3].DATA.AIM = 0) : (MOTOR[MOTOR_D_CHASSIS_3].DATA.AIM = (-ROTATE_VX + ROTATE_VY - VR * COMPONENT[0]) * COMPONENT[1] + PRIDICT);
(DBUS->IS_OFF) ? (MOTOR[MOTOR_D_CHASSIS_4].DATA.AIM = 0) : (MOTOR[MOTOR_D_CHASSIS_4].DATA.AIM = ( ROTATE_VX + ROTATE_VY - VR * COMPONENT[0]) * COMPONENT[1] + PRIDICT);
```

### 3.5 优点与注意事项
**优点**  

- 计算量小，只需一次旋转矩阵运算。  

- 控制逻辑清晰，遥控器指令直接对应世界系方向，便于操作手理解（推杆向前 = 朝场地前方移动，不受车头旋转影响）。

**注意事项**  

- 必须保证云台确实能稳定在世界坐标系下（即云台 PID 足够强，使云台 Yaw 角基本不变）。如果云台跟随目标移动，则世界系定义需要相应调整（例如以目标方向为参考）。  

- 底盘朝向角 \(\theta_{chassis}\) 的获取必须准确且平滑，建议使用 IMU 数据进行滤波，避免因角度跳变导致速度突变。  

- 小陀螺旋转速度 \(\omega_{spin}\) 的切换需平滑（如使用斜坡函数），防止电机电流过冲。

---