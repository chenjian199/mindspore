# Install MindSpore GPU with CUDA 12.0 in Docker

这是一个在 Docker 里安装 MindSpore GPU（CUDA 12.0 版本），并使用 COCO2017 数据集训练 YOLOv3 的教程。
*首先确保你宿主机可以连接外网。我们在宿主机上制作半成品镜像，上传到远端服务器上制作成品和训练。这样做是因为服务器上无连接外网权限*


1.进入下面这个网址：
  [镜像仓库](https://hub.docker.com/)
  
2.在搜索框里输入nvidia/cuda，然后在tags搜索栏里输入cuda12.0,选择你需要的版本，这里我们选择：
  12.0.0-cudnn8-devel-ubuntu20.04
  
3.下载镜像到本地：
  ```bash
  docker pull nvidia/cuda:12.0.0-cudnn8-devel-ubuntu20.04
  ```

4.启动镜像：
  ```bash
  sudo docker run -it --name CJ_MAKE_MINDSPORE --hostname cjmakemindspore --network host --privileged  nvidia/cuda:12.0.0-cudnn8-devel-ubuntu20.04 /bin/bash
  ```

5.进入容器后，输入下列命令，安装基础编辑器：
  ```bash
  apt update
  apt install vim -y
  vim ~/.bashrc
  ```
  在文件末尾输入：
  ```vim
  # 自定义 PS1：绿色用户主机名 + 蓝色目录
  PS1='\[\e[0;32m\]\u@\h\[\e[0m\]:\[\e[0;34m\]\w\[\e[0m\]\$ '
  ```
  然后保存退出，运行：
  ```bash
  source ~/.bashrc
  apt install nano -y
  ```
***

6.安装cuda依赖：
  ```bash
  apt install linux-headers-$(uname -r) gcc-9
  ```

7.安装cuda11.6:
  ```bash
  apt install wget -y
  ```
  进入下面这个网址：
    [nvidia官网](https://developer.nvidia.com/cuda-11-6-0-download-archive)
  选择合适的版本，这里我们选择ubuntu20.04版本，通过wget安装：
  ```bash
    wget -c https://developer.download.nvidia.com/compute/cuda/11.6.0/local_installers/cuda_11.6.0_510.39.01_linux.run
    bash cuda_11.6.0_510.39.01_linux.run
  ```
  安装时如缺少库，需手动安装，例如：
  ```bash
  apt install libxml2
  ```
  其中这个库的选项地区和时区自选，例如：Current default time zone: 'Asia/Shanghai'。
  安装完依赖库，然后再次运行安装命令即可。
  安装时只需选择安装 Toolkit:  Installed in /usr/local/cuda-11.6/
  
8.安装cudnn8.5.0:
  进入下面网址，选择cudnn8.5.0 Linux x84_64.tar下载
  [nvida官网](https://developer.nvidia.com/rdp/cudnn-archive)
  下载到本地后导入到容器中：
  ```bash
  sudo docker cp cudnn-linux-x86_64-8.5.0.96_cuda11-archive.tar.xz CJ_MAKE_MINDSPORE:/cudnn-linux-x86_64-8.5.0.96_cuda11-archive.tar.xz
  ```
  然后解压安装：
  ```bash
  tar -xvf 'cudnn-linux-x86_64-8.5.0.96_cuda11-archive.tar.xz'
  cp cudnn-linux-x86_64-8.5.0.96_cuda11-archive/include/cudnn*.h /usr/local/cuda-11.6/include
  cp cudnn-linux-x86_64-8.5.0.96_cuda11-archive/lib/libcudnn* /usr/local/cuda-11.6/lib64
  chmod a+r /usr/local/cuda-11.6/include/cudnn*.h /usr/local/cuda-11.6/lib64/libcudnn*
  ```

9.安装python3.9
  ```bash
apt-get update && \
apt-get install -y software-properties-common wget curl gnupg lsb-release && \
add-apt-repository ppa:deadsnakes/ppa -y && \
apt-get update && \
apt-get install -y python3.9 python3.9-dev python3.9-distutils python3-pip && \
update-alternatives --install /usr/bin/python python /usr/bin/python3.9 100 && \
python -m ensurepip --upgrade && \
python -m pip install --upgrade pip -i https://repo.huaweicloud.com/repository/pypi/simple   && \
update-alternatives --install /usr/bin/pip pip /usr/local/bin/pip3.9 100 && \
pip config set global.index-url https://repo.huaweicloud.com/repository/pypi/simple
 ```
  测试是否成功：
  ```bash
python --version     # 应输出 Python 3.9.x
pip --version        # 应输出 pip 相关版本
  ```
10.安装gcc-9
```bash
apt-get install gcc-9 -y
```
11.下载安装TensorRT
  进入这个网站：
  [nvidia官网](https://developer.nvidia.com/nvidia-tensorrt-8x-download)
  选择下载：适用于 Linux x86_64 和 CUDA 11.0、11.1、11.2、11.3、11.4、11.5、11.6 和 11.7 TAR 包的 TensorRT 8.4 GA
  ```bash
sudo docker cp TensorRT-8.4.1.5.Linux.x86_64-gnu.cuda-11.6.cudnn8.4.tar.gz CJ_MAKE_MINDSPORE:/TensorRT-8.4.1.5.tar.gz
  ```
  ```bash
tar xzf TensorRT-8.4.1.5.tar.gz
cd TensorRT-8.4.1.5
echo -e "export TENSORRT_HOME=$PWD" >> ~/.bashrc
echo -e "export LD_LIBRARY_PATH=\$TENSORRT_HOME/lib:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc
cd -```bash
  ```
12.保存为镜像，上传到远程服务器:
```bash
sudo docker commit CJ_MAKE_MINDSPORE cj_make_mindspore:temp
sudo docker save cj_make_mindspore | gzip -9 > cj_make_mindspore.tar.gz
```
13.在服务器上加载容器：
```bash
docker load -i cj_make_mindspore.tar.gz
docker run -it --name CJ_MINDSPORE --hostname cjmindspore --gpus all --network host --shm-size=8g -v /mnt/CJ_Workspace:/mnt/CJ_Workspace -v /mnt/Datasets:/mnt/Datasets -w /mnt/CJ_Workspace cj_make_mindspore /bin/bash
```
14.安装mindspore:
```bash
pip install https://ms-release.obs.cn-north-4.myhuaweicloud.com/2.2.14/MindSpore/unified/x86_64/mindspore-2.2.14-cp39-cp39-linux_x86_64.whl
```
-运行检查
```bash
python -c "import mindspore as ms; from mindspore import context; context.set_context(mode=context.GRAPH_MODE, device_target='GPU'); ms.run_check()"
```
如果看到（第一次会很慢，请耐心等待）：
```
MindSpore version:  2.2.14
The result of multiplication calculation is correct, MindSpore has been installed on platform [GPU] successfully!
```
安装成功！
-或运行下面这个脚本
```python
import numpy as np
import mindspore.context as context
from mindspore import Tensor
from mindspore.ops import functional as F

context.set_context(mode=context.PYNATIVE_MODE, device_target="GPU")

x = Tensor(np.ones([1,3,3,4]).astype(np.float32))
y = Tensor(np.ones([1,3,3,4]).astype(np.float32))
print(F.tensor_add(x, y))
```
若看到：
```
[[[[2. 2. 2. 2.]
   [2. 2. 2. 2.]
   [2. 2. 2. 2.]]

  [[2. 2. 2. 2.]
   [2. 2. 2. 2.]
   [2. 2. 2. 2.]]

  [[2. 2. 2. 2.]
   [2. 2. 2. 2.]
   [2. 2. 2. 2.]]]]
```
则安装成功！

15.克隆yolov3项目
```bash
git clone https://gitee.com/mindspore/models.git
```
跟着项目流程，测试运行项目。

训练成功，检查版本：
```bash
nvcc --version
```
显示：
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Fri_Jan__6_16:45:21_PST_2023
Cuda compilation tools, release 12.0, V12.0.140
Build cuda_12.0.r12.0/compiler.32267302_0
```
恭喜你得到了一个 ubuntu20.04+python3.9+cuda12.0+cuda11.6+cudnn8.8.0+cudnn8.5.0+mindspore-gpu:2.2.14容器。

  






  

