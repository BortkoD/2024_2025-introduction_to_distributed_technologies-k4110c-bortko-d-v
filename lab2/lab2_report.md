University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [PIN](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: K4110c  
Author: Bortko Daria Victorovna  
Lab: Lab2  
Date of create: 12.12.2024  
Date of finished: -  

## Описание   
В данной лабораторной работе вы познакомитесь с развертыванием полноценного веб сервиса с несколькими репликами. 

## Цель работы  
Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение. 

## Ход работы  

Напишем манифест для развертывания deployment'а, в котором будет 2 реплики контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/r/ifilyaninitmo/itdt-contained-frontend).  

``` 
kind: Deployment
apiVersion: apps/v1
metadata:
  name: front-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-app
  template:
    metadata:
      labels:
        app: front-app
    spec:
      containers:
      - name: front-container
        image:  ifilyaninitmo/itdt-contained-frontend:master
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
        ports:
        - containerPort: 3000
        env:
          - name: REACT_APP_USERNAME
            value: Bortko Daria
          - name: REACT_APP_COMPANY_NAME
            value: ITMO
```
Применим написанный манифест для создания реплик:  

[Создание_deployment](./img/kubectl_apply.jpj)  

Создадим сервис для доступа на "поды". Возможные типы сервисов: ClusterIP, NodePort, Load Balancer (вообще их больше, но прочие сейчас не рассматриваем).  
ClusterIP - тип сервиса по умолчанию. Сервис будет доступен внутри кластера, но не доступен снаружи и из сети Интернет. Для текущей задачи не подходит.  
NodePort - сервис будет доступен снаружи кластера по определенному порту. Можно использовать в данном случае.   
Load Balancer - сервис будет доступен извне, будет автоматически предоставлен внешний балансировщик нагрузки для приложений в кластере. Также может быть использован в этой лабораторной работе (хотя это кажется несколько избыточным в наших условиях)
Остановимся на типе NodePort.

```
apiVersion: v1
kind: Service
metadata:
  name: front-service
  labels:
    app: front-app
    name: front-service
spec:
  type: NodePort
  selector:
    app: front-app
  ports:
  - port: 3000
    targetPort: 3000
```

[Создание сервиса](./img/create_service.jpj)  

Пробросим порты, чтобы получить возможность доступа к контейнерам через веб-браузер.

[Проброс портов](./img/port_forward.jpj)  

Проверим переданные в контейнер переменные в веб-браузере. Имя пользователя и название компании остаются неизменными, т. к. они заданы в явном виде, а вот название контейнера может меняться, поскольку у нас две реплики и подключение может происходить к любой из них:  

[Проверка переменных](./img/value_check.jpj)  

Проверим логи контейнеров:  

[Проверка логов](./img/logs_check.jpj)  

Схема организации контейнеров и сервисов:

[Схема организации](./img/scheme_2.jpj) 
 
