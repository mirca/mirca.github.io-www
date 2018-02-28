---
layout: post
title: "L1 optimization using Majorization-Minimization in TensorFlow"
excerpt: ""
modified: 2018-02-27
tags: [optimization, majorization-minimization, tensorflow, python]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---

In this notebook I will show a simple example which demonstrates the power of a really clever optimization
technique, called Majorization-Minimization (MM). With MM, one is able to transform a non-differentiable
optimization problem (which often leads to unstable solutions) into a differentiable and therefore stable
optimization problem.

Note: for a theorectical and practical introduction of MM algorithms, see [Majorization-Minimization Algorithms in Signal Processing, Communications, and Machine Learning](http://ieeexplore.ieee.org/document/7547360/) by Y. Sun, P. Babu, and D. P. Palomar


```python
%matplotlib inline
%config IPython.matplotlib.backend = "retina"
import matplotlib.pyplot as plt
from matplotlib import rcParams
rcParams["figure.dpi"] = 150
rcParams["savefig.dpi"] = 150
```


```python
import numpy as np
np.random.seed(0)
import tensorflow as tf
```


## Toy data generation

To begin with, let's generate some toy data which is the result of
adding an independent, non-identically distributed, Poisson process:


```python
x = tf.placeholder(dtype=tf.float64)
y = tf.placeholder(dtype=tf.float64)
x_data = np.linspace(0, 5, 100)
```


```python
true_slope = 2
true_intercept = 10
y_data = np.random.poisson(x_data * true_slope + true_intercept)
```

Let's add some outliers as well:


```python
mask = np.arange(len(x_data)) % 10 == 0
y_data[mask] = 3 * y_data[mask]
```


```python
plt.step(np.arange(len(x_data)), y_data)
plt.ylabel("Data")
plt.xlabel("Bin number")
```




![png](../images/mm/output_11_1.png)


We want to estimate the slope and the intercept of the line that "best" describes
the mean of every Poisson random variable.


```python
def mean(theta):
    m, b = theta[0], theta[1]
    return x * m + b
```

# Direct Minimization of the L1-Norm

Let's assume that we don't know that the data is comes from a Poisson process,
and also that we are not interested in modeling the outliers, but only the linear
trend. Since our data have outliers, we choose to use the L1-Norm as our cost function:


```python
m = tf.Variable(np.random.normal(), dtype=tf.float64, name='m')
b = tf.Variable(np.random.normal(), dtype=tf.float64, name='b')
```


```python
# l1-norm
negloglike = tf.reduce_sum(tf.abs(mean((m, b)) - y))
grad = tf.gradients(negloglike, [m, b])
```


```python
sess = tf.Session()
sess.run(fetches=tf.global_variables_initializer())
optimizer = tf.contrib.opt.ScipyOptimizerInterface(loss=negloglike, method='BFGS')
optimizer.minimize(session=sess, feed_dict={x: x_data, y: y_data})
grad_opt = sess.run(grad, feed_dict={x: x_data, y: y_data})
l1_opt = [sess.run(m), sess.run(b)]
print(l1_opt, grad_opt)
```

    INFO:tensorflow:Optimization terminated with:
      Message: Desired error not necessarily achieved due to precision loss.
      Objective function value: 449.252608
      Number of iterations: 5
      Number of functions evaluations: 54
    [2.1050409239847778, 10.832882755026946] [2.020202020202019, 0.0]


**Message: Desired error not necessarily achieved due to precision loss.**


```python
plt.step(np.arange(len(x_data)), y_data)
plt.plot(np.arange(len(x_data)), sess.run(mean((m, b)), feed_dict={x:x_data}),
         label='best L1-norm line')
plt.legend()
plt.ylabel("Data")
plt.xlabel("Bin number")
```





![png](../images/mm/output_20_1.png)



```python
print("Value of the gradient at final parameter values: {}".format(grad_opt))
```

    Value of the gradient at final parameter values: [2.020202020202019, 0.0]


The gradient value at the solution is a good figure of merit to decide whether or not
the optimization has reached a local minima. However, the gradient value is not meaningful
in this case, since our cost function is non-differentiable in the first place.
In big data systems and high dimensional problems we can't afford looking at plots all the
time to see whether results are good or not.

# Majorization-Minimization

Majorization-minimization allow us to reframe a non-differentiable optimization problem into a differentiable one
by using a "surrogate" function to approximate the objective function.


```python
m = tf.Variable(np.abs(np.random.normal()), dtype=tf.float64, name='m')
b = tf.Variable(np.abs(np.random.normal()), dtype=tf.float64, name='b')
m_n, b_n = np.abs(np.random.normal()), np.abs(np.random.normal()) # initial guess
```

A possible surrogate function for the L1-norm is given as
\begin{align}
    g(\mathbf{r} | \mathbf{r}_n) = \dfrac{1}{2}\dfrac{||\mathbf{r}||^{2}_2}{||\mathbf{r}_n||_1} + \dfrac{1}{2}||\mathbf{r}_n||_1
\end{align}
which majorizes $||\mathbf{r}||_{1}$ at the point $\mathbf{r}_n$. See **Example 6** of [Majorization-Minimization Algorithms in Signal Processing, Communications, and Machine Learning](http://ieeexplore.ieee.org/document/7547360/) by Y. Sun, P. Babu, and D. P. Palomar.

Let's code up this surrogate function:


```python
r = y - mean((m, b)) # residual vector
abs_r_n = tf.abs(y - mean((m_n, b_n))) # abs value of the residual vector at the previous iteration
negloglike_surrogate = .5 * tf.reduce_sum(r * r / abs_r_n + abs_r_n) # surrogate function
grad = tf.gradients(negloglike_surrogate, [m, b]) # gradient of the surrogate function
```

Now we can iterate on minimizing $g(\mathbf{r} | \mathbf{r}_n)$ with respect to $(m, b)$ and updating
$(m_n, b_n)$ to the current optimal value.


```python
sess = tf.Session()
sess.run(fetches=tf.global_variables_initializer())
optimizer = tf.contrib.opt.ScipyOptimizerInterface(loss=negloglike_surrogate, var_list=[m, b],
                                                   method='BFGS')
i = 0
while i < 20:
    optimizer.minimize(session=sess, feed_dict={x: x_data, y: y_data})
    m_opt, b_opt = sess.run(m), sess.run(b)
    grad_opt = sess.run(grad, feed_dict={x: x_data, y: y_data})
    print("Solution: [{}, {}]\nPrevious solution: [{}, {}]\nGradient at solution: {}"
          .format(m_opt, b_opt, m_n, b_n, grad_opt))
    m_n, b_n = m_opt, b_opt
    # update surrogate function
    r = y - mean((m, b))
    abs_r_n = tf.abs(y - mean((m_n, b_n)))
    negloglike_surrogate = .5 * tf.reduce_sum(r * r / abs_r_n + abs_r_n)
    grad = tf.gradients(negloglike_surrogate, [m, b])
    optimizer = tf.contrib.opt.ScipyOptimizerInterface(loss=negloglike_surrogate, var_list=[m, b], method='BFGS')
    i+=1
```

    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 734.326830
      Number of iterations: 6
      Number of functions evaluations: 7
    Solution: [1.4496763621233597, 10.485754001760487]
    Previous solution: [2.16323594928069, 1.336527949436392]
    Gradient at solution: [2.2322854675849158e-10, 4.0236924903069848e-11]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 498.319702
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.4990495336473753, 10.570419248794593]
    Previous solution: [1.4496763621233597, 10.485754001760487]
    Gradient at solution: [8.6153306710912148e-14, 1.7763568394002505e-15]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 484.897254
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.6671918978343137, 10.626856860059574]
    Previous solution: [1.4990495336473753, 10.570419248794593]
    Gradient at solution: [1.1759482276829658e-12, -3.7214675785435247e-12]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 469.530050
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.8312888850551365, 10.691934629932456]
    Previous solution: [1.6671918978343137, 10.626856860059574]
    Gradient at solution: [-1.1368683772161603e-13, 1.2700951401711791e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 457.606473
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.8876297148592471, 10.86312811769006]
    Previous solution: [1.8312888850551365, 10.691934629932456]
    Gradient at solution: [1.5276668818842154e-13, 6.6613381477509392e-14]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 453.130012
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9013590751208442, 10.864915782095013]
    Previous solution: [1.8876297148592471, 10.86312811769006]
    Gradient at solution: [4.4408920985006262e-15, 1.2856382625159313e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 451.957296
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9377806534119828, 10.857428415441738]
    Previous solution: [1.9013590751208442, 10.864915782095013]
    Gradient at solution: [-2.7355895326763857e-13, -1.389999226830696e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 451.137567
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9365138465488718, 10.875750232024895]
    Previous solution: [1.9377806534119828, 10.857428415441738]
    Gradient at solution: [-1.1759482276829658e-12, -6.8389738316909643e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 450.821659
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9563832800773513, 10.858849543378488]
    Previous solution: [1.9365138465488718, 10.875750232024895]
    Gradient at solution: [3.2862601528904634e-13, 2.2559731860383181e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 450.245299
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9797591276137347, 10.84992912479451]
    Previous solution: [1.9563832800773513, 10.858849543378488]
    Gradient at solution: [2.5757174171303632e-13, 2.1671553440683056e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.814703
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9942019057908564, 10.85391260279468]
    Previous solution: [1.9797591276137347, 10.84992912479451]
    Gradient at solution: [2.7622348852673895e-13, 4.4630965589931293e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.596724
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9951841974758429, 10.863077741275276]
    Previous solution: [1.9942019057908564, 10.85391260279468]
    Gradient at solution: [-1.7408297026122455e-13, -2.8776980798284058e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.550010
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [1.9999276666195624, 10.856404281113537]
    Previous solution: [1.9951841974758429, 10.863077741275276]
    Gradient at solution: [4.4870773763250327e-12, 5.1070259132757201e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.512057
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [2.0073082918852077, 10.845620946027948]
    Previous solution: [1.9999276666195624, 10.856404281113537]
    Gradient at solution: [-2.5579538487363607e-13, -2.0019541580040823e-12]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.473694
      Number of iterations: 3
      Number of functions evaluations: 5
    Solution: [2.0167765090766285, 10.828229844857454]
    Previous solution: [2.0073082918852077, 10.845620946027948]
    Gradient at solution: [1.808331262509455e-12, 8.1801232454381534e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.430980
      Number of iterations: 3
      Number of functions evaluations: 5
    Solution: [2.0268943932192807, 10.81117726932769]
    Previous solution: [2.0167765090766285, 10.828229844857454]
    Gradient at solution: [-1.0338396805309458e-12, -4.7251091928046662e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.383032
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [2.0371619453822727, 10.7958722871897]
    Previous solution: [2.0268943932192807, 10.81117726932769]
    Gradient at solution: [-4.2810199829546036e-13, 3.390177027995378e-12]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.331424
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [2.047127451478164, 10.783079199451496]
    Previous solution: [2.0371619453822727, 10.7958722871897]
    Gradient at solution: [-6.1639582327188691e-13, -5.6576965334897977e-13]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.278922
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [2.0562542564766484, 10.773161806641019]
    Previous solution: [2.047127451478164, 10.783079199451496]
    Gradient at solution: [-5.666578317686799e-13, 1.1821654766208667e-12]
    INFO:tensorflow:Optimization terminated with:
      Message: Optimization terminated successfully.
      Objective function value: 449.230216
      Number of iterations: 2
      Number of functions evaluations: 5
    Solution: [2.063663005140765, 10.766644862881687]
    Previous solution: [2.0562542564766484, 10.773161806641019]
    Gradient at solution: [-1.723066134218243e-13, -1.7674750552032492e-13]



```python
sess.run(tf.reduce_sum(tf.abs(mean(l1_opt) - y)), feed_dict={x:x_data, y:y_data})
```




    449.25260792724191




```python
sess.run(tf.reduce_sum(tf.abs(mean((m_opt, b_opt)) - y)), feed_dict={x:x_data, y:y_data})
```




    449.20754720343052



Clear advantages:
1. Gradient is  close to zero on every iteration! => stable optimization!
2. Smaller cost function at reported "best" parameter values!

Remarks: Majorization-minimization algorithms enable us to approximate non-differentiable objective functions
    by differentiable ones which are easier to optimize. This fact gives rise to an iterative procedure to arrive
    at the local minima. Additionally, meaningful gradient information is available at every iteration.


```python
plt.step(np.arange(len(x_data)), y_data)
plt.plot(np.arange(len(x_data)), sess.run(mean(l1_opt), feed_dict={x:x_data}),
         label='direct L1-norm minimization')
plt.plot(np.arange(len(x_data)), sess.run(mean((m_opt, b_opt)), feed_dict={x:x_data}),
         label='majorization-minimization')
plt.plot(np.arange(len(x_data)), sess.run(mean((2., 10.)), feed_dict={x:x_data}),
         label='true line')
plt.legend()
plt.ylabel("Data")
plt.xlabel("Bin number")
```




![png](../images/mm/output_35_1.png)


The almost constant difference between the true line and the estimated ones is likely due to the fact that
we did not model the outliers properly (see Section 3 of [Hogg et al, Data Analysis Recipes](https://arxiv.org/pdf/1008.4686.pdf))
