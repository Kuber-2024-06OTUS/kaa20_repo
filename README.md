# Репозиторий для выполнения домашних заданий курса "Инфраструктурная платформа на основе Kubernetes-2024-06" 

## Пошаговая инструкция выполнения домашнего задания #1
1. Для выполнения домашнего задания вам потребуется установить Minikube. 
Если вы по каким-то причинам предпочитаете работать с другими локальными инсталляциями k8s можете использовать их, для выполнения ДЗ это не принципиально.
2. Установите утилиту kubectl
3. Необходимо создать манифест **namespace.yaml** для namespace с именем **homework**
4. Необходимо создать манифест **pod.yaml**. Он должен описывать под, который:
	- Будет создаваться в namespace homework
	- Будет иметь контейнер, поднимающий веб-сервер на 8000 порту и отдающий содержимое папки /homework внутри этого контейнера.
	- Будет иметь init-контейнер, скачивающий или генерирующий файл index.html и сохраняющий его в директорию /init
	- Будет иметь общий том (volume) для основного и init- контейнера, монтируемый в директорию /homework первого и /init второго
	- Будет удалять файл index.html из директории /homework основного контейнера, перед его завершением

## Пошаговая инструкция выполнения домашнего задания #2
1. Необходимо создать манифест namespace.yaml для namespace с именем homework
2. Необходимо создать манифест deployment.yaml. Он должен описывать deployment, который:
	- Будет создаваться в namespace homework
	- Запускает 3 экземпляра пода, полностью
	аналогичных по спецификации прошлому ДЗ.
	- В дополнение к этому будет иметь readiness пробу, проверяющую наличие файла /homework/index.html
	- Будет иметь стратегию обновления RollingUpdate, настроенную так, что в процессе обновления может быть недоступен максимум 1 под
3. Задание с *
	- Добавить к манифесту deployment-а спецификацию, обеспечивающую запуск подов деплоймента, только на нодах кластера, имеющих метку homework=true
 (документация https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/ )

### Создание манифестов
Создаем манифест namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: homework
```

Создаем манифест deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: homework
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
        <тут описание Pod>
```

Для нашего веб-сервера также потребуется создать конфиг для volume nginx
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
  namespace: homework
data:
  nginx.conf: |
    events {}
    http {
      server {
          listen       8000;
          server_name  localhost;
          location / {
            root   /homework;
            index  index.html index.htm;
        }
      }
    }
```

### Запуск и проверка работоспособности
Применяем манифесты
`kubectl apply -f namespace.yaml`
`kubectl apply -f nginx-cm.yaml -f deployment.yaml`

Проверяем что все 3 экземпляра запущены
`kubectl get po -A`

![Проверяем деплоймент](./assets/images/hw2/get_deploy.png)

![Проверяем поды](./assets/images/hw2/get_po.png)

Заходим в терминал пода веб-сервиса
`kubectl exec -it deploy/nginx-deployment --namespace homework -- bash`

Проверяем что применился наш конфиг nginx
`cat /etc/nginx/nginx.conf`

Делаем http запрос к сервису
`curl localhost:8000`
Если получаем страницу, которую ранее выгрузил инит-контейнер, то все сделано правильно.

![Запрос к веб-сервису](./assets/images/hw2/exec_and_curl.png)

### Добавляем спецификацию, обеспечивающую запуск подов деплоймента, только на нодах кластера, имеющих метку homework=true
Проверяем список нод и их лейблы:
`kubectl get nodes --show-labels`
Выбираем ноду и выставляем для нее лейбл homework=true:
`kubectl label nodes minikube homework=true`
Удалить лейбл можно командой:
`kubectl label nodes minikube homework-`

![Выставляем лейбл для ноды](./assets/images/hw2/label_nodes.png)

Добавляем спецификацию с меткой в разделе spec для Pod:
```
  template:
    metadata:
      labels:
        app: nginx-with-node-affinity
    spec:
      nodeSelector:
        homework: "true"
      containers:
```

!Значение для метки должно быть строковым иначе получим ошибку при создании деплоймента.

Для проверки работы метки изменим значение на "false" и применим деплоймент
![Деплоймент не готов](./assets/images/hw2/deployment_not_ready.png)
![Поды в статусе Pending](./assets/images/hw2/pods_pending.png)
![Фрагмент описания Pod](./assets/images/hw2/node_affinity_error.png)

Меняем значение на "true", применяем манифест деплоймента и убеждаемся, что Pod успешно запустились.

