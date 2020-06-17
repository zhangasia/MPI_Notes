#####  Cannon算法
* 算法过程
  假设矩阵$A，B$和$C$都可以分成$m\times m$块矩阵，即$A = (A_{(ij)})_{m\times m}，B = (B_{(ij)})_{m\times m}$和$C = (C_{(ij)})_{m\times m}$，其中$A_{ij}，B_{ij}$和$C_{ij}$是$n \times n$矩阵，进一步假设有$p = m \times m$个处理器。为了讨论**Cannon**算法，引入块置换矩阵$Q = (Q_{ij})$。即

$$
Q = \left [
\begin{matrix}
0 & 1 &0 & \cdots & 0\\ 
0 & 0 &1 & \cdots & 0 \\ 
\vdots & \vdots & \vdots & \ddots & \vdots \\ 
0 & 0 &0  & \cdots & 1 \\ 
1 & 0 &0 & \cdots & 0 
\end{matrix} 
\right ]
,\quad Q_{ij} = 
\begin{cases}
1,j \equiv (i+1)mod m\\
0,other
\end{cases}
$$
>$QA$就是将$A$的所有行向上移动一个位置，$AQ$则是将$A$的所有列向右移动一个位置。

定义块对角矩阵$D_A^{(l)} = diag(D_i^{(l)}) = diag(A_{i,i+1mod  m})$，容易证明$A = \sum_{l=0}^{m-1}D_A^{(l)}Q^l$，于是

$$
\begin{aligned}
C &=AB=\sum_{l=0}^{m-1}D_A^{(l)}Q^lB\\
&=D_{A}^{(0)}B^{(0)}+D_{A}^{(1)}B^{(1)}+...+D_{A}^{(m-1)}B^{(m-1)}
\end{aligned}
$$

其中$B^{(l)} = Q^lB = QB^{l-1}，l = 0,1,...,m-1$

假如：$A$是$3\times 3$的矩阵，则
$$
D^{(0)}_A = \left [
\begin{matrix}
A_{0,0} & 0 &0 \\ 
0 & A_{1,1} &0  \\ 
0 & 0 & A_{2,2} \\ 
\end{matrix} 
\right ] ，

 D^{(1)}_A = \left [
\begin{matrix}
A_{0,1} & 0 &0 \\ 
0 & A_{1,2} &0  \\ 
0 & 0 & A_{2,0} \\ 
\end{matrix} 
\right ] ，

D^{(2)}_A = \left [
\begin{matrix}
A_{0,2} & 0 &0 \\ 
0 & A_{1,0} &0  \\ 
0 & 0 & A_{2,1} \\ 
\end{matrix} 
\right ]
$$

$$
Q^0 = \left [
\begin{matrix}
1 & 0 &0 \\ 
0 & 1 &0  \\ 
0 & 0 & 1 \\ 
\end{matrix} 
\right ] ，

Q^1 = \left [
\begin{matrix}
0 & 1 &0 \\ 
0 & 0 &1  \\ 
1 & 0 &0 \\ 
\end{matrix} 
\right ] ，

Q^2 = QQ = \left [
\begin{matrix}
0 & 0 &1 \\ 
1 & 0 &0  \\ 
0 & 1 & 0 \\ 
\end{matrix} 
\right ]
$$
经过计算$A = \sum_{l=0}^{m-1}D_A^{(l)}Q^l$

**Cannon**算法是为了更加便于并行，可以把矩阵乘转化为若干个小的计算单元，分别用不同的进程去进行计算，而互不干扰。

Cannon算法采用了主从模式的同时也采用了分而治之的模式。一方面，0号线程作为Master，负责矩阵A和矩阵B以及矩阵C的I/O，也负责小矩阵的分发和结果的聚集。而其他节点作为Worker进行本地的小矩阵串行乘法计算。另一方面，Cannon算法将两个大矩阵的乘法运算分解为若干各小矩阵的乘法运算，最终计算结束后，将计算结果聚集回来，也采用了分而治之的思想。cannon算法不仅实现了矩阵乘法运算的并行化，也减少了分块矩阵乘法的局部存储量，节省了节点的内存开销。