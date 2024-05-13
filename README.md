# Репозиторий для выполнения домашних заданий курса "Инфраструктурная платформа на основе Kubernetes-2024-02" 

# Создание namespace homework
kubectl apply -f namespace.yaml

# Добавление label на node для точного развертывания
kubectl label nodes minikube homework=true

# Запуск
kubectl apply -f deployment.yaml

# Проверка "livenessProbe" провел сделав ошибку в имени файла index.html - в результате контейнеры перезапускаются каждые 25 сек
