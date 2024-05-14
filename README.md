# Репозиторий для выполнения домашних заданий курса "Инфраструктурная платформа на основе Kubernetes-2024-02" 

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


# Вопрос по пути монтирования если изменить пути с /usr/share/nginx/html/ то придется менять конфигурацию nginx и указать новый путь


# Добавление label на node для точного развертывания
kubectl label nodes minikube homework=true

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


# Разобрался в Liveness, Readiness and Startup Probes

# readinessProbe проходит до привязки к service и port forwarding здесь тоже либо менять порт на котором слушает nginx либо проверять на 80 а потом на service  менять на 8000 если я правильно понял 

kubectl apply -f service.yaml

# kubectl get service
# NAME                  TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
# homework-deployment   NodePort   10.98.255.212   <none>        8000:31643/TCP   97m

# С проверкой и сетью пока не разобрался проверял пробросом на pod коммандой "kubectl port-forward  homework2-6db46fd9b-d9vrl 8010:80" но с изменением путей отображается приветствие nginx