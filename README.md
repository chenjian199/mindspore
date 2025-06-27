这是一个在docker里安装mindspore-gpu-cuda12.0版本，然后使用coco2017数据集训练yolov3的教程。
首先确保能连接外网。

第一步，进入下面这个网址：
  https://hub.docker.com/
第二步，在搜索框里输入navidia/cuda，然后在tags搜索栏里输入cuda12.0,选择你需要的版本，这里我们选择：
  12.0.0-cudnn8-devel-ubuntu20.04
第三步，下载镜像到本地：
  docker pull nvidia/cuda:12.0.0-cudnn8-devel-ubuntu20.04
第四步，启动镜像：
  sudo docker run -it --name CJ_MAKE_MINDSPORE --hostname cjmakemindspore --network host --privileged  nvidia/cuda:12.0.0-cudnn8-devel-ubuntu20.04 /bin/bash
第五步，进入容器后，输入下列命令，安装基础编辑器：
  apt update
  apt install vim -y
  vim ~/.bashrc
  在文件末尾输入：
    # 自定义 PS1：绿色用户主机名 + 蓝色目录
    PS1='\[\e[0;32m\]\u@\h\[\e[0m\]:\[\e[0;34m\]\w\[\e[0m\]\$ '
  然后保存退出，运行：
    source ~/.bashrc
  apt install nano -y

第六步，安装cuda依赖：
  apt install linux-headers-generic gcc-9
第七步，安装cuda11.6:
  apt install wget -y
  进入下面这个网址：
    https://developer.nvidia.com/cuda-11-6-0-download-archive
  选择合适的版本，这里我们选择ubuntu20.04版本，通过wget安装：
    wget https://developer.download.nvidia.com/compute/cuda/11.6.0/local_installers/cuda_11.6.0_510.39.01_linux.run
    bash cuda_11.6.0_510.39.01_linux.run
