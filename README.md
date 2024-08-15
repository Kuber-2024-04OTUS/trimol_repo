Развернул кластер k8s на своем оборудование через kubespray
inventory.ini
[all]
master1 ansible_host=192.168.75.150  ip=192.168.75.150 etcd_member_name=etcd1
master2 ansible_host=192.168.75.151  ip=192.168.75.151 etcd_member_name=etcd2
master3 ansible_host=192.168.75.152  ip=192.168.75.152 etcd_member_name=etcd3


# ## configure a bastion host if your nodes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
master1
master2
master3


[etcd]
master1
master2
master3


[kube_node]
master1
master2
master3


[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr






Инфраструктурными нодами выбраны ноды Master2 и Master3
kubectl taint nodes master3 node-role=infra:NoSchedule
kubectl taint nodes master2 node-role=infra:NoSchedule





kubectl get node -o wide --show-labels
NAME      STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME     LABELS
master1   Ready    control-plane   19h   v1.30.3   192.168.75.150   <none>        Ubuntu 22.04.4 LTS   5.15.0-118-generic   containerd://1.7.20   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master1,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
master2   Ready    control-plane   19h   v1.30.3   192.168.75.151   <none>        Ubuntu 22.04.4 LTS   5.15.0-118-generic   containerd://1.7.20   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master2,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
master3   Ready    control-plane   18h   v1.30.3   192.168.75.152   <none>        Ubuntu 22.04.4 LTS   5.15.0-118-generic   containerd://1.7.20   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master3,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=




kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
NAME      TAINTS
master1   <none>
master2   [map[effect:NoSchedule key:node-role value:infra]]
master3   [map[effect:NoSchedule key:node-role value:infra]]



Установка helm
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm



helm version
version.BuildInfo{Version:"v3.15.3", GitCommit:"3bb50bbbdd9c946ba9989fbe4fb4104766302a64", GitTreeState:"clean", GoVersion:"go1.22.5"}

Добавление helm chart repository
helm repo add argo https://argoproj.github.io/argo-helm


curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3



chmod 700 get_helm.sh



./get_helm.sh
Downloading https://get.helm.sh/helm-v3.15.3-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm




kubectl create namespace argocd
kubectl get ns
NAME              STATUS   AGE
argocd            Active   4s
default           Active   23h
kube-node-lease   Active   23h
kube-public       Active   23h
kube-system       Active   23h





helm install argocd argo/argo-cd --namespace argocd --set nodeSelector.node-role=infra:NoSchedule
NAME: argocd
LAST DEPLOYED: Tue Aug 13 09:19:31 2024
NAMESPACE: argocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argocd-server -n argocd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)




helm list -n argocd
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
argocd  argocd          1               2024-08-13 09:19:31.688841169 +0000 UTC deployed        argo-cd-7.4.3   v2.12.0




kubectl get pod -n argocd
NAME                                                READY   STATUS    RESTARTS        AGE
argocd-application-controller-0                     1/1     Running   0               5m33s
argocd-applicationset-controller-65955b986d-kprnh   1/1     Running   2 (5m29s ago)   5m33s
argocd-dex-server-758f576bb7-sptkn                  1/1     Running   0               5m33s
argocd-notifications-controller-54dd575496-td6gt    1/1     Running   0               5m33s
argocd-redis-6ddc76cf75-wkk9c                       1/1     Running   0               5m33s
argocd-repo-server-8497dcbd4-xw5nk                  1/1     Running   0               5m33s
argocd-server-64c545845d-c7kqv                      1/1     Running   0               5m33s


Перенастроил svc на nodeport для открытия web интерфейса

Пересоздал branch network так как все файлы были в корне при создание приложения выходила ошибка 
Создал папку "kubernetis-network" в нее поместил yaml файлы


Создал Проект Otus в argoCD

Создал project в argocd с названием "otus"
Создал project в argocd с названием "helm"
Прикрепил манифесты в корнево директории
Скрины в приложении argocd


![Image alt](https://github.com/Kuber-2024-04OTUS/trimol_repo/blob/kubernetes-gitops/images/scrin0.png)
