# Создание namespace в helm не производится по этой причине создание namespace выполнил коммандой 
kubectl apply -f .\namespace.yaml


# Селектор node для развертывания {{.Values.nodeAffinity.key}} 
# Выставление данного key произвожу коммандой 
kubectl label nodes minikube homework=true

# необходим ingress addons 
minikube addons enable ingress

# и metric addons
minikube addons enable metrics-server

# добавление репозитория
helm repo add bitnami https://charts.bitnami.com/bitnami

# получение redis 
helm dependency build helm/


helm install homework helm\

# kubectl get po
# NAME                        READY   STATUS    RESTARTS      AGE
# homework-8f87db4cf-4bgfb    1/1     Running   0             81s
# homework-8f87db4cf-r6q6d    1/1     Running   0             81s
# homework-8f87db4cf-tfcxg    1/1     Running   0             81s
# homework-redis-master-0     1/1     Running   0             81s
# homework-redis-replicas-0   1/1     Running   1 (41s ago)   81s
# homework-redis-replicas-1   0/1     Running   0             16s



# Задание 2

helm install mykafka bitnami/kafka --namespace prod