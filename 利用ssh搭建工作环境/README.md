# 利用ssh搭建工作环境

> 原创：余俊卿 转载请:<yujunqing@meizu.com>

----


### 结果就是：

工作站的屏幕线可以拔掉了！

**再也再也不用管环境配置了！使用mac和windows作为主力机的同学必备！**

### 从此以后：
mac上执行

````
./makeandpush
````
 
就能编译并且push了
也只需要维护mac上的那一部分代码就可以了

### SSH讲解
此处省去一万字……

### 作案工具：

1、打通ssh隧道，请百度后开启ubuntu主机的ssh连入

2、编写几个脚本

3、利用rsync传输文件

4、利用screen跑脚本

### 作案要求：

* 两台主机上都需要有源码，服务器主机需要有可以编译通过的完整源码，可以写一个脚本定时更新，mac/windows主机只需要有模块源码即可，可以使用repo sync 模块路径同步下载模块源码而不必下载全部源码，节省硬盘空间

* 有额外一台电脑。。。和主机之间打通互相的SSH连接

### 作案手法：
1、成功用工作站ssh登录你自己的另外一台电脑，成功用这太电脑ssh登录你的工作站，总之就是打通两台电脑之间的ssh连接。

2、推荐用密钥验证的方式，安全性更好，附上一段复制密钥的代码

````
cat ~/.ssh/id_rsa.pub | ssh yourname@yourhost "mkdir ~/.ssh ; cat >> ~/.ssh/authorized_keys”
````
   
3、编写脚本 以systemUI为例

将mac上改过的文件和远程主机对应目录下的同步，然后执行远程主机的编译命令，最后将生成的APK传回mac，并push进设备然后重启systemui


````
#makeandpush

#!/bin/bash

HOST='yujunqing@shell.kenai.cc'

LOCALROOT='/Volumes/workspace/sourcespace/mtk/'

REMOTEROOT='/home/yujunqing/meizu/m76/'

APPPATH='flyme/frameworks/base/packages/SystemUI/'

REMOTEOUT='out/target/product/m76/system/priv-app/SystemUI.apk'

LOCALOUT='/Users/kenai/Desktop/SystemUI.apk'

INSTALL='/system/priv-app/'

#将mac上改过的文件和远程主机对应目录下的同步，然后执行远程主机的编译命令，最后将生成的APK传回mac
mRsync(){

    rsync -avzP --delete ${LOCALROOT}${APPPATH}  ${HOST}:${REMOTEROOT}${APPPATH}

    #copy ${REMOTEROOT}    ${APPPATH}
    ssh $HOST 'cd /home/yujunqing/meizu/m76/ && . build/envsetup.sh && lunch meizu_m76-eng &&  mmm flyme/frameworks/base/packages/SystemUI/ '

    rsync -avzP --delete $HOST:${REMOTEROOT}${REMOTEOUT}  ${LOCALOUT}

}

push(){
    adb push ${LOCALOUT} ${INSTALL} && sleep 1 && xadb-kill-systemui.sh && say 'ok' && exit
}

#>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

mRsync

push

adb remount && push

say 'check    please'

#<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
````

**顺便分享一下重启SystemUI的脚本**

````
#!/bin/bash

adb shell ps | grep 'systemui'  | cut -d ' ' -f 5 | xargs adb shell kill && echo 'SystemUI restart'
````

**如有疏漏，敬请指正！**