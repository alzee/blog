Title:  title
Author: Al Zee  
Email:  z@alz.ee  
Web:    https://alz.ee  
Date:   Mon, 11 Mar 2024 15:34:54 +0800
Link:   https://alz.ee/article/script_to_run_ec2_instance
Tags:   

# 一键创建/销毁ec2实例

### 目的
* ec2按需付费，使用时创建，用完销毁。
* 一个命令即可快速创建/销毁实例。
* 通过User data写入的开机脚本，在实例启动时自动部署游戏加速器/梯子/计算平台，开箱即用。
* 加速器和梯子通过wireguard实现。
* 通过spot Request获取最低价。

### 注意事项
* Spot Instance优点是低价，缺点是有中断的风险。但总体而言中断的可能性很小，所以对于价格敏感，而持久性要求不高的场景来说很合适。
* 如果你追求持久稳定且对价格无感，**请勿**选择Spot Instances。本文可能也不适合你的场景。你可能需要的是Saving Plans或Reserved Instances。
* 如果你是用作加速器或梯子，请选择离目标服务器最近的区域，比如Warmane服务器在巴黎，则区域选择巴黎。
* 如果你是用作计算，可选择价格相对较低的区域，比如美国的N. Virginia、亚马逊中国的宁夏区等。应避免使用香港、东京等价格较高的区域。
* 亚马逊(亚马逊中国除外)提供每月100GB的免费出口流量，对于普通用户通常是够用的。
* EBS是另外收费的，Root Volume请尽量小，8-10G通常够用。
* 本文主要介绍整体思路，未包含的技术细节请参考aws文档

### 创建Launch Template
我不希望在命令行输入大量参数，所以提前创建Launch Template。
1. `Instance Type`按价格排序，选择价格最低的。ARM平台相对较便宜，所以推荐选择t4g.nano。
1. OS选择自己喜欢的，同样选择ARM架构
1. 创建Key pair。
1. 创建Security Groups，tcp/22务必开启。另建议开启http/https/ICMP等以备不时之需。
1. EBS默认gp3 8G即可。
1. `Advanced details` > `Purchasing option`，选择`Spot instances`。
1. `Advanced details` > `Purchasing option` > `Interruption behavior`，如果不存储数据，建议选择`Terminate`，否则选择`Stop`。
1. `Advanced details` > `User data`，写入开机脚本。

### User data 中开机脚本实例
```
#!/bin/bash

curl -L https://ash.alz.ee | DEFAULT_USER=admin bash

curl -L https://wg.alz.ee/setup | PRIVATEKEY='YOUR_WIREGUARD_SERVER_PRIVATE_KEY_HERE' bash
```
代码主要实现两个功能：
1. 环境初始化。包括但不限于：创建用户、创建swap、添加仓库、安装常用软件、设置自动更新、增加用户组、设置时区、设置主机名，导入常用函数、环境变量、配置等。不同云平台及OS，默认用户名不同，所以通过环境变量`DEFAULT_USER`传递。
2. 配置wireguard。因为脚本部署在公网，为防止Private Key泄漏，这里使用环境变量`PRIVATEKEY`传递。

两个功能都封装成脚本，部署在公网，以便实例启动后可以读取并执行。这样做有两个好处：
1. 简洁。
1. 便于更新。我是通过github pages托管，修改脚本后push上去，实例访问的即是最新版本，不用再修改User data。

##### 环境初始化脚本(https://ash.alz.ee)
```
#!/bin/bash

# add user
user=al
sudo_group=sudo

if ! id $user &> /dev/null; then
    sudo useradd -m -s /bin/bash $user
    sudo usermod -aG $sudo_group $user

    # env DEFAULT_USER
    default_user=${DEFAULT_USER:-$(id -un)}
    default_user_home=/home/$default_user
    [ default_user = root ] && $default_user_home=/root

    if [ -d $default_user_home/.ssh/ ]; then
        sudo cp $default_user_home/.ssh/  /home/$user/ -a
        sudo chown -R $user:$user /home/$user/.ssh
    fi
    echo $user:zee | sudo chpasswd
fi


# get a.sh
# Get tarball
# https://unix.stackexchange.com/a/669552
url=https://github.com/alzee/ash/archive/master.tar.gz
f=ash-master.tar.gz
curl -L -o $f "$url"
tar xf $f && rm $f
mv ${f%%.*} ash

# Or using git
# git clone https://github.com/alzee/ash

sudo mv ash/ /home/$user/.ash
sudo chown -R $user:$user /home/$user/.ash

cd /home/$user
sudo -u $user .ash/a.sh -L
sudo -u $default_user .ash/a.sh -Y
```

