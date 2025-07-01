# 代码能力提升实战教程

## Git工作流实战

### 基础操作命令

=== "代码提交"
    ```bash
    # 添加修改文件
    git add .
    
    # 提交更改
    git commit -m "✨ 修复云台校准逻辑"
    
    # 推送到远程仓库
    git push 
    ```

=== "分支管理"
    ```bash
    # 创建新分支
    git branch xxx
    
    # 切换分支
    git checkout xxx
    
    # 创建并切换分支
    git checkout -b xxx
    ```


=== "拉取更新"
    ```bash
    # 拉取当前分支更新
    git pull
    
    # 获取所有分支更新
    git fetch --all
    
    # 合并特定分支
    git merge origin/xxx
    ```

!!! note "分支操作注意事项"
    在执行分支操作前，请务必使用 `git branch` 确认当前所在分支。  
    这里输出的名称是你当前所在分支的名称，后面的 `<current_branch_name>` 均应替换为你当前所在分支的名称，即上个命令的输出。

### 高级Git技巧

#### 1. 设置代理解决网络问题
```bash
# 设置代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890

# 查看当前代理配置
git config --global --get http.proxy
# 取消代理设置
git config --global --unset http.proxy
```

### .git目录结构解析
Git的核心是`.git`目录，包含版本控制的所有信息

```bash
.git/
├── HEAD            # 当前所在分支指针
├── config          # 仓库特定配置
├── description     # 仓库描述
├── hooks/          # 客户端钩子脚本
├── info/           # 全局排除模式
├── objects/        # 所有数据对象
├── refs/           # 分支和标签指针
│   ├── heads/      # 分支指针
│   └── tags/       # 标签指针
└── index           # 暂存区信息
```

#### 关键文件功能说明：

