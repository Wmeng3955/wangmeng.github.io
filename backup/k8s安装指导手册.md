# 在线安装k8s 以及kubesphere 容器平台
## 软件版本信息

- 操作系统版本：CentOS 7.x
- KubeSphere版本：v3.1.1
- KubeKey版本：v1.2.1
- K8s版本：v1.19.8
- Docker版本：v20.10.9

## 部署节点
1主2从
## 服务器基础配置 【主从节点一致】
### os 基础配置
*备注：参考前篇os操作系统基础配置【可使用自动化脚本】
1、关闭防火墙和SELinux
```shell
iptables -F
systemctl stop firewalld && systemctl disable firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
setenforce 0
```
2、配置主机名
hostnamectl set-hostname  master/worker1/worker2
3、系统参数修改
```shell 
sed -i 's/4096/40960/g' /etc/security/limits.d/20-nproc.conf
echo '* soft nofile 102400' >> /etc/security/limits.conf
echo '* hard nofile 102400' >> /etc/security/limits.conf
```
4、设置时间同步和时区
```shell
yum install -y chrony
systemctl enable chronyd
systemctl start chronyd
timedatectl set-ntp true
timedatectl set-timezone Asia/Shanghai
date -s  “2020-10-28 10:15:15”
加定时任务在线同步服务器时间
0 30 * * * ntpdate ntp.aliyun.com > /dev/null 2>&1 &
Ps：构建镜像的时候选择时区。

#构建应用镜像时设置时区
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \ 
&& echo 'Asia/Shanghai' >/etc/timezone \
```
5、安装依赖软件包
yum install -y openssl openssl-devel socat
yum install -y epel-release
yum install -y conntrack conntrack-tools ebtables ipset
6、重启操作：Reboot

### docker部署
1、配置Docker-Yum源 
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache
2、安装docker
yum install -y docker-ce-20.10.9 docker-ce-cli-20.10.9 containerd.io
3、设置开机自启
systemctl enable --now docker
4、设置镜像仓库加速器
```shell
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://79j3qj6x.mirror.aliyuncs.com","https://registry.aliyuncs.com"],
"insecure-registries": ["registry.wangmeng.cn"]
}
EOF
```
5、重新加载配置并重启Docker
`systemctl daemon-reload`
`systemctl restart docker`
6、验证docker服务
`docker info `
7、添加Harbor域名解析【每台机器】
```shell
echo '#Harbor' >> /etc/hosts
echo '172.23.32.90    registry.wangmeng.cn' >> /etc/hosts   #ip为harbor仓库的ip
```
8、登录仓库【每台机器】
`docker login registry.wangmeng.cn`
### 安装k8s&kubesphere 
1、下载kk【master节点】
```shell
mkdir –p /root/ks311 && cd /root/ks311
export KKZONE=cn
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.2.1 sh –
chmod +x kk
```
2、创建集群配置文件config-sample.yaml
3、修改配置后执行安装命令：
`./kk create cluster -f config-sample.yaml`
4、运行以下命令以检查安装日志。
`kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f`

### 修改集群全局配置
1、初始化集群内服务需要的namespace
`kubectl create ns svc-external`
验证是否创建成功，`kubectl get ns `查询是否上面命名空间创建成功；命名空间根据实际需要进行创建。
2、创建外部基础组件应用
`kubectl apply -f redis-svc.yaml`
`kubectl get serviceentry -n svc-external`
3、修改coredns映射
`kubectl edit configmap coredns -n kube-system`
ip改成对应基础服务所在节点ip
4、修改nodelocaldns配置
`kubectl edit configmap nodelocaldns -n kube-system`
#修改之后重启nodelocaldns
`kubectl -n kube-system rollout restart daemonset nodelocaldns`

