---
title: Mac OS之Android源码下载、编译、刷机
date: 2018-09-13 09:25:00
author: ThisMJ
img: /medias/images/2018-09-13.jpeg 
top: true
mathjax: false
categories: Android
tags:
  - AOSP
---
## 软硬件环境
* 系统版本：Mac OS High Sierra 10.13.2 (17C88)
* 刷机硬件：Nexus 6
* AOSP分支：android-7.1.1_r57（NGI77B）

## 搭建Mac编译环境
通过shell使用以下命令创建大小写敏感的稀疏磁盘映像（官网写的40g是远远不够的）:
```bash
hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 150g ~/android.dmg
```
在~/.bash_profile添加以下两个辅助函数，记得source一下：
```bash
# 装载Android磁盘映像
mountAndroid() { hdiutil attach ~/android.dmg.sparseimage -mountpoint /Volumes/android; }
    
# 卸载Android磁盘映像
umountAndroid() { hdiutil detach /Volumes/android; }
```
安装 Xcode 软件，可以选择 8.3.3 版本，安装最新的 9.x 后面编译会有问题。
安装 Xcode 命令行工具：
```bash
xcode-select --install
```
安装[MacPorts](https://www.macports.org/install.php)，确保 /opt/local/bin 在路径中显示在 /usr/bin 前面：
```bash
export PATH=/opt/local/bin:$PATH
```
通过 MacPorts 获取 Make、Git 和 GPG 软件包：
```bash
POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg
```
然后设置文件描述符数量上限：
```bash
ulimit -S -n 1024
```

## 下载AOSP(Android Open Source Project)
装载Android磁盘映像：
```bash
mountAndroid
```
安装配置repo，并初始化repo项目：
```bash
cd ~
mkdir ~/bin
export PATH=~/bin:$PATH

curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

cd /Volumes/android
mkdir AOSP
cd AOSP
git config --global user.name "Heyruad.Towne"
git config --global user.email "Heyruad.Towne@gmail.com"

#可以翻墙（Google）
repo init -u https://android.googlesource.com/platform/manifest -b android-7.1.1_r57
#翻不了墙（清华AOSP镜像）
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-7.1.1_r57
```
下载同步源代码，大概30多个g，这个时候只能慢慢等，公司网比较慢，花了10多个小时，家里不到3小时搞定：
```bash
repo sync
```

## 编译源代码
下载对应分支（android-7.1.1_r57）、对应设备（Nexus 6）的二进制驱动文件，如下图：
![](http://7xrpoc.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-09-13%20%E4%B8%8B%E5%8D%889.42.21.png)
下载完成后解压到AOSP源代码目录，得到3个sh脚本，依次执行，最后一行输入"I ACCEPT"同意许可证条款，二进制文件及其对应的 Makefile 将会安装在AOSP的vendor目录下

执行编译前清理：
```bash
make clobber
```
初始化编译环境：
```bash
source build/envsetup.sh
```
lunch选择[编译目标](https://source.android.com/setup/build/running.html#selecting-device-build)，因为我用的是 Nexus 6，所以执行命令如下：
```bash
lunch aosp_shamu-userdebug
```
使用 make 命令编译代码，借助 -jN 参数指定同时编译的任务数量，一般设置为CPU支持并发线程总数的1-2倍
```bash
make -j8
```
编译完成后终端输出，喜闻乐见只花了1小时4分25秒：
```bash
#### make completed successfully (01:04:25 (hh:mm:ss)) ####
```

## 刷机
首先在开发者模式下打开OEM解锁：
![oem-w320](http://7xrpoc.com1.z0.glb.clouddn.com/device-2018-09-14-110545.png)
adb进入 fastboot 模式：
```bash
adb reboot bootloader
```
解锁 bootloader：
```bash
fastboot oem unlock
```
刷机，-w 选项会清除 /data 分区：
```bash
fastboot flashall -w
```
不到1分钟，刷机成功，喜大普奔！检查下系统版本：
![oem-w320](http://7xrpoc.com1.z0.glb.clouddn.com/device-2018-09-14-121427.png)
## 坑坑洼洼
1、踩坑之一

错误：
```bash
Error: gnupg has been deprecated. If you absolutely want to stay on the classic version, install the gnupg1 port. All other users are recommended to install gnupg2.
Error: Failed to configure gnupg: obsolete port
Error: See /opt/local/var/macports/logs/_opt_local_var_macports_sources_rsync.macports.org_macports_release_tarballs_ports_mail_gnupg/gnupg/main.log for details.
Error: Follow https://guide.macports.org/#project.tickets to report a bug.
Error: Processing of port gnupg failed
```
解决办法：卸载gnupg1，安装gnupg2：
```bash
POSIXLY_CORRECT=1 sudo port uninstall gnupg1   
POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg2  
```
——————
2、踩坑之二
原因：AOSP比较庞大，repo sync容易发生错误导致同步中断，使用shell脚本
解决办法：使用sh脚本
```bash
#!/bin/sh
repo sync
while [ $? -ne 0 ]
do
repo sync
done
```
——————
3、踩坑之三

错误：
```bash
build/core/combo/mac_version.mk:26: none of the installed SDKs (ac_sdk_versions_installed) match supported versions (10.8 10.9 10.10 10.11), trying 10.8
build/core/combo/mac_version.mk:36: no SDK 10.8 at /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.8.sdk, trying legacy dir
build/core/combo/mac_version.mk:40: *****************************************************
build/core/combo/mac_version.mk:41: * Can not find SDK 10.8 at /Developer/SDKs/MacOSX10.8.sdk
build/core/combo/mac_version.mk:42: *****************************************************
build/core/combo/mac_version.mk:43: *** Stop..  Stop.
```
解决办法：

查看build/core/combo/mac_version.mk:
```bash
mac_sdk_versions_supported :=  10.8 10.9 10.10 10.11
```
下载[MacOSX10.11.sdk](https://github.com/phracker/MacOSX-SDKs/releases/download/10.13/MacOSX10.11.sdk.tar.xz)，完成后解压，放在下面两个目录：
```
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/
```
```
~/Document/
```
最后创建软链接：
```bash
ln -s ~/Document/MacOSX10.11.sdk /Applications/XCode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk
```
——————
4、踩坑之四

错误：
```bash
build/core/config.mk:600: *** Error: could not find jdk tools.jar at /System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/../lib/tools.jar, please check if your JDK was installed correctly.  Stop.
```
解决办法：设置ANDROID_JAVA_HOME环境变量
```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_172.jdk/Contents/Home
export ANDROID_JAVA_HOME=$JAVA_HOME
```
——————
5、踩坑之五

错误：
```bash
ninja: Entering directory `.'
ninja: warning: multiple rules generate out/target/product/shamu/system/etc/gps.conf. builds involving this target will not be correct; continuing anyway [-w dupbuild=warn]
[  0% 22/33752] Yacc: aidl <= system/tools/aidl/aidl_language_y.yy
FAILED: /bin/bash -c "prebuilts/misc/darwin-x86/bison/bison -d  --defines=out/host/darwin-x86/obj/STATIC_LIBRARIES/libaidl-common_intermediates/aidl_language_y.h -o out/host/darwin-x86/obj/STATIC_LIBRARIES/libaidl-common_intermediates/aidl_language_y.cpp system/tools/aidl/aidl_language_y.yy"
[  0% 22/33752] host Java: tagsouplib (out/host/common/obj/JAVA_LIBRARIES/tagsouplib_intermediates/classes)
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。
[  0% 22/33752] host Java: antlr-runtime (out/host/common/obj/JAVA_LIBRARIES/antlr-runtime_intermediates/classes)
注: 某些输入文件使用了未经检查或不安全的操作。
注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。
ninja: build stopped: subcommand failed.
make: *** [ninja_wrapper] Error 1

#### make failed to build some targets (38 seconds) ####
```
解决办法：
编辑prebuilts/sdk/tools/jack-admin文件
```bash
JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -cp $LAUNCHER_JAR $LAUNCHER_NAME"
```
修改为
```bash
JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -Xmx4096m -cp $LAUNCHER_JAR $LAUNCHER_NAME"
```
然后依次执行：
```bash
./prebuilts/sdk/tools/jack-admin stop-server
./prebuilts/sdk/tools/jack-admin start-server
```
——————
6、踩坑之六

错误：
```bash
Jack server installation not found
```
解决办法：
```bash
./prebuilts/sdk/tools/jack-admin install-server jack-launcher.jar jack-server-4.8.ALPHA.jar 
```
执行命令后提示警告，暂时没管。
```bash
Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore /Users/aero.tang/.jack-server/server.jks -destkeystore /Users/aero.tang/.jack-server/server.jks -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore /Users/aero.tang/.jack-server/client.jks -destkeystore /Users/aero.tang/.jack-server/client.jks -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。
```
——————
7、踩坑之七

错误：
```bash
external/libcxx/include/cstdlib:167:44: error: declaration conflicts with target of using declaration already in scope
external/libcxx/include/cstdlib:169:44: error: declaration conflicts with target of using declaration already in scope
external/libcxx/include/cstdlib:172:42: error: declaration conflicts with target of using declaration already in scope
external/libcxx/include/cstdlib:174:42: error: declaration conflicts with target of using declaration already in scope
......
```
解决办法：见[stackoverflow](https://stackoverflow.com/questions/47060910/error-declaration-conflicts-with-target-of-using-declaration-already-in-scope)，Xcode降级到8.3.3
——————
8、踩坑之八

原因：High Sierra系统下bison有关于动态格式化字符串的bug，具体见[Bison Patch](https://android-review.googlesource.com/c/platform/external/bison/+/517740)
```bash
FAILED: /bin/bash -c "prebuilts/misc/darwin-x86/bison/bison -d  --defines=out/host/darwin-x86/obj/STATIC_LIBRARIES/libaidl-common_intermediates/aidl_language_y.h -o out/host/darwin-x86/obj/STATIC_LIBRARIES/libaidl-common_intermediates/aidl_language_y.cpp system/tools/aidl/aidl_language_y.yy"
```
解决办法：AOSP源代码目录执行
```bash
cd /external/bison
git cherry-pick c0c852bd6fe462b148475476d9124fd740eba160
mm
cp /out/host/darwin-x86/bin/bison /prebuilts/misc/darwin-x86/bison/
```

## 参考文档
1. [搭建编译环境](https://source.android.com/setup/build/initializing)
2. [代号、标签和版本号](https://source.android.com/setup/start/build-numbers.html#source-code-tags-and-builds)
3. [下载源代码](https://source.android.com/setup/build/downloading)
4. [准备编译](https://source.android.com/setup/build/building)
5. [开始运行](https://source.android.com/setup/build/building#run-it)
6. [Repo 命令参考资料](https://source.android.com/setup/develop/repo.html)
7. [Nexus、Pixel设备的二进制驱动文件列表](https://developers.google.com/android/drivers)






