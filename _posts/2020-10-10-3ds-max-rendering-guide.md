---
layout: post
title:  "Руководство по использованию рендер-фермы на базе Microsoft Azure"
author: sergeype
tags: [ Azure, Batch, hpc, cad, 3dsmax ]
image: assets/users/sergeyperus/rendering-guide.png
featured: true
hidden: true
---

Целью данной статьи ставлю дать простое руководство для пользователей 3DS Max по рендерингу моделей на рендер ферме, созданной в Microsoft Azure

Процесс рендеринга выглядит как следующая последовательность:

1) Загрузка готовой модели в облачное хранилище
2) Запуск рендеринга
3) Скачивание результатов

Предполагается, что ферма уже создана и модель построена

## ПОДГОТОВКА

Необходимо установить [Azure Storage Explorer](https://azure.github.io/BatchExplorer/) и авторизоваться с помощью учетной записи, имеющей доступ к ферме. 
После этого можно начинать рендеринг готовой модели даже без установленного 3DS Max. Если уже он установлен, имеет смысл установить для него [плагин] (https://github.com/Azure/azure-batch-rendering/tree/master/plugins/3ds-max)




## СТАТЬЯ НЕ ЗАВЕРШЕНА