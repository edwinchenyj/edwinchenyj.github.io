---
title:  "An Introduction to Stiff ODEs with Python"
author: Ed
last_modified_at: 2021-07-24
categories:
        - Scientific Computing
tags:
        - Python
        - Differential Equations
toc: true
toc_sticky: true
excerpt: "In this post, I hope to make the concept of stiffness in ODEs a bit easier to understand by showing a few Python examples."
---


**Notice:** In my previous [post on stiff ODEs](https://edwinchenyj.github.io/scientific%20computing/stiff/), I demonstrated how different ODE solvers in Matlab perform with a few examples. The [live script](https://github.com/edwinchenyj/scientific-computing-notes/tree/main/stiff_ode) for the post is also provided for educational purpose. In this post, I will do the same in Python. You can find the `ipynb` file in the same [repository](https://github.com/edwinchenyj/scientific-computing-notes/tree/main/stiff_ode).
{: .notice}


**Info:** The solver interfaces provided by Matlab and SciPy are not exactly the same (SciPy uses `rtol*abs(y)+atol` while Matlab uses `max(rtol*abs(y),atol)`), so we will use different solvers and tolerances. . If you are interested, please refered to the [SciPy document](https://docs.scipy.org/doc/scipy/reference/generated/scipy.integrate.solve_ivp.html) and the [Matlab document](https://www.mathworks.com/help/matlab/ref/odeset.html).
{: .notice--info}

---

It’s well-known that stiff ODEs are hard to solve. Many books are written on this topic, and SciPy even provides solvers specialized for stiff ODEs. It is easy to find resources, including the wikipedia entry, with technical and detailed explanations. For example, one of the common descriptions for stiff ODEs may read:

*An ODE is stiff if absolute stability requirement is much more restrictive than accuracy requirement, and we need to be careful when we choose our ODE solver.*

However, it’s fairly abstract and hard to understand for people new to scientific computing. In this post, I hope to make the concept of stiffness in ODEs easier to understand by showing a few examples. Let’s start with a simple (non-stiff) example, and compare it with some stiff examples later on.

## Example 1

Let's consider a non-stiff ODE



<img src="https://latex.codecogs.com/gif.latex?\textrm{y'}=\textrm{Ay}"/>



where <img src="https://latex.codecogs.com/gif.latex?\inline&space;A=\lambda"/>




In the first case we have <img src="https://latex.codecogs.com/gif.latex?\inline&space;\lambda&space;\;=-0\ldotp&space;1"/>. The solution is 




<img src="https://latex.codecogs.com/gif.latex?\inline&space;y\left(t\right)=e^{-0\ldotp&space;1t}&space;y\left(0\right)"/>,




meaning we have a exponential decaying function.


```python
import numpy as np
from scipy.integrate import solve_ivp, RK45
import matplotlib.pyplot as plt

mlambda = -1e-1
A = np.matrix([mlambda])

F = lambda t,u: A.dot(u.flatten())

# initial condition
u0 = np.ones(A.shape[0])

# time points
t = [0,10]
```

### RK45
We can look at the solution from `RK45:`


```python
# solve ODE
sol = solve_ivp(F,t,u0,'RK45',rtol=1e-7,atol=1e-7)

# # # # plot results
plt.plot(sol.t,sol.y[0], 'o-')
plt.xlabel('t')
plt.ylabel('y')
plt.show()
```


    
![png](/assets/stiff_ode_images/output_3_0.png)
    


As we can see, it `RK45` gives us a decaying function. In this interval, `RK45` used


```python
sol.t.size
```




    9



steps to achieve the specified tolerance.

## Example 2

Let's consider the same equation



<img src="https://latex.codecogs.com/gif.latex?\textrm{y'}=\textrm{Ay}"/>



but now <img src="https://latex.codecogs.com/gif.latex?\inline&space;A=\left\lbrack&space;\begin{array}{cc}
\lambda_1&space;&space;&&space;\\
&space;&&space;\lambda_2&space;
\end{array}\right\rbrack"/>




In the first case we have <img src="https://latex.codecogs.com/gif.latex?\inline&space;\lambda_1&space;=-0\ldotp&space;1,\;\lambda_2&space;={10}^3&space;\lambda_1"/>. This means we have two decoupled equations. The solution is 




<img src="https://latex.codecogs.com/gif.latex?\inline&space;y\left(t\right)=\left\lbrack&space;\begin{array}{c}
y_1&space;\left(t\right)\\
y_2&space;\left(t\right)
\end{array}\right\rbrack&space;=\left\lbrack&space;\begin{array}{c}
e^{-0\ldotp&space;1t}&space;y_1&space;\left(0\right)\\
e^{-100t}&space;y_2&space;\left(0\right)
\end{array}\right\rbrack"/>,


meaning we have two exponential decaying functions.



```python
mlambda1 = -1e-1
mlambda2 = 1e3*mlambda1
A = np.diag([mlambda1, mlambda2])

F = lambda t,u: A.dot(u) 


# initial condition
u0 = np.ones(A.shape[0])

# time points
t = [0,10]
```

### RK45
We can use `RK45` to solve it in the same fashion:


```python
# solve ODE
sol = solve_ivp(F,t,u0,'RK45',rtol=1e-7,atol=1e-7)

# # plot results
plt.plot(sol.t,sol.y[0],'o-',sol.t,sol.y[1],'o-')
plt.xlabel('t')
plt.ylabel('y')
plt.show()
```


    
![png](/assets/stiff_ode_images/output_10_0.png)
    


This time we get 2 decaying functions, and <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_2"/> decays much faster then <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_1"/>. In this same interval, `RK45` used 


```python
sol.t.size
```




    333



steps to achieve the desired error tolerance. In this example, <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_1"/> is exactly the same as the solution in Example 1, but it take much longer to calculate. One may think the step size of `RK45` is limited by the *accuracy requirement* due to the addition of <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_2"/>. However, this is clearly not the case since <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_2"/> is almost identically <img src="https://latex.codecogs.com/gif.latex?\inline&space;0"/> on the entire interval. What is happening here is that, the step size of `RK45` is limited by the *stability requirement* of <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_2"/>, and we call the ODE in Example 2 ***stiff***.




SciPy provides specialized ODE solvers for stiff ODEs. Let's look at `BDF` and `Radau`

### BDF


```python
# solve ODE
sol = solve_ivp(F,t,u0,'BDF',rtol=1e-7,atol=1e-7)

# # plot results
plt.plot(sol.t,sol.y[0],'o-',sol.t,sol.y[1],'o-')
plt.xlabel('t')
plt.ylabel('y')
plt.show()
```


    
![png](/assets/stiff_ode_images/output_15_0.png)
    


`BDF` takes 


```python
sol.t.size
```




    134



### Radau


```python
# solve ODE
sol = solve_ivp(F,t,u0,'Radau',rtol=1e-7,atol=1e-7)

# # plot results
plt.plot(sol.t,sol.y[0],'o-',sol.t,sol.y[1],'o-')
plt.xlabel('t')
plt.ylabel('y')
plt.show()
```


    
![png](/assets/stiff_ode_images/output_19_0.png)
    


and `Radau` takes


```python
sol.t.size
```




    85



steps. Apparently, `BDF` and `Radau` is significantly more efficient than `RK45` for this example. From the figure above, we can also see that `BDF` and `Radau` stratigically used shorter step size when <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_2"/> is decaying fast, and larger step size when <img src="https://latex.codecogs.com/gif.latex?\inline&space;y_2"/> flattens out.

At this point you may think that if you don't know whether an ODE is stiff or not, it is always better to use `BDF` and `Radau`. However, this is not the case, as we will show in the next example.

# Ocsillatory ODE

## Example 3


Let's look at an oscillatory ODE



<img src="https://latex.codecogs.com/gif.latex?\textrm{y'}=Ly"/>



and <img src="https://latex.codecogs.com/gif.latex?\inline&space;L=\left\lbrack&space;\begin{array}{cc}
&space;&&space;\lambda&space;\;\\
-\lambda&space;\;&space;&&space;
\end{array}\right\rbrack&space;,\;\lambda&space;=-0\ldotp&space;1"/>

The eigenpairs of <img src="https://latex.codecogs.com/gif.latex?\inline&space;L"/> are



<img src="https://latex.codecogs.com/gif.latex?\left(\pm&space;\lambda&space;i\;,\left\lbrack&space;\begin{array}{c}&space;1\\&space;\pm&space;i&space;\end{array}\right\rbrack&space;\right)"/>  



The solution is oscillatory because the eigenvalues are imaginary.


```python
mlambda = -1e-1
L = np.matrix([[ 0, mlambda],[ -mlambda, 0]])

F = lambda t,u: L.dot(u.flatten())


# initial condition
u0 = np.ones(L.shape[0])

# time points
t = [0,50]
```

### RK45
Let's look at the solution from `RK45`


```python
# solve ODE
sol = solve_ivp(F,t,u0,'RK45',rtol=1e-7,atol=1e-7)

# # plot results
plt.plot(sol.t,sol.y[0],'o-',sol.t,sol.y[1],'o-')
plt.xlabel('t')
plt.ylabel('y')
plt.show()
```


    
![png](/assets/stiff_ode_images/output_27_0.png)
    



```python
sol.t.size
```




    34




As expected, we see two slow oscillatory functions.


## Example 4


Now let's look at a stiff oscillatory ODE



<img src="https://latex.codecogs.com/gif.latex?\textrm{y'}=Ly"/>



and <img src="https://latex.codecogs.com/gif.latex?\inline&space;L=\left\lbrack&space;\begin{array}{cc}
&space;&&space;A\\
-A&space;&&space;
\end{array}\right\rbrack&space;,\;A=\left\lbrack&space;\begin{array}{cc}
\lambda_1&space;&space;&&space;\\
&space;&&space;\lambda_2&space;
\end{array}\right\rbrack"/>




The eigenpairs of <img src="https://latex.codecogs.com/gif.latex?\inline&space;L"/> are




<img src="https://latex.codecogs.com/gif.latex?\inline&space;\left(\pm&space;\lambda_1&space;i,\left\lbrack&space;\begin{array}{c}
1\\
0\\
\pm&space;\;i\\
0
\end{array}\right\rbrack&space;\right)"/>, and <img src="https://latex.codecogs.com/gif.latex?\inline&space;\left(\pm&space;\lambda_2&space;i,\left\lbrack&space;\begin{array}{c}
0\\
1\\
0\\
\pm&space;i
\end{array}\right\rbrack&space;\right)"/> 




