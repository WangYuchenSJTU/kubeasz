# Setup a k8s Cluster in 10 minutes
Use 1 VM as deploy and master node, 2 VMs as worker nodes.

## Import VMs in Virtualbox
1. import 1 deploy VM and 2 worker VMs, reinitialize MAC address, rename and chose cpu/memory wanted for them
2. boot the 3 VM and get their IPs with `ifconfig`

### Deploy cluster on the deploy VM
0. run every things as root
```bash
sudo su
```
1. setup ssh key
```bash
# generate ssh key
ssh-keygen -t rsa -b 2048
# (press Enter Enter Enter)

# setup no password access to all VMs (including deploy VM and Worker VM)
ssh-copy-id #all node IPs
# (yes and then type root password for target VM)
```
2. open ansible hosts file
```bash
# change IPs in hosts file
cd /etc/ansible
vim hosts
```
3. replace the right IPs in the following location with your deploy and worker IPs
```bash
# 集群部署节点：一般为运行ansible 脚本的节点
# 变量 NTP_ENABLED (=yes/no) 设置集群是否安装 chrony 时间同步
[deploy]
192.168.1.1 NTP_ENABLED=no

# etcd集群请提供如下NODE_NAME，请注意etcd集群必须是1,3,5,7...奇数个节点
[etcd]
192.168.1.1 NODE_NAME=etcd1

[kube-master]
192.168.1.1

[kube-node]
192.168.1.2
192.168.1.3
```
4. test connection to VMs
```bash
ansible all -m ping
```
5. deploy
```bash
ansible-playbook 90.setup.yml
```
6. creat a new ssh connection to your deploy/master VM and test the k8s cluster with kubectl
```bash
kubectl version
kubectl get componentstatus # 可以看到scheduler/controller-manager/etcd等组件 Healthy
kubectl cluster-info # 可以看到kubernetes master(apiserver)组件 running
kubectl get node # 可以看到单 node Ready状态
kubectl get pod --all-namespaces # 可以查看所有集群pod状态，默认已安装网络插件、coredns、metrics-server等
kubectl get svc --all-namespaces # 可以查看所有集群服务状态
```

7. [optional] access k8s dashboard at`https://192.168.1.xxx:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy`
with username:`admin`and password:`123456`. Then follow [dashboard](guide/dashboard.md) for futhur authentification.


