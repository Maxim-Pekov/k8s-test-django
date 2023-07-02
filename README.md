# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).
## Рабочую версию приложения можно увидеть по адресу `max-k8s.site`

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./docker-compose.yml).

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

# Как развернуть DEV версию приложения в кластере k8s

## Установка

Используйте данную инструкцию по установке этого скрипта

1. Установить

```python
git clone https://github.com/Maxim-Pekov/k8s-test-django.git
```

2. Скачайте minikube по интсрукции `https://minikube.sigs.k8s.io/docs/start/`:

3. Запустите minikube командой:
```python
minikube start
```

4. Перейдите в директорию `kubernetes`

5. Создайте файл  `django-app-secret.yml`, со следующим содержимым, в 
   разделе data впишите данные своего приложения и данные подключения к 
   базе предварительно закодируйте по протоколу `base64`:
```shell
apiVersion: v1
kind: Secret
metadata:
  name: django-app-secret-v9
type: Opaque
data:
  SECRET_KEY: MTIzN
  DEBUG: VHJ1Z #True
  DATABASE_URL: cG9zdGdyZXM6Ly90ZXN0X2s4czpPd090QmVwOUZydXRAcG9zdGdyZXNxbC1zZXJ2OjU  #postgres://test:test@postgresql-serv:5432/test
  ALLOWED_HOSTS: d3d3LnN0YXIudGVzdA== #www.star.test
  POSTGRES_USER: dGVzdF
  POSTGRES_PASSWORD: T3dPdEJlcD
  POSTGRES_DB: dGVzdF9
```
6. Примените в кластере все манифесты командой:
```shell
kubectl apply -f ./ --validate=false
```

7. Узнайте IP для подключения к кластеру:
```shell
minikube ip
```

8. Выполните команду для задания локального доменного имени:
```shell
nano /etc/hosts
```

9. Напишите в верхней строчке ip полученый в прошлом шаге, а через пробел 
   нужный домен и сохраните файл:
```shell
<- ip ->  www.star.test
```

10. Теперь можете протестировать работу сайта, набрав в браузере локальной 
    машины `www.star.test`

# Как развернуть PROD версию приложения в кластере k8s

## Установка

Используйте данную инструкцию по установке этого скрипта

1. Установить

```python
git clone https://github.com/Maxim-Pekov/k8s-test-django.git
```

2. Разверните кластер k8s в облаке, например в [VK CLOUD](https://mcs.mail.ru/app/)

3. После того как развернули в облаке кластер, скачайте файл настроек подключения `kubernetes-cluster-5180_kubeconfig.yaml`, с похожим названием.
4. Поместите его у себя на локальной машине в директорию `/home/user/.kube/`
5. Скачайте на локальный комп `kubectl`, для работы с удаленным кластером
6. Для подключения к удаленному кластеру, сконфигурируйте настройки:
```python
export KUBECONFIG=/home/user/.kube/kubernetes-cluster-5180_kubeconfig.yaml
```
7. Перейдите в директорию `kubernetes`

8. Создайте файл  `django-app-secret.yml`, со следующим содержимым, в 
   разделе data впишите данные своего приложения и данные подключения к 
   базе предварительно закодируйте по протоколу `base64`:
```shell
apiVersion: v1
kind: Secret
metadata:
  name: django-app-secret-v9
type: Opaque
data:
  SECRET_KEY: MTIzN
  DEBUG: VHJ1Z #True
  DATABASE_URL: cG9zdGdyZXM6Ly90ZXN0X2s4czpPd090QmVwOUZydXRAcG9zdGdyZXNxbC1zZXJ2OjU  #postgres://test:test@postgresql-serv:5432/test
  ALLOWED_HOSTS: bWF4LWs4cy5zaXRl #max-k8s.site
  POSTGRES_USER: dGVzdF
  POSTGRES_PASSWORD: T3dPdEJlcD
  POSTGRES_DB: dGVzdF9
```
9. Примените в кластере все манифесты командой:
```shell
kubectl apply -f ./ --validate=false
```

10. Выполните команду `kubectl get svc`, скопируйте EXTERNAL-IP адрес сервиса с типом LoadBalancer, и привяжите к нему свое доменное имя.

11. Теперь можете протестировать работу сайта, набрав в браузере выбранное вами Доменное имя.