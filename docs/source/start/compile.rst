编译镜像
============

以 OpenHarmony 5.0.1 编译 RK3568 镜像 为例

安装WSL ubuntu虚拟机
-------------------------

powershell 管理员

.. note::

    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

或者通过开启windows功能: hyper-v，适用于linux的windows子系统

 
查看版本 

.. note::

    wsl -l -o 

安装 

.. note::

    wsl --install Ubuntu-20.04  

或者在windows store里安装

 
迁移到D盘：

.. note::

    wsl --shutdown

    wsl --export Ubuntu-20.04 d:\ub.tar

    wsl --unregister Ubuntu-20.04 

    wsl --import Ubuntu-20.04 d:\software\vm\ubuntu\  d:\ub.tar   


Ubuntu 环境
---------------

设置apt源 

.. note::

    sudo sed -i "s@http://.*archive.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list

    sudo sed -i "s@http://.*security.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list

 
切换至bash

.. note::

     dpkg-reconfigure dash


配置gitee
-------------

注册 gitee账号，登录。

配置ssh key

.. note::

    ssh-keygen -t rsa

在 https://gitee.com/profile/sshkeys 配置ssh公钥

打开 https://gitee.com/openharmony/manifest

从克隆-> SSH 中找到git config，配置user.name，user.email

.. note::

    git config --global user.name 'xxx'

    git config --global user.email 'xxx@xxx.com'

    git config --global credential.helper store


安装repo
------------

.. note::

    curl https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 | sudo tee /usr/local/bin/repo >/dev/null

    sudo chmod a+x /usr/local/bin/repo


安装apt包
-----------

 apt包列表参考 [OpenHamony 5.0.1编译纠错指南 - Withm - 博客园](https://www.cnblogs.com/Luvm/p/18619092)

.. note::

    apt-get update

    apt-get -f -y install apt-utils vim software-properties-common openssh-server iputils-ping curl net-tools bsdmainutils kmod bc rsync gawk ssh ccache zip python-dev make m4 gcc-multilib ca-certificates-java unzip python3-yaml perl openssl libssl1.1 gnupg xsltproc x11proto-core-dev tcl python3-crypto python-crypto libxml2-utils libxml2-dev libx11-dev libssl-dev libgl1-mesa-dev lib32z1-dev lib32ncurses5-dev g++-multilib flex bison doxygen git subversion tofrodos pigz expect python3-xlrd git-core gperf build-essential zlib1g-dev libc6-dev-i386 lib32z-dev openjdk-8-jdk ruby mtools python3-pip gcc-arm-linux-gnueabi genext2fs liblz4-tool libssl-dev autoconf pkg-config zlib1g-dev libglib2.0-dev libmount-dev libpixman-1-dev libncurses5-dev exuberant-ctags silversearcher-ag libtinfo5 device-tree-compiler libssl-dev libelf-dev dwarves gcc-arm-none-eabi default-jdk u-boot-tools mtd-utils scons automake libtinfo5 gcc-multilib libtool libgmp-dev texinfo mpc autotools-dev libmpc-dev libmpfr-dev libgmp-dev patchutils libexpat-dev libfdt-dev libncursesw5-dev cmake wget libelf-dev

 

设置python3
-------------

.. note::

    update-alternatives --install /usr/bin/python python /usr/bin/python3.8 1

    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1 

 

下载openharmony代码
----------------------

.. note::

    cd ~

    mkdir ohos

    cd ohos

    repo init -u git@gitee.com:openharmony/manifest.git -b OpenHarmony-5.1.0-Release --no-repo-verify

    repo sync -c -j4

    repo forall -c 'git lfs pull'

    repo sync -c -j4 --force-sync 


准备编译环境
--------------

.. note::

    bash ./build/prebuilts_download.sh


编译RK3568镜像
-----------------

.. note::

    ./build.sh --product-name rk3568 --ccache --no-prebuilt-sdk 

 
编译后在out/rk3568/packages/phone/images目录下

AMD Ryzen 9 5900，64G内存，编译一次约3小时 


编译问题：ohos-sdk
--------------------------

ninja: error: '../../prebuilts/ohos-sdk/linux/15/native/sysroot/usr/lib/arm-linux-ohos/libbundle_ndk.z.so', needed by 'web/webview/libarkweb_os_adapter.z.so', missing and no known rule to make it

    cd prebuilts/ohos-sdk/linux/

ls发现版本是20，不是15

    mkdir prebuilts/ohos-sdk/linux/15


在OpenHarmony-v5.0.3-release.md下载对应版本的标准系统full sdk或public sdk包，解压到 prebuilts/ohos-sdk/linux/15目录下


或者不mkdir，直接

    ln -sf 20 15


编译问题：permission_manager/dlp_manager
-----------------------------------------------

    [63333/79843] ACTION //applications/standard/permission_manager:permission_manager_compile_app(//build/toolchain/ohos:ohos_clang_arm)

    FAILED: obj/applications/standard/permission_manager/permission_manager/unsigned_hap_path_list.json

    /usr/bin/env ../../build/scripts/compile_app.py --nodejs ../../prebuilts/build-tools/common/nodejs/node-v16.20.2-linux-x64/bin/node --cwd ../../applications/standard/permission_manager/ --build-profile ../../applications/standard/permission_manager/build-profile.json5 --sdk-home /root/ohos/prebuilts/ohos-sdk/linux --output-file obj/applications/standard/permission_manager/permission_manager/unsigned_hap_path_list.json --ohpm-registry  --build-level module --assemble-type assembleHap --sdk-type-name sdk.dir --build-modules permissionmanager --hvigor-obfuscation

    build_profile:../../applications/standard/permission_manager/build-profile.json5; cwd:/root/ohos/applications/standard/permission_manager

    modules_list:[{'name': 'entry', 'srcPath': './entry', 'targets': [{'name': 'default', 'applyToProducts': ['default']}]}, {'name': 'permissionmanager', 'srcPath': './permissionmanager', 'targets': [{'name': 'default', 'applyToProducts': ['default']}]}]

 
编辑  applications/standard/permission_manager/build-profile.json5 文件，将其中的版本号从15改成20

       "compileSdkVersion": 20,

       "compatibleSdkVersion": 20,

       "targetSdkVersion": 20,

再单独编译permission_manager应用

.. note::

    ./build.sh --product-name rk3568 --build-target permission_manager


dlp_manager的处理与permission_manager类似。


编译问题：signal_handler encaps.json invalid
--------------------------------------

FileExistsError: [Errno kernel_permission json file /root/ohos/out/rk3568/../../base/hiviewdfx/faultloggerd/interfaces/innerkits/signal_handler/encaps.json invalid!] 0001

之前 OpenHarmony-5.0.3-Release 提示找不到文件，到hiviewdfx_faultloggerd

目录下看5.1.0-Release有，升上去，或者单独下载该模块。


编译问题：ninja lexing error
----------------------------

 [OHOS ERROR] [NINJA] ninja: error: toolchain.ninja:177177: expected newline, got lexing error

 rule __build_common_libcpp_libc++_shared.so___build_toolchain_ohos_ohos_clang_arm__rule

 把rule中的++替换成其他字符，例如__
