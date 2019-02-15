---
title: aapt命令行使用实践
tags:
---
## 环境
* 操作系统：Mac OS High Sierra 10.13.2 (17C88)
* Android SDK Build Tool版本：28.0.3

## aapt是什么
aapt 即 Android Asset Packaging Tool，它是 Android SDK 自带的一个命令行工具，是 Android 构建系统的一部分。aapt可以查看、创建以及更新兼容 ZIP 格式的归档文件（.zip、.jar、.apk），在 Android 构建流程中，我们使用它来编译项目的资源文件、打包apk。

可以在下面两个地方找到aapt工具：

```bash
Android SDK根目录/build-tools/xx.x.x/
```
  
```bash
AOSP/out/host/darwin-x86/bin/
```  
如果想要知道aapt的实现原理，可以在下面位置阅读相关的源码：
```bash
AOSP/frameworks/base/tools/aapt/
```

## aapt命令行实践
在命令行直接输入aapt，不带任何参数，我们可以看到详细的帮助文档：
```bash
➜ ~ aapt
Android Asset Packaging Tool

Usage:
 aapt l[ist] [-v] [-a] file.{zip,jar,apk}
   List contents of Zip-compatible archive.

 aapt d[ump] [--values] [--include-meta-data] WHAT file.{apk} [asset [asset ...]]
   strings          Print the contents of the resource table string pool in the APK.
   badging          Print the label and icon for the app declared in APK.
   permissions      Print the permissions from the APK.
   resources        Print the resource table from the APK.
   configurations   Print the configurations in the APK.
   xmltree          Print the compiled xmls in the given assets.
   xmlstrings       Print the strings of the given compiled xml assets.

 aapt p[ackage] [-d][-f][-m][-u][-v][-x][-z][-M AndroidManifest.xml] \
        [-0 extension [-0 extension ...]] [-g tolerance] [-j jarfile] \
        [--debug-mode] [--min-sdk-version VAL] [--target-sdk-version VAL] \
        [--app-version VAL] [--app-version-name TEXT] [--custom-package VAL] \
        [--rename-manifest-package PACKAGE] \
        [--rename-instrumentation-target-package PACKAGE] \
        [--utf16] [--auto-add-overlay] \
        [--max-res-version VAL] \
        [-I base-package [-I base-package ...]] \
        [-A asset-source-dir]  [-G class-list-file] [-P public-definitions-file] \
        [-D main-dex-class-list-file] \
        [-S resource-sources [-S resource-sources ...]] \
        [-F apk-file] [-J R-file-dir] \
        [--product product1,product2,...] \
        [-c CONFIGS] [--preferred-density DENSITY] \
        [--split CONFIGS [--split CONFIGS]] \
        [--feature-of package [--feature-after package]] \
        [raw-files-dir [raw-files-dir] ...] \
        [--output-text-symbols DIR]

   Package the android resources.  It will read assets and resources that are
   supplied with the -M -A -S or raw-files-dir arguments.  The -J -P -F and -R
   options control which files are output.

 aapt r[emove] [-v] file.{zip,jar,apk} file1 [file2 ...]
   Delete specified files from Zip-compatible archive.

 aapt a[dd] [-v] file.{zip,jar,apk} file1 [file2 ...]
   Add specified files to Zip-compatible archive.

 aapt c[runch] [-v] -S resource-sources ... -C output-folder ...
   Do PNG preprocessing on one or several resource folders
   and store the results in the output folder.

 aapt s[ingleCrunch] [-v] -i input-file -o outputfile
   Do PNG preprocessing on a single file.

 aapt v[ersion]
   Print program version.
```

为了便于实验，我们新建一个 Android Demo 工程，并生成一个 demo.apk 文件待用。下面对 aapt 工具的命令一一进行实践（注意！！！输出太长的地方加了省略符号）。

### 命令 v
打印aapt的版本号：
```bash
➜ ~ aapt v
Android Asset Packaging Tool, v0.2-5016651
```
### 命令 l
查看兼容 ZIP 格式归档文件的内容，可选的命令参数如下表：

