# Policy Base Reinforcement learning

在互动的问题中（例如打游戏），模型的输出将会影响接下来的的输入。

强化学习的三要素为Actor、Environment、Reward：玩家Actor观察环境Environment，并与环境Environment交互，得到反馈Reward来更新自己。例如在一个下棋的例子里，Actor是AlphaGo，Environment是棋盘，Reward是赢棋时1、输棋时-1、其他时候都是0，围棋的规则是产生Reward的依据，它被称为Reward Function。

在Policy Base的方法中Actor使用一个称为Policy的神经网络来决策，此神经网络Policy可写作$\pi$，其中的参数为$\theta$。以一个动作游戏为例，Actor是AI操控的角色，Policy的输入是游戏的画面，输出是各个操作的几率，例如：左移：0.5，右移：0.2，跳跃：0.2，攻击：0.1，Actor根据Policy输出的几率采取行动（否则它可能永远也不会尝试某些行为），它便会得到一个Reward，如果成功击杀敌人得得到reward 1，否则0，被杀死-1。

![image-20201111144401648](E:\WorkSpace\blog\md\DeepLearning\强化学习\Image\image-20201111144401648.png)

一场游戏（可能因Actor死亡结束，也可能因其他规则结束）称为一个episode，将这个episode中所有得到的Reward求和，便可以得到Actor在对应state的这一连串操作action（称作Trajectory，记作$\tau$）的合理性，称为R，显然，训练Actor的决策网络Policy的目的就是最大化R。但是（大多数）游戏本身是有随机性的，即便是街机游戏，出怪的位置都是固定的，但怪的出招也是随机的，因此即便Policy的参数$\theta$一样，得到的结果也不一定一样，所以$R(\theta)$是没有意义的，Reward Function作用的应该是具体的某个Trajectory，因此只能选择计算$\bar R(\theta)$，即$R(\theta)$在模型采用参数$\theta$时Reward的数学期望。

数学期望是概率加权平均数，离散型随机变量的一切可能的取值xi与对应的概率P(=xi)之积的和称为的数学期望（设级数绝对收敛）,记为E。

例如，假设老虎机投入1元得到100元的概率是0.1%，得到10元的概率是1%，得到1元的概率是40%，得到0.5元的概率是38.9%，得到0.1元的概率是50%，那么期望$E=100×0.001+10×0.01+1×0.4+0.5×0.089+0.1×0.5 \approx 0.7<1$，即虽然可能第一次玩投入1元钱得到100元，看起来血赚，但投多了从概率上讲一定会亏钱。

因此$E(R(\theta))=\sum_{\tau}p_{\theta}(\tau)R(\tau)$，意为$R(\theta)$的期望为在Policy参数为$\theta$时，与环境交互出现$\tau$的几率乘出现$\tau$时Reward的值。因为大多数游戏的$\tau$都是无法穷举的，所以采用采样的方法，让Actor玩N次游戏，获得N个$\tau$，相当于从所有的$\tau$中采N个样本，并将N个样本中出现某个$\tau$的几率当做它在所有$\tau$中出现的几率，即将N次采样的均值当做期望（例如在老虎机的例子里，投入大量硬币，获得的钱除以硬币的数量也可以得到0.7的近似值，此实际上为大数定律），因此求$R(\theta)$的期望公式如下：
$$
E(R(\theta))=\sum_{\tau}p_{\theta}(\tau)R(\tau)\approx \frac{1}{N}\sum_{i=1}^{N} R(\tau^i)
$$


因为$R(\tau)$其实就是Reward Function，即游戏规则，训练的过程中改变$\theta$时不能影响它，因此训练时计算梯度的公式为
$$
\nabla\bar R(\theta)=\sum_{\tau}R(\tau)\nabla p_{\theta}(\tau)
$$
其中
$$
p_{\theta}(\tau)=p(state_1)*p_\theta(action_1|state_1)*p[(reward_1,state_2)|(state_1,action_1)]*\cdots\\
=p(state_1)\prod_{t=1}^T p_\theta(action_t|state_t)p[(reward_t,state_{t+1})|(state_t,action_t)]
$$
概率论经常会出现这种连乘积，不方便求导，一般要使用对数函数将连乘积变成连加和，如下：
$$
log(p_{\theta}(\tau))=logp(state_1)+\sum_{t=1}^T logp_\theta(action_t|state_t)+logp[(reward_t,state_{t+1})|(state_t,action_t)]
$$
将与$\theta$无关的项删去（它们导数为0），求导得到：
$$
\frac{\nabla p_\theta(\tau)}{p_\theta(\tau)} = \sum_{i=1}^T\nabla logp_\theta(action_t|state_t)
$$


对$\bar R(\theta)$做等价代换：


$$
\nabla\bar R(\theta)=\sum_{\tau}R(\tau) \nabla{p_{\theta}(\tau)}=\sum_{\tau}R(\tau)p_\theta(\tau) \frac{\nabla{p_\theta(\tau)}}{p_\theta(\tau)}\\

\approx \frac{1}{N} \sum_{n=1}^{N} {R(\tau^n) }\frac{\nabla{p_\theta(\tau^n)}}{p_\theta(\tau^n)}\\

= \frac{1}{N} \sum_{n=1}^{N} {R(\tau^n) } \sum_{t=1}^{T}\nabla{logp_\theta(action_t|state_t)}
$$







因为要最大化$\nabla\bar R(\theta)$，所以采用梯度上升：
$$
\theta' = \theta+ \eta \nabla\bar R(\theta) \\
\theta' = \theta + \eta \frac{1}{N} \sum_{n=1}^{N} {R(\tau^n) } \sum_{t=1}^{T}\nabla{logp_\theta(action_t|state_t)}
$$

回想梯度上升和梯度下降的原理，在一维的情况下，导数大于0，那么x增大y增大，如果选择梯度上升就是正的。梯度$\nabla f(x)$的方向就是$f(x)$上升最快的方向（梯度上升），梯度的反方向$-\nabla f(x)$就是$f(x)$下降最快的方向，因此这个梯度上升的式子实际上会在$R(\tau^n)>0$时最大化$p_\theta(action_t|state_t)$，也就是采用本场对策的几率。

收集数据，当做分类任务，但是在loss上乘一个权重，即本场游戏获得的Reward，这样模型会偏向于Reward为正时的参数，而背离Reward为负时候的参数

> -b让Reward不总是正的

如果要用PyTorch等深度学习框架的话，它们默认都是梯度下降的，那么loss就是$-\eta \frac{1}{N} \sum_{n=1}^{N} {R(\tau^n) } \sum_{t=1}^{T}\nabla{logp_\theta(action_t|state_t)}$，显然这不是CrossEntropy更不是MSELoss，需要自行编写损失函数。

