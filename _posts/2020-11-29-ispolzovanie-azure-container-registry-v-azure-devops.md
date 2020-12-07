---
layout: post
title:  "Использование Azure Container Registry в Azure DevOps"
author: evgeny
tags: [ docker, Azure, Azure DevOps, Azure Container Registry ]
image: assets/users/erudinsky/azure-devops/azure-devops-container.jpg
featured: true
hidden: true
---

В данной статье я хочу поделиться опытом использования [Azure Container Registry](https://azure.microsoft.com/ru-ru/services/container-registry/) на проектах с [Azure DevOps](https://azure.microsoft.com/ru-ru/services/devops/). Реестр контейнеров используется для хранения образов Docker, Helm чартов и других артефактов OCI. Из дополнительных преимуществ ACR я бы выделил возможность сборки образов "на лету" (с помощью ACR task), возможность подключения сканера для поиска уязвимостей в образе и его зависимостях, а также георепликацию артефактов по выбранным ЦОДам (на момент написания статьи Azure предалагает 54 региона <i class="em em-muscle" aria-role="presentation" aria-label="FLEXED BICEPS"></i>). 

## CI для Docker с использованием Azure DevOps и ACR

![CI для Docker с использованием Azure DevOps и ACR](/assets/users/erudinsky/azure-devops/azure-devops-acr-ci.png)

Небольшое пояснения: 

* Разработчик использует git, который развернут в Azure DevOps ([Repos](https://azure.microsoft.com/ru-ru/services/devops/repos/))
* Приложение будет упаковано в Docker <i class="em em-whale" aria-role="presentation" aria-label="SPOUTING WHALE"></i> образ со следующими условиями:
  * Сборка будет производиться на агенте Azure DevOps (используется [Pipeline](https://azure.microsoft.com/ru-ru/services/devops/pipelines/));
  * Образ, после сборки и сканирования на уязвимости, отправляется в Azure Container Regisry;
* Процесс повторяется автоматически (например, при обновлении файлов приложения или `Dockerfile`) или запускается в ручком режиме при необходимости.

Я буду использовать [бесплатную версию Azure DevOps](https://azure.microsoft.com/ru-ru/pricing/details/devops/azure-devops-services/) (на момент написания статьи 1800 минут работы агента в месяц, неограниченный git, 2 Гб для хранения артифактов и Azure Boards на 5 пользователей можно использовать бесплатно <i class="em em-tada" aria-role="presentation" aria-label="PARTY POPPER"></i>).

## Начнём с git

Для начала нужна организация в Azure DevOps и проект. Если ещё нет, то можно достаточно просто сделать это [здесь](https://dev.azure.com/). 

![Azure DevOps - начало](/assets/users/erudinsky/azure-devops/azure-devops-getting-started.png)

Шаблон проекта не важен (я использую [Agile](https://docs.microsoft.com/ru-ru/azure/devops/boards/work-items/guidance/agile-process-workflow?view=azure-devops)). Если все проделано правильно, то на выходе должен получиться один дефолтный git репозиторий, используя git-клиент склонируем на локальную машину.

![Azure DevOps - Repos git](/assets/users/erudinsky/azure-devops/azure-devops-init-repo.png)

## Сделаем ACR и дадим доступ к нему из Azure DevOps

Создать Azure Container Registry [можно](https://docs.microsoft.com/ru-ru/azure/container-registry/) [несколькими](https://docs.microsoft.com/ru-ru/azure/container-registry/container-registry-get-started-portal) [способами](https://docs.microsoft.com/ru-ru/azure/container-registry/container-registry-get-started-powershell). Для данной статьи я использовал `west europe` в качестве региона, сам ресурс создавал в [базовой конфигурации](https://azure.microsoft.com/ru-ru/pricing/details/container-registry/) (basic tier) для нашей цели этого достаточно.

![Azure Container Registy - Overview](/assets/users/erudinsky/azure-devops/acr-dashboard.png)

1. Сервер нашего хранилища (можно использовать в командах `docker`);
2. SKU (aka tier) - базовый. Также доступны `стандартный` и `премиум` конфигурации;
3. Можно сменить tier при необходимости.

У нас есть ресурс в Azure, теперь можно добавить его в Azure DevOps для использования во время сборки. Для этого находим Service Connections в Azure DevOps > Project Settings > Pipelines > Service Connections. Добавим новый с типом Docker Registry:

![Azure DevOps - Docker service connection](/assets/users/erudinsky/azure-devops/azure-devops-docker-sc.png)

Я использовал Service Principal для подключения к моему ресурсу из AzDo Pipelines. После сохранения настроек в Azure AD в приложениях появится новое приложение. Исследуем его настройки, для этого через `az cli` попробуем посмотреть список всех service connections: 

```
PS C:\dev\azurefort> az devops service-endpoint list
```

![Azure DevOps - aad sp](/assets/users/erudinsky/azure-devops/azure-devops-aad-sp.png)

Выделен сверху тот самый SPN, который имеет `AcrPush` роль для нашего Azure Container Registry. 

> `AcrPush` - [позволяет отправить и получить образ](https://docs.microsoft.com/ru-ru/azure/container-registry/container-registry-roles) (push & pull), чего вполне достаточно для нашей задачи.

![Azure Container Registry - Roles assignments](/assets/users/erudinsky/azure-devops/azure-devops-acrpush.png)

Всё, service connection готов.

## Dockerfile и контент

Будем использовать `nginx` образ в качестве базового и добавим немного своего контента (html + картинка).

Вот тут [git](https://dev.azure.com/erudinsky/_git/azurefort).

Немного пояснений: 

```Dockerfile
FROM nginx
COPY src /usr/share/nginx/html
```

В `src` лежит контент сайт (рядом с Dockerfile), который копируется в контейнер во время его сборки в путь `/usr/share/nginx/html`. Сервер nginx с настройками по умолчанию загружает эту директорию и слушает порт 80. 

## Сборка приложения

Для сборки мы будем использовать Pipelines (синяя ракета! <i class="em em-rocket" aria-role="presentation" aria-label="ROCKET"></i>). В Azure DevOps можно использовать классический визард (всё происходит через UI), YAML и расширенный YAML ([Mutlistage](https://docs.microsoft.com/ru-ru/azure/devops/pipelines/get-started/multi-stage-pipelines-experience?view=azure-devops)). В данном примере я будут использовать последний, но сделаю короткий обзор первых двух.

### Классический редактор

![Azure DevOps - classic editor](/assets/users/erudinsky/azure-devops/azure-devops-pipelines-classic-editor.png)

С ним достаточно просто начать, необходимо пройти небольшой визард, ответить на вопросы: где хранится код (например, можно использовать Bitbucket или свой собственный git) и далее в графическом интерфейсе выбрать какие конкретно таски (task) будут запускаться (на каком агенте) в пачке каких конкретно задач (job). 

Отличный вариант конфигурации пайплайна, но хранить как код не получится, всё будет лежать где-то в БД Microsoft.. всё-таки yaml хочется, чтоб прям Инфраструктура-как-Код (IaC).

### YAML и multistage

Эти два варианта очень похожи друг на друга (в multistage добавляется подолнительные indent'ы и появляется понятие `stage`). Начинка задач (скоуп job) одинаковый. Ещё в multistage добавляется дополнительный тип задач (в дополнение к [традиционным](https://docs.microsoft.com/ru-ru/azure/devops/pipelines/process/phases?view=azure-devops)), который называется [deployment](https://docs.microsoft.com/ru-ru/azure/devops/pipelines/process/deployment-jobs?view=azure-devops). 

Вернёмся к нашей поставленной задаче. Посмотрим на наш yaml:

```yaml
trigger: none

stages:
- stage: build 
  pool:
    vmImage: 'ubuntu-latest'
  jobs:
    - job: build
      steps:
      - task: Docker@2
        inputs:
          containerRegistry: 'azurefort'
          repository: 'my-nginx'
          command: 'buildAndPush'
          Dockerfile: '**/Dockerfile'
```

У нас всего один `stage` с одной `job` и одной `task`. 

Лучше всего работать с этим файлов через редактор Pipelines, меньше будет опечаток и синтаксических ошибок. Рекомендую также использовать ассистент для задач. 

![Azure DevOps - Pipeline editor](/assets/users/erudinsky/azure-devops/azure-devops-pipeline-yaml-editor.png)

Сохраним и запустим. 

В результате сборки в реестре образов появился новый образ `my-nginx` с нашим приложением и тэгом `$(build.buildid)`. Рекомендую ознакомиться со [встроенными переменными](https://docs.microsoft.com/ru-ru/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml), они позволяет оптимизировать работу пайплайна, иметь меньше хардкода.

## Несколько полезных команд для работы с ACR / Docker

1. Подключиться к ACR: 

```shell

#  подключиться к реестру образов

az acr login -n <name> 
Login Succeeded

# получить список доступных образов в реестре

az acr repository list -n <name> // список образов

# скачать образ my-nginx:2 

docker pull <name>.azurecr.io/my-nginx:2

# запустить образ, слушать на ноде 8080, в контейнере - 80 
# (tip: curl -v localhost:8080)

docker run -d -p 8080:80 <name>.azurecr.io/my-nginx:2 

```

Список пополнятся! <i class="em em-i_love_you_hand_sign" aria-role="presentation" aria-label="I LOVE YOU HAND SIGN"></i>


## Что дальше? 

* Дальше можно автоматизировать сборку добавив `trigger: <branch>`. Допустим вы хотите собирать новый образ каждый раз когда вы делаете `push` или `merge` с `main` веткой; 
* Добавить Azure Key Vault для секретов, сертификатов итд; 
* С помощью дополнительных `stages` можно деплоить приложение. Вот здесь расписывал [как работать с docker-compose](/2020/04/04/build-and-release-docker-compose-usuing-azure-devops-copy/).

Удачи <i class="em em-wave" aria-role="presentation" aria-label="WAVING HAND SIGN"></i>