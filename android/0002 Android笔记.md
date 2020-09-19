
# SELINUX
+ class文件定义在system/sepolicy/secruity_classes和access_vectors：
class filesystem
class file

+ 权限定义在global_macros，如下：
define(`rw_file_perms', `{ r_file_perms w_file_perms }')


# Android 10属性文件
* 属性文件：
```
/default.prop
/system/build.prop
/vendor/default.prop
/vendor/build.prop
```

* 增加新属性，需修改文件
android/system/sepolicy/private/property_contexts

# handler
handler持有looper对象，looper持有MessageQueue对象
Looper.loop会进入循环，一直获取消息，有消息则交给handlerMessage处理. 

在自己的线程中创建handler，需如下步骤：  
1) 初始化looper： Looper.prepare();
2) 创建looper对象： mLooper = Looper.myLooper();
3) 启动looper循环： Looper.loop();

# Zygote启动过程
startVM启动虚拟机  
调用startReg注册JNI方法  
创建socket服务，等待AMS消息  
创建system_server进程，调用SystemServer.main函数，创建系统关键服务  