##### wireguard配置脚本(https://wg.alz.ee/setup)
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

### 安装并配置aws cli
1. 配置access key和secret
1. 配置default region。如果需要操作其它区域，可用cli参数`--region`，或`AWS_DEFAULT_REGION`环境变量。推荐后者，不用修改命令参数，更方便。

### 创建及销毁实例
```
jq_aws_instance_ip(){
    # use xargs only for removing quotes
    jq '.Reservations[0].Instances[0].PublicIpAddress' | xargs
}

jq_aws_instance_id(){
    # use xargs only for removing quotes
    jq '.Reservations[0].Instances[0].InstanceId' | xargs
}

append_to_hosts(){
    # example
    # append_to_hosts $(aws ec2 describe-instances | jq_aws_instance_ip) aws
    local ip=$1
    local hostname=$2
    local hosts=/etc/hosts
    sudo sed -i "/ $hostname$/d" $hosts
    sudo sed -i "\$a$ip $hostname" $hosts
}

run_a_aws_instance(){
    local id
    id=$(aws ec2 run-instances --launch-template LaunchTemplateName=min | jq '.Instances[0].InstanceId' | xargs | tee ~/.aws/instance_id)
    append_to_hosts $(aws ec2 describe-instances --instance-ids $id | jq_aws_instance_ip) aws
}

terminate_a_aws_instance(){
    local id
    # id=$(aws ec2 describe-instances | jq_aws_instance_id)
    id=$(< ~/.aws/instance_id)
    aws ec2 terminate-instances --instance-ids $id
}
```
代码由4个函数组成，主函数为：  
`run_a_aws_instance`：
* 以Launch Template为模板创建实例，`LaunchTemplateName`为所创建模板的名称，如代码中的`min`
* 获取实例ID，并将其写入`~/.aws/instance_id`，供`terminate_a_aws_instance`使用
* 将实例公网IP写入/etc/hosts，主机名`aws`。
* 此主机名和wireguard客户端配置中`Endpoint`主机名保持一致，以后就不用再修改wireguard客户端配置。

`terminate_a_aws_instance`：
* 从`~/.aws/instance_id`中读取实例ID
* 销毁实例

### Wireguard客户端配置
```
[Interface]
Address = 10.5.7.5/24
# SaveConfig = true
PrivateKey = YOUR_PRIVATE_KEY_HERE
DNS = 1.1.1.1

# PreDown = terminate_a_aws_instance

[Peer]
PublicKey = YOUR_PUB_KEY_HERE
# AllowedIPs = 10.5.7.0/24
# AllowedIPs = 188.138.40.87,62.138.7.219,51.91.106.148,51.178.64.97,51.178.64.87,10.5.7.0/24
AllowedIPs = 0.0.0.0/0
Endpoint = aws:55555
PersistentKeepalive = 30
```
如果只需部分数据通过wireguard，则AllowedIPs填写目标IP。比如我只希望与warmane服务器的连接走wireguard接口，其它连接走默认接口，则可通过tcpdump或wireshark抓包，得到warmane几台服务器的IP为`188.138.40.87`,`62.138.7.219`,`51.91.106.148`,`51.178.64.97`,`51.178.64.87`，将其写入AllowedIPs即可。另须加入Wireguard服务器网段，如`10.5.7.0/24`。  
如果希望所有连接都走Wireguard接口，AllowedIPs则写`0.0.0.0/0`。

### 日常使用
1. `run_a_aws_instance`创建实例
1. `sudo wg-quick up aws`启用Wireguard接口
1. `sudo wg-quick down aws`关闭Wireguard接口
1. `terminate_a_aws_instance`销毁实例

当然也可以把`terminate_a_aws_instance`放在Wireguard客户端配置的`PreDown`中，以达到关闭接口时自动销毁实例的效果。但前提是`root`用户可以读取到该函数。读者可自行调试。
