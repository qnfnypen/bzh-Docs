## Linux设置整理

### 一 Ubuntu
#### 1.1 查看系统信息
```bash
## 1. 版本信息
### 简单
cat /etc/issue
### 详细
cat /etc/lsb-release
### 查看内核信息
unmae -a

## 2. 内存信息
### 2.1 查看内存总量
cat /proc/meminfo | grep MemTotal
cat /proc/meminfo | grep -i memtotal | awk -F " " '{print $2/1024/1024 "GB"}'
cat /proc/meminfo | grep -i memfree | awk -F " " '{print $2 "KB"}'

## 3. 磁盘信息
df -Th

## 4. CPU信息
### 4.1 CPU核数
cat /proc/cpuinfo | grep -c "processor"
### 4.2 CPU是否支持虚拟化
egrep -o '(vmx|svm)' /proc/cpuinfo
```

#### 1.2 修改时间
[时间服务器列表](https://blog.csdn.net/weixin_42588262/article/details/82501488)
```bash
## 1. 硬件时间
### 1.1 查看硬件时间
sudo hwclock -r
### 1.2 将系统时间写入硬件
sudo hwclock -w

## 2. 时间服务器
### 2.1 安装chrony
apt install -y chrony
### 2.2 修改时间服务器
vim /etc/chrony/chrony.conf
> server ntp iburst
### 2.3 常用的时间服务器
☛ 上海：ntp.api.bz
```

#### 1.3 网络相关
```bash
## 1. 更改IP
### 1.1 修改为静态IP
vim /etc/netplan/50-cloud-init.yaml
> network:
>     version: 2
>     ethernets:
>           eth0:
>               addresses:
>               gateway4:
>               nameservers:
>                   addresses:
### 使更改生效
netplan apply 

## 2. 更改DNS
vim /etc/systemd/resolved.conf
> DNS=8.8.8.8

## 3. 更改网卡名称为eth0开始
vim /etc/default/grub
> GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
### 重新生成grub引导文件
grub-mkconfig -o /boot/grub/grub.cfg 
```

#### 1.4 允许SSH远程登陆
```bash
## 1. 允许普通用户SSH登陆
apt install openssh-server -y

## 2. 设置root用户密码
sudo passwd root

## 2. 允许root用户SSH登陆
vim /etc/ssh/sshd_conf
> PermitRootLogin yes

## 3. 重启ssh服务
systemctl restart ssh
```