Similar to before, we set <img src="https://latex.codecogs.com/gif.latex?\inline&space;\lambda_1&space;=0\ldotp&space;1,\lambda_2&space;=100\lambda_1"/>. Now we have both fast and slow oscillatory functions in our solution.





```python
mlambda1 = -1e-1
mlambda2 = 1e2*mlambda1
A = np.diag([mlambda1, mlambda2])
L = np.block([[np.zeros([2,2]),A],[-A,np.zeros([2,2])]])

F = lambda t,u: L.dot(u.flatten()) 


# initial condition
u0 = np.ones(L.shape[0])


# time points
t = [0,50]
```

### RK45


```python
# solve ODE
sol = solve_ivp(F,t,u0,'RK45',rtol = 1e-6, atol = 1e-6)

# # plot results
plt.plot(sol.t,sol.y[0],'o-',sol.t,sol.y[1],'-',sol.t,sol.y[2],'o-',sol.t,sol.y[3],'-')
plt.xlabel('t')
plt.ylabel('y')
```




    Text(0, 0.5, 'y')




    
![png](/assets/stiff_ode_images/output_33_1.png)
    


In the plots we can see both slow and highly oscillatory parts. Again, similar to the decaying case, now `RK45` is taking shorter step sizes because of the the fast oscillating part, even though  and  could have taken much shorter time steps like the example above. In this case,


