# Репозиторий для выполнения домашних заданий курса "Инфраструктурная платформа на основе Kubernetes-2024-02" 

# Создание sa
kubectl apply -f sa.yaml

# Свзяываем с ролью cluster-admin
kubectl create clusterrolebinding mysql-controller --clusterrole=cluster-admin --serviceaccount=default:mysql-controller

# Привязываем crd
kubectl apply -f crd.yaml

# Запускаем deploymrnt контроллера crd
kubectl apply -f mysql-controller.yaml

# Просмотр log
 kubectl logs -f ID -c main
# /usr/local/lib/python3.10/site-packages/kopf/_core/reactor/running.py:179: FutureWarning: Absence of either namespaces or cluster-wide flag will become an error soon. For now, switching to the cluster-wide mode for backward compatibility.
# warnings.warn("Absence of either namespaces or cluster-wide flag will become an error soon."
# [2024-06-24 04:08:00,960] kopf._core.engines.a [INFO    ] Initial authentication has been initiated.
# [2024-06-24 04:08:00,961] kopf.activities.auth [INFO    ] Activity 'login_via_client' succeeded.
# [2024-06-24 04:08:00,962] kopf._core.engines.a [INFO    ] Initial authentication has finished.
# [2024-06-24 04:08:00,977] kopf._core.reactor.o [WARNING ] Not enough permissions to watch for resources: changes (creation/deletion/updates) will not be noticed; the resources are only refreshed on operator restarts.



# Создаем обьект cr
kubectl apply -f cr.yaml
# [2024-06-24 04:08:18,813] kopf.objects         [INFO    ] [default/mysqll] Creating pv, pvc for mysql data and svc...
# [2024-06-24 04:08:18,833] kopf.objects         [INFO    ] [default/mysqll] Creating mysql deployment...
# [2024-06-24 04:08:18,845] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:08:28,860] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:08:38,875] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:08:48,889] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:08:58,903] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:09:08,917] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:09:18,935] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:09:28,948] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:09:38,964] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:09:48,979] kopf.objects         [INFO    ] [default/mysqll] Waiting for mysql deployment to become ready...
# [2024-06-24 04:09:58,993] kopf.objects         [INFO    ] [default/mysqll] MySQL instance mysqll and its children resources created!
# [2024-06-24 04:09:58,996] kopf.objects         [INFO    ] [default/mysqll] Handler 'mysql_on_create' succeeded.
# [2024-06-24 04:09:58,996] kopf.objects         [INFO    ] [default/mysqll] Creation is processed: 1 succeeded; 0 failed.


# Проверка service, PV и pvc
kubectl get service
# mysqll       ClusterIP   None         <none>        3306/TCP   2m20s
kubectl get pv
# mysqll-pv   1k         RWO            Retain           Bound    default/mysqll-pvc   standard       <unset>                          2m24s
kubectl get pvc
# mysqll-pvc   Bound    mysqll-pv   1k         RWO            standard       <unset>                 2m26s


# Проверка на удаление ресурсов с обьектом cr
kubectl delete -f cr.yaml
# Ресурсы service, po, pv и pvc были удаленны






# Задание c*
# Переключается на Cluster-wide и использование Role и RoleBinding вместо ClusterRole и ClusterRoleBinding приводит к ошибке
# /usr/local/lib/python3.10/site-packages/kopf/_core/reactor/running.py:179: FutureWarning: Absence of either namespaces or cluster-wide flag will become an error soon. For now, switching to the cluster-wide mode for backward compatibility.
# warnings.warn("Absence of either namespaces or cluster-wide flag will become an error soon."

# Создание sa
kubectl apply -f sa.yaml

# Создание ClusterRole
kubectl apply -f sa-r.yaml

# Создание clusterroleBinding
kubectl apply -f sa-rb.yaml

# Привязываем crd
kubectl apply -f crd.yaml

# Запускаем deploymrnt контроллера crd
kubectl apply -f mysql-controller.yaml

# Создаем обьект cr
kubectl apply -f cr.yaml
# Создание и удаление po, pvб pvc и service выполняется








#задание c**
# Для выполнения была развернута виртуальная машина с ubuntu 22 LTS установлени minikube, operator-sdk, go
# Задание выполнялось в папке operator

operator-sdk version
# operator-sdk version: "v1.33.0", kubernetes version: "1.27.0", go version: "go1.21.5", GOOS: "linux", GOARCH: "amd64"


sudo operator-sdk init --domain homework --plugins ansible
sudo operator-sdk create api --group otus --version v1 --kind MySQL --generate-role

# Внес изменения в файлы roles/mysql/tasks/main.yml на создание deployment, pv и service 
# pvc выдавал ошибку перенес его в /opt/oper/config/crd/bases изменив kustomization.yaml по пути config/crd для выполнения yaml pvc.yaml


#сборка
make docker-build IMG=trimolvl/mysql:v1
sudo make docker-push IMG=trimolvl/mysql:v1
make deploy IMG=trimolvl/mysql:v1

# применение cr
kubectl apply -f cr-operator.yaml

# лог выполнения прикрепил в фале log.txt