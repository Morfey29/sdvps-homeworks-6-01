# Домашнее задание к занятию «Kubernetes. Часть 1»

Это задание для самостоятельной отработки навыков и не предполагает обратной связи от преподавателя. Его выполнение не влияет на завершение модуля. Но мы рекомендуем его выполнить, чтобы закрепить полученные знания.

### Задание 1

**Выполните действия:**

1. Запустите Kubernetes локально, используя k3s или minikube на свой выбор.
1. Добейтесь стабильной работы всех системных контейнеров.


### Задание 2

Есть файл с деплоем:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: master
        image: bitnami/redis
        env:
         - name: REDIS_PASSWORD
           value: password123
        ports:
        - containerPort: 6379
```


**Выполните действия:**

1. Измените файл с учётом условий:

 * redis должен запускаться без пароля;
 * создайте Service, который будет направлять трафик на этот Deployment;
 * версия образа redis должна быть зафиксирована на 6.0.13.

2. Запустите Deployment в своём кластере и добейтесь его стабильной работы.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.0.13
        ports:
        - containerPort: 6379
        command: ["redis-server"]
        args: ["--requirepass", ""] 
```


### Задание 3

**Выполните действия:**

1. Напишите команды kubectl для контейнера из предыдущего задания:

 - выполнения команды ps aux внутри контейнера;
 - просмотра логов контейнера за последние 5 минут;
 - удаления контейнера;
 - проброса порта локальной машины в контейнер для отладки.
```
Вот команды kubectl для работы с контейнером Redis из предыдущего задания:

1. Выполнение команды ps aux внутри контейнера
bash
# Выполнить команду ps aux в контейнере redis
kubectl exec deployment/redis-deployment -- ps aux

# Если в поде несколько контейнеров, укажите конкретный
kubectl exec deployment/redis-deployment -c redis -- ps aux

# Альтернативный вариант - получить интерактивный shell
kubectl exec deployment/redis-deployment -it -- /bin/bash
# затем внутри контейнера: ps aux
2. Просмотр логов контейнера за последние 5 минут
bash
# Логи за последние 5 минут
kubectl logs deployment/redis-deployment --since=5m

# С потоковым выводом (follow)
kubectl logs deployment/redis-deployment --since=5m -f

# Логи конкретного пода (если несколько реплик)
kubectl logs -l app=redis --since=5m

# С таймстампами
kubectl logs deployment/redis-deployment --since=5m --timestamps
3. Удаление контейнера (пода)
bash
# Удаление deployment (удалит все связанные pods)
kubectl delete deployment redis-deployment

# Удаление конкретного пода (если нужно удалить только один)
kubectl delete pod -l app=redis

# Принудительное удаление пода
kubectl delete pod -l app=redis --force --grace-period=0
4. Проброс порта локальной машины в контейнер для отладки
bash
# Проброс порта 6379 контейнера на localhost:6379
kubectl port-forward deployment/redis-deployment 6379:6379

# Проброс в фоновом режиме
kubectl port-forward deployment/redis-deployment 6379:6379 &

# Проброс на другой локальный порт
kubectl port-forward deployment/redis-deployment 16379:6379

# Проброс к конкретному поду
kubectl port-forward pod/redis-deployment-xxxxx 6379:6379

# После проброса можно подключиться с локальной машины:
redis-cli -h localhost -p 6379
Дополнительные полезные команды:
bash
# Проверить статус deployment
kubectl get deployment redis-deployment

# Описание deployment
kubectl describe deployment redis-deployment

# Список подов с меткой app=redis
kubectl get pods -l app=redis

# Перезапуск deployment (создаст новые pods)
kubectl rollout restart deployment/redis-deployment
Все команды предполагают, что deployment и сервис развернуты в namespace по умолчанию. Если используется другой namespace, добавьте флаг -n <namespace-name> к командам.
```

### Задание 4

Есть конфигурация nginx:

```
location / {
    add_header Content-Type text/plain;
    return 200 'Hello from k8s';
}
```

**Выполните действия:**

Напишите yaml-файлы для развёртки nginx, в которых будут присутствовать:

 - ConfigMap с конфигом nginx;
 - Deployment, который бы подключал этот configmap;
 - Ingress, который будет направлять запросы по префиксу /test на наш сервис.


# Домашнее задание к занятию «Kubernetes. Часть 2»

Это задание для самостоятельной отработки навыков и не предполагает обратной связи от преподавателя. Его выполнение не влияет на завершение модуля. Но мы рекомендуем его выполнить, чтобы закрепить полученные знания.

### Задание 1

**Выполните действия:**

1. Создайте свой кластер с помощью kubeadm.
1. Установите любой понравившийся CNI плагин.
1. Добейтесь стабильной работы кластера.


### Задание 2

Есть файл с деплоем:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: master
        image: bitnami/redis
        env:
         - name: REDIS_PASSWORD
           value: password123
        ports:
        - containerPort: 6379
```
**Выполните действия:**

1. Создайте Helm Charts.
1. Добавьте в него сервис.
1. Вынесите все нужные, на ваш взгляд, параметры в `values.yaml`.
1. Запустите чарт в своём кластере и добейтесь его стабильной работы.


### Задание 3*

1. Изучите [документацию](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) по подключению volume типа hostPath.
1. Дополните деплоймент в чарте подключением этого volume.
1. Запишите что-нибудь в файл на сервере, подключившись к поду с помощью `kubectl exec`, и проверьте правильность подключения volume.