```python
sol.t.size
```




    1772



This time `BDF` and `Radau` are not that efficient either.

### BDF


```python
# solve ODE
sol = solve_ivp(F,t,u0,'BDF',rtol = 1e-6, atol = 1e-6)

# # plot results
plt.plot(sol.t,sol.y[0],'o-',sol.t,sol.y[1],'-',sol.t,sol.y[2],'o-',sol.t,sol.y[3],'-')
plt.xlabel('t')
plt.ylabel('y')
```




    Text(0, 0.5, 'y')




    
![png](/assets/stiff_ode_images/output_37_1.png)
    



```python
sol.t.size
```




    3853



### Radau


```python
# solve ODE
sol = solve_ivp(F,t,u0,'Radau',rtol = 1e-6, atol = 1e-6)

# # plot results
plt.plot(sol.t,sol.y[0],'o-',sol.t,sol.y[1],'-',sol.t,sol.y[2],'o-',sol.t,sol.y[3],'-')
plt.xlabel('t')
plt.ylabel('y')
plt.show()
```


    
![png](/assets/stiff_ode_images/output_40_0.png)
    



```python
sol.t.size
```




    3788



Notice highly oscillatory and stiff ODEs are generally hard to solve. All the solvers, `RK45`, `BDF`, and `Radau` take very short steps and become very expensive. 

This blog post is published at [https://edwinchenyj.github.io.](https://edwinchenyj.github.io.) The pdf version and the source code are available at [https://github.com/edwinchenyj/scientific-computing-notes](https://github.com/edwinchenyj/scientific-computing-notes).
