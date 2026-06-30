# Termux 安装 Alas 个人攻略 (Ubuntu24方案)

### 前言：

此攻略适用于：安卓11及以上安卓手机；

测试环境：Redmi Note 12T Pro 天玑8200-U 安卓13 

Termux 版本： 0.118

要求您有一定的代码基础、玩机经验与足够的折腾耐心

还有，受制于markdown格式，本教程内会有部分长代码被截断强制分行，请注意甄别 

#### 最好能详细了解过alas官方群内的几篇相关攻略

##### 包括但不限于：
- 《[教程] 在安卓真机上安装Alas》
- 《安卓部署ALAS[基于高版本AidLux]》
- 《[教程] 在云手机上安装Alas 20220920》

此教程基于《[教程] 在云手机上安装Alas 20220920》，在此教程基础上结合前两篇与ai提供的信息而完成，侧重点在于与实际安卓真机和Termux结合，剔除云手机安装流程中对于安卓真机不必要的部分，补充必要的步骤，修正部分遇到的错误

#### 其实aidlux版教程和本文差别不大，不过在termux上能用ssh了就方便很多，而且Ubuntu自带root不用加sudo了
#### 哦对了，这篇教程还参考了 “下游项目“ 的 [教程](https://github.com/wess09/AzurPilot/blob/master/README.md#termux-%E7%9C%9F%E6%9C%BA%E5%AE%89%E8%A3%85%E8%BF%90%E8%A1%8C%E6%8C%87%E5%8D%97)
###### 链接为2026-06-30有效，由于此[项目](https://github.com/wess09/AzurPilot)的结构经常变动，因此此链接可能在未来失效

下载Alas请认准GitHub官方地址：https://github.com/LmeSzinc/AzurLaneAutoScript


### 注意：此方案的Ubuntu24自带Python版本相对于alas过高(3.12),需要创建虚拟环境，这意味着需要更加折腾

- 如果确认您没有用真机运行游戏的需求，VMOS已经可以满足您的需求，请转向Ubuntu 20方案
- termux的Ubuntu 20 与24 两个方案可共存



#### 接下来的所有安装过程中如果遇到询问你的选项，直接回车



## 解除安卓 12 及以上的子进程限制

下面给出手动解除命令（或许有效吧）：

```bash
adb shell "/system/bin/device_config put activity_manager max_phantom_processes 2147483647"
adb shell "/system/bin/device_config set_sync_disabled_for_tests persistent" 

#如果希望恢复原本的限制状态：
#adb shell /system/bin/device_config set_sync_disabled_for_tests none
#adb shell /system/bin/device_config put activity_manager max_phantom_processes 32
```

​	安卓14及以上可尝试：开发者选项——停止限制子进程——关闭

## 更换termux的软件源为清华源
```bash
termux-change-repo
#大概就是形如 mirrors.tuna.tsinghua.edu.cn的选项
```
### 可选：使用电脑ssh连接代替手机操作
在termux内：
```bash
pkg upgrade -y
pkg install openssh
```
打开并配置ssh：
```bash
sshd
passwd
#这里输入两次你的密码，要自己能记住
whoami
#这里会弹出一个类似 u0_a(xxx)的名字，记住它，这里以u0_a260为例
```
接下来暂时断开网络代理（如果有的话）
在无线调试或者路由器后台界面查看并记住手机的本地局域网ip（形如 192.168.0.114 等）
一些路由器可以开启静态ip以固定这个值
在电脑上：打开powershell（推荐）或者cmd
```bash
ssh u0_a260@192.168.0.114 -p 8022
#这里@符号前面就是whoami提示的名字，后面就是局域网ip
```
输入密码，如果是第一次连接会提示让你输入“yes”三个字母
随后便能看到电脑终端上显示出termux刚启动时的“welcome to termux”界面
##### 报错：WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
需要清除旧缓存
```bash
ssh-keygen -R "[192.168.0.114]:8022"
```

## 配置termux
```bash
pkg install -y curl wget tsu proot tar git
curl -L https://its-pointless.github.io/setup-pointless-repo.sh | sh
```

