# 从零开始的异世界
学习研究安卓系统的第一步就是下载并编译自己的安卓源码。我在参与脱壳机项目之前，完全没有接触过安卓开发，所以我是从零开始搭建的。
## 前言
* 首先是给大家推荐一个工具叫docker，非常方便。如果你已经有docker的使用经验，可以直接使用下面的俩个image进行源码编译
    * 编译安卓4.3：unpacker/android-4.3:v1
    * 编译安卓6.0：unpacker/android-6.0:v2
* 文章内容参考自[自己动手编译Android源码](https://www.jianshu.com/p/367f0886e62b)，[repo安装](http://www.cnblogs.com/xiaoerlang/p/3549156.html)，[ubuntu16.04安装openjdk7](https://www.cnblogs.com/bluestorm/p/5677625.html)，[修改make版本](http://await.leanote.com/post/%E6%9B%B4%E6%96%B0make%E7%89%88%E6%9C%AC-2)，[android版本下载和切换](http://blog.csdn.net/bob_fly1984/article/details/61415575)

## 概述
* android源码编译的四个流程，下文也将按照该流程讲述：
 * 1.源码下载;
 * 2.构建编译环境;
 * 3.编译源码;
 * 4运行


&emsp;
## 源码下载

由于某墙的原因,这里我们采用国内的镜像源进行下载.
目前,可用的镜像源一般是科大和清华的,具体使用差不多,这里我使用的是科大源.(参考:[科大源](https://link.jianshu.com/?t=https://lug.ustc.edu.cn/wiki/mirrors/help/aosp),[清华源](https://link.jianshu.com/?t=https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/))。

&emsp;

## repo工具的下载及安装

repo是基于git基础开发，便于git资源管理的一个工具，所以在安装repo之前我们先要安装git
```
sudo apt-get install git
```

网上的资料一般都推荐执行以下命令实现repo工具的下载和安装
```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
下载完毕后创建源码工作目录
```
mkdir source
cd source
```

如果和我一样下载不成功，可以用下面的方法安装repo
```
mkdir ~/bin
PATH=~/bin:$PATH
```
首先是创建bin文件夹并配置为临时环境变量(也可以配置为永久),然后用git下载repo
```
git clone https://gerrit-googlesource.lug.ustc.edu.cn/git-repo
```
将git-repo的repo复制到先前创建的bin目录中并修改权限
```
cd git-repo
cp repo ~/bin/
chmod a+x ~/bin/repo
```
创建同步源码的工作目录，并在里面创建.repo
```
mkdir android_source
cd android_source
mkdir .repo
```
将下载的git-repo拷贝到.repo下并改名为repo

&emsp;

## 初始化仓库
我们将上面的source文件夹作为仓库,现在需要来初始化这个仓库了.通过执行初始化仓库命令可以获取AOSP项目master上最新的代码并初始化该仓库,命令如下:

```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
```
或者使用
```
repo init -u git://aosp.tuna.tsinghua.edu.cn/aosp/platform/manifest
```

两者实现的效果一致,仅仅只是协议不同.
如果执行该命令的过程中,如果提示无法连接到 gerrit.googlesource.com，那么我们只需要编辑 ~/bin/repo文件，找到REPO_URL这一行,然后将其内容修改为:
```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
```
##同步源码到本地
初始化仓库之后,就可以开始正式同步代码到本地了,命令如下:
```
repo sync
```
以后如果需要同步最新的远程代码到本地,也只需要执行该命令即可.在同步过程中,如果因为网络原因中断,使用该命令继续同步即可.不出意外,5个小时便可以将全部源码同步到本地.所以呢,这个过程可以放在晚上睡觉期间完成.

(提示:一定要确定代码完全同步了,不然在下面编译过程出现的错误会让你痛不欲生,不确定的童鞋可以多用repo sync同步几次)


&emsp;

## 查看源码版本
去make文件中查看
```
vim build\core\version_defaults.mk
```
搜索该文件中的 PLATFORM_VERSION值
## 修改源码版本
修改源码的安卓代码版本，以切换为4.0.3_r1为例
```
repo init -bandroid-4.0.3_r1
```
可以在目录 .repo/manifest.xml中查看repo客户端是在哪个分支上。
```
repo sync
```
## 构建编译环境
### 1.硬件
在安卓官网上要求写的很清楚：100GB以上硬盘空间，64位操作系统，如果是2.3x以下版本，需要32位的操作系统。


### 2.操作系统(我使用的是ubuntu)

* Android 6.0至AOSP master  ->  Ubuntu 14.04以上
* Android 2.3x至Android 5.x -> Ubuntu 12.04以上
* Android 1.5至Android 2.2x  -> Ubuntu 10.04以上


### 3.JDK
* Android6.0至AOSPmaster ->  OpenJDK 8
* Android 5.x至android 6.0  ->  OpenJDK 7
* Android 2.3.x至Android 4.4.x  ->  Oracle JDK 6
* Android 1.5至Android 2.2.x  ->  Oracle JDK 5

##### OpenJDK7

如果你要编译的是Android 5.x到android 6.0之间的系统版本,需要采用openjdk7.但是Ubuntu 15.04及之后的版本的在线安装库中只支持openjdk8和openjdk9的安装.因此,如果你想要安装openjdk 7需要首先设置ppa：
```
sudo add-apt-repository ppa:openjdk-r/ppa 
sudo apt-get update
```
安装jdk7
```
sudo apt-get install openjdk-7-jdk 
```
##### OpenJDK8
我现在在Ubuntu 16.04下编译AOSP主线代码,因此需要安装OpenJDK 8,执行命令如下:
```
sudo apt-get install openjdk-8-jdk
```
如果你需要在Ubuntu 14.04下编译AOSP主线代码,同样需要安装OpenJDK 8,此时需要执行如下命令:
```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```
##### Java版本切换
```
sudo update-alternative --config java
sudo update-alternative --config javac
```

&emsp;


### 4.Key packages

* Python 2.6 -- 2.7 from [python.org](https://www.python.org/downloads/)
* GNU Make 3.81 -- 3.82 from [gnu.org](ftp://ftp.gnu.org/gnu/make/)
* Git 1.7 or newer from [git-scm.com](https://git-scm.com/download)

注意，ubuntu15.04以上默认安装的是4.0的make，所以我们要手动更改make版本，在上面的链接中下载make3.81或者3.82的tar.gz。下载后解压，然后进入解压的目录中
```
./configure --prefix=/usr 
```
./configure --prefix=/usr 指定目录要不然安装完后就是在usr/local/bin/make 这样就不是在/usr/bin/make 存在2个make
```
type make
make check
make install
```
这样新的make就安装在了/usr/bin/make
```
make -v
```
发现make版本改变成功

### 5.Other required packages(only for ubuntu)
##### Unbuntu 12.04
```
sudo apt-get install git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386

sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so
```

##### Ubuntu 14.04
```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip
```
注意：如果要使用SELinux工具进，请同时安装python-networkx软件包。如果您使用的是LDAP并希望运行ART主机测试，请安装libnss-sss:i386 软件包。

##### Ubuntu 16.04
```
sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-dev g++-multilib 
sudo apt-get install -y git flex bison gperf build-essential libncurses5-dev:i386 
sudo apt-get install tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 
sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev
sudo apt-get install git-core gnupg flex bison gperf build-essential  
sudo apt-get install zip curl zlib1g-dev gcc-multilib g++-multilib 
sudo apt-get install libc6-dev-i386 
sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev 
sudo apt-get install libgl1-mesa-dev libxml2-utils xsltproc unzip m4
sudo apt-get install lib32z-dev ccache
```
注意：在安装libx11-dev:i386，libreadline6-dev:i386，libncurses5-dev:i386，zlib1g-dev:i386时也许会出现问题，请忽略

## 编译源码
#### 初始化编译环境
确认了源码的版本并确保上述过程完成后，我们开始初始化编译环境
```
source build/envsetup.sh
```
然后我们用lunch命令选择编译目标。所谓的编译目标就是生成的镜像要运行在什么样的设备上.这里我们设置的编译目标是aosp_arm64-eng,因此执行指令:
```
lunch aosp_arm64-eng
```
也可以只输入lunch，选择目标
```
lunch
```

#### 开始编译

通过make指令进行代码编译,该指令通过-j参数来设置参与编译的线程数量,以提高编译速度.比如这里我们设置8个线程同时编译:

```
make -j8
```
需要注意的是,参与编译的线程并不是越多越好,通常是根据你机器cup的核心来确定:core*2,即当前cpu的核心的2倍.比如,我现在的笔记本是双核四线程的,因此根据公式,最快速的编译可以make -j8.
(通过cat /proc/cpuinfo查看相关cpu信息)

如果一切顺利的化,在几个小时之后,便可以编译完成.看到### make completed successfully (01:18:45(hh:mm:ss)) ###表示你编译成功了


&emsp;


## 运行模拟器
在编译完成之后,就可以通过以下命令运行Android虚拟机了,命令如下:
```
source build/envsetup.sh
lunch(选择刚才你设置的目标版本,比如这里了我选择的是2)
emulator
```
如果你是在编译完后立刻运行虚拟机,由于我们之前已经执行过source及lunch命令了,因此现在你只需要执行命令就可以运行虚拟机:
```
emulator
```

## Ubuntu 16.04 编译android源码 error: unsupported reloc 42|error: unsupported reloc 43
①打开core/clang/HOST_x86——common.mk文件：

在大约15行的位置找到:
```
--sysroot $($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/sysroot
```
将其修改为：
```
--sysroot $($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/sysroot \
-B$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/x86_64-linux/bin
```
若①操作没有解决问题，则进行②操作

②打开 /art/build/Android.common_build.mk文件：
在大约80行的位置找到：
```
# Host.
ART_HOST_CLANG := false
ifneq ($(WITHOUT_HOST_CLANG),true)
  # By default, host builds use clang for better warnings.
  ART_HOST_CLANG := true
endif
```
修改为：
```
# Host.
ART_HOST_CLANG := false
ifeq ($(WITHOUT_HOST_CLANG),false)
  # By default, host builds use clang for better warnings.
  ART_HOST_CLANG := true
endif
```
若②操作仍没有用，则进行③操作

③原先的ld编译错误了，用我们本地的ld对2.11内的ld进行覆盖
```
cp /usr/bin/ld.gold prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.6/x86_64-linux/bin/ld
```
注意查看自己的目录，根据编译版本不同，文件路径可能不同。
根据glibc2.15文件目录是否存在，选择执行④操作
</br></br>
④将2.15内的ld文件重新命名，并设置链接
```
mv prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.6/x86_64-linux/bin/ld prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.6/x86_64-linux/bin/ld.OLD

ln -s /usr/bin/ld.gold prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.6/x86_64-linux/bin/ld
```
</br>
## 在编译完成目录下执行sudo emulator失败
* 进入root权限，重新执行脚本，然后执行lunch，打开emulator
</br></br>
```
source build/envsetup.sh
lunch
emulator
```
