# 操作系统OS 
## linux 
### centos镜像
#### 配置方法
1. 备份 
`mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup`
2. 下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/ 
`wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo`
3.  运行 yum makecache 生成缓存
4. 常见问题
`sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo`




### Centos 系统初始化
#### 系统初始化-centos7【联网】
```shell

iptables -F
systemctl disable firewalld && systemctl stop firewalld
systemctl disable postfix && systemctl stop postfix
setenforce 0

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sed -i 's/4096/40960/g' /etc/security/limits.d/20-nproc.conf

echo '* soft nofile 102400' >> /etc/security/limits.conf
echo '* hard nofile 102400' >> /etc/security/limits.conf
echo 'hive - nofile 102400' >> /etc/security/limits.conf
echo 'hive - nproc 102400' >> /etc/security/limits.conf

mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
yum makecache
yum install -y wget

wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum makecache fast

yum groupinstall 'development tools' -y
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel -y

yum install lvm2 vim lrzsz ntp chrony wget zip unzip git screen telnet net-tools traceroute lsof -y
yum install yum-utils yum-plugin-downloadonly device-mapper-persistent-data -y

```
#重启服务器#
`reboot`

#修改hostname#
`hostnamectl set-hostname  *设置主机名* `



