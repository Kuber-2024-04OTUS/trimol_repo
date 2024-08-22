Kubeadm

Для выплнения дз развернул через vmware workstation 4 vm с ОС ubuntu 22.04
с характеристика 2CPU 8Гб RAM


Master       192.168.66.150
worker1      192.168.66.161
worker2      192.168.66.162
worker3      192.168.66.163

Проверил доступ к интернету и связь между vm

Последняя версия k8s на момент выполнения дз v1.31. Установка будет произведенена версии v1.30

Установка необходимых пакетов на master
apt update && apt -y upgrade && apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common

Отключение swap
nano /etc/fstab
закомментировал создание swap
Отключил swap 
swapoff -a


Загружаю дополнительные сетевые модули

**cat <<EOF | tee /etc/modules-load.d/k8s.conf**
**overlay**
**br_netfilter**
**EOF**

**modprobe overlay**
**modprobe br_netfilter**

![image alt](https://github.com/Kuber-2024-04OTUS/trimol_repo/blob/kubernetes-prod/image/lsmod.png)



**cat <<EOF | tee /etc/sysctl.d/k8s.conf**
**net.bridge.bridge-nf-call-iptables  = 1**
**net.bridge.bridge-nf-call-ip6tables = 1**
**net.ipv4.ip_forward                 = 1**
**EOF**

![image alt](https://github.com/Kuber-2024-04OTUS/trimol_repo/blob/kubernetes-prod/image/cni_plugins.png)


Перезапускаем параметры ядра

**sysctl --system**

![image alt](https://github.com/Kuber-2024-04OTUS/trimol_repo/blob/kubernetes-prod/image/checkparam.png)


Отключаем UFW
systemctl stop ufw && systemctl disable ufw


Установка CRI-O
export OS=xUbuntu_22.04
export CRIO_VERSION=1.25


Обновляем списки репозиториев и устанавливаем CRIO, а также дополнительные утилиты
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"| tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"| tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | apt-key add -

curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -

apt update && apt -y install cri-o cri-o-runc cri-tools


Запускаем crio и добавляем его в автозагрузку
systemctl start crio && systemctl enable crio

Проверяем статус crio
-----------------------------------------------------------------Criostatus.png


Установка Kubernetis v1.30
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list


apt update && apt -y install kubelet kubeadm kubectl && apt-mark hold kubelet kubeadm kubectl





Выполняем только на master. Инициализируем мастер-ноду а также выделяем подсеть
kubeadm init --pod-network-cidr=10.244.0.0/16

-----------------------------------------------------------------------kubeinit.png


  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
  
  
Подключил  3 worker node


---------------------------------------------------------------------------k8sstatus.png

--------------------------------------------------------------------------k8spods.png


Установка CNI flanell
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml

----------------------------------------------------------------------------flanellstaus.png


------------------------------------------------------------------------------------nodeswide.png




Обновление 


curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list



apt-get update
apt-get update && sudo apt-get install -y kubelet='1.31.*'
apt-get update && sudo apt-get install -y kubeadm='1.31.*'
apt-get update && sudo apt-get install -y kubectl='1.31.*'


kubectl drain master --ignore-daemonsets


kubeadm upgrade apply v1.31.0

systemctl restart kubelet
systemctl status kubelet

--------------------------------------------------------------------------------kubeletstatus.png
kubelet uncordon master
kubectl get nodes
-------------------------------------------------------------------------------masterupdate.png


на master выполняю
kubectl drain worker1 --ignore-daemonsets
на worker1 выполняю
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get update && sudo apt-get install -y kubelet='1.31.*' kubectl='1.31.*'
apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet


на master выполняю
kubectl uncordon worker1



на master выполняю
kubectl drain worker2 --ignore-daemonsets
на worker2 выполняю
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get update && sudo apt-get install -y kubelet='1.31.*' kubectl='1.31.*'
apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet


на master выполняю
kubectl uncordon worker2



на master выполняю
kubectl drain worker3 --ignore-daemonsets
на worker3 выполняю
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get update && sudo apt-get install -y kubelet='1.31.*' kubectl='1.31.*'
apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet


на master выполняю
kubectl uncordon worker3



Проверка node после обновления
--------------------------------------------------------------------------------------resupdate.png





Задание c*
Развернул виртуальные машины  с ОС ubuntu 22.04
3 мастера 
master1    192.168.66.151
master2    192.168.66.152
master3    192.168.66.153


2 worker
worker1 192.168.66.161
worker2 192.168.66.162



Создал root пользователей и разрешил доступ root по ssh изменив конфигурацию в фале /etc/ssh/sshd_config раскоментирова и изменив значение на всех vm
PermitRootLogin yes
перезапускаю службу 
service ssh restart
service sshd restart


Отключаю swap на всех vm
swapoff -a
комментирую swap.img в /etc/fstab


Генерирую ssh ключ на master1
ssh-keygen -t rsa


Настраиваю доступ по SSH без пароля на все vp с master1
ssh-copy-id root@192.168.66.151
ssh-copy-id root@192.168.66.152
ssh-copy-id root@192.168.66.153
ssh-copy-id root@192.168.66.161
ssh-copy-id root@192.168.66.162


На master1 устанавливаю python
apt-get update
apt-get install python



Включим переадресацию IPv4
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf

Перехожу в директорию 
cd /opt/
Создаю папку kubespray
mkdir kubespray
cd kubespray/
git clone git@github.com:kubernetes-sigs/kubespray.git
cd kubespray
cp -rfp inventory/sample inventory/homework


объявим переменную IPS
declare -a IPS=(192.168.66.151 192.168.66.152 192.168.66.153 192.168.66.161 192.168.66.162)

CONFIG_FILE=inventory/homework/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
nano inventory/homework/hosts.yaml
-----------------------------------------------------------------------inventory.png


на master node выполнил
pip install -r requirements.txt
ansible-playbook -i inventory/homework/hosts.yaml cluster.yml
----------------------------------------------------------------------res_kubespray.png
