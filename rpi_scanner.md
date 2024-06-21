Title:  基于树莓派的智能扫描枪  
Author: Al Zee  
Email:  z@alz.ee  
Web:    https://alz.ee  
Date:   Jun 9, 2022  
Link:   https://alz.ee/article/rpi_scanner  
Tags:   raspberry_pi, scanner

# 基于树莓派的智能扫描枪

扫描枪做的事情简单直接，读取条码/二维码，还原出字符串，将其填入输入框。就相当于一个能看懂条码的人拿着键盘。   
所以它也有着和键盘一样的局限性————必须把焦点定在输入框中，否则扫描/按键盘不会有反应。就像我们有时低着头打字，等抬起头才发现没有点到输入框，刚才字白打了。  
这也是为什么早期的收银系统界面都相当简洁，且是固定的界面，无法轻易切换界面和焦点。其中一个主要原因就是为了防止丢失焦点，出现扫空的情况。  

这也合乎常理。

但有一个不合乎常理的问题：没有选中输入框的情况下，怎么让扫描枪工作？  

是的，客户的需求就是这样。客户甚至希望界面还没有打开的时候扫描枪就能工作。 

我做了一些资讯，请教了些朋友，包括在扫描枪/条码行业有丰富经验的。得到的答案都是————可能要上一套昂贵的系统。

昂贵就是不可行。

变换下思路，我们一定能在各个的层面找到相应的解决办法。  

键盘输入的原理大致为：按下键盘 > 开关接通 > 键盘驱动生成扫描码并送入寄存器 > 向CPU发出中断 > CPU执行中断处理键盘输入 > 具有键盘焦点的窗口接收键盘消息。扫描枪类似。  
所以这个问题的本质就是必须有一个模块来接受并处理键盘/扫描枪的输入。  

我们不太可能在操作系统上做文章，但我们可以在扫描枪和操作系统间再介入一个系统。  

我想到的是单板机，比如树梅派。只需要几百元就可以搞定。当然也可以国产品牌，性能不输树梅派，价格还有优势。不过这是题外话。  

下面以树梅派为例。

要达到目的，需要做如下配置：
树梅派：
1. 一个开机引导至命令行界面(`tty1`)，而不是图形界面的Linux系统。最好不要安装图形界面。后面以Debian为例
1. 开机自动登录
1. 登录后自动**在前台**运行处理字符串的脚本
1. 脚本携带字符串向业务系统API发送请求

业务系统：
1. 提供API处理请求，完成业务逻辑，生成扫码记录

### 开机自动登录：
```bash
# 编辑并覆盖systemd gettty@tty1默认配置
systemctl edit getty@tty1.service
```

```systemd
# systemd gettty@tty1配置内容
# 开机自动登录用户al
# Tip: `ExecStart` 在重新赋值前要先清空[^drop-in-examples]

[Service]
ExecStart=
ExecStart=-/sbin/agetty -o '-p -- \\u' --noclear -a al %I $TERM
```

```bash
# 清除用户al的密码
sudo passwd -d al
```

### 登录后自动**在前台**运行处理字符串的脚本

```bash
# .bash_profile

...

~/scan.sh
```

### 脚本代码

```bash
# scan.sh

# 无限循环，等待输入，处理输入
while :
do
    read i
    curl -d "i=$i" https://server/api/scan
done
```
## References

[^drop-in-examples]: [Drop-in files Examples](https://wiki.archlinux.org/title/systemd#Examples)
[^autologin]: [Automatically Login on Debian 9.2.1 Command Line](https://unix.stackexchange.com/a/401798/274163)
[^autologin-2]: [Automatic root login in Debian 8.0 (console only))](https://superuser.com/a/1423805)
