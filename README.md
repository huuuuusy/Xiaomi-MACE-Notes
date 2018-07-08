# MACE 环境配置及DEMO运行

[MACE](https://github.com/XiaoMi/mace) 是小米开发的移动端深度学习平台，其[官方文档](https://mace.readthedocs.io/en/latest/)提供了安装和开发的步骤.笔者的配置流程如下：

## Part I: Docker 使用

推荐使用Docker配置MACE开发环境．有关于Docker的基础知识可以参考如下教程：
1. [Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
2. [深入Docker：容器和镜像](https://segmentfault.com/a/1190000002766882)　

笔者电脑是Ubuntu 16.04系统，安装Docker指令为：

    sudo apt install docker.io

因为Docker需要的空间比较大(完整的环境大约有5G)，如果根目录没有足够的空间，可以外接移动硬盘，并使用软链接与根目录Docker路径相连．

***以下步骤为挂载硬盘及软链接操作步骤，非MACE安装必须.***

/var/lib/docker是原地址（默认Docker镜像存放地址），/media/hu/hu-disk/docker是移动硬盘的新地址．

操作方法：

关闭Docker服务：

    service docker stop
    
将默认的Dokcer目录移动到移动硬盘中：

    sudo mv /var/lib/docker /media/hu/hu-disk/docker
    
设置软链接：

    sudo ln -s /media/hu/hu-disk/docker /var/lib/docker
    
重启Docker服务：

    service docker start

## Part II: MACE 安装

参照[官网教程](https://mace.readthedocs.io/en/latest/getting_started/how_to_build.html)，完整镜像大概有5G,根据网速自行选择镜像制作方法：

方法一：直接下载docker镜像：

    sudo docker pull registry.cn-hangzhou.aliyuncs.com/xiaomimace/mace-dev-lite

方法二：下载[MACE源码](https://github.com/XiaoMi/mace.git),手动制作镜像：

    sudo docker build -t registry.cn-hangzhou.aliyuncs.com/xiaomimace/mace-dev-lite ./docker/mace-dev-lite

运行Docker(记住加sudo 权限）:

    sudo docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb --net=host \
               -v /local/path:/container/path \
               registry.cn-hangzhou.aliyuncs.com/xiaomimace/mace-dev-lite \
               /bin/bash
               
获取MACE源码：

    git clone https://github.com/XiaoMi/mace.git
    cd mace
    git fetch --all --tags --prune
    tag_name=`git describe --abbrev=0 --tags`
    git checkout tags/${tag_name}

![](http://static.zybuluo.com/huuuuusy/onkgv2mcuuf1bkt633riexon/image.png)


## Part III: MACE Demo运行

进入demo所在文件夹：

    cd mace/examples/android
    
查看build.sh脚本：

    #!/usr/bin/env bash
    
    set -e -u -o pipefail
    
    pushd ../../../
    python tools/converter.py build --config=docs/getting_started/models/demo_app_models.yaml
    
    cp -r builds/mobilenet/include/ mace/examples/android/macelibrary/src/main/cpp/
    cp -r builds/mobilenet/lib/ mace/examples/android/macelibrary/src/main/cpp/
    
    popd
    
    ./gradlew installAppRelease

因为原始Docker镜像没有配置SDK开发环境，直接运行脚本会报SDK路径错误，因此首先单步执行指令，然后在自己的主机上进行最后的app编译．

![](http://static.zybuluo.com/huuuuusy/xyc7r5k268dyyohf4seqmevn/image.png)

    set -e -u -o pipefail
    pushd ../../../
    python tools/converter.py build --config=docs/getting_started/models/demo_app_models.yaml
    
![](http://static.zybuluo.com/huuuuusy/z0p5eennhzekgyll0shc5mfq/image.png)

先下载MobileNet_v1模型：

![](http://static.zybuluo.com/huuuuusy/tgacsr155ou5qoox0eva83yp/image.png)

然后下载MobileNet_v2模型：

![](http://static.zybuluo.com/huuuuusy/fxhxxwa9498jcmgqwr563vdj/image.png)

构建库：

![](http://static.zybuluo.com/huuuuusy/gp35rxpvnqled3yvf53671lv/image.png)

![](http://static.zybuluo.com/huuuuusy/bhyzvu6lhcwccgr6dpi4f7f9/image.png)

    cp -r builds/mobilenet/include/ mace/examples/android/macelibrary/src/main/cpp/
    cp -r builds/mobilenet/lib/ mace/examples/android/macelibrary/src/main/cpp/

安装tree命令，以树形结构打印mace/builds/mobilenet文件目录作为验证：

    apt-get install tree
    
![](http://static.zybuluo.com/huuuuusy/3hcbsf7q0kk6dcjo0i3x693t/image.png)

从Docker下载mace文件夹到主机的Ubuntu 16.04系统下，利用主机的SDK环境完成最后的app生成．

首先在主机中打开新终端查看Docker镜像的ID:

![](http://static.zybuluo.com/huuuuusy/il09rtlry67gmx8j3loyr168/image.png)

    sudo docker cp f617db573cd4:/mace /media/hu/hu-disk

其中，f617db573cd4是ID,/mace是镜像中待复制的目录，/media/hu/hu-disk是主机目录．

![](http://static.zybuluo.com/huuuuusy/c5y30xok7uqgo8xiy9zolr3s/image.png)

因为复制操作是在sudo权限下进行的，所以在主机目录下修改文件夹权限：

    sudo chmod -R 777 mace

进入android项目的目录，新建local.properties文件，填写主机SDK和NDK路径信息：

    ## This file must *NOT* be checked into Version Control Systems,
    # as it contains information specific to your local configuration.
    #
    # Location of the SDK. This is only used by Gradle.
    # For customization when using a Version Control System, please read the
    # header note.
    #Sat Jul 07 19:31:12 HKT 2018
    ndk.dir=/media/hu/hu-disk/package/android/ndk/android-ndk-r15c
    sdk.dir=/media/hu/hu-disk/package/android/Sdk

然后打开终端，执行：

    ./gradlew installAppRelease

![](http://static.zybuluo.com/huuuuusy/m585wphlbumlwlbthul3myml/image.png)

此时，USB口连接的是小米8,通过ADB可以识别设备并将编译后的APP传入手机中(手机需要提前解开BL锁，刷成开发版本以获得root权限，电脑需要安装ADB工具并可以通过USB接口调试手机).

最终得到的demo是名为识物的app,可以选择MobileNet_v1, MobileNet_V2两种网络和CPU，GPU两种原件进行物体识别计算．

MobileNet_v1+CPU

![](http://static.zybuluo.com/huuuuusy/xa9cus3nbpjulg3w8hc57jft/Screenshot_2018-07-07-22-15-19-705_com.xiaomi.mace.demo.png)

MobileNet_v1+GPU

![](http://static.zybuluo.com/huuuuusy/oy8xv91ofi1geuc137ez8m26/Screenshot_2018-07-07-22-15-29-558_com.xiaomi.mace.demo.png)

MobileNet_v2+CPU

![](http://static.zybuluo.com/huuuuusy/ibw8ycvx17ueu9wc3w87edli/Screenshot_2018-07-07-22-15-47-615_com.xiaomi.mace.demo.png)

MobileNet_v2+GPU

![](http://static.zybuluo.com/huuuuusy/fezrp2x3une2sgexi6p3oqsz/Screenshot_2018-07-07-22-15-39-996_com.xiaomi.mace.demo.png)

可以看到两种网络利用Mobile GPU的物体识别速度均快于Mobile CPU.
