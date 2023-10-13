---
layout: cover
---
# Chap3 Convolutional Codes
---

# 卷积编码概述

- 卷积编码也属于线性分组码,因为卷积编码也是通过对information symbol进行**线性变换**得到code symbol(因此,也有”生成矩阵”)
- 与线性分组码不同的是,卷积编码具有memory
- 卷积码流行的主要原因是
    - 存在高效的译码算法
        - 维特比算法
        - BCJR算法
        - 上面的两种算法都基于trellis,而trellis高度重复的结构使其非常适于被pipelined hardware实现
    - 可以使用软输入进行译码,并产生软输出
        - 软输入: 译码前的demod模块不输出一个确定的symbol,而是输出一个置信度
        - 软输出: 译码后的结果不是一个确定的symbol,而是一个置信度
        - 这种译码方式对于构造级联卷积码(turbo码)很重要

---
layout: cover
---

# 3.1 卷积码的编码和基本性质

---
layout: two-cols-header
---

# LTI系统/FIR视角下的卷积编码

- 卷积编码过程是有限域$\mathbb{F}_2$上的FIR滤波过程,卷积编码的生成多项式就是其冲激响应,卷积编码的输出是其输入序列与生成多项式进行卷积的结果

::left::

- 在右图中有
    
    $\begin{aligned}
    & b_i^{(1)}=u_i+u_{i-1}+u_{i-2}, \\
    & b_i^{(2)}=u_i+u_{i-2}
    \end{aligned}$


- 将其改写为卷积形式有
    
    $\begin{aligned}& b_i^{(1)}=\sum_{l=0}^m u_{i-l} g_l^{(1)} \leftrightarrow \mathbf{b}^{(1)}=\mathbf{u} * \mathbf{g}^{(1)}, \\& b_i^{(2)}=\sum_{l=0}^m u_{i-l} g_l^{(2)} \leftrightarrow \mathbf{b}^{(2)}=\mathbf{u} * \mathbf{g}^{(2)} . \\&\end{aligned}$
    

::right::

![Untitled](/Untitled.png)

其中,$\mathbf{g}^{(1)} = (1,1,1,0,\cdots)$,$\mathbf{g}^{(2)} = (1,0,1,0,\cdots)$


---
layout: two-cols-header
---

# 卷积编码参数

::left::

- 卷积编码的”尺寸”可以被一个三元组$(n, k, m)$刻画,这样的卷积编码器有:
    - k路输入
    - 连接着k组移位寄存器,移位寄存器的最大深度为m,寄存器总数$v \le k\times m$
    - n路输出
- 这样的卷积编码器,编码效率为$R=\frac{k}{n}$
- 卷积编码的每一路输出都是k路输入分别与生成多项式卷积后相加的结果,因此,卷积编码可以被$k \times n$个生成多项式完全刻画

::right::

![Untitled](/Untitled%201.png)

---

# 矩阵运算视角下的卷积编码

- 与线性分组码类似地,卷积编码过程可以被描述为一个矩阵乘法$\mathbf{b}=\mathbf{u G}$
- 与线性分组码不同地,卷积编码的输入和输出都是semi-infinite序列,因此,生成矩阵也应当是semi-infinite矩阵
- 生成矩阵具有下面的形式

$$\mathbf{G}=\left(\begin{array}{ccccccc}\mathbf{G}_0 & \mathbf{G}_1 & \ldots & \mathbf{G}_m & \mathbf{0} & \mathbf{0} & \ldots \\\mathbf{0} & \mathbf{G}_0 & \mathbf{G}_1 & \ldots & \mathbf{G}_m & \mathbf{0} & \ldots \\\mathbf{0} & \mathbf{0} & \mathbf{G}_0 & \mathbf{G}_1 & \ldots & \mathbf{G}_m & \ldots \\\mathbf{0} & \mathbf{0} & \mathbf{0} & \ddots & \ddots & & \ddots\end{array}\right)$$

