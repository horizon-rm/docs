# RoboMaster 电控进阶培训：旋转矩阵与欧拉角

## 1. 引言

在机器人控制中，描述姿态与坐标变换是基础中的基础。无论是云台控制、底盘运动学，还是 IMU 数据处理，都离不开旋转矩阵与欧拉角这两个核心工具。本教程将从二维旋转出发，逐步推导三维旋转矩阵，并详细介绍欧拉角的定义、旋转顺序、万向锁问题及其在 RoboMaster 工程实践中的应用。

---

## 2. 二维平面旋转

### 2.1 旋转矩阵推导

考虑二维平面中一个向量 \(\vec{v}\) 绕原点逆时针旋转角度 \(\theta\)，得到新向量 \(\vec{v}'\)。设 \(\vec{v}\) 在原始坐标系下的坐标为 \((x, y)\)，极坐标形式为：

\[
\begin{aligned}
x &= r \cos\alpha \\
y &= r \sin\alpha
\end{aligned}
\]

旋转后，角度变为 \(\alpha + \theta\)，因此：

\[
\begin{aligned}
x' &= r\cos(\alpha + \theta) = r(\cos\alpha\cos\theta - \sin\alpha\sin\theta) = x\cos\theta - y\sin\theta \\
y' &= r\sin(\alpha + \theta) = r(\sin\alpha\cos\theta + \cos\alpha\sin\theta) = x\sin\theta + y\cos\theta
\end{aligned}
\]

写成矩阵形式：

\[
\begin{bmatrix} x' \\ y' \end{bmatrix} = 
\begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}
\begin{bmatrix} x \\ y \end{bmatrix}
\]

称该矩阵为 **旋转矩阵** \(R(\theta)\)，它满足正交性 \(R^{-1}=R^T\) 且 \(\det(R)=1\)。

### 2.2 坐标系旋转 vs. 向量旋转

旋转矩阵可以有两种理解方式：
- **主动旋转**：将向量本身旋转 \(\theta\) 角（如上所述）。
- **被动旋转**：将坐标系旋转 \(-\theta\) 角，得到向量在新坐标系下的坐标。

两种表示互为转置（或逆）。在机器人控制中，通常采用 **被动旋转** 的观点：世界坐标系固定，我们描述机器人在世界坐标系下的朝向，或者将向量在不同坐标系之间转换。本教程后续均采用被动旋转约定。

---

## 3. 三维旋转矩阵

### 3.1 绕基本坐标轴的旋转

三维空间中，旋转矩阵是 \(3\times 3\) 正交矩阵。绕 \(x\)、\(y\)、\(z\) 轴旋转 \(\theta\) 角的基本旋转矩阵分别为：

\[
R_x(\theta) = \begin{bmatrix}
1 & 0 & 0 \\
0 & \cos\theta & -\sin\theta \\
0 & \sin\theta & \cos\theta
\end{bmatrix}
\]

\[
R_y(\theta) = \begin{bmatrix}
\cos\theta & 0 & \sin\theta \\
0 & 1 & 0 \\
-\sin\theta & 0 & \cos\theta
\end{bmatrix}
\]

\[
R_z(\theta) = \begin{bmatrix}
\cos\theta & -\sin\theta & 0 \\
\sin\theta & \cos\theta & 0 \\
0 & 0 & 1
\end{bmatrix}
\]

注意 \(R_y\) 中符号的位置与 \(R_x\)、\(R_z\) 不同，这是由右手定则和坐标轴方向决定的。通常我们采用右手坐标系，绕轴正方向旋转时，从轴的正向看向原点，逆时针旋转为正。

### 3.2 复合旋转

多次旋转的复合可通过旋转矩阵相乘实现。例如，先绕 \(x\) 轴旋转 \(\alpha\)，再绕 \(y\) 轴旋转 \(\beta\)，最后绕 \(z\) 轴旋转 \(\gamma\)，总旋转矩阵为：

\[
R = R_z(\gamma) R_y(\beta) R_x(\alpha)
\]

注意：矩阵乘法顺序从右至左对应旋转的顺序（右乘表示在原始坐标系下的连续旋转）。若旋转是相对于当前坐标系（即每次旋转后坐标轴发生变化），则顺序相反。本教程采用 **固定轴** 旋转顺序，即每次旋转相对于固定世界坐标系。

---

## 4. 欧拉角

### 4.1 定义与旋转顺序

欧拉角用三个独立的角度来描述刚体在三维空间中的姿态，常见的有 ZYX、ZYZ、XYZ 等顺序。在 RoboMaster 中，常用 **ZYX 欧拉角**（即航向-俯仰-滚转，Yaw-Pitch-Roll）。旋转顺序为：先绕 \(z\) 轴转 \(\psi\)（yaw），再绕 \(y\) 轴转 \(\theta\)（pitch），最后绕 \(x\) 轴转 \(\phi\)（roll）。对应的旋转矩阵为：

