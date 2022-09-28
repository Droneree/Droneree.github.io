---
title: AI学习笔记--Numba+GPU加速
date: 2020-11-21 15:18:00
categories:
- 计算机科学
tags: 
- python
- 人工智能
- 机器学习
- GPU
---

使用Numba + GPU加速你的python程序。<!--more-->

<br/>

### Numba加速器

我把一段matlab代码重写迁移到python之后，运行速度慢了近100倍。经过代码分析，找到最耗时间的是python里的for循环。这可能与python语言本身的运行效率有关系，python是一门解释型的高级语言，在python被操作系统理解之前，要先转化为pyc字节码给python虚拟机，通过虚拟机与硬件操作系统交互。相比于一些更底层的语言，python的设计使得它更容易被程序员理解和使用，但trade-off是对机器来说更不容易理解，运行效率降低了。

Numba就是为了解决python计算效率低的痛点开发的JIT（just-in-time）编译器。简单来说，Numba把一部分python代码编译成机器语言，只需在第一次调用时编译一次，之后再调用都能以机器语言的速度运行。使用Numba之后，我的同一段的代码运算速度和matlab相近甚至更快了。

#### Numba安装 

用conda直接装

```python
$ conda install numba
```



#### 在CPU上使用Numba

使用Numba只需要把需要加速的那部分代码封装成function，在function前加一个`@jit`装饰器（decorator）就行了。是的，就这么简单。

```python
from numba import jit

@jit
def f(x, y):
    return x + y
```