| 参数 | 作用 |
| :--- | :---- | 
| -v | 列出归档文件内容的详细信息，类似'unzip -l -v' | 
| -a | 列出apk文件的资源表以及Android manifest的详细内容 | 
</br>我们使用上面生成的 demo.apk 来查看命令执行结果：
```bash
➜ aapt l app/build/outputs/apk/debug/demo.apk 
AndroidManifest.xml
META-INF/CERT.RSA
META-INF/CERT.SF
META-INF/MANIFEST.MF
assets/assets.txt
classes.dex
res/drawable-anydpi-v21/ic_launcher_background.xml
res/drawable-xxxhdpi-v4/ic_launcher_background.png
res/layout/activity_main.xml
res/mipmap-hdpi-v4/ic_launcher.png
......
res/raw/raw.txt
resources.arsc
```
带-v参数：
```bash
➜ aapt l -v app/build/outputs/apk/debug/demo.apk
Archive:  app/build/outputs/apk/debug/demo.apk
 Length   Method    Size  Ratio   Offset      Date  Time  CRC-32    Name
--------  ------  ------- -----  -------      ----  ----  ------    ----
    2096  Deflate     772  63%    121428  11-30-79 00:00  ffffffffdf4f467e  AndroidManifest.xml
     773  Deflate     603  22%    195990  11-30-79 00:00  09bb8857  META-INF/CERT.RSA
    2006  Deflate     957  52%    194987  11-30-79 00:00  ffffffff89d38cd0  META-INF/CERT.SF
    1969  Deflate     938  52%    193999  11-30-79 00:00  fffffffff46ceacc  META-INF/MANIFEST.MF
      16  Stored       16   0%    112384  11-30-79 00:00  ffffffffe2d2eeaf  assets/assets.txt
  231704  Deflate  112343  52%         0  11-30-79 00:00  ffffffff8149f570  classes.dex
    5696  Deflate     982  83%    122249  11-30-79 00:00  0c9d0923  res/drawable-anydpi-v21/ic_launcher_background.xml
    4708  Stored     4708   0%    160564  11-30-79 00:00  ffffffffc0e2b4a8  res/drawable-xxxhdpi-v4/ic_launcher_background.png
     876  Deflate     389  56%    112525  11-30-79 00:00  029eb45b  res/layout/activity_main.xml
    2963  Stored     2963   0%    133287  11-30-79 00:00  78bc849d  res/mipmap-hdpi-v4/ic_launcher.png
    4905  Stored     4905   0%    125444  11-30-79 00:00  ffffffffac8a9f01  res/mipmap-hdpi-v4/ic_launcher_round.png
......
      17  Stored       17   0%    112456  11-30-79 00:00  fffffffffd969fde  res/raw/raw.txt
    8412  Stored     8412   0%    112972  11-30-79 00:00  ffffffffeb638dbe  resources.arsc
--------          -------  ---                            -------
  323429           195293  40%                            21 files

```
带-a参数：
```bas
➜ aapt l -a app/build/outputs/apk/debug/demo.apk
AndroidManifest.xml
META-INF/CERT.RSA
META-INF/CERT.SF
META-INF/MANIFEST.MF
assets/assets.txt
classes.dex
res/drawable-anydpi-v21/ic_launcher_background.xml
res/drawable-xxxhdpi-v4/ic_launcher_background.png
......
res/raw/raw.txt
resources.arsc

Resource table:
Package Groups (1)
Package Group 0 id=0x7f packageCount=1 name=cn.thismj.demo
  Package 0 id=0x7f name=cn.thismj.demo
    type 0 configCount=1 entryCount=57
      spec resource 0x7f010000 cn.thismj.demo:attr/barrierAllowsGoneWidgets: flags=0x00000000
      spec resource 0x7f010001 cn.thismj.demo:attr/barrierDirection: flags=0x00000000
      ......
      spec resource 0x7f010038 cn.thismj.demo:attr/layout_optimizationLevel: flags=0x00000000
      config (default):
        resource 0x7f010000 cn.thismj.demo:attr/barrierAllowsGoneWidgets: <bag>
        resource 0x7f010001 cn.thismj.demo:attr/barrierDirection: <bag>
        ......
        resource 0x7f010037 cn.thismj.demo:attr/layout_goneMarginTop: <bag>
        resource 0x7f010038 cn.thismj.demo:attr/layout_optimizationLevel: <bag>
    ......
    type 2 configCount=2 entryCount=1
      spec resource 0x7f030000 cn.thismj.demo:drawable/ic_launcher_background: flags=0x00000500
      config xxxhdpi:
        resource 0x7f030000 cn.thismj.demo:drawable/ic_launcher_background: t=0x03 d=0x00000002 (s=0x0008 r=0x00)
      config anydpi-v21:
        resource 0x7f030000 cn.thismj.demo:drawable/ic_launcher_background: t=0x03 d=0x00000001 (s=0x0008 r=0x00)
    ......
    type 8 configCount=1 entryCount=1
      spec resource 0x7f090000 cn.thismj.demo:style/AppTheme: flags=0x00000000
      config (default):
        resource 0x7f090000 cn.thismj.demo:style/AppTheme: <bag>

Android manifest:
N: android=http://schemas.android.com/apk/res/android
  E: manifest (line=2)
    A: android:versionCode(0x0101021b)=(type 0x10)0x1
    A: android:versionName(0x0101021c)="1.0" (Raw: "1.0")
    ......
    E: uses-sdk (line=7)
      A: android:minSdkVersion(0x0101020c)=(type 0x10)0x13
      A: android:targetSdkVersion(0x01010270)=(type 0x10)0x1c
    E: application (line=11)
      A: android:theme(0x01010000)=@0x7f090000
      A: android:label(0x01010001)=@0x7f080000
      ......
      A: android:roundIcon(0x0101052c)=@0x7f060001
      E: activity (line=20)
        A: android:name(0x01010003)="cn.thismj.demo.MainActivity" (Raw: "cn.thismj.demo.MainActivity")
        E: intent-filter (line=21)
          E: action (line=22)
            A: android:name(0x01010003)="android.intent.action.MAIN" (Raw: "android.intent.action.MAIN")
          E: category (line=24)
            A: android:name(0x01010003)="android.intent.category.LAUNCHER" (Raw: "android.intent.category.LAUNCHER")
```
### 命令 d
指定不同的子命令dump出对应的内容，如下表：

