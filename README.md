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


# Добавление label на node для точного развертывания
kubectl label nodes minikube homework=true



# установка ingress addons 
minikube addons enable ingress
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






# /var/run/secrets/kubernetes.io/serviceaccount


# APISERVER=https://kubernetes.default.svc
# SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
# NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)
# TOKEN=$(cat ${SERVICEACCOUNT}/token)
# CACERT=${SERVICEACCOUNT}/ca.crt
# curl --cacert ${CACERT} --header "Authorization: Bearer fdgdfgdfgfdg" -X GET ${APISERVER}/api





# miniku ssh
# sudo mkdir -p /cert
# sudo openssl genrsa -out /cert/homework.pem 2048
# sudo openssl req -new -key /cert/homework.pem -out /cert/homework-csr.pem -subj "/CN=homework/O=team1/"
# sudo openssl x509 -req -in /cert/homework-csr.pem -CA /var/lib/minikube/certs/ca.crt -CAkey /var/lib/minikube/certs/ca.key -CAcreateserial -out /cert/homework.crt -days 10000
# sudo chmod +r /cert/homework.pem

#  cat /cert/homework-csr.pem

-----BEGIN CERTIFICATE REQUEST-----
MIICaDCCAVACAQAwIzERMA8GA1UEAwwIaG9tZXdvcmsxDjAMBgNVBAoMBXRlYW0x
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2t7gIwXjC1362cGlFwQD
BHvTWasexeDtFXehVBcMu2GCNg9oV2sAzRkFvksLTvEgUm3r67rhnAJLZ6Z1Fw6q
hc3/97dApLb0n7KSDoxf6VMxl1eaXi/SiWPQx3azs13kK05q6QnqL5nkuTskw+5P
P9V/Gglt9qU3is9eBne8GPmv2Qz4V+z+TNOopIEianyPG4/K35kzm78HmAd0GJ+L
R9gFRZoSMPiVvnNxW9vTA2AFLBPVmvO/RRGJlB4hMb6StrmC3KsbEynxVe2Y9RXD
rZzoHZJZ+HxSqqpTCUwvHKLyVRFSNT7MdDcS65YWfX2tNq9y7Z/dLGgeZXbV30OE
SwIDAQABoAAwDQYJKoZIhvcNAQELBQADggEBAMPmihrioZqlxye5DySk4bazQ0fF
aWK8N7ss7FpwajJNHFBn7259S9cx6RrmWNP6P8a/I/gRx0fmPn/3WhXGYNKdTrXg
+waKG7CfAoGXLD/j22FCDiMkyYqVFbMB/HAQfkPUXneBvLBJIkB1SpS8Sd3rasYt
R7X+skr54u7YaKL4IxsrLN0uvOZ97W/GWJQZNH+GqZ5ec/30GxcZGFHCH/hOnC2E
1yGkCUkIEvgii2W8PMTZvuucj8Ac9PH8g3vX5qs98fZsrGzTOW9b0qpfBFdwnrpW
UTeY+ayLV3/8Ndjxg7qFDCvsY8kyxuH3TZdMrzWWD4rR3upOgG+k5jcpm7A=
-----END CERTIFICATE REQUEST-----


# cat /cert/homework.crt
-----BEGIN CERTIFICATE-----
MIICwTCCAakCFFrUCEgkyUEy/9D7TsMfnDxP5fcqMA0GCSqGSIb3DQEBCwUAMBUx
EzARBgNVBAMTCm1pbmlrdWJlQ0EwIBcNMjQwNTI1MTgxMTUwWhgPMjA1MTEwMTEx
ODExNTBaMCMxETAPBgNVBAMMCGhvbWV3b3JrMQ4wDAYDVQQKDAV0ZWFtMTCCASIw
DQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANre4CMF4wtd+tnBpRcEAwR701mr
HsXg7RV3oVQXDLthgjYPaFdrAM0ZBb5LC07xIFJt6+u64ZwCS2emdRcOqoXN//e3
QKS29J+ykg6MX+lTMZdXml4v0olj0Md2s7Nd5CtOaukJ6i+Z5Lk7JMPuTz/VfxoJ
bfalN4rPXgZ3vBj5r9kM+Ffs/kzTqKSBImp8jxuPyt+ZM5u/B5gHdBifi0fYBUWa
EjD4lb5zcVvb0wNgBSwT1Zrzv0URiZQeITG+kra5gtyrGxMp8VXtmPUVw62c6B2S
Wfh8UqqqUwlMLxyi8lURUjU+zHQ3EuuWFn19rTavcu2f3SxoHmV21d9DhEsCAwEA
ATANBgkqhkiG9w0BAQsFAAOCAQEAT5whAcnOfMEUJhVCKegTw1I83SJC8frdx2Ua
paBtTYhpP+AwKtFW8KoZWWSYxtDowPwSru/+j7VXIEwSHNOstsz5NFWiq9WDz2x6
dzFww+4vKPPf5mdfjv0+TkeEyh2x1dwirqhSnrjGWeiYwZRn58Lb2gs3RZsPqen3
b2t4aYZ6kulHqFY5j7ptstK3rPfd8LDOuH8gD1+wCDHu4iwmaoFlxwfEfhTx987o
5mIA9AOIjvxBvzfYmwnsyu2iYVJj3k1r8GbhEA1Yx5C8ukCU3N0I64aXEcqXbrSa
w4f7cDx8tCFRGC0oou9EZkoMzF/njv4IL9gM0DhZgf3v7aJKCQ==
-----END CERTIFICATE-----

