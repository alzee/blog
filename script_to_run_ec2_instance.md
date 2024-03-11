Title:  title
Author: Al Zee  
Email:  z@alz.ee  
Web:    https://alz.ee  
Date:   Mon, 11 Mar 2024 15:34:54 +0800
Link:   https://alz.ee/article/script_to_run_ec2_instance
Tags:   

# 一键启动/终止ec2实例

### 目的
* 一个命令即可快速启动/终止ec2实例
* 通过User data写入的开机脚本，在实例启动时自动部署游戏加速器/梯子/计算平台，开箱即用
* 通过spot Request获取最低价

### 注意事项
* Spot Instance优点是低价，缺点是有中断的风险。但总体而言中断的可能性很小，所以对于价格敏感，而持久性要求不高的场景来说很合适。
* 如果你追求持久稳定且对价格无感，请勿选择Spot Instances。本文可能也不适合你的场景。你可能需要的是Saving Plans或Reserved Instances。
* 如果你是用作加速器或梯子，请选择离目标服务器最近的区域，比如Warmane服务器在巴黎，则区域选择巴黎。
* 如果你是用作计算，可选择价格相对较低的区域，比如美国的N. Virginia、亚马逊中国的宁夏区等。应避免使用香港、东京等价格较高的区域。
* 亚马逊(亚马逊中国除外)提供每月100GB的免费出口流量，对于普通用户通常是够用的。
* EBS是另外收费的，Root Volume请尽量小，8-10G通常够用。
* 本文主要介绍整体思路，未包含的技术细节请参考aws文档

### 安装并配置aws cli
1. Instance Type按价格排序，选择价格最低的。ARM平台相对较便宜，且够用，所以推荐选择t4g.nano
1. OS选择自己喜欢的，同样选择ARM架构
1. 创建Key pair
1. 创建Security Groups，tcp/22务必开启。另建议开启http/https/ICMP等以备不时之需。
1. EBS默认gp3 8G即可
1. Advanced details > Purchasing option，选择Spot instances，
1. Advanced details > Purchasing option > Interruption behavior，如果不存储数据，建议选择Terminate，否则选择Stop。
1. Advanced details > User data，写入开机脚本

### 开机脚本举例
```
#!/bin/bash

curl -L https://ash.alz.ee | DEFAULT_USER=admin bash

curl -L https://wg.alz.ee/setup | PRIVATEKEY='YOUR_WIREGUARD_SERVER_PRIVATE_KEY_HERE' bash
```
该代码主要实现两个功能：
1. 环境初始化。包括但不限于：创建用户、创建swap、添加仓库、安装常用软件、设置自动更新、增加用户组、设置时区、设置主机名，导入常用函数、环境变量、配置等
2. 配置wireguard。因为脚本部署在公网，为防止Private Key泄漏，这里使用环境变量PRIVATEKEY传递

两个功能都封装成脚本，部署在公网，以便实例启动后可以读取并执行。这样做有两个好处：
1. 简洁
1. 便于更新。我是通过github pages托管，修改脚本后push上去，实例即可访问最新的版本，不用再修改User data.


```
#!/bin/bash
#
# vim:ft=sh

apt update
apt install wireguard -y

# env PRIVATEKEY
key=${PRIVATEKEY:-private_key_not_given}
conf=wg-server.conf

# 下载配置文件模板
curl -L https://wg.alz.ee/server.conf > $conf

# 修改网络接口名
iface=$(ip r show default | awk '{print $5}')

sed -i s/eth0/$iface/ $conf

# 修改private key
sed -i s/SERVER_PRIVATEKEY/$key/ $conf

mv $conf /etc/wireguard/

# 开机自动启动该wireguard接口
systemctl enable --now wg-quick@${conf%.conf}
```

### 