| 子命令 | 作用 |
| :--- | :---- | 
| strings | 列出apk中资源表的字符串池 |
|  badging | 列出apk中清单文件声明的属性标签跟应用图标 | 
|  permissions | 列出apk清单文件声明的权限 | 
|  resources | 列出apk文件的资源表 |
| configurations | 列出apk资源声明的所有配置|
| xmltree | 解析并打印指定被编译的二进制xml文件的元素 |
| xmlstrings | 解析并打印指定被编译的二进制xml文件的字符串池 |

#### strings
```bash
➜ aapt d strings app/build/outputs/apk/debug/demo.apk
String pool of 15 unique UTF-8 non-sorted strings, 15 entries and 0 styles using 664 bytes:
String #0: Demo
String #1: res/drawable-anydpi-v21/ic_launcher_background.xml
String #2: res/drawable-xxxhdpi-v4/ic_launcher_background.png
String #3: res/layout/activity_main.xml
String #4: res/mipmap-hdpi-v4/ic_launcher.png
......
String #12: res/mipmap-xxxhdpi-v4/ic_launcher.png
String #13: res/mipmap-xxxhdpi-v4/ic_launcher_round.png
String #14: res/raw/raw.txt
```
#### badging
```bash
➜ aapt d badging app/build/outputs/apk/debug/demo.apk
package: name='cn.thismj.demo' versionCode='1' versionName='1.0' compileSdkVersion='28' compileSdkVersionCodename='9'
sdkVersion:'19'
targetSdkVersion:'28'
uses-permission: name='android.permission.WRITE_EXTERNAL_STORAGE'
uses-permission: name='android.permission.INTERNET'
application-label:'Demo'
application-icon-160:'res/mipmap-mdpi-v4/ic_launcher.png'
application-icon-240:'res/mipmap-hdpi-v4/ic_launcher.png'
application-icon-320:'res/mipmap-xhdpi-v4/ic_launcher.png'
application-icon-480:'res/mipmap-xxhdpi-v4/ic_launcher.png'
application-icon-640:'res/mipmap-xxxhdpi-v4/ic_launcher.png'
application-icon-65534:'res/mipmap-mdpi-v4/ic_launcher.png'
application: label='Demo' icon='res/mipmap-mdpi-v4/ic_launcher.png'
testOnly='-1'
application-debuggable
launchable-activity: name='cn.thismj.demo.MainActivity'  label='' icon=''
uses-permission: name='android.permission.READ_EXTERNAL_STORAGE'
uses-implied-permission: name='android.permission.READ_EXTERNAL_STORAGE' reason='requested WRITE_EXTERNAL_STORAGE'
feature-group: label=''
  uses-feature: name='android.hardware.faketouch'
  uses-implied-feature: name='android.hardware.faketouch' reason='default feature for all apps'
main
supports-screens: 'small' 'normal' 'large' 'xlarge'
supports-any-density: 'true'
locales: '--_--'
densities: '160' '240' '320' '480' '640' '65534'
```
#### permissions
```bash
➜ aapt d permissions app/build/outputs/apk/debug/demo.apk
package: cn.thismj.demo
uses-permission: name='android.permission.WRITE_EXTERNAL_STORAGE'
uses-permission: name='android.permission.INTERNET'
```
#### resources
```bash
➜ aapt d resources app/build/outputs/apk/debug/demo.apk
Package Groups (1)
Package Group 0 id=0x7f packageCount=1 name=cn.thismj.demo
  Package 0 id=0x7f name=cn.thismj.demo
    type 0 configCount=1 entryCount=57
      spec resource 0x7f010000 cn.thismj.demo:attr/barrierAllowsGoneWidgets: flags=0x00000000
       ......
      spec resource 0x7f010038 cn.thismj.demo:attr/layout_optimizationLevel: flags=0x00000000
      config (default):
        resource 0x7f010000 cn.thismj.demo:attr/barrierAllowsGoneWidgets: <bag>
        ......
        resource 0x7f010038 cn.thismj.demo:attr/layout_optimizationLevel: <bag>
    ......
    type 8 configCount=1 entryCount=1
      spec resource 0x7f090000 cn.thismj.demo:style/AppTheme: flags=0x00000000
      config (default):
        resource 0x7f090000 cn.thismj.demo:style/AppTheme: <bag>
```
#### configurations
```bash
➜ aapt d configurations app/build/outputs/apk/debug/demo.apk

mdpi
hdpi
xhdpi
xxhdpi
xxxhdpi
anydpi-v21
```
#### xmltree
```bash
➜ aapt d xmltree app/build/outputs/apk/debug/demo.apk res/layout/activity_main.xml
N: android=http://schemas.android.com/apk/res/android
  N: app=http://schemas.android.com/apk/res-auto
    E: android.support.constraint.ConstraintLayout (line=2)
      A: android:layout_width(0x010100f4)=(type 0x10)0xffffffff
      A: android:layout_height(0x010100f5)=(type 0x10)0xffffffff
      E: TextView (line=9)
        A: android:layout_width(0x010100f4)=(type 0x10)0xfffffffe
        A: android:layout_height(0x010100f5)=(type 0x10)0xfffffffe
        A: android:text(0x0101014f)="Hello World!" (Raw: "Hello World!")
        A: app:layout_constraintBottom_toBottomOf(0x7f01000c)=(type 0x10)0x0
        A: app:layout_constraintLeft_toLeftOf(0x7f01001f)=(type 0x10)0x0
        A: app:layout_constraintRight_toRightOf(0x7f010023)=(type 0x10)0x0
        A: app:layout_constraintTop_toTopOf(0x7f010028)=(type 0x10)0x0
```
#### xmlstrings
```bash
➜ aapt d xmlstrings app/build/outputs/apk/debug/demo.apk res/layout/activity_main.xml
String pool of 14 unique UTF-8 non-sorted strings, 14 entries and 0 styles using 436 bytes:
String #0: layout_width
String #1: layout_height
String #2: text
String #3: layout_constraintBottom_toBottomOf
String #4: layout_constraintLeft_toLeftOf
String #5: layout_constraintRight_toRightOf
String #6: layout_constraintTop_toTopOf
String #7: Hello World!
String #8: TextView
String #9: android
String #10: android.support.constraint.ConstraintLayout
String #11: app
String #12: http://schemas.android.com/apk/res-auto
String #13: http://schemas.android.com/apk/res/android
```
### 命令 p
Android构建流程就是用这个命令来编译资源打包apk的，可选参数如下：

| 参数 | 作用 |
| :--- | :---- | 
| -d | 列出归档文件内容的详细信息，类似'unzip -l -v' | 
| -f | 列出apk文件的资表以及Android manifest的详细内容 | 
| -f | 列出apk文件的资表以及Android manifest的详细内容 |
| -m | 列出apk文件的资表以及Android manifest的详细内容 |
| -u | 列出apk文件的资表以及Android manifest的详细内容 |
| -v | 列出apk文件的资表以及Android manifest的详细内容 |
| -x | 列出apk文件的资以及Android manifest的详细内容 |
| -z | 列出apk文件的资表以及Android manifest的详细内容 |
| -M | 列出apk文件的资以及Android manifest的详细内容 |
















