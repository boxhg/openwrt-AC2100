# 红米路由器AC2100刷不死鸟breed、OpenWRT

## 一、从AC2100官方固件后台开启SSH

### 1.1 登录web后台，获取token
- 首次登录路由器后台，会进行初始配置，设置好wifi及后台管理密码。

- 确认路由器固件版本是：2.0.23

- 在浏览器中获取url地址

  ```
  http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/web/home#router
  ```
  \<STOK\>是登录后自动生成的token，32个字符，下面要用到；

### 1.2 在url中执行命令，开启ssh

用上面获取的token, 替换url中的<STOK>后，复制到浏览器的URL中打开，即可执行命令

```
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B
```
返回{"code":0}即代表成功

上面是利用了官方固件的web注入漏洞执行命令
```
nvram set ssh_en=1;
nvram commit;
sed -i 's/channel=.*/channel=\"debug\"/g' /etc/init.d/dropbear; 
/etc/init.d/dropbear enable
/etc/init.d/dropbear start;
```
### 1.3 修改root密码为admin（可选操作）

echo -e 'admin\nadmin' | passwd root
```
> http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=gallifrey&user_id=doctor&ssid=-h%0Aecho%20-e%20%27admin%5Cnadmin%27%20%7C%20passwd%20root%0A
```
### 1.4 使用ssh登录

> ssh root@192.168.31.1

密码是上面初始化配置的密码，如果不对可以参考1.3步骤修改root密码

```
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B
```



## 二、刷入不死鸟Breed，配置环境变量

### 2.1.刷入breed_boot

使用ssh客户端登录后台后，上传breed-mt7621-xiaomi-r3g.bin 到   /tmp  
```
nvram set uart_en=1
nvram set bootdelay=5
nvram set flag_try_sys1_failed=1
nvram commit

cd /tmp
mtd -r write breed-mt7621-xiaomi-r3g.bin Bootloader
```

执行命令后，路由器自动重启，等重启重新入web控制台；

如果路由器在60秒内重启则代表刷BREED成功(灯会从蓝变橘，最终变蓝进入系统)。

### 2.2.刷入成功后拔掉电源，按住reset同时接上电源等10秒即可进入breed。

### 2.3.设置PC为自动获取IP地址，访问http://192.168.1.1 进入Breed Web恢复控制台；

### 2.4. 添加环境变量xiaomi.r3g.bootfw",值设置为 2

### 2.5 (可选步骤) 进入“小米 R3G 设置”找到字段“normal_firmware_md5”，点击最右边的删除并保存，完后再刷固件

## 三、在Breed中刷入openwrt过渡包

### 3.1.恢复openwrt过渡包 AC2100-Breed-MiddleRom.bin, 等待设备重启；

### 3.2.重启后访问 http://192.168.1.1/，即可进入openwrt后台；

## 四、升级到最新OpenWRT官方固件

### 4.1 进入openwrt后台，System -> Backup / Flash Fireware

### 4.2. Flash new firmware image

-  择WRT完整升级固件openwrt-21.02.0-rc4-ramips-mt7621-xiaomi_redmi-router-ac2100-squashfs-sysupgrade.bin，
-  不要勾选 Keep settings and retain the current configuration，
-  点击 “Flash Image...”上升刷入固件, 
-  最后点Continue，等几分钟让路由器自己重启



## 相关url

- OpenWRT官方固件下载
  > https://downloads.openwrt.org/releases/21.02.0-rc4/targets/ramips/mt7621/
  > https://firmware-selector.openwrt.org/?version=21.02.0-rc4&target=ramips%2Fmt7621&id=xiaomi_redmi-router-ac2100

- Wiki 

  > https://openwrt.org/toh/xiaomi/xiaomi_redmi_router_ac2100#tab__shell_script

- 工具包下载

  > http://openwrt.ink:8666/

- 参考论坛帖子

  > [AC2100(RM2100)] 红米(小米)AC2100Breed和Padan固件教程及刷回官方教程 https://www.right.com.cn/FORUM/thread-4059522-1-1.html
  >
  > [AC2100(RM2100)] 红米/小米AC2100刷入r3g breed以及恢复官方boot详细教程（更新查坏块方法、pb-boot）https://www.right.com.cn/forum/thread-4023907-1-1.html

- 




