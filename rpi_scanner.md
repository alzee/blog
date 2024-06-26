Title:  基于树莓派的智能扫描枪  
Author: Al Zee  
Email:  z@alz.ee  
Web:    https://alz.ee  
Date:   Jun 9, 2022  
Link:   https://alz.ee/article/rpi_scanner  
Tags:   raspberry_pi, scanner

# 基于树莓派的智能扫描枪

扫描枪做的事情简单直接，读取条码/二维码，还原出编码，并追加一个回车。就相当于一个能看懂条码的人拿着键盘。   
所以它也有着和键盘一样的局限性——必须把焦点定在输入框中，否则扫描/按键盘不会有反应。就像我们有时低着头打字，等抬起头才发现没有点到输入框，字白打了。  
早期的收银系统界面都很简洁，且是固定的界面，无法轻易切换界面和焦点。其中一个主要原因就是为了防止丢失焦点，出现扫空的情况。  

这也合乎常理。

但有一个不合乎常理的问题：没有选中输入框的情况下，怎么让扫描枪有效输入？   

是的，客户的需求就是这样，他们甚至希望界面还没有打开的时候扫描枪就能工作。    
他们的应用场景就是简单的通过扫码实现进出库，扫一次出库，再扫一次进库，不需要录入其它信息。

我做了一些资讯，请教了些朋友，包括在扫描枪/条码行业有丰富经验的。得到的答案都是——可能要上一套昂贵的系统。

昂贵对我们来说就是不可行。

变换下思路，我们一定能在各个层面找到相应的解决办法。  

键盘输入的原理大致为：按下键盘 > 开关接通 > 键盘驱动生成扫描码并送入寄存器 > 向CPU发出中断 > CPU执行中断处理键盘输入 > 具有键盘焦点的窗口接收键盘消息。   
扫描枪类似。   
总之，最终必须有一个模块来接收并处理键盘的输入。这个模块通常是前端界面。

所以这个问题的本质就是：没有前端界面的时候，谁来接收键盘消息？ 

我们不太可能在操作系统上做文章，但我们可以在扫描枪和操作系统间再介入一个系统。  

我想到的是单板机，比如树梅派。只需要几百元就可以搞定。当然也可以选国产品牌，性能不输树梅派，价格还有优势。不过这是题外话。  

下面以树梅派为例。

### 要达到目的，需要做如下配置：

**树梅派**：
1. 一个开机引导至命令行界面(`tty1`)，而不是图形界面的Linux系统。最好不要安装图形界面
1. 开机自动登录
1. 登录后自动**在前台**运行处理编码的脚本
1. 脚本携带编码向业务系统API发送请求

**业务系统**：
1. 提供API来处理请求，完成业务逻辑，生成扫码记录

### 树梅派的具体配置如下。系统以Debian为例，其它`systemd`系统基本相同

#### 开机自动登录[^autologin][^autologin-2]：
```bash
# 编辑并覆盖systemd gettty@tty1.service默认配置
systemctl edit getty@tty1.service
```

```bash
# systemd gettty@tty1配置内容
# 开机自动登录用户al

[Service]
ExecStart=
ExecStart=-/sbin/agetty -o '-p -- \\u' --noclear -a al %I $TERM
```
Tip: `ExecStart` 在重新赋值前要先清空[^drop-in-examples]

```bash
# 清除用户al的密码
sudo passwd -d al
```

#### 在`~/.bash_profile`中插入脚本路径`~/scan.sh`，使其在登录后自动**在前台**运行

```bash
# .bash_profile

...

~/scan.sh
```

#### 脚本`scan.sh`代码

```bash
# scan.sh

# 无限循环
while :
do
    # 等待输入
    read i
    # 向API发送POST请求，payload为输入的编码
    curl -d "sn=$i" https://server/api/scan
done
```

#### 运行逻辑：
1. 扫描枪接在树梅派的USB口
1. 扫描枪扫描条码，并将解码出的编码输入树梅派系统命令行前台
1. 运行在前台的脚本读取编码，向API发送POST请求，payload为编码
1. 业务系统接收请求，处理数据，生成记录

这样就达到了预期效果。无需打开日常操作电脑，无需打开操作界面。只要业务系统在线，树梅派上电，保证树梅派能通过外网或内网访问业务系统，即可通过扫码完成录入/登记等的记录。也可以根据需求在API中写更复杂的业务逻辑。如此，借助单板电脑的可编程性，一个功能简单的扫描枪就摇身一变成了智能扫描枪。

但随之而来又有另外一个问题：我要通过操作界面录入怎么办？   

至于这个问题，当然可以再通过一些奇技淫巧来达到兼顾二者的目的，比如`webSocket`。但我的建议是，花一百元再配一把扫描枪直接插电脑上，搭配使用。  

以上方法已在实践中证明可行。   

这里抛砖引玉，如果您有其它好的想法，欢迎讨论。

[^drop-in-examples]: [Drop-in files Examples](https://wiki.archlinux.org/title/systemd#Examples)
[^autologin]: [Automatically Login on Debian 9.2.1 Command Line](https://unix.stackexchange.com/a/401798/274163)
[^autologin-2]: [Automatic root login in Debian 8.0 (console only))](https://superuser.com/a/1423805)