- 其中每个子矩阵$\mathbf{G}_i$具有下面的形式,子矩阵的每个元素$g_{l, i}^{(j)}$为从第$l$路输入去向第$j$路输出的生成多项式上的第$i$个系数

$$\mathbf{G}_i=\left(\begin{array}{cccc}g_{1, i}^{(1)} & g_{1, i}^{(2)} & \ldots & g_{1, i}^{(n)} \\g_{2, i}^{(1)} & g_{2, i}^{(2)} & \ldots & g_{2, i}^{(n)} \\\vdots & \vdots & & \vdots \\g_{k, i}^{(1)} & g_{k, i}^{(2)} & \ldots & g_{k, i}^{(n)}\end{array}\right) \quad \text { for } i \in[0, m]$$

---
layout: two-cols-header
---

# 有限状态机视角下的卷积编码

::left::

- 卷积编码器中所有$v$个寄存器的状态构成了卷积编码器的状态
- 在右侧的状态转移图中,我们为每一种可能的状态转移进行连线,并在状态转移路径上标明了input/output

::right::

<img src='/Untitled%202.png' class='m-10 h-100'>

---
layout: two-cols-header
---

# 有限状态机视角下的卷积编码

::left::

- 状态转移图的一个变体是trellis图,使用不同列上的顶点表示不同时刻的状态,使用边表示状态转移,在$i$时刻和$i+1$时刻间,我们可以画出一个trellis module;重复这一module,我们可以得到随时间延伸的trellis图

::right::

<img src='/Untitled%203.png' class='m-10 h-100'>


---

# 卷积编码的termination问题

- 虽然从定义出发,卷积编码过程的输入是semi-infinite的,但实际实现时,往往是对一个确定长度的序列进行卷积编码,为构造确定长度的序列,有下面三种方法:

![Untitled](/Untitled%204.png)

---

# 卷积编码的termination问题

|  | 处理方法 | 起始状态 | 结束状态 | 误码率 | 编码效率 | 译码复杂度 |
| --- | --- | --- | --- | --- | --- | --- |
| Truncation | 直接编码 | all-zero | 不确定 | 末尾bits的误码率下降 | R | 不变 |
| Termination | 在末尾填充$k \cdot m$个零比特,然后直接编码 | all-zero | all-zero | 不变 | 下降为$R \frac{L}{L+m}$ | 不变 |
| Tail-biting | 通过最后$k \cdot m$个输入比特决定起始状态,然后直接编码 | 由最后$k \cdot m$个输入比特决定 | 与起始状态一致 | 不变 | R | 上升 |

---

# 卷积编码puncturing

- 尽管上文中给出了k路输入,n路输出的通用编码器模型,在实践中,一般只使用1/n卷积码(即k=1).其中一个原因是,高编码效率的卷积码可以通过一种简单的方法从1/n卷积码上构造得到,这种方法被称为puncturing(中译穿孔).同等编码效率下,穿孔1/n卷积码的译码比k/n卷积码要简单得多.
- 穿孔是通过周期性地删除编码后序列的部分bits实现的,可以通过一个$n \times T$的puncturing matrix $\mathbf{P}$来刻画,其中$T$为周期,矩阵中的1代表保留,0代表删除.
    - 一个例子是,通过矩阵$\mathbf{P}=\left(\begin{array}{ll}1 & 1 \\1 & 0\end{array}\right)$对$\mathbb{B}(2,1,2)$进行穿孔可以得到$\mathbb{B}_{\text {punctured }}(3,2,1)$.

      ![puncturing](/puncturing.png)
---

# D-Domain视角下的卷积编码

- 在本书的前面章节中,并没有对D-Domain和D-transform进行介绍,在这里进行补充
    - D-Domain上的运算是binary sequence之间的运算,运算律遵循在$\mathbb{F}_2$上引入delay operator D扩展得到的多项式运算规则
    - D-transform是通过D域多项式来表达离散时域序列的方法,如下:
        
        $\mathbf{x}=x_i, x_{i+1}, x_{i+2}, \ldots \circ \bullet X(D)=\sum_{i=j}^{+\infty} x_i D^i, j \in \mathbb{Z}$
        
