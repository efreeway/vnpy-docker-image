# vnpy-docker-image  
vnpy docker image （ubuntu-18.04 + novnc + vnpy-1.9.1)  
  
## 使用方法：  
  
#本镜像同样可以在windows环境下的docker中使用，但是注意windows环境下docker的存储空间比较有限，本镜像解压后有5.5G，注意删除不需要的images，以便腾出有效空间  
  
#先pull该镜像，并运行，然后在本机用浏览器通过novnc登录容器  
docker pull efreeway/ubuntu-vnpy-novnc:vnpy-1.9.1  
docker tag efreeway/ubuntu-vnpy-novnc:vnpy-1.9.1  vnpynovnc  
docker run -p 6080:80 vnpynovnc  
  
http://localhost:6080/  #logon the container   
#在windows环境下，要用http://docker本身的虚拟机地址:6080  
因为，如果是在windows环境下用docker，docker启动的时候，实际上是启动了一台虚拟机，并且明确告知了这个虚拟机的的地址，例如：192.168.99.100。  
所以，容器是运行在这个虚拟机上的，在通过novnc登录vnpy容器的时候，应该用虚拟机的ip地址：  
http://192.168.99.100:6080   而不是  http://localhost:6080  
注意，连接novnc的时候，浏览器一定要全屏，novnc是根据浏览器的大小决定桌面的尺寸的，vnpy的界面很大，如果不全屏，界面会被遮挡。  
  
登录后用 菜单中的system tools/LXterminal开启terminal  
在terminal中：  
cd Desktop/vnpy-1.9.1/examples/VnTrader  
conda_python run.py  
就可以看到vnpy的图形界面。  
  
  
在linux下面使用docker没有上述转换问题  
  
文字编辑可以用镜像中菜单中的accessories/leafpad，这是一个很方便的文字编辑软件。  
打开system tools/file manager pcmanfm，有图像界面，定位到要修改的文件，右键，选择leafpad，就可以很方便的修改。  
  
  
## 存在的问题  
1、xtp接口在运行的时候报错  
2、由于python多个版本，可能会造成互相影响，需要时可以修改这些配置  
在~/.bashrc  中有一句： alias python='/usr/bin/python2.7'  
conda_python的实际文件是/opt/conda/bin/python (由ln -s /opt/conda/bin/python /usr/local/bin/conda_python命令产生的链接)  
比如，如果要重装vnpy，可能需要把~/.bashrc中的alias改为：  
alias python='/usr/local/bin/python'  
并使用source ~/.bashrc使配置生效  
安装完毕之后，为了novnc的正常使用，不要忘了改回来。  
  
  
## 镜像制作过程  
  
  
#本镜像基于dorowu/ubuntu-desktop-lxde-vnc制作，操作系统为ubuntu-18.04  
#based on: dorowu/ubuntu-desktop-lxde-vnc  
#docker run -p 6080:80 dorowu/ubuntu-desktop-lxde-vnc  
#http://localhost:6080/  logon the container   
#在windows环境下，要用http://docker本身的虚拟机地址:6080  
  
  
### step1: 系统设置  
systemd-machine-id-setup  
  
#使用传统的 bash 作为 shell 解释器  
rm /bin/sh  ln -s /bin/bash /bin/sh  
  
#时区设置  
tzselect  
  
  
### step2  更新软件  
#在本地构建镜像时，使用163的 apt 源  
在/etc/apt/sources.list 文件中，把原有的源注释掉，增加下面的源。这里面“bionic”是ubuntu-18.04的代号，如果是其他版本，比如16.04，要用“xenial”  
deb http://mirrors.163.com/ubuntu/ bionic main multiverse restricted universe   
deb http://mirrors.163.com/ubuntu/ bionic-backports main multiverse restricted universe   
deb http://mirrors.163.com/ubuntu/ bionic-proposed main multiverse restricted universe   
deb http://mirrors.163.com/ubuntu/ bionic-security main multiverse restricted universe   
deb http://mirrors.163.com/ubuntu/ bionic-updates main multiverse restricted universe  
deb-src http://mirrors.163.com/ubuntu/ bionic main multiverse restricted universe   
deb-src http://mirrors.163.com/ubuntu/ bionic-backports main multiverse restricted universe   
deb-src http://mirrors.163.com/ubuntu/ bionic-proposed main multiverse restricted universe   
deb-src http://mirrors.163.com/ubuntu/ bionic-security main multiverse restricted universe   
deb-src http://mirrors.163.com/ubuntu/ bionic-updates main multiverse restricted universe   
  
#apt更新  
apt-get clean   
apt-get update  
  
#从 apt 获取软件" \  
     apt-get install -y bzip2 wget libgl1-mesa-glx qt5-default ttf-wqy-microhei \  
#安装编译环境" \  
     apt-get install -y build-essential libboost-all-dev python-dev cmake  
  
  
### step3 安装 anaconda  
     mkdir /tmp/conda/ \  
     cd /tmp/conda/ \  
#下载 Miniconda by Python2 ，安装，然后删除下载的包  
wget -t 0 https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda2-latest-Linux-x86_64.sh   
bash Miniconda*.sh -b -p /opt/conda   
rm Miniconda*.sh \  
  
#设置 conda 和 python 的环境路径，为了避免和原来的python环境冲突，修改名称为conda_python：  
     ln -s /opt/conda/bin/python /usr/local/bin/python  
     ln -s /opt/conda/bin/conda /usr/local/bin/conda  
     ln -s /opt/conda/bin/pip /usr/local/bin/pip  
  
  
### step4 设置 conda 国内源, 从 conda 安装 python 库  
#下面这2个环节中，有一个好像是直接用缺省的源就好了。好像是conda的源是可以用的，反而是tsinghua的源没有针对ubuntu-18.04的。  
#conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/   
#conda config --set show_channel_urls yes   
conda install -y pymongo pyzmq numpy msgpack-python qtpy pyqt   
conda install -c https://conda.anaconda.org/quantopian ta-lib   
conda clean -ay  
  
#从 pip 安装 python 库  
mkdir ~/.pip \  
echo [global]" >> ~/.pip/pip.conf   
echo index-url = http://pypi.douban.com/simple" >> ~/.pip/pip.conf   
#使用 pip 安装 python 库  
     pip --trusted-host pypi.douban.com install ta-lib websocket-client qdarkstyle psutil quantopian-tools   
     pip --trusted-host pypi.douban.com install zipline  
  
  
#安装 mongodb 服务  
     mkdir -p /data/db \  
     apt-get install -y mongodb \  
     systemctl enable mongodb.service \  
     sed -i 's/bind_ip = 127.0.0.1/\#bind_ip = 127.0.0.1/g' /etc/mongodb.conf  
  
#为确保novnc可用，需要修改默认python版本，并修改vnpy所用版本的快捷方式  
     在~/.bashrc文件中，添加：  
	 alias python='/usr/bin/python2.7'  
	 并保存。  
	 rm /usr/local/bin/python  
     ln -s /opt/conda/bin/python /usr/local/bin/conda_python  
	 source ~/.bashrc  #确保新的配置生效  
#安装配置结束  
