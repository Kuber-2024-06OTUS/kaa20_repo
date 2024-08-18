Пошаговая инструкция выполнения домашнего задания
1. В namespace homework создать service account monitoring и дать ему доступ к эндпоинту /metrics вашего кластера
2. Изменить манифест deployment из прошлых ДЗ так, чтобы поды запускались под service account monitoring
3. В namespace homework создать service account с именем cd и дать ему роль admin в рамках namespace homework
4. Создать kubeconfig для service account cd
5. Сгенерировать для service account cd токен с временем действия 1 день и сохранить его в файл token
6. Задание с * 
Модифицировать deployment из прошлых ДЗ так, чтобы в процессе запуска pod происходило обращение к эндпоинту /metrics вашего кластера (механика вызова не принципиальна), результат ответа сохранялся в файл metrics.html и содержимое этого файла можно было бы получить при обращении по адресу /metrics.html вашего сервиса

