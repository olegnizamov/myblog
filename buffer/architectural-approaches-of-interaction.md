---
title: Архитектурные подходы взаимодействия систем
date: 2023-01-01 15:16:37
image: images/posts/4questions.jpg
tags: [Разработка]
---
Вы о них ничего не знаете. Ну правда! Когда на собеседование задаешь о взамио

Какой архитектурный подход выбрать, что надо знать? Одного REST маловато будет, нужны ещё как минимум RPC/SOAP и
GraphQL.

Микро-ликбез, что надо уметь озвучивать на собеседовании по бэкенду.

Самый плохой тут подход, по сути карго-культ — это начитаться википедии и бездумно говорить, что "REST — это
архитектурный стиль", а "SOAP — это протокол".

= RPC (remote procedure call) — удалённый вызов процедур родом ещё из 1970-х годов: здорово конечно вызывать в вакууме
функцию как локально, так и по сети единообразным способом, оставаясь в одном процессе. Однако на практике многое пошло
не так :) Сетевой вызов может сработать, а может и не сработать например, и что делать? Увеличить таймаут, ждать не пять
миллисекунд, а пять минут? Сгенерировать исключение? Попробовать снова вызвать? А вдруг удалённая функция сработала (
например, денежку со счёта списала), базу пофиксила, а мы её снова дёргаем? Придётся дедупликацию прикручивать, и т. д.

Т.е. главная цель RPC — делать и локальные, и удалённые вызовы функций одним макаром, провалилась.

Поэтому был разработан SOAP.

= SOAP — это стандартизованный протокол для обмена сообщениями в формате XML, основанный на RPC. Работает через HTTP как
транспортный протокол который теоретически можно заменять.
В SOAP были учтены все слабые стороны чистого RPC, и получился очень качественно структурированный протокол с чёткой
спецификацией и хорошей безопасностью (хотя реализации SOAP разных корпораций существенно отличаются и плоховато
совместимы). В него входит протокол WSDL для исчерпывающего описания веб-сервиса, позволяющий фактически описывать
весьма сложные схемы взаимодействия и парсинг без модификации исходников RPC-функций и автоматизировать многие нюансы (
например, без проблем организовывается туннелирование через прокси и межсетевые экраны).

А прикончил SOAP, пожалуй, бурный рост мобильных приложений. Объёмные XML-сообщения тяжело на клиентах парсить и
накладно передавать по слабой сети (хотя, возможно, ситуацию спасут сети 5G :), сама спецификация сложная, порог входа
высокий, и разрабатывать SOAP-приложения в разы дольше, нежели REST.

Сегодня SOAP стал по сути легаси-технологией, иногда поверх его гонится облегчённый stateless JSON-RPC например.

70-80% систем Amazon и Google в последние десятилетия переведены с SOAP на REST.

= REST (Representational state transfer) — это так называемый "архитектурный стиль" :) для создания API, единого
стандарта нету, но имеется неформальный набор правил проектирования. Грубо говоря, если вы написали обращение к
урлу/uri, который возвращает json, можете считать это REST.

Важный момент, что REST API не должен зависеть от истории: каждый вызов "чистый", не завязан на предыдущие, а не то что
сперва "запомни 2", потом "запомни 3", и потом "посчитай результат": нет, сразу "возьми 2 и 3, сложи и покажи
результат", только так. Ну то есть правильный REST всегда stateless.

REST в отличие от RPC уже явно подразумевает работу через сеть, тесно завязан на HTTP (кэширование используется
например, коды ошибок...), предлагает стандартный CRUD через GET/PUT/POST/DELETE, простой в понимании, лёгкий в
реализации.

К его стандартным недостаткам относят прежде всего необходимость часто поддерживать много endpoints на клиенте, ну или
выстраивать цепочки запросов, поддерживать некие логические последовательности на клиенте, и это всегда плохо.
Потребовалось последовательность таких шагов немного подправить — и придётся пересобирать и апгрейдить все клиентские
приложения.

Например, сперва надо запросить всё по основной сущности (пользователь), затем запросить по его id связанные с ним
счета, затем запросить выписку по id выбранного пользователем счёта, и т. д. По сути, мы организовываем с клиентов
запросы к условной БД, каким-то кривым самопальным CRUD.

Ну и другой довольно серьёзный недостаток REST — это отсутствие поддержки типов.

Можно сказать, что REST — это больше неформальное хакерство, а SOAP — это больше бюрократическая корпоративщина.

Поэтому следующим логическим шагом, нацеленным на повышение гибкости клиентов, стал GraphQL.

