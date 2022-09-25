## 简介

### 什么是svFSI

svFSI是SimVascular项目旗下的开源软件包，提供了传热、流体、固体、流固耦合甚至电生理学（心电）的求解模块。此外，SimVascular项目的SimVascular程序主题不在本文介绍范围之内。

### 为什么使用svFSI

svFSI目前组内比较适合使用svFSI提供流固耦合模块（强耦合）以及0D-3D耦合功能（支持开循环、闭循环），使用难点仅仅在使用者对linux环境的不熟悉与编写配置文件。

### 使用前提

#### 简单了解linux系统（仅使用可略过）

可以把操作系统视为一个大程序，里面运行的不同程序负责不同的功能，windows程序也可以等效看待

例：

- linux的桌面环境一般有gnome或者xfce
- linux与操作系统交互的shell，windows下有cmd、powershell等

不同的linux开源系统版本，有centos、ubuntu、debian、openSUSE、archlinux等，推荐ubuntu20.04，简单常用

#### 使用linux系统

下载一个ubuntu20.04的镜像包（ISO文件），安装好vmware虚拟机，将ubuntu安装到虚拟机内，建议设置密码为简单密码。也可以采用物理机直接安装ubuntu系统，这里不介绍。

#### 简单了解命令行（仅使用可略过）

命令行命令就是通过shell执行与系统交互的命令。涉及到系统，因此普通权限和管理员权限的区别是个很重要的知识点，如果你采用管理员权限就有改动系统文件的可能，所以对于linux系统命令以及文件的权限问题需要认真学习。下面这个命令就是一个危险的命令

```bash
sudo rm -rf /*
```

总之，采用管理员权限以及涉及删改的命令都应该谨慎。

需要熟悉的命令有

> sudo cd ls mkdir chmod echo source alias rm kill top export git make cmake apt ssh gedit vi/vim 等

linux命令可以查看https://www.runoob.com/linux/linux-command-manual.html

其中apt是ubuntu下的包管理软件，很方便安装软件以及解决软件间的依赖问题

#### 简单了解环境变量（仅使用可略过）

PATH等，这个windows下同理

#### 编译安装svFSI前的要求（仅使用可略过）

svFSI的依赖包括（由管理同学配置好）

- C, Fortran编译器 (GNU下的gcc/gfortran, intel下的icc/ifort,..)，建议简单化直接安装sudo apt install gcc g++ gfortran
- MPI (mpich, openmpi, intel MPI,..)
- CMake
- LAPACK and BLAS（netlib的LAPCK/BLAS，openblas，AMD的blis/flame，intel的MKL）
- Trilinos (可选，个人测试采用0D边界条件调用trilinos进行计算容易不收敛，采用svFSI内置的求解器更好，并且trilinos的编译比较麻烦，这里略过)

#### 使用svFSI的要求

不使用0D3D耦合模块可以不用熟悉编程语言，使用0D3D耦合模块应该熟悉fortran语言。

### 下载svFSI（仅使用可略过）

svFSI的github网址

https://github.com/SimVascular/svFSI

svFSI案例的github网址

https://github.com/SimVascular/svFSI-Tests

下载svFSI源代码：

```bash
git clone https://github.com/SimVascular/svFSI.git
```

下载svFSI的案例：

```bash
git clone https://github.com/SimVascular/svFSI-Tests.git
```

> 涉及到github的地方都可以通过替换"github.com"为“hub.fastgit.xyz”来加快github的访问以及下载程序包，否则可以通过科学上网加快github访问

### 安装svFSI（仅使用可略过）

使用cmake进行编译安装，执行

```
cd svFSI
mkdir build
cd build
cmake ..
make
```

第一行是进入“git clone”下载的svFSI源代码目录下，第二行新建一个叫“build”的目录，然后第三行进入该目录，然后执行cmake构建编译项目，“..”指的是上级目录，因为“CmakeLists.txt”文件在上级目录下。

编译无错误完成后，编译好的程序在如下目录下

```
/home/chen/svFSI/build/svFSI-build/bin
```

需要将该目录放入PATH环境变量：

```
export PATH="/home/chen/svFSI/build/svFSI-build/bin:${PATH}"
```

或者用alias将该程序取一个别名

```
alias svFSI=/home/chen/svFSI/build/svFSI-build/bin/svFSI
```

### 使用svFSI

```bash
mpirun -np N svFSI svFSI_master.inp
```

mpirun是mpi并行程序，-np是指定进程数的前置标记，N是N个进程来运行程序，svFSI是编译好的svFSI程序名，svFSI_master.inp是配置文件，svFSI程序读取该配置文件来运行计算。

#### 运行官方算例

进入到下载的svFSI-Tests下，选择一个算例，这里选择

```
cd svFSI-Tests/04-fluid/01-pipe3D_RCR/
```

执行

```
mpirun -np 4 svFSI svFSI.inp
```

可以在`4-procs`目录下看到新计算的结果

##### 后处理

采用paraview或者tecplot进行后处理均可，不在本文范围内。

##### 前处理

模型文件的准备不在本文范围，这里只讨论怎么生成`svFSI`可以使用的网格文件

`svFSI`可以使用三角形网格和四边形网格，并且支持二阶网格（有限元法只需要这两种网格）。这里给出几个可行的流程：

1. `fluent`生成网格导入`icem`导出为`Gambit`格式通过`mesh_converter`转化为`svFSI`需要的`vtk`格式
2. `icem`生成网格导出为`Gambit`格式通过`mesh_converter`转化为`svFSI`需要的`vtk`格式
3. `pointwise`生成网格导出为`Gambit`格式通过`mesh_converter`转化为`svFSI`需要的`vtk`格式
4. `gmsh`生成网格通过`mesh_converter`转化为svFSI需要的vtk格式
5. 采用`SimVascular`程序进行前处理

其中`mesh_converter`为`SimVascular`官方提供的网格转换小程序，需要执行`make`编译，通过如下命令下载

```bash
git clone https://github.com/SimVascular/svFSI-Tools.git
```

#### 配置文件详解

[配置文件详解](配置文件详解.md)

#### 流固耦合模块

[流固耦合模块](流固耦合模块/流固耦合模块.md)

#### 0D与3D耦合模块

[0D与3D耦合模块](0D与3D耦合模块.md)

