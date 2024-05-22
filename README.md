# Репозиторий для выполнения домашних заданий курса "Инфраструктурная платформа на основе Kubernetes-2024-02" 


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



# Добавление label на node для точного развертывания
kubectl label nodes minikube homework=true



# установка ingress addons 
minikube addons enable ingress
# The 'ingress' addon is enabled
# ingress-nginx   ingress-nginx-controller-84df5799c-tw5ld   1/1     Running     0          20m

# Добавление configmap
kubectl apply -f .\configMap.yaml
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


# задание со * не понял что требуется 301 редирект на nginx выполнить 