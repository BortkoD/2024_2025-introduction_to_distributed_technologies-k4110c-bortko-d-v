University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [PIN](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: K4110c  
Author: Bortko Daria Victorovna  
Lab: Lab4  
Date of create: 26.12.2024  
Date of finished: -  

## Описание   
Это последняя лабораторная работа в которой вы познакомитесь с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают underlay и overlay сети, а управление может быть организованно различными CNI.  

## Цель работы  
Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.    

## Ход работы  

CNI - Container Network Interface, интерфейсы, отвечающие за сетевую связность между узлами и контейнерами.  
Calico - open-source проект. Плагин, позволяющий расширить "коробочные" функции K8s в контексте обеспечения сетевой связности и управления доступностью.

Поднимаем minikube в режиме Multi-Node Clusters с двумя нодами и плагином CNI=calico:  

![Minikube_start]('./img/start.jpg') 

Проверяем, что поднялись обе ноды, а также поды с calico:  

![Two_nodes]('./img/nodes.jpg')  

Проверим работу Calico, используя функцию IPAM plugin - первой ноде назначим географическую метку SPb, второй - Karelia.  
IPAM - IP-address management, плагин службы управления IP-адресами.

![Calico_check]('./img/calico_check.jpg')  

Проверим уже имеющиеся в системе пулы IP-адресов, удалим лишнее:

![IP_pools]('./img/ip_pools.jpg') 

Создаем два пула IP-адресов, которые будут применяться в зависимости от метки ноды:

```
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: spb-ippool
spec:
  cidr: 192.168.0.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: zone == "SPb"
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: karelia-ippool
spec:
  cidr: 192.168.1.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: zone == "Karelia"
```

Воспользуемся утилитой calicoctl для создания собственных пулов (перед этим утилиту нужно поставить, если ранее этого сделано не было). Поскольку работа выполняется под ОС Windows, нужно дополнительно передать в calico kube-конфиг, т.к. по умолчанию он будет искать их по путям Linux и, разумеется, не найдет. Также требуется использовать флаг  --allow-version-mismatch, чтобы избежать несоответствия версий.

![create_pools]('./img/create_pools.jpg') 

Для создания deployment и сервиса воспользуемся файлами из прошлой лабораторной с небольшими модификациями: [configmap.yaml]('./yaml/configmap.yaml'), [deployment.yaml]('./yaml/deployment.yaml'), [service.yaml]('./yaml/service.yaml')

Проверим, какие IP-адреса были назначены подам:

![IP_check]('./img/ip_check.jpg')  

Проверим, что будет отображаться на странице в браузере. Переменные Container name и Container IP могут меняться в зависимости от состояния пода, из которого мы запускаемся. Под работоспособен => переменные постоянные, под умер (или оказался слишком загружен и балансировщик отправил нас на другой) => данные на странице изменились.

![localhost_check]('./img/localhost.jpg')  

Проверим ping между подами:

![ping_check]('./img/ping.jpg')  

Схема организации контейнеров и сервисов:  

![scheme]('./img/scheme.jpg') 
