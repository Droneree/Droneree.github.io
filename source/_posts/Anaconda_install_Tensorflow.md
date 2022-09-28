---
title: AI学习笔记--安装Tensorflow框架
date: 2020-09-10 21:27:00
categories:
- 计算机科学
tags: 
- python
- 人工智能
- 机器学习
- 深度学习
---

Tensorflow是Google开发的开源机器学习框架，基于Tensorflow的Keras是一个更高级的API，Keras的易用性使得它对新手非常友好。<!--more-->另一个流行的深度学习框架是Facebook开发的Pytorch。关于深度学习框架的选择可参考这篇[Keras vs PyTorch：谁是「第一」深度学习框架？](https://www.jiqizhixin.com/articles/keras-or-pytorch)

<br/>

### Anaconda安装Tensorflow

[Tensorflow官网](https://www.tensorflow.org/install?hl=zh-cn)给出的安装方式为pip安装，但其实用Anaconda也可以安装，conda强大的虚拟环境管理功能允许不同的项目、不同的框架独立区隔，便于管理。Anaconda的安装指导见[官网文档](https://docs.conda.io/projects/conda/en/latest/user-guide/install/download.html)。

#### CPU版本

CPU版本是在我的macOS系统上安装的。首先，为Tensorflow新建一个专门的python环境，取名为”tf“

`conda create -n tf python=3.6`

注意这里指定的python版本为3.6，这是因为**目前Tensorflow仅支持python 3.5-3.7的版本**（不装3.7的原因是anaconda官网源安装python3.7太太太慢了，但清华的镜像源最新只有python3.6的版本）。

听说conda已经支持Tensorflow2.0了，但试了一下好像还是不行，直接conda install的话安装的是Tensorflow1.1.0。

`conda install tensorflow`

[Stack overflow上给出了一种解决办法](https://stackoverflow.com/questions/55392100/install-tensorflow-2-0-in-conda-enviroment)，先用conda安装，再用pip更新Tensorflow至2.0。

`pip install --upgrade tensorflow==2.0.0`

装好后可以`import tensorflow as tf`试一下，用`tf.__version__`查看版本。

<br/>

#### GPU版本

GPU版本是在服务器上尝试安装的，直接conda安装的2.1.0版本，竟然异常的顺利。

`conda create -n tf python=3.7`

`conda install tensorflow-gpu=2.1.0`

Tensorflow在GPU上使用还需要一些支持的驱动程序和加速库，包括CUDA 和 cuDNN等。Tensorflow官网建议使用[Docker容器](https://www.tensorflow.org/install/docker?hl=zh-cn)为Tensorflow提供GPU支持和虚拟环境。一些论坛上还有建议[手动为Tensorflow-GPU配置支持](https://zhuanlan.zhihu.com/p/60924644)

的。但是，conda其实已经为我们做了这些工作，conda install时已经包含了cudatoolkit、cudnn这两个包。也是异常的顺利。

 `tf.__version__ ` 检查Tensorflow版本

 `tf.test.is_gpu_available()`  检查Tensorflow-GPU是否可用

检查的结果是可以的，但用MXnet官网的方法pip安装了MXnet之后gpu就都不能用了：

![image-20200923171733370](/images/image-20200923171733370.png)

报错显示nvidia的gpu driver不在运行。

`nvidia-smi`检查gpu driver的状态

```
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
```

似乎服务器上没有装gpu driver，不知道是环境变量被修改了还是什么。关于这个问题Stack overflow有[解答](https://stackoverflow.com/questions/56470424/nvcc-missing-when-installing-cudatoolkit)，conda安装的cuda是不包含gpu driver的，下面要自己装一下gpu driver，参考的是知乎的[这篇](https://zhuanlan.zhihu.com/p/59618999)。

检查GPU及推荐的服务器：` ubuntu-drivers devices`，显示

```
== /sys/devices/pci0000:53/0000:53:00.0/0000:54:00.0/0000:55:08.0/0000:5a:00.0 ==
modalias : pci:v000010DEd00001E04sv000010DEsd000012FAbc03sc00i00
vendor  : NVIDIA Corporation
model  : TU102 [GeForce RTX 2080 Ti]
driver  : nvidia-driver-435 - distro non-free
driver  : nvidia-driver-440-server - distro non-free
driver  : nvidia-driver-450 - distro non-free recommended
driver  : nvidia-driver-450-server - distro non-free
driver  : nvidia-driver-418-server - distro non-free
driver  : xserver-xorg-video-nouveau - distro free builtin
```

看到这里推荐的是nvidia-driver-450，使用下面的指令自动安装

```
sudo ubuntu-drivers autoinstall
```

安装后重启系统（`sudo shutdown -r now `），输入`nvidia-smi`检查是否安装成功，若成功会显示GPU的信息。`dpkg -l | grep nvidia`显示相关nvidia相关包的信息。

再次测试tensorflow是否可用GPU，这次显示

<img src="/images/image-20201012154534615.png" alt="image-20201012154534615" style="zoom:80%;" />

若系统上曾经装过gpu driver，没有卸载干净之前的就装新的会很mess，nvida官方也建议卸载干净旧版再装新版，相关问题看[这篇](https://forums.developer.nvidia.com/t/nvidia-smi-has-failed-because-it-couldnt-communicate-with-the-nvidia-driver-ubuntu-16-04/48635)。因为之前安装的nvidia driver导致无法安装新版本的情况，参考[这篇](https://unix.stackexchange.com/questions/620141/cant-install-nvidia-driver-455-upgrade-from-450-version)。

---

重装过nivdia gpu driver之后，今天发现又`nvidia-smi`又找不到driver了，找了半天原因，发现不知道什么时候ubuntu的内核偷偷更新过了……以前是5.4.0-48-generic，现在是5.4.0-53-generic。内核更新后就和原来的driver不匹配了，参考[这篇](https://codeleading.com/article/2604429676/)，和[这篇](https://www.pianshen.com/article/7487258025/)。

`uname -r`查看当前内核版本

`dkms status`    查看匹配的ubuntu内核和nvidia驱动版本

`grep menuentry /boot/grub/grub.cfg`查看系统有哪些内核可用

![ubuntu_core](/images/ubuntu_core.png)

找到旧版的5.4.0-48-generic内核是'Advanced options for Ubuntu'下的第3个menuentry，所以`sudo vi /etc/default/grub`，找到

```
GRUB_DEFAULT=0
```

改为

```
GRUB_DEFAULT="1 >2"
```

2是第3个menuentry的编号。

更新一下

```
sudo update-grub
```

之后`sudo reboot`重启，`nvidia-smi`就可以用啦。

<br/>

<br/>

<br/>