| 文件/目录       | 功能说明                                                                 |
|-----------------|--------------------------------------------------------------------------|
| **HEAD**        | 指向当前分支的引用，内容如 `ref: refs/heads/main`                        |
| **config**      | 仓库级配置，优先级高于全局配置，可设置远程仓库URL、分支跟踪等             |
| **hooks/**      | 客户端钩子脚本，可自定义Git操作前后触发的事件（如pre-commit, post-merge）|
| **objects/**    | 数据库对象存储（blob-文件内容, tree-目录结构, commit-提交信息）           |
| **refs/heads/** | 分支指针文件，每个文件对应一个分支，内容为最新提交的SHA-1值              |
| **index**       | 暂存区索引，记录下次提交的文件状态（二进制文件）                         |

> **比喻理解**：`.git`目录就像机器人的"黑匣子"，完整记录项目的所有历史变更

### VSCode高级操作指南

1. **解决代码冲突**：
   - 打开冲突文件，选择"Accept Current Change"或"Accept Incoming Change"
   - 或手动编辑解决冲突后标记为已解决

2. **提交Pull Request**：
   ```mermaid
   graph LR
   A[推送分支到远程] --> B[在GitHub创建PR]
   B --> C[团队成员审查代码]
   C --> D[解决审查意见]
   D --> E[合并到主分支]
   ```

3. **可视化版本回退**：
   - 打开VSCode源代码管理视图
   - 右键提交记录选择"撤销提交"
   - 或使用"重置当前分支到此次提交"

!!! success "最终交付要求"
    - 在组织仓库提交至少1个PR
    - 完成1次代码冲突解决
    - 演示版本回退操作
    - PR需包含有意义的描述和修改说明


以下是针对代码双平台备份（GitHub+Gitee）的完整解决方案，整合自动同步、仓库管理、免密登录及跨平台Hook配置：

---

### 一、GitHub Actions同步方案

#### **配置步骤**
=== ".github/workflows/sync.yml"
    ```yaml
    name: Sync to Gitee
    on: [push]  # 代码推送时触发

    jobs:
      mirror:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout
            uses: actions/checkout@v4

          - name: Mirror to Gitee
            uses: wearerequired/git-mirror-action@v1
            env:
              SSH_PRIVATE_KEY: ${{ secrets.GITEE_SSH_KEY }}  # 私钥存入GitHub Secrets
            with:
              source-repo: "[email?protected]:yourname/repo.git"
              destination-repo: "[email?protected]:yourname/repo.git"
    ```

#### **密钥配置流程**
1. **生成密钥对**（所有系统通用）：
    ```bash
    ssh-keygen -t rsa -C "[email?protected]"
    # 默认路径：~/.ssh/id_rsa（私钥）和 id_rsa.pub（公钥）
    ```
2. **公钥添加至Gitee**：  
   `设置 -> SSH公钥` 粘贴 `id_rsa.pub` 内容  
3. **私钥存入GitHub Secrets**：  
   `仓库Settings -> Secrets -> New repository secret`  
   名称填 `GITEE_SSH_KEY`，值粘贴私钥文件**完整内容**（含`-----BEGIN RSA PRIVATE KEY-----`）

> ✅ **验证同步**：GitHub推送后查看Actions日志，成功同步后Gitee仓库更新时间应与GitHub一致

---

### 二、仓库创建与克隆指南
#### **1. 创建仓库**
- **GitHub**：主页 → "+" → New repository  
- **Gitee**：项目 → "+" → 创建仓库 → **不初始化**（避免冲突）

然后对着终端输入相应命令即可。

#### **2. 克隆仓库（SSH协议）**
```bash
# 通用命令（需提前配置SSH密钥）
git clone [email?protected]:yourname/repo.git
```

---

### 三、免密登录配置
#### **核心原理**
通过SSH公钥认证，客户端用私钥签名，服务端用公钥验证  
![](https://www.wsisp.com/ssh-auth-flow.png)

#### **操作步骤**
1. **本地生成密钥**（Windows示例）：
   ```powershell
   # PowerShell
   ssh-keygen -t rsa  # 三次回车跳过密码
   ```
2. **上传公钥到远程**：
   ```powershell
   scp C:\Users\user\.ssh\id_rsa.pub user@gitee.com:~/.ssh/
   ssh user@gitee.com "cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys"
   ```
3. **权限修正（关键！）**：  
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys  # 权限不足会导致认证失败
   ```

---

### 四、Git Hook跨平台适配
#### **操作系统差异对比**
| 要素         | Linux/Mac                  | Windows                     |  
|--------------|----------------------------|-----------------------------|
| **脚本语言** | Bash                       | PowerShell/Batch           |  
| **路径分隔符**| `/`                        | `\`                         |  
| **执行权限** | `chmod +x .git/hooks/*`    | 无需权限设置               |  

#### **Hook脚本示例（提交信息自动加Emoji）**
=== "Linux/Mac (.git/hooks/pre-commit)"
    ```bash
    #!/bin/sh
    MSG=$(cat $1)
    if [[ ! $MSG =~ ✨|🐛|🔧 ]]; then
      sed -i '1s/^/🔧 /' $1  # 无Emoji时自动添加
    fi
    ```

=== "Windows (.git/hooks/pre-commit.ps1)"
    ```powershell
    $MSG = Get-Content $args[0]
    if ($MSG -notmatch "✨|🐛|🔧") {
      "🔧 $MSG" | Set-Content $args[0]
    }
    ```
> ⚠️ **注意事项**：  
> 1. Windows需删除脚本后缀（`.ps1`→无后缀）  
> 2. 跨团队协作时Hook**不纳入版本控制**（避免执行冲突）  

---

### 五、备份方案对比
| 方案               | 自动化 | 安全性 | 适用场景          |  
|--------------------|--------|--------|-------------------|  
| **GitHub Actions** | ✅      | ★★★★   | 开源项目/团队协作 |  
| 本地双remote推送   | ❌      | ★★☆    | 个人小项目        |  
| 定时脚本同步       | ✅      | ★★★☆   | 私有服务器备份    |  

> 💎 **推荐策略**：  
> - 主仓库用 **GitHub Actions 实时同步**到 Gitee  
> - 关键版本额外备份至**私有NAS**（通过`rsync` + `cron`）  

通过此方案，开发者只需维护GitHub仓库，即可实现Gitee镜像自动更新，同时保障跨平台开发体验一致性。


## Markdown文档规范

### 基础语法速查

```markdown
# 一级标题
## 二级标题

- 无序列表项
1. 有序列表项

**粗体** *斜体* ~~删除线~~

[链接文本](https://example.com)

![图片描述](image.png)
```

### 代码块与表格

=== "代码块"
    ```c
    // 底盘控制参数示例
    #define CHASSIS_MAX_VELOCITY 3.0f // m/s
    #define CHASSIS_ROTATE_RATIO 0.6f
    ```

=== "表格"
    | 模块        | 功能描述          | 依赖项     |
    | ----------- | ----------------- | ---------- |
    | `can.c`     | CAN总线通信       | bsp_can    |
    | `gimbal.c`  | 云台控制逻辑      | pid, motor|
    | `shoot.c`   | 发射机构控制      | friction, feed|

!!! tip "文档规范要求"
    1. 使用三级标题结构组织内容
    2. 关键参数必须使用代码块标注
    3. 模块依赖关系用表格清晰展示
    4. 复杂流程配合mermaid流程图说明


## 湖南大学框架解析

### 三层架构解析

```mermaid
graph TD
A[硬件层] -->|提供驱动| B[BSP抽象层]
B -->|标准接口| C[MODULE驱动层]
C -->|功能模块| D[APP应用层]
D -->|控制指令| C
C -->|设备操作| B
```

> **比喻理解**：框架就像机器人身体的神经系统：
> - 硬件层是四肢和感官（电机、传感器）
> - BSP层是脊髓（基础反射）
> - MODULE层是小脑（协调运动）
> - APP层是大脑（高级决策）

### 关键修改实践

=== "底盘参数调整"
    ```c
    // 文件路径：modules/chassis/chassis.c
    void chassis_task() {
        // 修改最大速度参数
        chassis.max_velocity = 3.5f; // 原值3.0f
        
        // 调整旋转灵敏度
        chassis.rotate_sensitivity = 0.7f; // 原值0.6f
    }
    ```

=== "云台校准流程"
    ```c
    // 文件路径：applications/gimbal_task.c
    void gimbal_calibration() {
        // 新增校准超时检测
        if (timeout > 5000) { // 增加5秒超时判断
            ERROR_HANDLE(CALIBRATION_TIMEOUT);
        }
        
        // 修改零点检测阈值
        if (fabs(current_angle) < 0.01f) { // 原值0.05f
            set_zero_position();
        }
    }
    ```

=== "开发板配置切换"
    ```c
    // 在board.h中选择开发板配置
    #define BOARD_TYPE_BOARD_A
    // #define BOARD_TYPE_BOARD_B
    
    #ifdef BOARD_TYPE_BOARD_A
        #include "board_a_config.h"
    #elif defined BOARD_TYPE_BOARD_B
        #include "board_b_config.h"
    #endif
    ```

!!! warning "重要注意事项"
    1. 修改参数后必须进行完整编译 `make clean && make`
    2. 条件编译切换后需重新初始化外设
    3. 所有修改必须通过git提交并推送到远程仓库

### 最终交付要求

1. 提交三层架构图（mermaid格式）
2. 演示底盘/云台参数修改效果
3. 展示条件编译切换开发板配置
4. 所有代码修改通过PR提交到组织仓库

## 实战练习题

### 练习仓库任务
```bash
# 克隆练习仓库
git clone https://github.com/RoboMaster/training-lab.git

# 切换到练习分支
git checkout -b week1-training
```

### Git任务
1. 配置Git代理并测试连接速度
2. 添加备份仓库并设置自动同步
3. 实现提交信息自动添加Emoji
4. 修改`.git/config`添加新的远程仓库
5. 解决预设冲突文件merge_conflict.c

### 框架修改任务
1. 在gimbal.c中：
   ```c
   // TODO: 添加云台角度软限制
   // 要求: yaw范围[-π, π], pitch范围[0, π/2]
   void gimbal_angle_limit(float* yaw, float* pitch) {
       // 你的代码
   }
   ```
   
2. 在chassis.c中：
   ```c
   // TODO: 添加底盘功率限制
   // 当功率超过80W时线性降低速度
   void chassis_power_limit() {
       // 你的代码
   }
   ```

### 文档任务
1. 为新增功能编写文档说明
2. 创建参数配置表格
3. 绘制数据流程图：
   ```mermaid
   graph LR
   A[遥控器输入] --> B[底盘控制]
   B --> C[电机驱动]
   C --> D[速度反馈]
   D --> B
   ```

### 提交要求
```bash
# 完成所有任务后
git add .
git commit -m "feat: 完成第一周训练任务"
# 提交时会自动添加✨前缀

git push origin week1-training
git push backup week1-training  # 推送到备份仓库

# 创建Pull Request并标记导师审核
```

> **专业提示**：完成任务时保持"小步提交"原则，每个功能点单独提交，便于问题追踪和代码审查