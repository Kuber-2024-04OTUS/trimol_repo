# Запустил minikube через hyperV для более удобной проверки так как пока работаю на windows 11
minikube start --vm-driver=hyperv



# Создание namespace homework
kubectl apply -f namespace.yaml

# Результат
# kubectl get namespace          
# NAME              STATUS   AGE
# default           Active   23h
# homework          Active   23h
# kube-node-lease   Active   23h
# kube-public       Active   23h
# kube-system       Active   23h



# смена namespace с def на homework
kubectl config set-context --current --namespace=homework



# Создание storageclass
kubectl apply -f storageClass.yaml
# kubectl get sc     
# NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE    ALLOWVOLUMEEXPANSION   AGE
# homework-sc          k8s.io/minikube-hostpath   Retain          Immediate           false                  18s
# standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  4h23m 

# создание PVC
kubectl apply -f pvc.yaml
# kubectl get pvc
# NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
# homework-pvc   Bound    pvc-f9a3a1ba-5a0c-4f17-87ff-2601fd3287c6   1Gi        RWO            homework-sc    <unset>                 33s



kubectl apply -f sa-monitoring.yaml
kubectl apply -f croleBinding-monitoring.yaml
kubectl apply -f crole-monitoruing.yaml




# Добавление label на node для точного развертывания
kubectl label nodes minikube homework=true



# установка ingress addons 
minikube addons enable ingress

# включение сервиса метрик
minikube addons enable metrics-server
# The 'ingress' addon is enabled
# ingress-nginx   ingress-nginx-controller-84df5799c-tw5ld   1/1     Running     0          20m

# Добавление configmap
kubectl apply -f configMap.yaml
# kubectl get configmap
# NAME               DATA   AGE
# cm-nginx           1      4m14s
# kube-root-ca.crt   1      4m44s

# Добавление configmap
kubectl apply -f cm.yaml
# kubectl get configmap
# NAME               DATA   AGE
# cm                 1      24m
# cm-nginx           1      12h
# kube-root-ca.crt   1      12h



# создание serviceIP 
kubectl apply -f service.yaml
# kubectl get service          
# NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
# homework-deployment   ClusterIP   10.101.79.147   <none>        8000/TCP   2s




# Запуск
kubectl apply -f deployment.yaml
# kubectl get po
# NAME                        READY   STATUS    RESTARTS   AGE
# homework2-6db46fd9b-dv6d6   1/1     Running   0          65s
# homework2-6db46fd9b-lvqrk   1/1     Running   0          65s
# homework2-6db46fd9b-qhqwz   1/1     Running   0          65s


# kubectl get deployment
# NAME        READY   UP-TO-DATE   AVAILABLE   AGE
# homework2   3/3     3            3           70s


# Проверка функционирования
# minikube service <имя сервиса> --url -n homework
kubectl get service
# NAME                  TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
# homework-deployment   NodePort   10.96.4.249   <none>        8000:30999/TCP   5m45s


# Применение ingress 
kubectl apply -f ingress.yaml

# Получение ip для проверки 
kubectl get ingress 

# NAME                  CLASS   HOSTS           ADDRESS         PORTS   AGE
# homework-deployment   nginx   homework.otus   172.29.205.64   80      49s

# добавил значение в файлл C:\Windows\System32\drivers\etc\hosts
# 172.29.205.64   homework.otus

# При открытие в браузере по url http://homework.otus/ открывается успешно

# При открытие по url http://homework.otus/conf/file отображются переменные созданные в configmap cm




# включение сервисв метрик
minikube addons enable metrics-server










# Создание пользователя cd с правами cluster-admin для namespace homework
start  cd create user
kubectl create clusterrolebinding cb_role_binding --clusterrole=cluster-admin --serviceaccount=homework:cd
kubectl config set-context cd --cluster=minikube --user cd

# создание token на сутки
kubectl create token cd --duration=24h
# Name:         secret-cd
# Namespace:    homework
# Labels:       <none>
# Annotations:  kubernetes.io/service-account.name: cd
#               kubernetes.io/service-account.uid: 7dd9d2e0-dbc4-4db6-8cce-f3c6631a507e

# Type:  kubernetes.io/service-account-token

# Data
# ====


# Создан файл token




# При переходе на url http://homework.otus/metric.html выводится метрики nodes
