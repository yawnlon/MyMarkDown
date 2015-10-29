# Android 6.0 Changes
Android 6.0 (API level 23) 有很多在系统和API层面上面的改变<br>
see [`Android 6.0 Changes`](http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html)<br>

## Runtime Permissions

权限管理是 Android M 最大的改变，权限管理更加精细，并且由以前的安装时静态授权，改为现在的运行时动态授权。主要改变有：

* 系统设置中可以对 APP 各个权限单独控制
* 权限根据内容进行分组了
* 普通权限还是在安装时授权
* 其他权限在运行时系统弹窗授权，并且要解析使用这个权限的目的

对于开发者来说，需要小心处理权限相关的问题。在使用某个功能的时候，需要总是判断是否有权限(call the new [`checkSelfPermission()`](http://developer.android.com/reference/android/content/Context.html#checkSelfPermission(java.lang.String)) method)，并且通过合适的方式请求用户授权(call the new [`requestPermissions()`](http://developer.android.com/reference/android/app/Activity.html#requestPermissions(java.lang.String[], int)) method.)。<br>
文章[`android 6.0 权限管理的学习资料和使用例子`](http://blog.csdn.net/yangqingqo/article/details/48371123) 有一个比较详细的解释。

## Apache HTTP Client Removal
Android 6.0 取消了对Apache HTTP Client的支持。如果App minSdkVersion > 9，可以使用[`HttpURLConnection`](http://developer.android.com/reference/java/net/HttpURLConnection.html) 替代，因为HttpURLConnection效率更高，because it reduces network use through transparent compression and response caching, and minimizes power consumption.<br>
如果想要继续使用HttpClient，解决方案：<br>

* Eclipse：libs中加入`org.apache.http.legacy.jar`上面的jar包在：`android-sdk\platforms\android-23\optional`下（需要下载android 6.0的SDK）<br>
* android studio: 在相应的module下的build.gradle中加入
```ini
android {
		useLibrary 'org.apache.http.legacy'
}
```

## Camera Service Changes
在Android6.0当中，访问照相机服务的模型由原先的“先来先服务”变为了“高优先级的进程先被服务”，具体如下：

* 访问相机子系统资源，包括打开和配置一个相机设备，被授予基于客户端应用程序的“优先级”。用户可视的或前景Activity的应用进程通常给予了更高的优先级，使相机资源获取和使用更可靠。
* 当有高优先级的应用进程启动时，正在使用相机的低优先级进程将会被”驱逐“。在过时的[`Camera`](http://developer.android.com/reference/android/hardware/Camera.html)API中，会result in [`OnError()`](http://developer.android.com/reference/android/hardware/Camera.ErrorCallback.html#onError(int, android.hardware.Camera))方法；而API23中的[`Camera2`](http://developer.android.com/reference/android/hardware/camera2/package-summary.html)中，会result in [`OnDisconnected()`](http://developer.android.com/reference/android/hardware/camera2/CameraDevice.StateCallback.html#onDisconnected(android.hardware.camera2.CameraDevice))方法
* 在有合适相机硬件的设备中，不同的应用进程可以同时独立使用不同的相机。然而，在多进程的情况下，如果每个进程使用的相机设备的性能都会显著降低，在Camera Service里是禁止的。这时会导致低优先级的应用进程的相机使用权被”驱逐“，即使没有其他进程和它竞争该相机设备。
* 改变当前设备的用户会使原先使用中的相机设备进程被”驱逐“

## BoringSSL
Android终于不用OpenSSL了TAT，替代的是[`BoringSSL`](https://boringssl.googlesource.com/boringssl/)。
需要注意的是如果在Android APP里使用了NDK，不要链接到NDK API中没有的密码库，such as `libcrypto.so` and `libssl.so`

