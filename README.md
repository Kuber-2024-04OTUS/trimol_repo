# Запустил minikube через hyperV для более удобной проверки так как пока работаю на windows 11
minikube start --vm-driver=hyperv



# Создание namespace homework
kubectl apply -f namespace.yaml

# Результат
# kubectl get namespace          
# NAME              STATUS   AGE
# default           Active   23h
# homework        Active   23h
# kube-node-lease   Active   23h
# kube-public       Active   23h
# kube-system       Active   23h



# смена namespace с def на homework
kubectl config set-context --current --namespace=homework



# активация ingress addons 
minikube addons enable ingress
# The 'ingress' addon is enabled


# Добавление configmap
kubectl apply -f configMap.yaml
# kubectl get configmap
# NAME               DATA   AGE
# cm-nginx           1      4m14s
# kube-root-ca.crt   1      4m44s

# создание serviceIP 
kubectl apply -f service.yaml
# kubectl get service          
# NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
# homework-deployment   ClusterIP   10.101.79.147   <none>        8000/TCP   2s

# Запуск
kubectl apply -f deployment.yaml
# kubectl get po
#kubectl get po
#NAME                         READY   STATUS    RESTARTS   AGE
#homework2-7c8547df8c-zp87k   1/1     Running   0          4d



# kubectl get deployment
# NAME        READY   UP-TO-DATE   AVAILABLE   AGE
#homework2   1/1     1            1           4d


kubectl get service
# NAME                  TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
# homework-deployment   NodePort   10.96.4.249   <none>        8000:30999/TCP   5m45s


# Применение ingress 
kubectl apply -f ingress.yaml

# Получение ip для проверки 
kubectl get ingress 

# NAME                  CLASS   HOSTS           ADDRESS         PORTS   AGE
# homework-deployment   nginx   homework.otus   192.168.49.2   80      49s

# добавил значение в файлл /etc/hosts
# 192.168.49.2   homework.otus

#Проверка получения 
curl http://homework.otus/basic_status
#Active connections: 2
#server accepts handled requests
# 4 4 5
#Reading: 0 Writing: 1 Waiting: 1


#Установка Prometheus
LATEST=$(curl -s https://api.github.com/repos/prometheus-operator/prometheus-operator/releases/latest | jq -cr .tag_name)
curl -sL https://github.com/prometheus-operator/prometheus-operator/releases/download/${LATEST}/bundle.yaml | kubectl create -f -
kubectl wait --for=condition=Ready pods -l  app.kubernetes.io/name=prometheus-operator -n default
#pod/prometheus-operator-59f5dbf4f7-6vfbq condition met

#создаем sa
kubectl apply -f sa-prometheus.yaml

#создаем role
kubectl apply -f rb-prometheus.yaml

#создаем cluster role
kubectl apply -f clusterrole.yaml

# Прминяем crd 
kubectl apply -f prometheus.yaml


kubectl get po
\NAME                         READY   STATUS    RESTARTS   AGE
homework2-7c8547df8c-zp87k   1/1     Running   0          4d1h
prometheus-prometheus-0      2/2     Running   0          2m33s


kubectl get -n default prometheus prometheus -w
NAME         VERSION   DESIRED   READY   RECONCILED   AVAILABLE   AGE
prometheus                       1       True         True        4d



kubectl apply -f podmonitor.yaml


