Пошаговая инструкция выполнения домашнего задания
1. В namespace homework создать service account monitoring и дать ему доступ к эндпоинту /metrics вашего кластера
2. Изменить манифест deployment из прошлых ДЗ так, чтобы поды запускались под service account monitoring
3. В namespace homework создать service account с именем cd и дать ему роль admin в рамках namespace homework
4. Создать kubeconfig для service account cd
5. Сгенерировать для service account cd токен с временем действия 1 день и сохранить его в файл token
6. Задание с * 
Модифицировать deployment из прошлых ДЗ так, чтобы в процессе запуска pod происходило обращение к эндпоинту /metrics вашего кластера (механика вызова не принципиальна), результат ответа сохранялся в файл metrics.html и содержимое этого файла можно было бы получить при обращении по адресу /metrics.html вашего сервиса

Создаем SA c именем "monitoring". Чтобы дать ему доступ к эндпоинту /metrics:
- Создаем ClusterRole c правилами 
```
rules:
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
```
- Создаем ClusterRoleBinding, чтобы связать ClusterRole с нашим ServiceAccount
```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: metrics-reader
subjects:
- kind: ServiceAccount
  name: monitoring
  namespace: homework
```

Чтобы поды запускались под SA monitoring, добавляем в спеку параметр
```
spec:
  serviceAccountName: monitoring
```

Создаем SA с именем "cd". Чтобы дать ему роль admin в рамках NS:
- Создаем RoleBinding связывая стандартую ClusterRole admin c SA cd
```
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: cd
  namespace: homework
```

Пример выполнения
```
[kaa24@vm-b7e133ee-fe90-4d0b-b887-c024868cb530 homework]$ kubectl apply -f cd-sa.yaml 
serviceaccount/cd created
rolebinding.rbac.authorization.k8s.io/cd-admin created

[kaa24@vm-b7e133ee-fe90-4d0b-b887-c024868cb530 homework]$ kubectl get sa -n homework
NAME         SECRETS   AGE
cd           0         40s
default      0         70d
monitoring   0         49d

[kaa24@vm-b7e133ee-fe90-4d0b-b887-c024868cb530 homework]$ kubectl get rolebinding -n homework
NAME       ROLE                AGE
cd-admin   ClusterRole/admin   3m20s
```

Генерируем токен
```
[kaa24@vm-b7e133ee-fe90-4d0b-b887-c024868cb530 homework]$ kubectl create token cd -n homework --duration=24h
eyJhbGciOiJSUzI1NiIsImtpZCI6IkE0b19TV3UwcG1qZUFIV3lpOWZrM3NZRWVGVC1nbzNLU1RkRlRHVkFjZ2sifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzI4MzMwNzI3LCJpYXQiOjE3MjgyNDQzMjcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiY2Q4NTVjM2ItNWRlOS00ZmI1LWIzODktMmUzNDZjZmI1M2IyIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJob21ld29yayIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJjZCIsInVpZCI6ImY3MmI3ZWU2LTFjNjItNDM5Yy05MWM5LTA3ODRhM2I5OTk5NSJ9fSwibmJmIjoxNzI4MjQ0MzI3LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6aG9tZXdvcms6Y2QifQ.DaOCUrCAVwT6Nf6lBTlEljdSSAJuBbzqxdP0B_2ozvNHGYTjf533CWpmodYrJOiPNYliai1PAnSRqLB0TNXL8yvTzTVeaRu60P4ZoYP6CtFEQrDHLGgFPW0J7JUUGljiTZopQlBVIvQen_Ud0hwxiPpUJBcn_0kw55mDWOsGAXkeG_Ozjg0909RKmTQmD31_nVpZn1vkICspfIADEYWpdpEBXvHOBF3EVHb1anRcAAjMwlY-cYOAkdN83eJyEkVJR-ZI90F4KAHNr8mS7iwW5mZe7ZvxwIsegS1pfZw0rlWbF8B1HVavBnMMFtDJR7om1DqaEsBhPrHZVdD7vVwM0w
```
