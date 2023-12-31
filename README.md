# PiKVM_Prebuild_image_ROCK64

此固件未测试，预计会有OTG问题，谨慎使用！

这是一个ROCK64的预构建镜像，该镜像可以让Pikvm运行在ROCK64上。

[PiKVM](https://github.com/pikvm/pikvm)是一个基于树莓派的开源、低成本的IP-KVM系统。不同于向日葵、teamviewer、Todesk这类远程控制软件，他可以实现硬件层的远程控制管理各类设备。  

当然这些年树莓派4B成为理财产品，便有了在各类便宜的ARM开发板上运行pikvm的需求。  

感谢[xe5700](https://github.com/xe5700)、[srepac](https://github.com/srepac)等大佬的开发移植，项目[kvmd-armbian](https://github.com/srepac/kvmd-armbian)已经实现在Allwinner全志, Amlogic晶晨以及Rockchip瑞芯微为核心的电视盒子以及开发板上运行PiKVM。  

当然能力有限，还是做不到刷入即用，还需要手动执行些步骤。  

## 使用步骤  
    
### 刷入镜像  
前往[发布页](https://github.com/Road-tech/PiKVM_ROCK64/releases)下载镜像，一直解压直到获得.img文件，并用[Etcher](https://etcher.balena.io/)将镜像刷入TF卡。 

### 插电开机
第一次开机用时会比较长，请耐心等待。  
以SSH的方式通过IP登入ROCK64,第一次启动默认账户为`root`，密码为`1234`。进入系统后会完成PiKVM的安装。     
等待显示PiKVM安装完成的提示，系统会让你设定新密码以及System command shell。    
接下来系统还会让你新增一个用户账户，这是可选项，你可按需设置新的用户账户或`Ctrl+C`退出。    

此时你可以在输入命令`kvmd -m`查看PiKVM运行情况。       
同时在浏览器访问Nanopi2的ip地址，如`https://192.168.8.197/`亦可看到PiKVM的登录界面， 账号密码皆为`admin`。     

![设置用户](https://github.com/Road-tech/Road-blog-Figure/blob/main/PiKVM_Prebuild_image_NanoPi-Neo/PiKVM_Prebuild_image_NanoPi-Neo-12.png?raw=true)  

### 调整MSD分区
查看分区信息：   
```
fdisk -l
```  

如图，我这里的TF卡名叫`/dev/mmcblk0`，装系统的主分区为`/dev/mmcblk0p1`,用来存放系统镜像的MSD分区为`/dev/mmcblk0p2`,下同。  
![占用分区](https://github.com/Road-tech/Road-blog-Figure/blob/main/PiKVM_Prebuild_image_NanoPi-Neo/PiKVM_Prebuild_image_NanoPi-Neo-07.png?raw=true)  

调整分区大小：  

```
cfdisk /dev/mmcblk0
```   

用`Delete`最开始预分配的`/dev/mmcblk0p2`分区，然后用`Resize`调整系统主分区`/dev/mmcblk0p1`,接着用`New`将剩余空间创建MSD分区，最后记得用`Write`保存修改,`Quit`推出。
![占用分区](https://github.com/Road-tech/Road-blog-Figure/blob/main/PiKVM_Prebuild_image_NanoPi-Neo/PiKVM_Prebuild_image_NanoPi-Neo-08.png?raw=true)  

重建主分区空间：  
```
resize2fs /dev/mmcblk0p1
```  

格式化MSD分区：  
```
mkfs -t ext4 /dev/mmcblk0p2
```  

### 挂载MSD分区
编辑文件：  
```
vi /etc/fstab
```  

在文件最下方新增一行，补充下面内容：  
```
/dev/mmcblk0p2  /var/lib/kvmd/msd   ext4  nodev,nosuid,noexec,ro,errors=remount-ro,data=journal,X-kvmd.otgmsd-root=/var/lib/kvmd/msd,X-kvmd.otgmsd-user=kvmd  0  0
```  

挂载分区：  
``` 
mount -a
```  

### 开启PiKVM的MSD功能
编辑文件：    
```
vi /etc/kvmd/override.yaml
```  

删除或注释以下内容：  
```
    msd:
       type:  disabled
```

重启PiKVM或者ROCK64：      

```
systemctl restart kvmd 或 reboot
```  