\[
R_{\text{ZYX}}(\phi, \theta, \psi) = R_x(\phi) R_y(\theta) R_z(\psi)
\]

展开后得到：

\[
R_{\text{ZYX}} =
\begin{bmatrix}
c\theta c\psi & c\theta s\psi & -s\theta \\
s\phi s\theta c\psi - c\phi s\psi & s\phi s\theta s\psi + c\phi c\psi & s\phi c\theta \\
c\phi s\theta c\psi + s\phi s\psi & c\phi s\theta s\psi - s\phi c\psi & c\phi c\theta
\end{bmatrix}
\]

其中 \(c = \cos\)，\(s = \sin\)。

### 4.2 从旋转矩阵到欧拉角

已知旋转矩阵 \(R = [r_{ij}]\)，可由其元素反解欧拉角：

\[
\begin{aligned}
\theta &= \arcsin(-r_{13}) \\
\phi &= \operatorname{atan2}(r_{23}, r_{33}) \\
\psi &= \operatorname{atan2}(r_{12}, r_{11})
\end{aligned}
\]

注意当 \(\theta = \pm 90^\circ\) 时会出现奇异（万向锁），此时 \(\phi\) 和 \(\psi\) 无法唯一确定。在工程中，通常通过选择不同旋转顺序（如 ZYZ）来避免奇异，或者使用四元数。

### 4.3 万向锁

当 pitch 角 \(\theta = \pm 90^\circ\) 时，yaw 和 roll 轴重合，导致一个自由度丢失。这是因为旋转矩阵退化为：

\[
R_{\text{ZYX}}(\phi, \pm 90^\circ, \psi) = 
\begin{bmatrix}
0 & 0 & \mp 1 \\
\sin(\phi \pm \psi) & \cos(\phi \pm \psi) & 0 \\
\cos(\phi \pm \psi) & -\sin(\phi \pm \psi) & 0
\end{bmatrix}
\]

此时只能确定 \(\phi \pm \psi\) 的和或差，而无法单独确定 \(\phi\) 和 \(\psi\)。在云台控制中，若要求 pitch 接近 \(90^\circ\)（如枪口指天），则需改用四元数或避免该工况。

---

## 5. 坐标变换在机器人控制中的应用

### 5.1 云台坐标系与底盘坐标系的转换

在 RoboMaster 中，云台坐标系（枪口指向）与世界坐标系之间的转换通常由云台电机和 IMU 共同完成。而底盘坐标系（车头方向）相对于世界坐标系的旋转由底盘旋转角 \(\theta_c\) 决定。

设遥控器指令 \(v_{\text{cmd}}\) 定义在云台坐标系下（推杆向前即朝枪口方向），则首先将其转换到世界坐标系：

\[
\vec{v}_w = R_z(\theta_g) \, \vec{v}_{\text{cmd}}
\]

其中 \(\theta_g\) 为云台 yaw 角。再将其转换到底盘坐标系（用于电机运动学）：

\[
\vec{v}_c = R_z^T(\theta_c) \, \vec{v}_w = R_z(-\theta_c) \, \vec{v}_w
\]

合并得：

\[
\vec{v}_c = R_z(-\theta_c) R_z(\theta_g) \, \vec{v}_{\text{cmd}} = R_z(\theta_g - \theta_c) \, \vec{v}_{\text{cmd}}
\]

即底盘系下速度 = 云台系速度旋转 \((\theta_g - \theta_c)\) 角度，这正是我们之前教程中得到的结论。

### 5.2 IMU 数据与欧拉角

IMU 通常输出以欧拉角表示的姿态（经过滤波融合）。在代码中，我们直接使用 IMU 提供的 yaw、pitch、roll 角进行控制。需要注意的是，IMU 内部使用的旋转顺序应与控制系统一致（通常也是 ZYX）。

### 5.3 旋转矩阵的几何意义

旋转矩阵的每一列表示旋转后的坐标轴在原坐标系中的方向。例如，若 \(R = [\mathbf{u} \ \mathbf{v} \ \mathbf{w}]\)，则 \(\mathbf{u}\)、\(\mathbf{v}\)、\(\mathbf{w}\) 分别是旋转后新的 \(x'\)、\(y'\)、\(z'\) 轴在世界坐标系中的单位向量。这一性质可用于将局部速度转换为全局速度。


## 6. 总结

- 旋转矩阵是描述坐标变换的标准工具，满足正交性。
- 欧拉角用三个角度直观表示姿态，但存在万向锁问题。
- 在 RoboMaster 控制中，常用 ZYX 欧拉角，并利用旋转矩阵进行云台、底盘之间的速度变换。
- 理解旋转矩阵的几何意义有助于正确推导坐标变换公式。

