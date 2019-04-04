---
title: AAPT源码分析（二）
categories: Android
tags:
- AAPT
---
## 准备
此篇接着 [AAPT源码分析（一）](http://thismj.cn/2019/03/15/aapt-yuan-ma-fen-xi-yi/) 继续分析 AAPT 的资源编译打包流程。

在资源编译之前，首先需要收集所有有效的资源文件，AAPT 用一个 AaptAssets 对象来描述收集到的资源集合。AaptAssets 类继承于 AaptDir 类，是一种树的递归层级结构。我们先看一下涉及到相关类的类图：

{% plantuml %}
@startuml
skinparam roundcorner 8
skinparam backgroundColor #FFFFFE
skinparam monochrome true

class AaptDir{
 -mFiles: DefaultKeyedVector<String8, sp<AaptGroup>>
 -mDirs: DefaultKeyedVector<String8, sp<AaptDir>>
}
class AaptAssets{
 -mGroupEntries: SortedVector<AaptGroupEntry>
 -mResDirs: Vector<sp<AaptDir>>
 -mIncludedAssets: AssetManager
 -mOverlay: sp<AaptAssets>
 -mRes: KeyedVector<String8, sp<ResourceTypeSet>>*
 -......
}

class AaptGroup{
  -......
  -mFiles: DefaultKeyedVector<AaptGroupEntry, sp<AaptFile>>
}

class AaptGroupEntry{
 -mParams: ConfigDescription
}
class AaptFile{
 -mPath: String8
 -mGroupEntry: AaptGroupEntry
 -mResourceType: String8
 -......
}

class KeyedVector<String8,sp<AaptGroup>>{
}

class ResourceTypeSet{
}

class ConfigDescription{
}

class ResTable_config{
 -mcc: uint16_t
 -mnc: uint16_t
 -locale: uint32_t
 -orientation: uint8_t
 -touchscreen: uint8_t
 -density: uint16_t
 -keyboard: uint8_t
 -screenSize: uint32_t
 -version: uint32_t
 -......
}

KeyedVector <|-- ResourceTypeSet
AaptAssets *-- ResourceTypeSet
AaptFile --> AaptGroupEntry
AaptDir <|-- AaptAssets
ResTable_config <|-- ConfigDescription
AaptGroupEntry --> ConfigDescription
AaptFile --* AaptGroup
AaptGroup --* AaptDir

@enduml
{% endplantuml %}

上面类图中有一些 Android 源码里面定义的集合和工具类，其中，KeyedVector 是 Android 封装的一种用来储存 Key-Value 格式的数据结构，类似于 Java 的 Map，但是内部是通过动态数组来实现的；DefaultKeyedVector 继承于 KeyedVector，当找不到对应的 Key 时，会返回一个设定的默认值；sp 是 Android 提供的一种智能指针，在普通指针上加了一层封装，通过引用计数法来自动管理内存的回收。

在分析源码之前，我们需要对上图中相关的类以及其成员变量做一个说明，以此来了解一个 AaptAssets 对象是如何储存收集到的资源文件的。

---
一个 AaptFile 对象描述一个资源文件，例如 assets 目录下的一个 .db 文件、res/drawable 目录下的一张 PNG 图片等，AaptFile 类的部分成员变量如下表：

| 变量名  | 类型 | 描述 |
| --- | --- | --- |
| mPath | String8 | 资源文件的路径（相对于aapt执行的位置）|
| mGroupEntry | AaptGroupEntry | 资源文件的配置信息|
| mResourceType | String8 | 资源文件的类型（drawable、layout等）|
---
一个 AaptGroupEntry 对象用来描述一个资源文件的[配置信息](https://developer.android.com/guide/topics/resources/providing-resources.html#AlternativeResources)，它只有一个类型为 ConfigDescription 的 mParams 成员变量，而 ConfigDescription 类继承于 ResTable_config，ResTable_config 才是真正保存配置信息的类。另外，AaptGroupEntry、ConfigDescription 都重载了 >、<、== 等操作符，方便对象进行比较。ResTable_config 类的部分信息如下表：

| 变量名  | 类型 | 描述 |
| --- | --- | --- |
| mcc | uint16_t | 移动国家代码 |
| mnc | uint16_t | 移动网络代码 |
| locale | uint32_t | 语言区域 |
| orientation | uint8_t | 布局方向 |
| density | uint16_t | 屏幕像素密度 |
---
一个 AaptGroup 对象用来描述一个同名同类型但是不同配置信息的资源文件集合，内部用一个类型为 DefaultKeyedVector 的 mFiles 成员变量来保存。例如项目中有 3 个类型为 drawable，名字为 a.png 的不同配置信息的资源文件：
```
res/drawable/a.png
res/drawable-xxhdpi/a.png
res/drawable-port-hdpi/a.png
``` 
则会存在有一个 AaptGroup 对象来保存这 3 个资源文件，Key 为 a.png 对应的 AaptGroupEntry 对象，Value 为 a.png 对应的 AaptFile 对象。AaptGroup 类的成员变量如下表：

| 变量名  | 类型 | 描述 |
| --- | --- | --- |
| mFiles | DefaultKeyedVector <AaptGroupEntry, sp<AaptFile\>> | 同名同类型但是不同配置信息的资源文件集合 |
---
一个 AaptDir 对象用来描述一个同类型的资源文件集合，例如以下 3 个资源文件夹对应了一个类型为 drawable 的 AaptDir 对象：
```
res/drawable
res/drawable-xxhdpi
res/drawable-port-hdpi
``` 
需要注意的是，AaptDir 类是一种树形结构，它会递归地去收集遵循命名规范的子文件夹的资源。AaptDir 类的成员变量如下表：

| 变量名  | 类型 | 描述 |
| --- | --- | --- |
| mFiles | DefaultKeyedVector<String8, sp<AaptGroup\>> | 同类型资源的 AaptGroup 的集合，Key 为资源文件名 |
| mDirs | DefaultKeyedVector<String8, sp<AaptDir\>> | 子文件夹 AaptDir 的集合，Key 为资源类型名 |
---
一个 ResourceTypeSet 对象描述了一个 res 类型的资源文件集合，有几种 res 类型的资源就有几个 ResourceTypeSet 对象，列举所有的 res 资源类型如下：

```
anim
animator
interpolator
transition
drawable
layout
values
xml
raw
color
menu
mipmap
```
ResourceTypeSet 类继承于 KeyedVector<String8,sp<AaptGroup\>>，它跟 AaptDir  类是有本质区别的，AaptDir 类是递归地描述所有的资源文件，包括 assets、res类型，而ResourceTypeSet 只收集 res 类型的资源。实际上， AaptAssets 是先把所有资源文件都收集到一个 类型为 DefaultKeyedVector<String8, sp<AaptGroup\>> 的 mFiles 变量以及一个类型为 DefaultKeyedVector<String8, sp<AaptDir\>> 的 mDirs 变量里面，并同时把 res 类型的 AaptDir 集合保存在一个类型为 Vector<sp<AaptDir\>> 的 mResDirs 变量里面，然后才从 mResDirs 变量里面把资源收集到一个类型为 KeyedVector<String8, sp<ResourceTypeSet\>> 的 mRes 指针变量里面，AaptAssets 类的部分成员变量如下表：

| 变量名  | 类型 | 描述 |
| --- | --- | --- |
| mFiles | DefaultKeyedVector<String8, sp<AaptGroup\>> | 从 AaptDir 类继承 |
| mDirs | DefaultKeyedVector<String8, sp<AaptDir\>> | 从 AaptDir 类继承 |
| mGroupEntries | SortedVector<AaptGroupEntry> | 顺序收集到的所有 AaptGroupEntry 的集合 |
| mResDirs | Vector<sp<AaptDir>> | mDirs 变量中 Key 为 res 类型名的 AaptDir 对象同时会保存进这个变量里面 |
| mIncludedAssets | AssetManager | 用来解析查询引用资源包的（"-I" 参数指定的资源包）|
| mOverlay | AaptAssets | Overlay资源（ "-S" 参数指定的多个资源路径） |
| mRes | KeyedVector<String8, sp<ResourceTypeSet\>>* | 收集 res 类型的资源集合 |
---

## 源码分析











