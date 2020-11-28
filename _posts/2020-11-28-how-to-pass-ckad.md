---
layout: post
title:  "Как сдать CKAD (Certified Kubernetes Application Developer)"
author: evgeny
tags: [ kubernetes, CKAD, AKS ]
image: assets/users/erudinsky/ckad/ckad.png
featured: true
hidden: true
---

За последний год я участвовал в десятке интересных проектов где использовался Docker и Kubernetes. Замечательно то, что я смог разобраться с технологиями на примерах из реального мира - от [the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way) до управляемого [AKS](https://docs.microsoft.com/en-us/azure/aks/), от ручного приема манифеста [yaml](https://yaml.org/) / json до полностью автоматизированного CI / CD с использованием [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) (или / и некоторых других инструментов) со всякими наворотами. Я почти уверен, что для любого технического специалиста практики имеет решающее значение и всегда является приоритетом, особенно в такой сложной и деликатной теме, как микросервисы.

Tl;dr .. Несколькими месяцами ранее я нашёл Cloud Native Computing Foundation (CNCF) с двумя интересными экзаменами - [CKAD](https://www.cncf.io/certification/ckad/) и [CKA](https://www.cncf.io/certification/cka/) и понял, что мне нужно сесть и сдать их, поскольку это именно то, чем я занимаюсь. Иногда, особенно когда вы имеете дело с чем-то новым, вы действительно хотите изучить чужую конфигурацию и убедиться, что вы на правильном пути или / и внести улучшения в свою конфигурацию. В этой статье я хотел бы выделить некоторые вещи, которые мне очень помогли (сначала я сдавал CKAD). Я не буду нарушать NDA, поэтому в статье вы не найдете конкретных вопросов, которые встречаются на экзамене.

Важно! Убедитесь, что вы делаете это не только из-за сертификата. Вам это действительно не нужно, если вы не используете / не планируете использовать эту технологию в своей повседневной деятельности. Все движется так быстро, поэтому, если вы пройдете его сегодня и будете заниматься чем-то другим в течение следующих месяцев, вам, вероятно, придется заново изучать много нового.

## Сертифицированный разработчик приложений Kubernetes (CKAD)

Успешному кандидату будет комфортно пользоваться:

* OCI-совместимая среда выполнения контейнера, например Docker или rkt;
* Концепции и архитектуры облачных приложений;
* Язык программирования, например Python, Node.js, Go или Java.

[Экзаменационная программа](https://github.com/cncf/curriculum) включает следующие категории вопросов (их ихпримерное распределение):

* 13% - Основные концепции;
* 18% - Конфигурация;
* 10% - многоконтейнерные контейнеры;
* 18% - наблюдаемость;
* 20% - Дизайн капсул;
* 13% - Услуги и сети;
* 8% - Постоянство состояния.

> Самая большая проблема - время! У вас 120 минут и 19 задач (~ 6,5 минут на задачу).
> Проходной бал - 66 из 100.

## Рекомендации для CKAD

Используйте alias-ы. Например:

<pre>
<code class="language-bash">
alias k="kubectl"
alias kgd="k get deploy"
alias kgp="k get pods"
alias kgn="k get nodes"
alias kgs="k get svc"
</code>
</pre>

Пример использования выше:

<pre>
<code class="language-bash">
kpg -ns web-development # get all pods from web-development namespace
kgs nginx -o yaml --export # exports yaml for gninx service
</code>
</pre>

Если вы хотите изменить редактор текста по умолчанию для объектов в K8s, измените переменную env:

<pre>
<code class="language-bash">
export KUBE_EDITOR="nano" # this sets editing resource to use nano
</code>
</pre>

Знайте, как использовать генератор для получения ресурсов в yaml / json. См. Пример ниже, эти четыре ниже дают вам другой тип объекта.

<pre>
<code class="language-bash">
kubectl run nginx --image=nginx --replicas=3 --restart=Never --dry-run -o yaml
kubectl run nginx --image=nginx --replicas=3 --restart=Always --dry-run -o yaml
kubectl run nginx --image=nginx --replicas=3 --restart=OnFailure --dry-run -o yaml
kubectl run nginx --image=nginx --replicas=3 --restart=OnFailure --schedule="0/1 * * * *" --dry-run -o yaml
</code>
</pre>


> NB! Обратите внимание на вопрос. Если они просят вас создать `pod`, создавайте его, а не `deployment`. Я использовал генератор и выводил в файл с номером задачи, а затем переходил в файл манифест и производил дальнейшие настройки (например, если некоторые из спецификаций невозможно добавить с помощью CLI). 

## Пример конфигурации в Azure

Для CKAD (поскольку он в основном ориентирован на объекты Kubernetes и способы их работы) я бы рекомендовал развернуть управляемый кластер в Azure. Даже для тестирования он не такой уж и дорогой (цена за месяц с учетом круглосуточной работы). Чтобы поиграть с разными типами объектов, у меня есть [небольшое приложение](https://github.com/erudinsky/dotodo), написанное на nodejs, и я объяснил, как его можно развернуть в AKS. В этом примере также есть pipeline для Azure DevOps (YAML), который использует ACR (Реестр контейнеров) для хранения созданных образов и извлечения из куба, а также имеет [шлюз приложений](https://azure.github.io/application-gateway-kubernetes-ingress/) для входа.

![Azure Price - стоимость работы с AKS и дополнительными сервисами в Azure](/assets/users/erudinsky/ckad/azure-price.png)

Удачи!