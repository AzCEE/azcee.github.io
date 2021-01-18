---
layout: post
title:  Как создать свой dedicated сервер Minecraft (за пару минут)?
author: evgeny
tags: [ minecraft, docker, bedrock ]
image: assets/users/erudinsky/minecraft/minecraft-title.jpg
featured: false
hidden: false

---
	
Майнкрафт — одна из самых увлекательных игр в современном геймдеве. За счет своих низких системных требований и огромной базы фанатов контент в популярной пиксельной вселенной разросся до небывалых масштабов. Мы играем на *мобильных* телефонах и, в случае подключения к одной сети wifi можем бегать вдвоем-троем на одной карте. Недавно эту возможность нужно было расширить и пригласить игрока не из домашней сети - нужен был [dedicated сервер](https://www.minecraft.net/en-us/download/server/bedrock/). В статье я написал как можно быстро решить эту задачу с помощью облака Azure.

## Что требуется?

![Minecraft dedicated server](/assets/users/erudinsky/minecraft/minecraft-dedicated-server.png)

Я буду использовать [Bedrock Dedicated Server](https://hub.docker.com/r/itzg/minecraft-bedrock-server) в [Container Instances](https://azure.microsoft.com/en-us/services/container-instances/).

## Создаем сервер

Для создания сервера будем использовать [Azure портал](https://portal.azure.com/). В каталоге доступных сервисов ищем по клюву `aci` продукт с названием *Container Instances*. Создаем новый.

![Azure Container Instance](/assets/users/erudinsky/minecraft/aci.png)

Запустим экземпляр сервера, для этого нужно лишь следующее:

1. Выбрать [правильный Azure регион](https://azure.microsoft.com/en-us/global-infrastructure/geographies/) для создания ACI. Определиться с выбором поможем [Azure Speed Test 2.0](https://azurespeedtest.azurewebsites.net/). На момент написания статьи я был в Санкт-Петербурге и использовал голландский ЦОД (West Europe).
2. В качестве Docker образа указать `itzg/minecraft-bedrock-server` из Dockerhub (image source: 	Docker Hub or other registry).
3. В Networking убедиться, что Networking type используется Public, задать DNS name label, и добавить порт 19132 UDP - это порт на котором сервер ожидает подключение игроков.
4. Последнее - в environment variables задать переменную `EULA` = `TRUE`. Это означает, что вы принимаете условия [Minecraft End User License Agreement](https://account.mojang.com/terms).

Для сохранения данных игры рекомендуется использовать `volume`. Через Azure портал монтировать их к ACI нет возможности (нужно использовать CLI или PS). Но там все [просто](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-volume-azure-files): потребуется создать Azure Storage Account и в нем использовать SMB, подключается через аргументы `--azure-file-volume-account-name`, `--azure-file-volume-account-key`, `--azure-file-volume-share-name` и `--azure-file-volume-mount-path`. Внутри контейнера контент сервера и сам сервер работает из `/data` ... 

Ниже - пример рабочего сервера с минимально-необходимыми характеристиками. Контейнер создается и запускается несколько секунд.

![Minecraft Dedicated Server на Azure Container Instances](assets/users/erudinsky/minecraft/minecraft-aci.png)

Для подключения к серверу нам нужно узнать публичный IP адрес, который был присвоен запущенному экземпляру контейнера.

![Как узнать IP адрес ACI](assets/users/erudinsky/minecraft/get-public-ip-aci.png)

## Результат

Ping до сервера получился в пределах 100 мс. 

![Ping до сервера Minecraft](assets/users/erudinsky/minecraft/minecraft-server-ping.png)
![Ping до сервера Minecraft](assets/users/erudinsky/minecraft/minecraft-game.png)

Расчет из [калькулятора](https://azure.microsoft.com/en-us/pricing/calculator/), час работы такого сервера ~ $0.05.

