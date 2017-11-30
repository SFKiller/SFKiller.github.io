---
title: Android N New Features———Multi-Window
date: 2016-02-22 20:43:01
tags: Android-N MultiWindow
---
最近在研究Android N的新特性，对于Android App开发者有很多需要关注的更新和改变，包括一些广播*（CONNECTIVITY_CHANGE、ACTION_NEW_PICTURE、ACTION_NEW_VIDEO）*的使用的限制，权限的改动*（GET_ACCOUNTS、ACTION_OPEN_EXTERNAL_DIRECTORY）*，Doze模式的完善，JobScheduler机制，多窗口模式(Multi-Window)等。这里记录下多窗口模式的开发使用以及遇到的问题。

##**编译和运行环境配置**
研究Android N的相关特性就需要在Android N的环境下：
###编译环境：

 - **Java 8**
 - **android:targetSdkVersion="N"**  *//(在AndroidManifest.xml中)*
 - **compileSdkVersion 'android-N'**  *//(在build.gradle中)*
 - **targetSdkVersion 'N'**  *//(在build.gradle中)*
 - **buildToolsVersion '24.0.0 rc3'**  *//在build.gradle中，这个配24.0.0 rc1及以上就行)*
 - **classpath 'com.android.tools.build:gradle:2.1.0'**  *//gradle最新的是2.2，只要配2.1.0以及以上版本就可以*

###运行环境：

 - **手机：** Nexus 6
 - **系统：** Android N
 - **版本号：** NPD35K 
 
##**Multi-Window介绍**

Android N 支持同时显示多个应用窗口。也就是说一块屏幕可以同时显示多个Activity，而且每个Activity窗口可在指定的范围内放大和缩小。Android N的多窗口模式有三种，分别是*分屏模式（split-screen）、自由模式（freeform）和画中画模式（Picture-in-Picture）*。

一般手机只支持分屏模式，这里也只研究分屏模式，其他两个模式一般在Android Pad和Android TV支持。所以本文下面提到的MultiWindow模式或者多窗口模式就是指*分屏模式*。

##**MultiWindow配置**
想让你的App支持多窗口模式，这里分targetSDKVersion为N和targetSDKVersion低于N两种情况，本文的研究是基于targetSDKVersion为N的情况，对于targetSDKVersion低于N的情况只做简单说明。

 - **targetSDKVersion低于N**
对于设置targetSDKVersion低于N的App，如果App能支持横竖屏旋转，则这个App能支持分屏模式(split-screen)，但是不是完美适配，在分屏模式下打开时系统会有Toast提示*“应用可能无法在分屏模式下正常运行”*。如果你的App不支持横竖屏旋转，那么这个App不支持分屏模式，在分屏模式下打开时，会导致当前的分屏模式退出并且系统会有Toast提示*“应用不支持分屏”*。
 - **targetSDKVersion为N**
对于设置targetSDKVersion为N的App，该App的所有Activity默认均支持分屏模式。在Android N的SDK中Activity有个新的属性*resizeableActivity*，这个属性用来设置一个Activity是否支持MultiWindow模式。所以在设置targetSDKVersion为N的App中，所有Activity的*resizeableActivity*这个属性默认是true，如果想让一个Activity启动后不支持MultiWindow模式并全屏显示的话，可以选择如下两种做法：

1、在AndroidManifest中设置

    android:resizeableActivity="false"
    android:taskAffinity=""
并且需要在启动该Activity的时候设置FLAG_ACTIVITY_NEW_TASK这个flag
`intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK)`。

2、在AndroidManifest中设置

    android:resizeableActivity="false"
并且需要在启动该Activity的时候设置FLAG_ACTIVITY_NEW_TASK和FLAG_ACTIVITY_MULTIPLE_TASK这个两个flag
`intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK|Intent.FLAG_ACTIVITY_MULTIPLE_TASK)`。

上面所说是在targetSDKVersion为N的App中不进入分屏模式的Activity启动的设置。对于需要进入分屏模式的Activity，启动有三种模式。

##**MultiWindow模式特点**
在MultiWindow模式下，同时处于可见状态下的两个Activity只有一个处于Resume状态，另外一个处于Pause状态，由于这里的处于Pause状态的Activity还是可见的，所以google的官方建议是，各种要支持MultiWindow的App需要将暂停业务的逻辑放在onStop中执行，而不是onPause中执行，并且将业务开始逻辑放在onStart中执行，而不是onResume中。这样就是保证在MultiWindow模式下，那个处于onPause状态的Activity能继续正常的处理业务，比如播放视频，播放音乐，实时消息等等。

但是，但是，但是。。。。。。有一个问题，当两个Activity处于MultiWindow模式的时候，这时你按下HOME键，两个Activity都会变成不可见，但是，但是，但是，这时位于上半屏的Activity的生命周期不会走到onStop，只有下半屏的Activity会走到onStop，上半屏Activity固定走到onPause，也就是说，不管之前上半屏的Activity是onResume还是onPause，你短按HOME键后，都只是走到onPause，如果之前已经是onPause，就不会回调任何生命周期方法；下半屏的Activity在短按HOME键的时候生命周期的变化跟不处于MultiWindow模式的Activity在短按HOME键的时候的生命周期的变化是一样的。

这个看官方文档貌似是google的设计就是这样，但是这样会影响到视频App的用户体验，比如你上半屏正在放视频（这个视频App的播放和暂停逻辑已经按照google的建议从onResume和onPause移到了onStart和onStop中），然后你短按HOME键到桌面去做一些其他事，这时由于它处于onPause，所以视频不会暂停，这就直接导致，按下HOME键后视频还在播放，等用户干完其他事回来后，就错过了一段时间的视频，这个体验对视频App非常不友好。

这个坑不知道怎么填，目前有一些比较蹩脚的措施去弥补这个问题，但是感觉google对于MultiWindow这个特性的设计不是perfect。

**未完待续~~**