- 在D域上,卷积编码可以被表示为:
    
    $\begin{aligned}& \mathbf{u}(D)=\mathbf{u}_0+\mathbf{u}_1 D+\mathbf{u}_2 D^2+\cdots \\& \mathbf{b}(D)=\mathbf{b}_0+\mathbf{b}_1 D+\mathbf{b}_2 D^2+\cdots\end{aligned}$
    
    $\mathbf{b}(D)=\mathbf{U}(D) \mathbf{G}(D)$
    

---

# 卷积编码的性质

### encoder,generator和codes

- code是所有输出序列的集合,如果两个encoder具有相同的code,则称它们为equivalent encoders,对于生成矩阵同理,equivalent generators之间有如下关系:
    
    $\mathbf{G}^{\prime}(D)=\mathbf{T}(D) \mathbf{G}(D)$,其中$\mathbf{T}(D)$是一个non-sigular $k \times k$矩阵
    

### catastrophic generator matrix

- 使含有大量1s的输入能够产生几乎不含1s的输出的生成矩阵
- 我们希望避免使用这样的生成矩阵,因为少量传输错误能够导致大量译码错误

### systematic generator matrix

- 包含单位阵的生成矩阵(因此,输出序列中包含输入序列)
- 这一结构的生成矩阵在concatenated codes中很有用
- 这一结构的生成矩阵不可能是catastrophic的(因为输出中至少包含与输入相同数量的1s)
- 不过,Codes with polynomial systematic generator matrices usually have poor distance- and error-correcting capabilities.

---
layout: cover
---

# 3.2 卷积码的trellis表示和维特比算法

---

# 译码准则:从最小错误概率到最短距离

- 我们的出发点是寻找译码方法
    - 使译码错误概率$\Sigma{\operatorname{Pr}(\hat{\boldsymbol{b}} \neq \boldsymbol{b} \mid \boldsymbol{r})\operatorname{Pr}(\boldsymbol{r})}$最小
    - 使$\Sigma{(1 - \operatorname{Pr}(\hat{\boldsymbol{b}} = \boldsymbol{b} \mid \boldsymbol{r}))\operatorname{Pr}(\boldsymbol{r})}$最小
    - 使$\Sigma{\operatorname{Pr}(\hat{\boldsymbol{b}} = \boldsymbol{b} \mid \boldsymbol{r})\operatorname{Pr}(\boldsymbol{r})}$最大
- $\operatorname{Pr}(\boldsymbol{r})$与译码方法无关
- 上面的准则等价于$\hat{\mathbf{b}}=\underset{\mathbf{b}}{\operatorname{argmax}}\{\operatorname{Pr}\{\mathbf{b} \mid \mathbf{r}\}\}$,称为最大后验概率译码准则(MAP)
- 由贝叶斯公式$\operatorname{Pr}(\mathbf{b} \mid \mathbf{r}) = \frac{\operatorname{Pr}\left(\boldsymbol{b}\right) \operatorname{Pr}\left(\boldsymbol{r} \mid \boldsymbol{b}\right)}{\operatorname{Pr}(\boldsymbol{r})}$
- $\operatorname{Pr}(\boldsymbol{r})$与译码方法无关
- 所有码字的出现概率$\operatorname{Pr}(\boldsymbol{b})$相同时,$\operatorname{Pr}(\boldsymbol{b})$也与译码方法无关
    - 在BSC信道中,这一条件成立
    - 在AWGN信道+BPSK调制中,这一条件可以成立
- 上面的准则等价于$\hat{\mathbf{b}}=\underset{\mathbf{b}}{\operatorname{argmax}}\{\operatorname{Pr}\{\mathbf{r} \mid \mathbf{b}\}\}$,称为最大似然译码准则(ML)

---

