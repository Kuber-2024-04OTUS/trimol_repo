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


# смена namespace с def на homework
kubectl config set-context --current --namespace=homework


# Добавление label на node для точного развертывания
kubectl label nodes minikube homework=true



# Добавление configmap
kubectl apply -f .\configMap.yaml
# kubectl get configmap
# NAME               DATA   AGE
# cm-nginx           1      4m14s
# kube-root-ca.crt   1      4m44s





# создание service 
kubectl apply -f service.yaml
# kubectl get service          
# NAME                  TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
# homework-deployment   NodePort   10.96.4.249   <none>        8000:30999/TCP   7s




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
minikube service homework-deployment --url -n homework
# http://127.0.0.1:59156

curl http://127.0.0.1:59156

# StatusCode        : 200
# StatusDescription : OK
# Content           : <html><head><title>homework</title></head><body><h1>Homework</h1></body></html>
#
# RawContent        : HTTP/1.1 200 OK
#                    Connection: keep-alive
#                    Accept-Ranges: bytes
#                    Content-Length: 80
#                    Content-Type: text/html
#                    Date: Tue, 14 May 2024 15:18:42 GMT
#                    ETag: "66437eec-50"
#                    Last-Modified: Tue, 14 May 2024 15...
# Forms             : {}
# Headers           : {[Connection, keep-alive], [Accept-Ranges, bytes], [Content-Length, 80], [Content-Type, text/html]...}
# Images            : {}
# InputFields       : {}
# Links             : {}
# ParsedHtml        : System.__ComObject
# RawContentLength  : 80