# Домашнее задание к занятию «`Хранение в K8s. Часть 1`» - `Живарев Игорь`

### Цель задания

В тестовой среде Kubernetes нужно обеспечить обмен файлами между контейнерам пода и доступ к логам ноды.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке MicroK8S](https://microk8s.io/docs/getting-started).
2. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).
3. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.

------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---


## Ответ


### Задание 1. Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Манифест `Deployment` и запуск `Pod`
![manifest](img/k8s-06_01.png)

2. Подключаемся к `Pod` с `busybox`, проверяем наполняемость файла
![busybox](img/k8s-06_02.png)

Подключаемся к `Pod` с `multitool`, проверяем возможность чтения и наполнение  
![multitool](img/k8s-06_03.png)


Листинг `Deployment`:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: netology
  labels:
    app: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo HNetology! >> /output/output.txt; sleep 5; done']
        volumeMounts:
        - name: volume
          mountPath: /output
      - name: multitool
        image: wbitt/network-multitool:latest
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "8080"
        volumeMounts:
        - name: volume
          mountPath: /input
      volumes:
      - name: volume
        emptyDir: {}
```


### Задание 2. Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Манифест `DaemonSet` приложения и запуск `Pod`
![DaemonSet](img/k8s-06_05.png)

2. Подключаемся к `Pod` с `DaemonSet`, проверяем возможность чтения `/var/log/syslog`
![syslog](img/k8s-06_04.png)


Листинг `DaemonSet`:
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-multitool
  namespace: netology
  labels:
    app: ds-multitool
spec:
  selector:
    matchLabels:
      app: ds-multitool
  template:
    metadata:
      labels:
        app: ds-multitool
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
        - name: volume
          mountPath: /input/logs
        ports:
        - containerPort: 8080
      volumes:
      - name: volume
        hostPath:
          path: /var/log/syslog
```

---