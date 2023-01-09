# 基于树莓派的智能扫描枪

一个项目要用到扫描枪，但


```bash
# .bash_profile
...
~/scan.sh
```

```bash
# scan.sh

while :
do
    read i
    curl -d "i=$i" https://
done
```

```bash
sudo passwd -d al
```

```bash
systemctl edit getty@tty1
```

Tip: `ExecStart` 在重新赋值前要先清空[^drop-in-examples]

```
[Service]
ExecStart=
ExecStart=-/sbin/agetty -o '-p -- \\u' --noclear -a al %I $TERM
```

## References

[^drop-in-examples]: [Drop-in files Examples](https://wiki.archlinux.org/title/systemd#Examples)
[^autologin]: [Automatically Login on Debian 9.2.1 Command Line](https://unix.stackexchange.com/a/401798/274163)
[^autologin-2]: [Automatic root login in Debian 8.0 (console only))](https://superuser.com/a/1423805)

Tags: raspberry_pi scanner