## 安装 Ubuntu20
```bash
proot-distro install -n ubuntu https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu-base/releases/24.04/release/ubuntu-base-24.04.4-base-arm64.tar.gz
#长代码截断
```
这时你会看到终端变成了“root@localhost:~#”
先退出
```bash
exit
```
真机文件权限（手机会有弹窗，必须同意）：
```bash
termux-setup-storage
```
回到Ubuntu
```bash
proot-distro login ubuntu
```
改掉 Andronix 脚填的 DNS
```bash
echo -e "nameserver 114.114.114.114\nnameserver 8.8.8.8\n" > $folder/etc/resolv.conf
```
更新
```bash
apt update && apt upgrade -y
```
安装Python 和 Git 等：
```bash
apt install -y python3 python3-pip git android-tools-adb nano
```
## 更改时区
```bash
DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata
```
```bash
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
```bash
date -R
```
如果看到 +0800 的字符就说明成功了

## 获取 Alas

遇到 timeout 等问题请自行寻找镜像源，或使用网络代理

```bash
git clone https://github.com/LmeSzinc/AzurLaneAutoScript.git
```

原教程的gitee不知为什么无法打开了（其实即使能打开也要你先注册登录），这里换成GitHub原版链接

## 下载编译安装python 3.8.10
安装编译所需库
```bash
apt update && apt upgrade -y
apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
下载python 3.8.10源码
```bash
cd /usr/local/src
wget https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tgz
tar -zxvf Python-3.8.10.tgz
cd Python-3.8.10
```
配置编译
```bash
./configure --prefix=/usr/local/python3.8.10 --enable-optimizations --with-ensurepip=install
```
编译安装
```bash
make -j$(nproc)
make altinstall
```
###### 这里-j 后填 CPU 核心数，可以加速，如-j4、-j8等，保持原样则为程序自动分配；但这个参数不要超过cpu物理核心数，否则得不偿失
创建软链接
```bash
ln -s /usr/local/python3.8.10/bin/python3.8 /usr/local/bin/python3.8
ln -s /usr/local/python3.8.10/bin/pip3.8 /usr/local/bin/pip3.8
```
验证版本
```bash
python3.8 -V
pip3.8 -V
```
###### 输出 Python 3.8.10 即成功

项目部署
```bash
cd /root/AzurLaneAutoScript
python3.8 -m venv venv_alas
#启动虚拟环境
source venv_alas/bin/activate
pip install --upgrade pip
```
###### 看到终端前缀出现 (venv_alas) 代表激活成功
###### 这里“venv_alas”可以改成自己想要的名字，但以后的命令必须同步改成自定义的名字
此后，激活虚拟环境命令为
```bash
source AzurLaneAutoScript/venv_alas/bin/activate
```
退出虚拟环境
```bash
deactivate
```

## 安装依赖库

```bash
apt install -y gfortran libopenblas-dev liblapack-dev libatlas-base-dev
apt install -y build-essential python3-dev curl git wget cmake clang
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install wheel
python3 -m pip install --upgrade pip setuptools wheel pbr packaging --user
```

## 安装其他库

```bash
apt install python3-dev libopenblas-dev libopencv-dev libzmq3-dev -y
pip3 install --upgrade pip setuptools wheel -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
pip3 install --only-binary=:all: gevent
apt-get install -y ffmpeg libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libavdevice-dev pkg-config
apt install -y gfortran libopenblas-dev liblapack-dev
apt install libxml2-dev libxslt-dev python3-dev gcc g++ -y
#长代码被截断
```

## 配置 Alas

切换到 alas 所在目录

```bash
cd
cd AzurLaneAutoScript
cp ./config/deploy.template-AidLux-cn.yaml ./config/deploy.yaml
```
这里直接拿原本的requirements.txt安装会有严重冲突，按照接下来的方式进行微调
```bash
nano ./deploy/docker/requirements.txt
```
##### 进行如下修改：

- numpy→1.9.5
- scipy→1.10.1
- 注释掉pillow这一行
- 删除 mxnet这一行
- av==10.0.0→av>=10.0.0
然后crtl+x，按y，再按回车即可退出

安装alas依赖库

```bash
pip3 install --use-pep517 -r ./deploy/docker/requirements.txt
#看到successfully installed (xxx库) 就行，安装完成后的不兼容报错不用管（或许吧）
#这一步极易报错，个人测试这样可以安装，但实际如果真的发生报错导致无法继续安装，还是交给ai看看吧
```
如果安装到某个库就报错，可以尝试先把它在./deploy/docker/requirements.txt里面注释掉，并在安装完成后单独pip3 install xx库==xx版本，版本请参考alas根目录的requirements.txt
比如如果不注释掉pillow，实测安装pillow时可能会报错：packaging.version.InvalidVersion: Invalid version: '1.1-linux32'
就等Successfully installed xxx之后单独执行
```bash
pip3 install pillow==8.3.2
```
###### 简直是玄学

## 安装 MXNet

