# Домашнее задание к занятию «Обновление приложений»

### Цель задания

Выбрать и настроить стратегию обновления приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment).
2. [Статья про стратегии обновлений](https://habr.com/ru/companies/flant/articles/471620/).

-----

### Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Вам нужно объяснить свой выбор стратегии обновления приложения.

```
В данном случае подходит политика развертывания - Recreate, т.к. новые версии приложения не умеют работать со старыми, 
необходимо удалить существующие экземпляры, а затем воссоздать новые, при минимальном простое приложения, во время
низкой загруженности.
```

### Задание 2. Обновить приложение

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.
2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
4. Откатиться после неудачного обновления.

**Подготовим ВМ на Yandex Cloud**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_ter.png">
</p>

**Развернём кластер kubeadm при помощи ansible-playbook**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_ans.png">
</p>

**Подключим воркер-ноды**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_add_wn1.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_add_wn2.png">
</p>

**Установим flannel, скопируем deployment на мастер-ноду**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_fl_scp.png">
</p>

[deployment-rollup](deployment-rolling.yml)

**Пять реплик, nginx:1.19**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_v19.png">
</p>

**Обновляем версию до nginx:1.20**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_v20.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_proc_v19-20.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_v19-20.png">
</p>

**Записываем в историю обновлений**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_rec_v20.png">
</p>

**Обновляем до nginx:1.28**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_v28.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_v28_err.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_v28_err1.png">
</p>

**Откатываем до сохранённой версии**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_rollout.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_rollout_done.png">
</p>

## Дополнительные задания — со звёздочкой*

Задания дополнительные, необязательные к выполнению, они не повлияют на получение зачёта по домашнему заданию. **Но мы настоятельно рекомендуем вам выполнять все задания со звёздочкой.** Это поможет лучше разобраться в материале.   

### Задание 3*. Создать Canary deployment

1. Создать два deployment'а приложения nginx.
2. При помощи разных ConfigMap сделать две версии приложения — веб-страницы.
3. С помощью ingress создать канареечный деплоймент, чтобы можно было часть трафика перебросить на разные версии приложения.

**Создадим новый namespace**

```bash
kubectl create namespace canary
```

**Подготовим два деплоймента с разными версиями**

[deployment-1](canary/deployment-1.yml)
<br>
[deployment-2](canary/deployment-2.yml)
<br>

**Конфигурация для nginx**

[configmap](canary/caonfigmap.yml)
<br>

**Сервис доступа**

[service](canary/service.yml)
<br>

**Для организации канареечного развертывания воспользуемся инструкциями по установке и настройке Istio**

[Install with Helm](https://istio.io/latest/docs/setup/install/helm/)
<br>
[Configure Istio Ingress Gateway](https://istio.io/latest/docs/examples/microservices-istio/istio-ingress-gateway/)
<br>

**Подготовим канареечный деплоймент с перебросом трафика к версиям приложения - 10/90**

[deployment-canary](canary/deployment-canary.yml)
<br>

**Проверим результат, выводом запроса - массив 10 обращений к виртуальному hostname по порту nodeport из istio-gateway**

<p align="center">
    <img width="1200 height="600" src="/img/upd_app_canary.png">
</p>

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
