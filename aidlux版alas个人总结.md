# AidLux 安装 Alas 个人攻略

### 前言：

此攻略适用于：安卓11及以上安卓手机；

测试环境：Redmi Note 12T Pro 天玑8200-U 安卓13 

Aidlux 版本： 2.1.0

要求您有一定的代码基础、玩机经验与足够的折腾耐心

还有，受制于markdown格式，本教程内会有部分长代码被截断强制分行，请注意甄别 `

#### 最好能详细了解过alas官方群内的几篇相关攻略

##### 包括但不限于：
- 《[教程] 在安卓真机上安装Alas》
- 《安卓部署ALAS[基于高版本AidLux]》
- 《[教程] 在云手机上安装Alas 20220920》

此教程基于《[教程] 在云手机上安装Alas 20220920》，在此教程基础上结合前两篇与ai提供的信息而完成，侧重点在于与实际安卓真机和aidlux结合，剔除云手机安装流程中对于安卓真机不必要的部分，补充必要的步骤，修正部分遇到的错误



## 解除安卓 12 及以上的子进程限制

aidlux启动时会进行询问并自动处理，一般遵循指示即可

但这个坑爹玩意在于它只给了自动处理，不给手动处理，一但抽风报错就永远无法为aidlux解除限制

（新能源汽车の神秘车机智驾处理逻辑：这个我熟！）

下面给出手动解除命令（或许有效吧）：

```bash
adb shell "/system/bin/device_config put activity_manager max_phantom_processes 2147483647"
adb shell "/system/bin/device_config set_sync_disabled_for_tests persistent" 

#如果希望恢复原本的限制状态：
#adb shell /system/bin/device_config set_sync_disabled_for_tests none
#adb shell /system/bin/device_config put activity_manager max_phantom_processes 32
```

​	安卓14及以上可尝试：开发者选项——停止限制子进程——关闭

## 更改时区

```bash
tzselect
4
10
1
1
date -R
#时区显示0800即为正常东八区（北京/上海时间）
```

## 更换软件源

因为默认清华源有无法访问的问题，故有换源操作

```bash
sudo sed -i 's/mirrors.tuna.tsinghua.edu.cn/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo apt update
```

## 安装 Adb、Python、Git 和 nano 等工具

```bash
sudo apt install -y python3 python3-pip git android-tools-adb nano
```

## 获取 Alas

遇到 timeout 等问题请自行寻找镜像源，或使用网络代理

```bash
git clone https://github.com/LmeSzinc/AzurLaneAutoScript.git
```

原教程的gitee不知为什么无法打开了（其实即使能打开也要你先注册登录），这里换成GitHub原版链接

## 安装依赖库

```bash
sudo apt install -y gfortran libopenblas-dev liblapack-dev libatlas-base-dev
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install wheel
python3 -m pip install --upgrade pip setuptools wheel pbr packaging --user
```

## 安装其他库

```bash
sudo apt install python3-dev libopenblas-dev libopencv-dev libzmq3-dev -y
sudo pip3 install --upgrade pip setuptools wheel -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
sudo pip3 install --only-binary=:all: gevent
sudo apt-get install -y ffmpeg libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libavdevice-dev pkg-config
sudo apt install -y gfortran libopenblas-dev liblapack-dev
sudo apt install libxml2-dev libxslt-dev python3-dev gcc g++ -y
#长代码被截断
```

## 配置 Alas

切换到 alas 所在目录

```bash
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
- 删除 mxnet这一行
- av==10.0.0→av>=10.0.0

然后crtl+x，按y，再按回车即可退出

安装alas依赖库

```bash
sudo pip3 install --use-pep517 -r ./deploy/docker/requirements.txt
#看到successfully installed (xxx库) 就行，安装完成后的不兼容报错不用管（或许吧）
#这一步极易报错，个人测试这样可以安装，但实际如果真的发生报错导致无法继续安装，还是交给ai看看吧
```

## 安装 MXNet

原教程的 whl 包获取方式太复杂，这里贴上对应的直链，感谢来自 [@binss](https://www.binss.me/)编译的 aarch64 架构下的 mxnet==1.9.1 wheel 文件

原帖链接： https://www.binss.me/blog/run-azurlaneautoscript-on-arm64/

```bash
sudo pip3 install https://github.com/LmeSzinc/AzurLaneAutoScript/raw/7ac25c06dc1e9841f771d8a2060d420d7e6f0a87/dev_tools/arm64/mxnet-1.9.1-py3-none-any.whl
#长代码被截断
```

## 编译安装 OpenCV

不要为了偷懒而直接 pip install ，老老实实地编译一遍吧

安装前置依赖

```bash
sudo apt install -y build-essential cmake qt5-default libvtk6-dev zlib1g-dev libjpeg-dev libwebp-dev libpng-dev libtiff5-dev libopenexr-dev libgdal-dev libdc1394-22-dev libavcodec-dev libavformat-dev libswscale-dev libtheora-dev libvorbis-dev libxvidcore-dev libx264-dev yasm libopencore-amrnb-dev libopencore-amrwb-dev libv4l-dev libxine2-dev libtbb-dev libeigen3-dev python-dev python-tk pylint python-numpy python3-dev python3-tk pylint3 python3-numpy flake8 ant default-jdk doxygen unzip wget
#长代码被截断
```

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
cmake -DWITH_QT=ON -DWITH_OPENGL=ON -DFORCE_VTK=ON -DWITH_TBB=ON -DWITH_GDAL=ON -DWITH_XINE=ON -DENABLE_PRECOMPILED_HEADERS=OFF -DWITH_OPENJPEG=OFF ..
#长代码被截断
```

make，这里需要 1~2 小时以上。

```bash
make -j2
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

### 附加：opencv-python 的安装

当性能测试报错cv2相关错误时可尝试

```bash
sudo pip install opencv-python==4.5.3.56
#何意味，最终不还是pip install 了一个opencv-python，那我前面编译源码的意义是什么
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


到这里，Alas 已经安装完成了。

## 启动 Alas

回到 aidlux 根目录

```bash
cd
cd AzurLaneAutoScript
python3 gui.py
```

或者在任意位置

```bash
python3 AzurLaneAutoScript/gui.py
```

其余操作(VMOS\真机运行游戏、调试alas、游戏内设置等)与旧版教程一致

## 注意事项

注意：性能测试结果仅供参考，不代表万事无忧
即使性能测试通过，后续也存在报错的风险（比如一些文字识别的情景）

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