# 译码准则:从最小错误概率到最短距离

![Untitled](/Untitled%205.png)

---

# Trellis图和最短距离准则

![Untitled](/Untitled%206.png)

- 在上文中,我们提到了有限状态机视角下,卷积编码可以被刻画为一个trellis module,它刻画了从$i$时刻到$i+1$时刻的过程,将这一结构平行重复$n$次,我们可以得到包含时刻$0$~$n$的trellis图,它刻画了$n \times k$个码字被编码的过程.
- 在trellis图视角下,译码等价于寻找一条从$0$时刻到达$n$时刻的路径,通过距离的恰当定义,以最大似然准则译码等价于寻找一条最短路径,下面对两种信道模型分别讨论

---

# BSC信道与汉明距离

- 接收到的码字$\mathbf{r}$为一个二进制序列$\overline{r_1\cdots r_n}$
- 两点间的距离被定义为两点间应输出的码字$\mathbf{b}$与实际接收的$\mathbf{r}$之间的汉明距离,记为$\operatorname{dist}(\mathbf{r}, \mathbf{b})$
- BSC信道下有
    - 由于信道是无记忆的,$\operatorname{Pr}\{\mathbf{r} \mid \mathbf{b}\}=\prod_{i=1}^n \operatorname{Pr}\left\{r_i \mid b_i\right\}$
    - $\operatorname{Pr}\left\{r_i \mid b_i\right\}=\varepsilon^{\operatorname{dist}(r_i, b_i)}(1-\varepsilon)^{n-\operatorname{dist}(r_i, b_i)}$
    - $\operatorname{Pr}\{\mathbf{r} \mid \mathbf{b}\} = (1-\varepsilon)^n\left(\frac{\varepsilon}{1-\varepsilon}\right)^{\operatorname{dist}(\mathbf{r}, \mathbf{b})}, \varepsilon < {1\over 2},\left(\frac{\varepsilon}{1-\varepsilon}\right)<1$
    - 因此,$\operatorname{Pr}\{\mathbf{r} \mid \mathbf{b}\}$在$\operatorname{dist}(\mathbf{r}, \mathbf{b})$最小时最大,最短路径译码等价于最大似然译码

---

# AWGN信道(BPSK调制)与欧氏距离

- 接收到的码字$\mathbf{r}$为一个实数序列$\overline{r_1\cdots r_n}$
- 两点间的距离被定义为两点间应输出的码字$\mathbf{b}$与实际接收的$\mathbf{r}$之间的squared Euclidean distance $\sum_i\left(r_i-b_i\right)^2$,记为$\operatorname{dist}(\mathbf{r}, \mathbf{b})$
- AWGN信道下有:
    - 由于信道是无记忆的,$\operatorname{Pr}\{\mathbf{r} \mid \mathbf{b}\}=\prod_{i=1}^n \operatorname{Pr}\left\{r_i \mid b_i\right\}$
    - $\operatorname{Pr}\{r_i \mid b_i\}=\frac{1}{\sqrt{2 \pi \sigma^2}} \exp \left(-\frac{\left(r_i-b_i\right)^2}{2 \sigma^2}\right)$
    - $\operatorname{Pr}\{\mathbf{r} \mid \mathbf{b}\}=\prod_{i=1}^n \frac{1}{\sqrt{2 \pi \sigma^2}} \exp \left(-\frac{\left(r_i-b_i\right)^2}{2 \sigma^2}\right)$
    - 由于目的是判决,可以忽略常数项$\frac{1}{\sqrt{2 \pi \sigma^2}}$,$\frac{1}{2 \sigma^2}$并取对数,最大似然条件为
        
        
        $\hat{\mathbf{b}}=\underset{\mathbf{b}}{\operatorname{argmax}}\left\{\sum_i-\left(r_i-b_i\right)^2\right\}=\underset{\mathbf{b}}{\operatorname{argmin}}\left\{\sum_i\left(r_i-b_i\right)^2\right\}$
        
    - 因此,$\operatorname{Pr}\{\mathbf{r} \mid \mathbf{b}\}$在$\operatorname{dist}(\mathbf{r}, \mathbf{b})$最小时最大,最短路径译码等价于最大似然译码

