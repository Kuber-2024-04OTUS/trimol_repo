# Перешел на driver vmware
minikube start --driver vmware




# Добавление configmap
kubectl apply -f ConfigMap.yaml
# kubectl get configmap
# NAME               DATA   AGE
# nc                 1      4m14s
# kube-root-ca.crt   1      4m44s


# Запуск
kubectl apply -f deployment.yaml
# kubectl get po
# NAME                     READY   STATUS    RESTARTS   AGE
# nginx-6f4f49c478-8hdsx   2/2     Running   0          7m59s


# создание service 
kubectl apply -f Service.yaml
# kubectl get service          
# NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)
# nginx   NodePort   10.103.188.241   <none>        80:30415/TCP,9113:30869/TCP   4s







kubectl get deployment
# NAME    READY   UP-TO-DATE   AVAILABLE   AGE
# nginx   1/1     1            1           10m




# Установка Prometheus 
helm install my-release oci://registry-1.docker.io/bitnamicharts/kube-prometheus


kubectl get po
# NAME                                                            READY   STATUS    RESTARTS   AGE
# alertmanager-my-release-kube-prometheus-alertmanager-0          1/2     Running   0          116s
# my-release-kube-prometheus-blackbox-exporter-7dfbb4f4bc-5jrks   1/1     Running   0          2m38s
# my-release-kube-prometheus-operator-5d698878f6-gnrw7            1/1     Running   0          2m38s
# my-release-kube-state-metrics-6df5584646-rtggt                  1/1     Running   0          2m38s
# my-release-node-exporter-vhkrm                                  1/1     Running   0          2m38s
# nginx-6f4f49c478-8hdsx                                          2/2     Running   0          13m
# prometheus-my-release-kube-prometheus-prometheus-0              2/2     Running   0          116s


# Запускаем ingree для подключение к web интерфейсу prometheus
minikube addons enable ingress


kubectl apply -f ingree-prometheus.yaml



kubectl get ingress
# NAME                  CLASS   HOSTS        ADDRESS          PORTS   AGE
# prometheus-operated   nginx   prometheus   192.168.66.140   80      76s

# добавляю запись в hosts


# Запускаем service monitor
kubectl apply -f serviceMonitor.yaml

# В web интерфейсе prometheus переходим в status->Targets 
# Созданный service monitor добавился и target up
# serviceMonitor/default/nginx/0 (1/1 up)