# cat /cert/homework.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA2t7gIwXjC1362cGlFwQDBHvTWasexeDtFXehVBcMu2GCNg9o
V2sAzRkFvksLTvEgUm3r67rhnAJLZ6Z1Fw6qhc3/97dApLb0n7KSDoxf6VMxl1ea
Xi/SiWPQx3azs13kK05q6QnqL5nkuTskw+5PP9V/Gglt9qU3is9eBne8GPmv2Qz4
V+z+TNOopIEianyPG4/K35kzm78HmAd0GJ+LR9gFRZoSMPiVvnNxW9vTA2AFLBPV
mvO/RRGJlB4hMb6StrmC3KsbEynxVe2Y9RXDrZzoHZJZ+HxSqqpTCUwvHKLyVRFS
NT7MdDcS65YWfX2tNq9y7Z/dLGgeZXbV30OESwIDAQABAoIBAD748TQidXjHyWzt
sjo1BUk890pNWcVFOfF7QgeXuioPnA9Bz5uXRdu831Io9U/5QHt29PIWCuOAJYHk
+rtvlNB6vLLhHtBFc4yfLDbTXKUdMp+ArQF8cO7msSMym/F3ijaLkWcLWz3m4VAK
HEHjRxnuq2sqW2EjmB9wGnVx7cShSf8If5+u2VvEBdHNCgNcMRqNtgRJggrhfDm8
g8Vn2WAiD1rJhOmkRsujRDHluFWE0A5yxLREE5inM0hNphIYkHCHnPJ3mMygQ/df
YdO9wCZXEBmENoSjH6WKSOQfCBXw8uxSLy3rkgidR/FEHYu/0ghk5AmaVkkR1OxA
DSmQMwECgYEA+JhKr1xmaIVIUiM5AHc4epOQy8D0LFEX5dnyRbWcOIkksNXVs9ew
UuDks7CPqMUo9m0zFFEEPktZfg6kVjZ2+EkiRn5fcV+vuY96kTtt6D1XRaZqPhgn
94y3BMpPmU81b05ef0nFYYIcaBpy2ORA3BCiCQ+23C4iVDkIJxyWQzECgYEA4WPq
YaKXu1LuC5eCMEgd5YTcxp0vUKd4AYynaFwfCTbjV/R614R+c/WkP5u9au6kB2ye
rP3JmZD6dmcpAdWcEcdN2RFx2aaJvc7gK2qXZ2J9KCjlozIzfyrZXlekjUNd9+4w
+/lW4SsGyJi2bUgH8+bavWkQpiWWu8EONyfwiDsCgYEAosrwFZg3w/iMHKXOPTzV
cofSCWwpOiA8uxuXaQj97ZP5wAe4M1ZqtEtr2TQlT4sVQRLPoV1Qnw6u4zrpaT+v
dvZFiM5W6CKWK7kGtbaqSaxpy0WoS1N8UMMIUw29RJM3VdWHUmnX5PvUaGPxk5Ed
3D7ULYTp5ZQcjPTwHtS8nfECgYB8RsNtomFXgJqQ+bFnPdx+OYwiV3lHV4/sCsoj
2OekBQfF75/sRboT5lXyXMVMuNjo3xN+/1CxxCbWMnuB725mvyZFkkDcad95MSCZ
z+Q7tSqdgi9clMmgTNgeFOU+nu6pTkTkKs+kyDytTscH6re33Iqv9cagVgmO7RwR
fYw7EQKBgQCIwJCb8FjMVezqEDjuff0wRJbs3GUAO8webpfaSclT9aMaoUlmVIqM
vMwbpdj/U7GJeyRhp3g293Hcnqc/W69wW6gLnXjWJ3Po9gFs73mjB+sfWn8aNG0g
XLOHbmIpwon+4256+/SRNWFwgxG/vfEkKQC7hqUcz30kpDS144mE1g==
-----END RSA PRIVATE KEY-----

# Перенес сертификаты


# kubectl config set-credentials cd --client-certificate=C:\Users\trimo\.kube\homework.crt --client-key=C:\Users\trimo\.kube\homework.pem

# kubectl config set-context cd --cluster=minikube --user cd

# C:\Users\trimo\.kube