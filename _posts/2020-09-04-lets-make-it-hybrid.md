---
layout: post
title:  "----В РАЗРАБОТКЕ-------
Использование Azure для локальной инфраструктуры"
author: sergeype
tags: [ Azure, Automation, Arc, LogAnalytics, Updates, Monitoring ]
image: assets/users/sergeyperus/hybrid.png
featured: true
hidden: true
---

Итак, вы не используете облака, у вас все локально, ну максимум немного местного хостинга. 
Публичные облака - это пока непонятно, опасно, да еще и не нужно. Цель этой статьи - показать, как в вашем случае можно просто, безопасно и бесплатно/недорого быстро получить преимущества от облака Microsoft Azure. Я пошагово буду показывать все этапы, давать ссылки на документацию, показывать результат и стоимость (насколько это возможно)

# Этап первый: мониторинг


## Цель - получить информацию о состоянии систем:
  - обновления
  - метрики (CPU/MEM/Disk/Network) и прочие, в том числе специализированные (SQL/Exchange ...)
  - взаимодействие серверов друг с другом, проблемы участников
  - централизация хранения логов/системных журналов с возможностью поиска и анализа собранной информации

## Стоимость

Первые 5 ГБ собираемой и хранимой 30 дней информации каждый месяц - бесплатно. Далее - по тарифу 186 рублей за 1ГБ. Здесь и далее все цены я буду брать из [калькулятора Azure](https://azure.microsoft.com/en-us/pricing/calculator/). Подробнее про [ценообразование мониторинга ](https://docs.microsoft.com/ru-ru/azure/azure-monitor/platform/usage-estimated-costs)
До начала работы сервисы вы, скорей всего, не знаете - какой объем логов генерят сервера. Поэтому мы сразу ограничим ежедневный объем принимаемой информациию чтоб избежать лишних затрат.

## Реализация

1. Создадим [Рабочую Область](https://docs.microsoft.com/ru-ru/azure/azure-monitor/learn/quick-create-workspace) в регионе West Europe. Она будет использована для хранения данных, все остальные сервисы будут опираться на нее. Кроме того, мы сможем вручную посмотреть/поискать/анализировать накопленные данные

![Image](assets/users/sergeyperus/createlaworkspace.gif)




<!-- 














I've been fighting a bit with [some](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/docker-compose?view=azure-devops) of the Azure DevOps pipeline [tasks](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/docker?view=azure-devops) trying to configure end-to-end solution for one of my side project. It is based on a good old Docker Compose and I am pretty happy with how it works in [production](https://docs.docker.com/compose/production/). What I wanted to do is schematically described down below.

![Docker Compose CI/CD with Azure DevOps](/assets/users/erudinsky/azure-devops/docker-compose-cicd-with-azure-devops-microsoft.png)

## What is Azure DevOps

[Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) helps to plan smarter, collaborate better, and ship faster with a set of modern dev services. It's a end-to-end solution for any software development cycle. Anyone can use it even for [free](https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/) with some limitations/conditions of usage (public projects, limited pipeline minutes per month etc). And it's free unlimited git!

### Multistage pipeline

If you are not familiar with multistage pipeline concept, [have a look](https://docs.microsoft.com/en-us/learn/modules/create-multi-stage-pipeline/). In short, it brings all CI/CD experience into `yaml`, where you define all your stages (no UI.. ). It's been really big missing thing for a while since only build pipeline could be explain like this.. 

![Multistage pipeline (Azure DevOps)](/assets/users/erudinsky/azure-devops/multistage-pipeline-yaml.png)

<pre class="language-yaml" data-language="yaml">
<code>
stages:
  - stage: Build_docker_containers
    jobs:
    - job: Build
      pool:
        vmImage: 'Ubuntu-16.04'
      continueOnError: true
      steps:
      - task: Docker@2
        inputs:
          containerRegistry: 'AZURE-CONTAINER-REGISTRY-NAME'
          repository: 'AZURE-CONTAINER-REGISTRY-REPOSITORY-NAME'
          command: 'buildAndPush'
          Dockerfile: '**/Dockerfile'
      - task: PublishPipelineArtifact@1
        inputs:
          targetPath: '$(Pipeline.Workspace)'
          artifact: 'docker-compose'
          publishLocation: 'pipeline'
  
  - stage: 'Deploy_to_production'
    jobs:
    - deployment: Production
      pool:
        vmImage: 'Ubuntu-16.04'
      environment: 'Production'
      strategy:
        runOnce:
          deploy:
            steps:
            - task: CopyFilesOverSSH@0
              inputs:
                sshEndpoint: 'SSH-END-POINT-NAME-FROM-SERVICE-CONNECTIONS'
                sourceFolder: '$(Pipeline.Workspace)/docker-compose/s/'
                contents: |
                  docker-compose.yaml
                  .env
                targetFolder: 'TARGET-PATH'
            - task: SSH@0
              inputs:
                sshEndpoint: 'SSH-END-POINT-NAME-FROM-SERVICE-CONNECTIONS'
                runOptions: 'inline'
                inline: |
                  sed -i 's/##BUILD##/$(Build.BuildId)/g' docker-compose.yaml
            - task: SSH@0
              inputs:
                sshEndpoint: 'SSH-END-POINT-NAME-FROM-SERVICE-CONNECTIONS'
                runOptions: 'inline'
                inline: |
                  docker-compose up -d 2> docker-compose.log
                  cat docker-compose.log
</code>
</pre>

Let's break down the above into small parts and explain what was going on there. In the first stage **Build_docker_containers** there are two tasks: [build image](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/containers/build-image?view=azure-devops#create-pipeline-with-build-step) (it's actually three actions in one: build, tag and push) and [publish pipeline artifact](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/publish-pipeline-artifact?view=azure-devops). Use this task in a pipeline to publish artifacts for the Azure Pipeline (note that publishing is NOT supported in release pipelines. It is supported in multi stage pipelines, build pipelines, and yaml pipelines). <code class="language-yaml">AZURE-CONTAINER-REGISTRY-NAME</code> and <code class="language-yaml">AZURE-CONTAINER-REGISTRY-REPOSITORY-NAME</code> both have to be changed accordingly. Note that a built image gets tag which is by default built-in <code class="language-yaml">Build.BuildId</code> [predefined variable](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/publish-pipeline-artifact?view=azure-devops) which helps me properly roll out my app update on the second stage.

For the sake of simplicity this pipeline (stages) has been simplified.

![Real engineers test in production](/assets/users/erudinsky/azure-devops/real-engineers-test-on-production.jpg)

The second stage is **Deploy_to_production** and it rolls out built image to my production server. All it's tasks are based on [SSH deployment tasks](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/ssh?view=azure-devops). The SSH endpoint has to be configured first in service endpoints (there used to be some issues in new service endpoint experience with >2048 public keys in 2019, but Microsoft team has fixed this).

<code class="language-bash">sed -i 's/##BUILD##/$(Build.BuildId)/g' docker-compose.yaml</code> replaces build number, so <code class="language-bash">docker-compose up -d 2> docker-compose.log</code> brings something to update in docker-machine.

## Docker and Docker Compose

I use docker for packaging an app and docker registry ([Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/)). There is no problem to use dockerhub, just a corresponding endpoint has to be present in service endpoints. Compose helps me to combine multiple containers and define the logic between them and some other objects, as well as their behavior.

### Docker compose for Azure DevOps

The following <code class="language-yaml">docker-compose.yaml</code> is in my project. 

<pre class="language-yaml">
<code>
version: '3'
volumes:
  postgres_data: {}
services:
  app:
    image: AZURE-CONTAINER-REGISTRY-NAME.azurecr.io/AZURE-CONTAINER-REGISTRY-REPOSITORY-NAME:##BUILD##
    restart: always
    environment: 
      RAILS_SERVE_STATIC_FILES: 'true'
      COMPOSE_PROJECT_NAME: 'PROJECT-PREFIX'
    depends_on:
      - db
  db:
    restart: always
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
</code>
</pre>

Let's break down the above into small parts and explain what was going on there. There are two services (of course there are more, but for the sake of simplicity of this exercise there are only two). <code class="language-yaml">AZURE-CONTAINER-REGISTRY-NAME.azurecr.io/AZURE-CONTAINER-REGISTRY-REPOSITORY-NAME:##BUILD##</code> has to be changes slightly except **##BUILD##** which is being changed every pipeline execution.

## Summary

Any feature or bug fix can be delivered to production in less than 6 minutes.

![Azure DevOps multistage pipeline summary](/assets/users/erudinsky/azure-devops/multistage-pipeline-overview.png)

With 1800 free minutes of pipeline per month and 6 minutes of all my stages total duration I can do 300 cycles building and releasing my software. 

Love it!

> All templates are generalized and uploaded to this [<i class="github icon"></i> repository](https://github.com/erudinsky/azure-pipelines).

-->