---

# Viterbi算法

- Viterbi算法是一种动态规划算法,这类算法依赖于最优子结构性质,即”全局最优解中必然包含局部最优解”
- Viterbi算法利用了最短路径问题的最优子结构,即最短路径的子路径也是最短路径.因此,在Viterbi算法的搜索过程中,只保留到达每一时刻所有状态的最短路径,而非所有路径.

![Untitled](/Untitled%207.png)

---

# Viterbi算法

- 将trellis图中时刻$i$的第$j$个节点记为$\sigma_{j,i}$,Viterbi算法的过程如下(对于以termination方式中止的卷积编码):
    1. 为初始节点$\sigma_{0,0}$赋节点度量$\Lambda\left(\sigma_{0,0}\right) = 0$
    2. (forward pass)对下一时刻$i$的所有节点$\sigma_{j,i}$:
        1. 加: 为到达该节点的分支$\sigma_{j^{\prime}, i-1} \rightarrow \sigma_{j, i}$,计算分支度量$\Lambda\left(\sigma_{j',i-1}\right) + \operatorname{dist}\left(\mathbf{r}_i, \mathbf{b}_i^{\prime}\right)$
        2. 比: 比较所有分支度量,找到最小的分支度量作为新的节点度量$\Lambda\left(\sigma_{j,i}\right)$
        3. 选: 保存带来最小分支度量的分支,具体来说,在$\sigma_{j,i}$上记录其前驱节点即可
    3. 当$i \le n$时,重复步骤2,否则执行步骤4
    4. (backward pass)从最终节点$\sigma_{0,n}$开始,回溯前驱节点并产生译码结果,直到$\sigma_{0,0}$为止

---
layout: two-cols-header
---

# Viterbi算法的复杂度分析

::left::

- 对于$(n, k, m)$卷积编码
    - 每个trellis module中有$2^v$个不同的状态,其中$v$是寄存器总数,这一数量小于等于$k \times m$
    - 每个状态节点产生$2^k$个分支,共有$2^{v+k}$个分支
- Viterbi算法在forward pass的每一步要:
    - 计算$2^{v+k}$个分支度量,,每个分支度量的计算需要:
        - 查一个规模为$2^v$的表,得到前驱节点的分支度量$\Lambda\left(\sigma_{j',i-1}\right)$
        - 查一个规模为$2^{v+k}$的表,得到分支所产生的码字$\mathbf{b}_i^\prime$
        - 计算距离$\operatorname{dist}\left(\mathbf{r}_i, \mathbf{b}_i^{\prime}\right)$
    - 从$2^{v+k}$个分支度量中找出最小值
    - 存储$2^v$个节点度量和$2^v$个前驱节点

::right::

- Viterbi算法在backward pass的每一步要:
    - 查一个规模为$2^v$的表,得到前驱节点
    - 查一个规模为$2^{v+k}$的表,得到分支所产生的码字
- 结论:
    - Viterbi算法的时空复杂度在每一步是一个常量,其复杂度与码长是线性关系,是一种高效算法
    - 然而,这一常量对于卷积编码的尺寸参数$k$和$m$非常敏感,随它们的增长呈指数增长

---

# 穿孔的Viterbi算法

- Viterbi算法和最短路径译码对穿孔卷积码同样适用,其要点在于不要让被删除的bits对判决产生影响

- 对于AWGN + BPSK情景,可以预先通过一个depuncturing模块为穿孔部分填$r_i = 0$,这样,对于每个分支,其贡献的距离是相同的($b_i = 1|-1, (0-1)^2 = (0+1)^2$)
- 类似地,对于BSC情景,也可以通过填入$r_i = {1\over2}$或是其它手段,使其对于每个贡献相同的距离