原教程的 whl 包获取方式太复杂，这里贴上对应的直链，感谢来自 [@binss](https://www.binss.me/)编译的 aarch64 架构下的 mxnet==1.9.1 wheel 文件

原帖链接： https://www.binss.me/blog/run-azurlaneautoscript-on-arm64/

```bash
pip3 install https://github.com/LmeSzinc/AzurLaneAutoScript/raw/7ac25c06dc1e9841f771d8a2060d420d7e6f0a87/dev_tools/arm64/mxnet-1.9.1-py3-none-any.whl
#长代码被截断
```

## 编译安装 OpenCV

不要为了偷懒而直接 pip install ，老老实实地编译一遍吧

安装前置依赖

```bash
apt install -y build-essential cmake unzip wget yasm doxygen default-jdk flake8 libtbb-dev libeigen3-dev zlib1g-dev libjpeg-dev libwebp-dev libpng-dev libtiff-dev libopenexr-dev libgdal-dev libavcodec-dev libavformat-dev libswscale-dev libtheora-dev libvorbis-dev libxvidcore-dev libx264-dev libopencore-amrnb-dev libopencore-amrwb-dev libxine2-dev qtbase5-dev qttools5-dev libqt5opengl5-dev libv4l-dev libdc1394-dev libvtk9-dev
#长代码被截断
```
###### 相比原教程，这里为适配Ubuntu 24 做了调整

下载 opencv 4.5.3 源码

```bash
OPENCV_VERSION='4.5.3'
git clone https://github.com/LmeSzinc/opencv-archive.git
unzip ./opencv-archive/opencv-${OPENCV_VERSION}.zip
rm -rf opencv-archive
mv opencv-${OPENCV_VERSION} OpenCV
cd OpenCV
mkdir build
cd build
```

cmake，这里可能需要 5 分钟。

```bash
cmake -DWITH_QT=ON -DWITH_OPENGL=ON -DFORCE_VTK=ON -DWITH_TBB=ON -DWITH_GDAL=ON -DWITH_XINE=ON -DENABLE_PRECOMPILED_HEADERS=OFF -DWITH_OPENJPEG=OFF -DCMAKE_CXX_FLAGS="-Wno-stringop-overread -Wno-stringop-overflow -Wno-error" -DWITH_ADE=OFF -DWITH_FFMPEG=OFF -DCMAKE_CXX_STANDARD=14 ..
#长代码被截断
```
###### 同样为了顺利编译而做了调整

make，这里需要 1~2 小时以上。

```bash
make -j4
#这里原教程的参数 j2还是太保守了，本地真机可以尝试 j4甚至 j8暴力加速
```

安装

```bash
pip3 uninstall -y opencv-python-headless
make install
ldconfig
```

## 测试 OpenCV 与 MXNet

测试 OpenCV 与 MXNet，输入以下命令

先清空临时变量

```bash
unset LD_PRELOAD
```

测试 opencv

```bash
export LD_PRELOAD="/usr/lib/aarch64-linux-gnu/libgomp.so.1:/usr/lib/aarch64-linux-gnu/libGLdispatch.so.0" && python3 -c "import cv2"
#长代码被截断
```

测试 mxnet

```bash
export MXNET_LIBRARY_PATH="/usr/local/mxnet/libmxnet.so" && python3 -c "import mxnet"
```

没有结果返回就是成功导入

有返回错误：请参照原云手机安装教程进行排查
附加：此处mxnet的系统变量设置
```bash
export LD_LIBRARY_PATH=/root/AzurLaneAutoScript/venv_alas/mxnet:$LD_LIBRARY_PATH
```


### 附加：opencv-python 的安装

当性能测试报错cv2相关错误时可尝试

```bash
pip install opencv-python==4.5.3.56
#何意味，最终不还是pip install 了一个opencv-python，那我前面编译源码的意义是什么
#史山项目能跑起来就不错了
```

## 添加环境变量

```bash
nano ~/.bashrc	
```

在文件末尾添加这两行：

```bash
export LD_PRELOAD="/usr/lib/aarch64-linux-gnu/libgomp.so.1:/usr/lib/aarch64-linux-gnu/libGLdispatch.so.0"
export MXNET_LIBRARY_PATH="/usr/local/mxnet/libmxnet.so"
#长代码被截断
```
然后保存退出

```bash
source ~/.bashrc
```
安装排查出来的冲突
```bash
pip3 install typing-extensions==4.3.0
pip3 install pydantic==1.10.2
```

到这里，Alas 已经安装完成了。

## 启动 Alas

回到根目录

```bash
cd
cd AzurLaneAutoScript
python3 gui.py
```

或者在执行cd 之后直接

```bash
python3 AzurLaneAutoScript/gui.py
```

其余操作(VMOS、调试alas、游戏内设置等)与旧版教程一致

## 注意事项

注意：性能测试结果仅供参考，不代表万事无忧
即使性能测试通过，后续也存在报错的风险（比如一些文字识别的情景）

##### 实测：三星zflip5 (骁龙8Gen2) 8G内存版魔改无屏幕盒子+主动散热运行工况：整机功耗6~8W左右，VMOS虚拟机和termux可以长期同时运行，可以满足长期无人值守挂机需求

## 后记
#### 为什么会有这篇文章？
个人最近入手了相关安卓设备用于运行游戏本体和alas，但alas官方现有的教程要么已经过时，要么根本牛头不对马嘴，新版教程甚至只确保到打开主界面？？！

（并非确保，里面一堆版本冲突）



旧版教程使用的aidlux0.92在高版本安卓无法正常使用，表现为安装后操作一番，彻底关闭应用再打开，会卡死在启动界面
新版教程已经吐槽过了
毫无疑问，alas开发组的努力与心血是值得也应当被尊重的，但教程过时、步骤分散、疏漏与错误的问题也的确客观存在
很难想象一篇在真机上运行的攻略，最后是在云设备安装的攻略里找到了关键步骤
故而本人结合实际探索与原本的教程，总结出了这篇攻略

### 哦对了

我不太会写markdown，所以这篇文章的格式会看着怪怪的



#### 这篇文章如果帮助到你，将会是我莫大的荣幸



2026-06-27
by	XDista && 平行之倾彻
