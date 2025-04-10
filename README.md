# 1、论文的目标和贡献

文章解决了什么问题：提升点云对抗性攻击的不可感知性  

创新点：引入P2S场，将扰动点拖回原始表明

# 2、剖析论文

### 论文的核心算法框架:
#### 1、构建点到表明场（P2S Field）：通过深度神经网络（DNN）学习点云表明的梯度场，用来指导扰动方向
    构建P2S场（Section IV.A）
         目标：训练一个去噪网络，学习点云表明对数密度函数的梯度场
         步骤：
            1、输入：带噪声的点云       
            2、输出：每个点的梯度方向，指向表面  
            3、方法：用深度神经网络预测梯度，最小化预测梯度和真实梯度之间的l_2损失

    构建P2S场伪代码:
            Input: 点云 P, Noise level σ
            Output:训练好的P2S场 F
            For each epoch:
                添加噪声: P_noisy = P + N(0, σ)
                计算目标梯度: G_target = P - P_noisy
                预测梯度: G_pred = F(P_noisy)
                         Loss = ||G_pred - G_target||_2
                         用损失优化F
#### 2、P2场指导的对抗性攻t：结合初始扰动生成和P2S场调整，生成不可感知的对抗性点云
    P2S场指导的对抗性攻击
        目标: 生成对抗性点云P'，误导分类器，同时保持不可感知性
        步骤: 
            1、初始化扰动(使用IFGM)
            2、用P2S场调整扰动方向
            3、迭代优化，平滑误差分类损失和不可感知性约束  

        伪代码:
            Input: 干净的点云P, 标签y, 分类器 f，P2S场F, ε, 迭代次数T
            输出: 对抗点云 P'
                P'(0) = P + 随机噪声
                for t = 1 to T:
                    计算损失: L_mis = -CrossEntropy(f(P'(t)), y) // 交叉熵损失函数
                    计算梯度: ▽P'(t) L_mis
                    初始扰动: ▲P = ε * sign(▽P'(t) L_mis) / T
                /*  
                    sign 用于提取输入值的符号（正、负或零）,sign 函数的作用是简化梯度信息，用于生成初始扰动
                    sign(x) = 1, if x > 0; 0, if x = 0; -1, if x < 0
                */
                    P'(t+1) = P'(t) + ▲P
                    用P2S进行调整: P'(t+1) = P'(t+1) + F(P'(t+1))
                    限制 P'(t+1) to [-1, 1]
                // 表示对对抗性点云P'(t+1)的每个元素，即云中每个点的(x, y, z)坐标应用一个范围约束，确保他的值不会超过[ -1, 1]，实现为: P'(t+1)_i = max(-1, min(1, P'(t+1)_i))
                return P'(T)

#### 3、确定实验设置
    论文中用来什么数据？怎么评估？
        数据: ModelNet40  评估指标: ASR、Chamfer Distance
    
    论文方法分解:
        数据加载模块、模型定义模块(P2S场、)、训练模块、攻击模块
