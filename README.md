### 基于3.1.1修改eadown脚本中指定k8s安装版本

### 做免密
```
ssh-keygen -t rsa -C 'admin@admin.com' -f ~/.ssh/admin_id_rsa
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.8.121
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.8.160
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.8.162
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.8.161
```

### 如需要其他版本可复制 eadown 到可联网机器上执行以下命令
### 下载默认位置为/etc/kubeasz/
```
#./eadown   -D
```
### 测试连通性
```
ansible -i example/hosts.ini all -m ping
```
### 文件复制到/etc/kubeasz/
- ps: -e "@example/config.yml" 指定playbook变量文件，-v 指定日志信息
```
\cp -fr /root/ansible/kubeasz-3.1.1/* /etc/kubeasz/

ps: -e "@example/config.yml" 指定playbook变量文件，-v 指定日志信息
1、生成证书
ansible-playbook -i example/hosts.ini playbooks/01.prepare.yml -e "@example/config.yml" -v

2、安装etcd集群
ansible-playbook -i example/hosts.ini playbooks/02.etcd.yml -e "@example/config.yml" -v

3、安装runtime运行时
ansible-playbook -i example/hosts.ini playbooks/03.runtime.yml -e "@example/config.yml" -v

4、安装master
ansible-playbook -i example/hosts.ini playbooks/04.kube-master.yml -e "@example/config.yml" -v

5、安装node节点
ansible-playbook -i example/hosts.ini playbooks/05.kube-node.yml -e "@example/config.yml" -v

6、安装ex-lb
ansible-playbook -i example/hosts.ini playbooks/10.ex-lb.yml -e "@example/config.yml" -v

7、 安装网络组件
ansible-playbook -i example/hosts.ini playbooks/06.network.yml -e "@example/config.yml" -v

8、安装集群主要插件
ansible-playbook -i example/hosts.ini playbooks/07.cluster-addon.yml -e "@example/config.yml" -v


删除集群
ansible-playbook -i example/hosts.ini playbooks/99.clean.yml -e "@example/config.yml" -v
```

### 离线安装kuboard
```
https://www.kuboard.cn/install/v3/install-in-k8s.html#%E5%AE%89%E8%A3%85
安装kuboard需要给master节点打上 etcd的标签，否则启动不了
kubectl label nodes k8s-master1 k8s.kuboard.cn/role=etcd
kubectl label nodes k8s-master2 k8s.kuboard.cn/role=etcd
kubectl label nodes k8s-master3 k8s.kuboard.cn/role=etcd

----------------------
docker pull eipwork/kuboard-agent:v3
docker pull eipwork/etcd-host:3.4.16-1
docker pull eipwork/kuboard:v3
docker pull questdb/questdb:6.0.5

docker tag eipwork/kuboard-agent:v3 myharbor.com/kuboard/kuboard-agent:v3
docker tag eipwork/etcd-host:3.4.16-1 myharbor.com/kuboard/etcd-host:3.4.16-1
docker tag eipwork/kuboard:v3 myharbor.com/kuboard/kuboard:v3
docker tag questdb/questdb:6.0.5 myharbor.com/kuboard/questdb:6.0.5

docker push myharbor.com/kuboard/kuboard-agent:v3
docker push myharbor.com/kuboard/etcd-host:3.4.16-1
docker push myharbor.com/kuboard/kuboard:v3
docker push myharbor.com/kuboard/questdb:6.0.5

docker rmi eipwork/kuboard-agent:v3
docker rmi eipwork/etcd-host:3.4.16-1
docker rmi eipwork/kuboard:v3
docker rmi questdb/questdb:6.0.5
```
### 让k8s集群具有拉取私有仓库的权限
```
kubectl create secret generic harbor-registry \
    --from-file=.dockerconfigjson=/root/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
```