= GraphQL — это разработанный относительно недавно в фейсбуке и затем ставший опенсорсным язык запросов + спецификация
API. GraphQL использует единую точку endpoint для обработки данных как SOAP, и формат типизирован, умеет работать в
реалтайме, но при этом лёгкий по нагрузке на сеть и клиента. В одном запросе можно получить сложный набор данных, формат
запроса гибкий (декларативщина), и многие вещи можно настраивать, условно, скриптами на клиенте без его пересборки.

Но, у всего есть цена :) Если "SOAP это протокол" и "REST это архитектурный стиль", то "GraphQL это технология".
Достаточно сложная, нагрузочная на сервер, нету готового кэширования (так как нету явной привязки к HTTP в отличие от
REST) и т. д. По сути, GraphQL просто переводит сложность с уровня клиент-сервер на сервер-СУБД, особо её не понижая.

Однако достаточно легко встраивается в существующие проекты, поэтому, если приходится постоянно организовывать новые и
новые endpoints на клиенте, или урлы разрастаются в нечто подобное:
users/posts?sortBy=date&fields=title,description&likes_above=3
то вам явно надо вот сюда — реализация GraphQL на питоне например:
[https://graphene-python.org/](https://vk.com/away.php?to=https%3A%2F%2Fgraphene-python.org%2F&post=-152484379_3074&cc_key=)

Большое сравнение разных подходов к API:
[https://goodapi.co/blog/rest-vs-graphql](https://vk.com/away.php?to=https%3A%2F%2Fgoodapi.co%2Fblog%2Frest-vs-graphql&post=-152484379_3074&cc_key=)

Наконец, всё большую актуальность по понятным причинам сегодня приобретает распределённый бэкенд сам по себе, и
возвращаемся к дедам на полвека назад: явление RPC 2.0.

=  *gRPC* , разработанный в Google, наследник RPC и SOAP. Это такой распределённый серверный энвайронмент сервер-сервер,
когда одна система должна вызвать асинхронную фунцию, заэкспозенную другой системой. Расширяет RPC концепцией стримов,
когда один вызов это не просто запрос/ответ, а целая последовательность общений, растянутая во времени, ну и вообще
качественно оригинальную версию улучшает.

Когда вы пилите прикладной проект с веб/мобильными клиентами, сервер чаще всего один. До миллиона запросов в сутки
например я делал с весьма сложной серверной логикой на облачном сингле без проблем. Тут GraphQL хороший выбор. Но если
создаётся сервисо-ориентированная архитектура, для коммуникации между серверами подходящим будет скорее  *gRPC* .

Имеет смысл и на другие версии RPC поглядывать, поизучать в частности промисы и фьючеры (инкапсуляция асинхронных сбоев,
интеграция ответов от разных серверов...).
Например,
[Rest.li](https://vk.com/away.php?to=http%3A%2F%2FRest.li&post=-152484379_3074&cc_key=) от линкедина
[https://github.com/linkedin/rest.li](https://vk.com/away.php?to=https%3A%2F%2Fgithub.com%2Flinkedin%2Frest.li&post=-152484379_3074&cc_key=)
твиттеровская Finagle
[https://github.com/twitter/finagle](https://vk.com/away.php?to=https%3A%2F%2Fgithub.com%2Ftwitter%2Ffinagle&post=-152484379_3074&cc_key=)
*gRPC*
[https://github.com/grpc](https://vk.com/away.php?to=https%3A%2F%2Fgithub.com%2Fgrpc&post=-152484379_3074&cc_key=)

все активно развиваются.

Более подробно мы разберём все эти темки в моём отдельном курсе (базы данных и высоконагруженные проекты), по роадмапу
должен выйти этой осенью:
[https://skillsmart.ru/roadmap/](https://vk.com/away.php?to=https%3A%2F%2Fskillsmart.ru%2Froadmap%2F&post=-152484379_3074&cc_key=)
и одновременно с научной точки зрения посмотрим на это всё с позиции парадигм программирования (тоже курс отдельный
готовлю будет скоро).

===

В заключение, зарезюмирую наверное две самые важные вещи, которые надо помнить всегда:
— REST — это stateless, xRPC это statefull.
— REST — это синхронная "технология" (хотя делаются конечно попытки расширить её асинхронщиной, но в мякотке своей она
синхронная: “A client sends a request, the server sends a response” по словам авторов Scala и Akka),
а xRPC это прежде всего полноценная асинхронная "технология".
Почему кстати для xRPC достаточно поддерживать только обратную совместимость по запросам и прямую совместимость по
ответам?
