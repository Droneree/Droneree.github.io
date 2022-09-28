---
title: Python环境（重）配置
date: 2020-05-28 02:00:00
categories:
- 计算机科学
tags: 
- python

---



今天使用conda更新python pkg的时候出现了[问题](https://github.com/conda/conda/issues/9038),导致jupyter notebook、之前写过的python程序，在import module的时候报出各种各样的错误，所以打算重装miniconda，重建一遍环境。<!--more-->

### 安装miniconda3

Miniconda是anaconda的精简版本，只包含python和conda，需要用到什么包时再安装。PC机上装miniconda非常轻便灵活。

重装前把之前的卸载干净，卸载方法见[miniconda官网](https://conda.io/projects/conda/en/latest/user-guide/install/macos.html)

到[miniconda官网](https://docs.conda.io/en/latest/miniconda.html)或国内的镜像网站（[清华镜像](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)）下载对应的Mac pkg，目的宗卷选择Macintosh HD

在终端输入`conda -V`检查是否安装成功。

<br/>

### 创建新的python虚拟环境

在终端输入`conda create -n condapy3 python`，创建一个名为condapy3的虚拟环境，miniconda3默认创建python3.8.3版本

进入虚拟环境`conda activate condapy3`

退出虚拟环境`conda deactivate`

查看虚拟环境列表`	conda env list`

删除虚拟环境`conda remove -n my_env --all`

<br/>

### 安装Jupiter notebook

选择在根环境下安装jupyter，安装指令`conda install jupyter notebook`

为了使Jupyter notebook可以使用不同的Conda虚拟环境，需要在notebook所在的环境中安装`nb_conda_kernels`包，并在其他需要用到的环境下安装`ipykernel`包。

```bash
conda install nb_conda_kernels
conda activate condapy3
conda install ipykernel
```

重启jupyter notebook，就可以在服务-改变服务里找到目标虚拟环境。

<br/>

### 安装各种包

这次打算把各种python pkg都装在虚拟环境里，逐渐习惯使用虚拟环境

安装列表：

- 科学计算（numpy, pandas, scipy）

- 绘图（matplotlib, seaborn）

- 网页爬取（requests, beautifulsoup4)

- 语词分析（jieba, wordcloud) 不能直接从conda官方库安装，安装方法见[CSDN](https://blog.csdn.net/zhaohaibo_/article/details/79253740)

`conda list`查看已安装的包列表

<br/>

### 使用镜像源(可选)

使用anaconda官网的源时经常出现下载慢、中断的情况，解决办法是为conda添加国内的镜像源，如添加清华的镜像源：

`conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/`

设置搜索时显示通道地址(生效)

`conda config --set show_channel_urls yes`

检查是否添加成功

`conda config --show-sources`

移除源

`conda config --remove channels xxxx`

<br/>

### 在Pycharm中使用conda虚拟环境

打开Pycharm的preference-project intepreter-conda env，设置python解释器为

![](/images/image-20200528015900321.png)

<br/>

### 在python中使用Matlab

[Matlab官网](https://ww2.mathworks.cn/help/matlab/matlab_external/install-the-matlab-engine-for-python.html)写的很详细，还有[知乎](https://zhuanlan.zhihu.com/p/47655091)这篇。

<br/>

<br/>