需要注意的是，并不是所有的python对象Numba都支持，**尽量只加速最耗时的那部分代码**，这样既能避免代码中包含Numba不支持的python类型导致报错，又可以给代码瘦身提高编译速度。详见[官网的说明](https://numba.pydata.org/numba-doc/dev/user/troubleshoot.html#numba-troubleshooting)。

编译有两个模式可选：object mode和nopython mode。后者默认你对要加速的代码充分了解，不包含Numba不支持的python类型，直接编译成机器语言。否则会直接报错。而object mode则不会，遇到不能编的就退出JIT，按python原生的来。所以**想要发挥Numba最好的性能，尽量使用nopython mode**。

```python
@jit(nopython=True)
def f(x, y):
    return x + y
```



Cpu parallel



Vectorize

### Numba+GPU并行

#### GPU与并行



#### Numba使用GPU

##### - 简单并行



##### - 多维并行

并行适合处理互相独立的运算，如果是对同一个数的循环累加，用单个线程代替一次循环可能会出问题。比如，我要并行化的原代码是一个两层嵌套循环，其中外层相互独立，内层是对同一个数的累加

```python
def calx(x, ss2, model_size, a, zwrap, smooth_steps):
    for i in range(ss2, ss2 + model_size):
        x[i - ss2, 0] = a[0] * zwrap[i + 1 - (- smooth_steps), 0] / 2.00
        for j in range(- smooth_steps + 1, smooth_steps):
            x[i - ss2, 0] = x[i - ss2, 0] + a[j + smooth_steps] * zwrap[i + 1 - j, 0]
        x[i - ss2, 0] = x[i - ss2, 0] + a[2 * smooth_steps] * zwrap[i + 1 - smooth_steps, 0] / 2.00
    return x
```

假如我把两层循环都并行化，这里我令block的维度是2*smooth_steps，用block里的一个thread计算一次内循环；令grid的维度是model_size，用grid里的每个block计算一次外循环。

```python
@cuda.jit
def gpu_calx(x_device, ss2, model_size, a_device, zwrap_device, smooth_steps):
    a = a_device
    zwrap = zwrap_device
    i = cuda.blockIdx.x + ss2
    j = cuda.threadIdx.x - smooth_steps

    if j == (- smooth_steps):
        x_device[i - ss2, 0] = x_device[i - ss2, 0] + a[0] * zwrap[i + 1 - (- smooth_steps), 0] / 2.00
    elif j == smooth_steps:
        x_device[i - ss2, 0] = x_device[i - ss2, 0] + a[2 * smooth_steps] * zwrap[i + 1 - smooth_steps, 0] / 2.00
    else:
        x_device[i - ss2, 0] = x_device[i - ss2, 0] + a[j + smooth_steps] * zwrap[i + 1 - j, 0]
```

输出cpu的calx函数和gpu的gpu_calx函数计算结果的前三个数，发现两者不相同，有1e-2的差异

```python
calx_result[0:3]=
[[0.23222521]
 [0.42714764]
 [0.63248611]] 
gpu_result[0:3]=
[[0.17172628]
 [0.36771521]
 [0.5903021 ]]
```

或许可以这样来解释：用并行的线程取代累加循环时，每个线程并不关心其他线程是否计算完成。假设线程a从内存中取出x_device[i - ss2, 0]执行加法运算时，可能同时有线程b也在执行相同的操作，而两者取出的x_device[i - ss2, 0]是同一值。这样两个线程执行完加法返回值的时候，其实相当于只执行了一次加法运算，线程a或b的其中一个运算是无效的。所以造成了最后结果的差异。

于是，我只能放弃并行内层的累加循环

```python
@cuda.jit(device=True)
def gpu_calx(x_device, ss2, model_size, a_device, zwrap_device, smooth_steps):
    a = a_device
    zwrap = zwrap_device
    i = cuda.threadIdx.x + ss2

    x_device[i - ss2, 0] = a[0] * zwrap[i + 1 - (- smooth_steps), 0] / 2.00
    for j in range(- smooth_steps + 1, smooth_steps):
        x_device[i - ss2, 0] = x_device[i - ss2, 0] + a[j + smooth_steps] * zwrap[i + 1 - j, 0]
    x_device[i - ss2, 0] = x_device[i - ss2, 0] + a[2 * smooth_steps] * zwrap[i + 1 - smooth_steps, 0] / 2.00
```

然鹅，并行后的代码实际运行时间并不比加了`@jit`的cpu函数快。原因可能是计算本身的dimension不够大，gpu并行节省的时间还打不过gpu和cpu之间数据传输花费的时间。并不是gpu并行就一定比cpu快，这里有一个**数据计算量**和**调用线程、数据传输时间**之间的trade-off，cpu虽然不如gpu核数多，但往往优化做的很好，所以小数据量的计算在cpu上可能更好。

##### -复杂函数的并行

刚刚并行化的是最里层的一个简单函数，只涉及数组元素的算术运算。除此之外，我还想要把最外层的一个独立for循环并行化，这个for循环计算的是独立的40个集合预报成员。但问题是，这层for循环是相当外层的循环了，在它内部还有**多层函数嵌套**和一些**numpy方法**。

```python
@cuda.jit
def gpu_step_z(delta_t, zens, model_size, ss2, smooth_steps, a, model_number, K4, H, K, K2, sts2, coupling, space_time_scale, forcing):
    iens = cuda.blockIdx.x
    #i = cuda.threadIdx.x
    z = zens[iens, :].T
        
    z_save = z
    # gpu_comp_dt_L04 is the 1st layer of gpu_step_z, more layers of functions lies inside gpu_comp_dt_L04
    dz = gpu_comp_dt_L04(z, model_size, ss2, smooth_steps, a, model_number, K4, H, K, K2, sts2, coupling, space_time_scale, forcing)  # Compute the first intermediate step
    z1 = np.multiply(delta_t, dz)
    z = z_save + z1 / 2.0

    dz = gpu_comp_dt_L04(z, model_size, ss2, smooth_steps, a, model_number, K4, H, K, K2, sts2, coupling, space_time_scale, forcing)  # Compute the second intermediate step
    z2 = np.multiply(delta_t, dz)
    z = z_save + z2 / 2.0

    dz = gpu_comp_dt_L04(z, model_size, ss2, smooth_steps, a, model_number, K4, H, K, K2, sts2, coupling, space_time_scale, forcing)  # Compute the third intermediate step
    z3 = np.multiply(delta_t, dz)
    z = z_save + z3

    dz = gpu_comp_dt_L04(z, model_size, ss2, smooth_steps, a, model_number, K4, H, K, K2, sts2, coupling, space_time_scale, forcing)  # Compute fourth intermediate step
    z4 = np.multiply(delta_t, dz)

    dzt = z1 / 6.0 + z2 / 3.0 + z3 / 3.0 + z4 / 6.0
    z = z_save + dzt
    
    zens[iens, :] = z.T
```



为了解决多层函数嵌套的编译问题，我在每个内层函数前加上`@cuda.jit(device=True)`装饰，在[这篇文章](https://towardsdatascience.com/speed-up-your-algorithms-part-2-numba-293e554c5cc1)有提到device function：

> On the other hand, a `device function` can only be invoked from inside a device (by a kernel or another device function). The plus point is, you can return a value from a `device function`. So, you can use this return value of the function to compute something inside a `kernel function` or a `device function`.

但是在`@cuda.jit`装饰的函数内不能使用numpy方法是硬伤，[numba官方](https://numba.pydata.org/numba-doc/dev/cuda/cudapysupported.html)不支持在cuda并行编译的程序里使用numpy methods

>Unsupported numpy features:
>
>- array creation APIs.
>- array methods.
>- functions that returns a new array.

所以只能，放弃了！！！Orzzzzz...



##### - 内存

Cuda.to_device

Shared memory

<br/><br/><br/>

<small>*参考*</small>

<small>*[Java天堂-GPU加速的思想，图解，和经典案例](https://www.javatt.com/p/41851)*</small>

<small>*[github-harrism/numba_examples](https://github.com/harrism/numba_examples/blob/master/mandelbrot_numba.ipynb)*</small>

<small>*[towards data science-speed up your algorithms-part2 numba](https://towardsdatascience.com/speed-up-your-algorithms-part-2-numba-293e554c5cc1)*</small>

