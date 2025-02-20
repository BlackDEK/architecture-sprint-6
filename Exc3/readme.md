# Задание 3

## Стартовые условия

- Сервисы core-app и ins-comp-settlement получают данные о доступных продуктах через REST API сервиса ins-product-aggregator.
- В момент вызова ins-product-aggregator запрашивает информацию из всех страховых компаний и агрегирует в список, который возвращает в рамках синхронного запроса.
- Чтобы ускорить работу сервисов, при изначальном проектировании команда решила хранить локальные реплики данных о продуктах и тарифах в сервисах core-app и ins-comp-settlement.
- Сервис core-app осуществляет запрос к ins-product-aggregator раз в 15 минут, а ins-comp-settlement — раз в сутки (ночью), при формировании реестра оформленных страховок.
- Иногда команда сталкивается с ошибками взаимодействия между этими сервисами. Они связаны с задержками ответов или ошибками при взаимодействии с API страховых компаний.
- Дополнительно сервис ins-comp-settlement раз в сутки осуществляет запрос в core-app по REST API для получения всех оформленных за день страховок.

## Потенциальные проблемы

- При увеличении нагрузки возможны следующие проблемы.
  - Агрегатор опрашивает все внешние api и формирует список, через синхронный запрос, однако если хоть одно api не отвечает, это может привести к простою и длительному ожиданию.
  - Агрегатор должен обработать более большой объем данных в следствии чего время ожидания при синхронном запросе увеличивается.
  - Агрегатор передаёт все данные а не дельту данных, что может сильно сказываться на нагрузке сервиса. 

## Решение

- Сейчас сервисы core-app и ins-comp-settlement хранят локальные реплики данных, в следствии чего можно сделать так, чтобы агрегатор отправлял не все данные, а только те, которые изменились, по сути он будет отправлять delta-у, а не все данные.
- Как это будет происходить.
  - Агрегатор будет запрашивать данные сравнивать их с уже имеющимися и на основе этого изменять свою БД.
  - После этого он опубликует событие, которые будут обрабатывать core-app и ins-comp-settlement.
  - Для проблемы двойной записи будем использовать Transaction log tailing, он позволит избежать не консистентного состояния при репликации.

## Схема

- [Схема до](InsureTech_C4_сontainer-diagram_as_is.drawio)
- [Схема после](InsureTech_C4_сontainer-diagram_to_be.drawio)

