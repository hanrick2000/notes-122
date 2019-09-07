# Gradient Descent / Gradient Boosting

ML里的很多问题都可以简化为求解优化问题，让loss function最小的 $$\hat{\theta}=\underset{\theta}{\arg \min } L(\theta)$$ 。

简单的model中L可以直接求解得到，但是对于参数多的L，求解过程困难。因为实际的loss function多是连续、可导的，那就有了gradient descent求解的思路。

$$
\begin{array}{l}{\text { 1: } \theta \leftarrow \theta_{0}} \\ {\text { 2: while } \nabla L(\theta)>\epsilon \text { do }} \\ {\text { 3: } \quad \theta \leftarrow \theta-\alpha \cdot \nabla L(\theta)} \\ {\text { 4: return } \theta}\end{array}
$$

$$\epsilon$$ 是tolerance，很小的数值（1e-6）， $$\alpha$$ 是learning rate。

{% hint style="info" %}
基于导数的优化问题（不只是gradient descent）都要求提前做好standardization 
{% endhint %}

## 实现Gradient Descent

```python
import numpy as np

#gradient descent
def gd(theta0, L, D, alpha, epsilon):
    theta = theta0
    it=0
    while True:
        it+=1
        l=L(theta)
        d=D(theta)
        if np.linalg.norm(d) < epsilon: #eucladian distance
            return theta
        theta-=alpha*d
        if (it%100)==0:
            print(f'{it}, theta: {theta}, l:{l}')

#Test Case L(x,y) = (x-3)**2 + (y-9)**2
ret = gd(
theta0=np.array([10.0, 20.0]),
L = lambda a:(a[0]-3)**2+(a[1]-9)**2,
D = lambda x: np.array([2*(x[0]-3), 2*(x[1]-9)]),
alpha = 1e-2,
epsilon = 1e-7
)

print(ret)
```

## Stochastic Gradient Descent  

从上面的过程可以看出，如果要实现一次gradient descent，需要load进去所有的x。如果我们只load部分的dataset，每次换不同的batch去算derivative，这个时候我们叫它out of core learning 在sklearn就是partial\_fit 



## Gradient Boosting 

### 从general的boosting vs bagging说起

![](../.gitbook/assets/image%20%283%29.png)

Bagging: independent classifier并行运算，一群high variance, low bias的model，用一些avg的技巧得到最后的结果（因为low bias，所以大家犯不同的错误取平均后应该得到一个真实值），比如weighted average, majority vote, normal average  最后的结果是reduce variance，variance变成了原来的1/N。

Boosting: sequential classifiers串行运算，可以理解成 精英教育，不停培养同一个model，让这个model的error最小。每一个model都在学习在此之前每一个model的mistake。Boosting的问题在于我们需要设置一个stop time or iteration time，因为它有overfitting的问题。

### Gradient Boosting

我们此前的想法都是given x，predict y。y本身有一个predicted value, 也有一个true value. 现在转变一下想法，我们还有一个叫做residual的参数 $$r_{j, i} $$ ， residual of $$f_{j}$$ on data i.

$$r_{0, i} \leftarrow y_{i}-f_{0}\left(x_{i}\right)$$ 

已知 $$x_{i}$$ ，预测每个$$r_{j, i} $$ , $$f 1 : x_{i} \rightarrow r_{0, i}$$ , 用预测结果改进 $$f_{0}$$.

$$
\begin{array}{l}{\text { }} \\ {\text{ 1: } f_{0} : x \rightarrow 0 } \\ {\text { 2: } f \leftarrow f_{0}} \\ {\text { 3: } i \leftarrow 1}  \\ {\text { 4: while } i<T \text { do }} \\ {\text { 5: } e_{i} \leftarrow y_{i}-f\left(x_{i}\right)} \\ {\text { 6: } \text { fit } f_{i} \text { using }\left\{<x_{i}, e_{i}>\right\}} \\ {\text { 7: } f(x) \leftarrow f(x)+f_{i}(x)} \\ {\text { 8: } \text { i } \leftarrow i+1} \\ {\text { 9: return } f}\end{array}
$$

其实，MSE是 $$L=\sum_{i}\left(y_{i}-\hat{y_{i}}\right)^{2}$$ , 而 $$\frac{d L}{d \hat{y_{i}}}=2\left(y_{i}-\hat{y_{i}}\right) = 2\times residual $$ 

$$
\begin{array}{l}{\text { 1: } f \leftarrow f_{0}} \\ {\text { 2: } i \leftarrow 1} \\ {\text { 3: while } i<T \text { do }} \\ {\text { 4: }  e_{i} \leftarrow \nabla L(f)} \\ {\text { 5: } \text { fit } f_{i} \text { using }\left\{<x_{i}, e_{i}>\right\}} \\ {\text { 6: } f(x) \leftarrow f(x)-\alpha_{i} \cdot f_{i}(x)} \\ {\text { 7: } i \leftarrow i+1} \\ {\text { 8: return } f}\end{array}
$$

### Adaboost 

用exponential loss的gradient boosting
