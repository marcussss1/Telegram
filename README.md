# Telegram

## Содержание

* ### [Тема и целевая аудитория](#1)
* ### [Расчет нагрузки](#2)
* ### [Глобальная балансировка](#3)
* ### [Локальная балансировка](#4)
* ### [Логическая схема БД](#5)
* ### [Физическая схема](#6)
* ### [Алгоритмы](#7)
* ### [Технологии](#8)
* ### [Обеспечение надежности](#9)
* ### [Схема проекта](#10)
* ### [Расчет ресурсов ] (#11)

## 1. Тема и целевая аудитория <a name="1"></a>

**Telegram** — мессенджер, который предоставляет возможность обмена голосовыми, видео и текстовыми сообщениями между пользователями, а также позволяет передавать файлы и мультимедийный контент.

### MVP

- Регистрация нового пользователя
- Отправка текстовых сообщений
- Отправка голосовых сообщений
- Отправка фотографий
- Список чатов
- История сообщений

### Целевая аудитория

- Суммарная аудитория 731 млн пользователей [^1]
- Месячная аудитория 550 млн пользователей [^1]
- Дневная аудитория 150 млн пользователей [^1]
- 12 млрд сообщений в день в среднем [^2]

Страны лидеры по размеру аудитории на момент августа 2022 года:

| Страна      | Пользователей в год, млн. |
| ----------- | ------------------------- |
| Индия       | 70.48                     |
| Россия      | 24.15                     |
| Сша         | 20.03                     |
| Индонезия   | 19.61                     |
| Бразилия    | 18.04                     |
| Египет      | 11.05                     |

## 2. Расчет нагрузки <a name="2"></a>

### Продуктовые метрики <a name="prods"></a>

* Месячная аудитория 550 млн пользователей [^1]
* Дневная аудитория 150 млн пользователей [^1]
* 12 млрд сообщений в день в среднем [^2]
* 1.5 млн людей ежедневно регистрируется в телеграме [^3]
* Количество заходов в день: 24 [^4]

#### Подсчет RPS по метрикам

Регистрация новых пользователей в день: *17.4 RPS*
> 1 500 000 / (24 * 60 * 60) = 17.4

Авторизация пользователей в день: *29 RPS*
> 150 000 000 / (2 * 24 * 30 * 60 * 60) = 29   

Количество сообщений в день на одного пользователя: *80 RPS*
> 12 000 000 000 / 150 000 000 = 80

Отправка сообщения в день: *138 889 RPS*
> 150 000 000 * 80 / (24 * 60 * 60) = 138 889    

Получение списка чатов в день: *41 166 RPS*
> 150 000 000 * 24 / (24 * 60 * 60) = 41 166

Получение истории сообщений в день: *13 722 RPS*
> 150 000 000 * 8 / (24 * 60 * 60) = 13 722

### Технические метрики

### 1. Расчет нагрузки на отправку сообщений

Средняя длинна сообщения в мессенджере 70 символов.
Возьмем кодировку UTF-8, так как у каждой страны свои алфавиты, возьмем средний размер символа - 3 байта.
Считать нагрузку будем среднюю за сутки и в часы наибольшей активности аудитории с 9 до 23 итого 14 часов.

Среднее за сутки: *0.21 Гбит/с*
> 70 * 3 * 12 000 000 000 / (24 * 60 * 60) = 0.21 Гбит/с

В период наибольшей активности: *0.36 Гбит/с*
> 70 * 3 * 12 000 000 000 / (14 * 60 * 60) = 0.36 Гбит/с

Учтем еще то, что часть сообщений будет отправлена в групповые чаты, следовательно и всем пользователям в этом чате. Возьмем, что 1/3 всех сообщений отправляется в чаты и среднее количество участников в чате 20 человек.

Среднее за сутки:
> 70 * 3 * 8 000 000 000 / (24 * 60 * 60) + 70 * 4 000 000 000 * 20 / (24 * 60 * 60) = 1.2 Гбит/с
    
В период наибольшей активности:
> 70 * 3 * 8 000 000 000 / (14 * 60 * 60) + 70 * 4 000 000 000 * 20 / (14 * 60 * 60) = 2.1 Гбит/с
    
Зная, что одно сообщение в среднем весит 70 байт, то нам нужно следующего размера хранилище на пользователей в месяц:
> 70 * 3 * 12 000 000 000 * 30 = 68.7 Тбайт

### 2. Расчет нагрузки на отправку фото

Возьмем, что ежедневно отправляется 175 миллионов фото.
Возьмем вес одной картинки 500 Кбайт получаем.
Считать нагрузку будем среднюю за сутки и в часы наибольшей активности аудитории с 9 до 23 итого 14 часов.

#### Расчет RPS

Среднее за сутки:
> 175 000 000 * 1 / (24 * 60 * 60) = 2 026 RPS
    
В период наибольшей активности:
> 175 000 000 * 1 / (14 * 60 * 60) = 3 472 RPS

#### Расчет нагрузки
    
Среднее за сутки:
> 500 * 175 000 000 * 1 / (24 * 60 * 60) = 7.7 Гбит/с
    
В период наибольшей активности:
> 500 * 175 000 000 * 1 / (14 * 60 * 60) = 13.3 Гбит/с

#### Расчет нагрузки отправки фотографий

Среднее за сутки:
> 500 * 116.666.667 / (24 * 60 * 60) + 500 * 58.333.333 * 20 / (24 * 60 * 60) = 41.2 Гбит/с
    
В период наибольшей активности:
> 500 * 116.666.667 / (14 * 60 * 60) + 500 * 58.333.333 * 20 / (14 * 60 * 60) = 70.3 Гбит/с
    
Тогда аналогично расчету выше посчитаем размер хранилища на всех пользователей в месяц:
> 500 * 175 000 000 * 30 = 2.39 Тбайт

### 3. Расчет авторизации пользователей

Возьмем, что в день пользователь авторизуется 0.25 раз за день.

#### Расчет RPS

Среднее за сутки:
> 150 000 000 * 0.25 / (24 * 60 * 60) = 434 RPS
    
В период наибольшей активности:
> 150 000 000 * 0.25 / (14 * 60 * 60) = 744 RPS

#### Расчет нагрузки

Возьмем, что в день пользователи авторизуются по логину и паролю, такой запрос не будет превышать 128 байт.

Среднее за сутки:
> 0.128 * 150 000 000 * 0.25 / (24 * 60 * 60) = 0.0013 Гбит/с

В период наибольшей активности:
> 0.128 * 150 000 000 * 0.25 / (14 * 60 * 60) = 0.0022 Гбит/с

В качестве базы для хранения сессий возьмем NoSQL базу, в которой будет хранится ключ + значение (логин + пароль)
Для хранения будем использовать NoSQL базу данных, в которой будет хранится ключ + значение (логин + пароль)
В таком базе поле не будет превышать 128 байт. В этом случае для хранения базы всех пользователей нам потребуется:
> 550 000 000 * 0.128 = 0.067 Тбайт

### 4. Расчет получения сообщений в чате

Возьмем, что в день пользователь нажимает на какой-либо чат 7 раз в день и ему выдается первые 50 сообщений из которых 1 картинка.

#### Расчет RPS

Среднее за сутки:
> 150 000 000 * 7 / (24 * 60 * 60) = 12153 RPS

В период наибольшей активности:
> 150 000 000 * 7 / (14 * 60 * 60) = 20833 RPS

#### Расчет нагрузки

Учитывая то, что в среднем сообщение занимает 70 байт, сообщений 50 из которых одна картинка и картинка 500 кБайт. Получим, что в среднем один запрос занимает:
> 70 * 50 * 500 000 * 1 = 1 750 000 байт

Среднее за сутки:
> 1 750 000 * 150 000 000 * 7 / (24 * 60 * 60) = 21 Гбит/с

В период наибольшей активности:
> 1 750 000 * 150 000 000 * 7 / (14 * 60 * 60) = 36 Гбит/с

Рассчитаем хранилище для пользователей в месяц:
> 1 750 000 * 150 000 000 * 30 = 7 186 Тбайт

## 3. Глобальная балансировка <a name="3"></a>

Схема глобальной балансировки:
* Latency-based DNS - при помощи него мы определяем наиболее отзывчивый ЦОД на основе задержки или времени отклика.
* BGP Anycast - для распределения трафика между серверами в ЦОДе при помощи метрик минимальных хопов.

### Регионы, имеющие наибольшее количество клиентов

<img alt="global_load_balancing" src="https://github.com/marcussss1/Telegram/blob/main/images/global_load_balancing.jpeg">

Исходя из изображения и статистики по странам лидерам, ДЦ расположим следующим образом:
- Северная Америка: 1 ДЦ + 1 резервный
- Южная Америка: 1 ДЦ + 1 резервный
- Европа: 2 ДЦ + 2 резервных
- Россия: 1 ДЦ + 1 резервный
- Азия: 4 ДЦ + 4 резервных

## 4. Локальная балансировка <a name="4"></a>

Схема локальной балансировки:

### L4

Для балансировки нагрузки на уровне L4 используем технологию Virtual Server via Direct Routing. Все запросы клиента должны пройти путь через сетевое оборудование и дойти до виртуального IP-адреса. Все запросы с виртуальных IP-адреса будет передаваться на один из серверов.

### L7

Здесь будем использовать k8s ingress controller-ы, благодаря нему мы сможем обеспечить:

1) Балансировка нагрузки: Ingress контроллер выполняет балансировку нагрузки на основе определенных правил маршрутизации. Он может использовать различные алгоритмы балансировки, такие как round-robin, least-connections и другие, чтобы равномерно распределить трафик между сервисами.
2) Унифицированный точка входа: Ingress controller предоставляет точку входа для внешнего трафика в наш кластер.
3) Поддержка маршрутизации на основе пути и домена: Ingress контроллер позволяет настраивать маршрутизацию на основе пути URL и домена.
4) Мониторинг и логирование: Ingress контроллеры предоставляют возможности мониторинга и логирования, которые позволяют отслеживать состояние и производительность балансировщика трафика.
5) Масштабируемость и отказоустойчивость: Ingress controller-ы могут быть масштабированы и развернуты в нескольких экземплярах для обеспечения отказоустойчивости. Если один из контроллеров выходит из строя, другие экземпляры могут продолжать обслуживать трафик. Это позволяет улучшить доступность нашего приложения и предотвратить простои.
6) SSL/TLS терминация: Ingress controller-ы поддерживают SSL/TLS терминацию, что позволяет расшифровывать SSL-трафик на контроллере и перенаправлять его на сервисы в зашифрованном или незашифрованном виде. Это упрощает управление сертификатами и обеспечивает безопасность передачи данных.
7) Ресурсы: Мы можем настроить количество ресурсов, таких как CPU и память, выделенных для Ingress controller-а. Это позволяет оптимизировать его производительность и обеспечить достаточные ресурсы для обработки трафика.

## 5. Логическая схема БД <a name="5"></a>

<img alt="database logic diagram" src="https://github.com/marcussss1/Telegram/blob/main/images/121.jpg">

## 6. Физическая схема <a name="6"></a>

<img alt="physical schema" src="https://github.com/marcussss1/Telegram/blob/main/images/12.jpg">

Для хранения основных данных(чатов, сообщений и профилей) используется база данных PostgreSQL, так как она является одной из наиболее функциональных, производительных и широко распространённых реляционных БД. Для обеспечения отказоустойчивости и повышения производительности базы на чтение, сделаем master-slave репликацию. Для повышения производительности базы на запись сделаем шардинг таблицы сообщений, чатов и профилей. Шардинг таблицы сообщений будем выполнять по полю id. Для более быстрого доступа к данным будет необходимо использовать индексы. Так как для поля id индексы в таблицах создадутся автоматически, навесим индекс на поле chat_id в таблице message.

Для хранения информации о сессиях используем базу данных Redis по причине того, что она осуществляет хранение данных in-memory, имеет поддержку неблокирующей репликации master-slave и возможность организации кластера Redis cluster.

Для хранения фотографий будем использовать amazom S3 хранилище, а в хранилище будем хранить лишь ссылку, которую и будем отдавать на клиент.

## 7. Алгоритмы <a name="7"></a>

## Чат для двух человек

Рассмотрим путь одного сообщения:

#### Изначальные условия

Чат для двух пользователей представляет из себя два отдельных чата, поэтому:

##### Пользователи

Profile {
    id: "576f9a1a-7b68-40db-90b1-0e45177b70ff",
    username: "Вася"
}

Profile {
    id: "a2802d62-b006-4949-8fa0-07328bd26719",
    username: "Петя"
}

##### Чаты

Chat {
    id: "0a9a7480-7db0-42cd-80a3-1ede4516b636",
    title: "Вася"
}

Chat {
    id: "3011c279-f5fe-4152-9b04-6e5d10907c45",
    title: "Петя"
}

#### Путь сообщения

Вася отправил сообщение Пете, при этом в чатах 0 сообщений:

1) Структуры сообщений генерирует клиент, поэтому нам придёт вот такие структуры(в это время в чате отправителя появляется сообщение с одной галочкой):

Сообщение для отправителя:

Message {
    id: 1,
    chat_id: "0a9a7480-7db0-42cd-80a3-1ede4516b636"(чат Васи),
    sender_id: "576f9a1a-7b68-40db-90b1-0e45177b70ff"(Вася),
    views_id: nil,
    is_read: true,
    body: "Привет Петя!",
    voice_message: nil,
    attachments: nil,
    created_at: NOW(),
    updated_at: nil
}

Сообщение для получателя:

Message {
    id: 1,
    chat_id: "3011c279-f5fe-4152-9b04-6e5d10907c45"(чат Пети),
    sender_id: "576f9a1a-7b68-40db-90b1-0e45177b70ff"(Вася),
    views_id: nil,
    is_read: false,
    body: "Привет Петя!",
    voice_message: nil,
    attachments: nil,
    created_at: NOW(),
    updated_at: nil
}

2) Нам нужно понять, что Вася действительно тот, за кого себя выдаёт, у него в cookie будет session_id, зная его и sender_id идём в сервис авторизации и убеждаемся, что Вася тот, за кого себя выдаёт.

3) Далее мы идём в Producer-service, использовав в качестве ключа chat_id, асинхронно пишем в базу сообщение в шард отправителя(смотрим поле is_read).

4) Кладём сообщение в очередь, если получилось, то отдаем клиенту ок(в это время в чате отправителя у сообщения появляется вторая галочка).

5) Если это сообщение отправителя(смотрим поле is_read), ему уже отдалось ок, если сообщение для получателя, то кладем его сообщение в очередь.

6) Как только мы сказали клиенту, что сообщение в очереди, мы должны любыми способами достичь того, чтобы получатель его 100 % получил.

7) Идем в Consumer-service, использовав в качестве ключа chat_id, асинхронно пишем в базу сообщение в шард получателя.

8) Читаем сообщения из Kafka и отправляем в Centrifugo, если получаем ок от Centrifugo, то помечаем сообщение прочитанным из очереди.

9) В Messages-service приходит сообщение из Centrifugo, которое он отдает клиенту.

<img alt="algorithms" src="https://github.com/marcussss1/Telegram/blob/main/images/algorithm_schema.jpg">

## Чат для группы

Рассмотрим путь одного сообщения:

#### Изначальные условия

Чат для трех пользователей представляет из себя один чат, поэтому:

##### Пользователи

Profile {
    id: "576f9a1a-7b68-40db-90b1-0e45177b70ff",
    username: "Вася"
}

Profile {
    id: "a2802d62-b006-4949-8fa0-07328bd26719",
    username: "Петя"
}

Profile {
    id: "1a12e412-bb0d-49be-ab4c-6ef69abd8a25",
    username: "Коля"
}

##### Чаты

Chat {
    id: "9340f6e4-6eec-4516-bdc7-79d150bf1cf4",
    title: "Друзья и товарищи"
}

#### Путь сообщения

Вася отправил сообщение в группу, при этом в чате 0 сообщений:

1) Структуры сообщений генерирует клиент, поэтому нам придёт вот такие структуры(в это время в чате отправителя появляется сообщение с одной галочкой):

Сообщение для пользователей:

Message {
    id: 1,
    chat_id: "9340f6e4-6eec-4516-bdc7-79d150bf1cf4"(Друзья и товарищи),
    sender_id: "576f9a1a-7b68-40db-90b1-0e45177b70ff"(Вася),
    receiver_id: "a2802d62-b006-4949-8fa0-07328bd26719"(Петя),
    views_id: nil,
    is_read: false,
    body: "Привет друзья!",
    voice_message: nil,
    attachments: nil,
    created_at: NOW(),
    updated_at: nil
}

Message {
    id: 1,
    chat_id: "9340f6e4-6eec-4516-bdc7-79d150bf1cf4"(Друзья и товарищи),
    sender_id: "576f9a1a-7b68-40db-90b1-0e45177b70ff"(Вася),
    receiver_id: "1a12e412-bb0d-49be-ab4c-6ef69abd8a25"(Коля),
    views_id: nil,
    is_read: false,
    body: "Привет друзья!",
    voice_message: nil,
    attachments: nil,
    created_at: NOW(),
    updated_at: nil
}

2) Нам нужно понять, что Вася действительно тот, за кого себя выдаёт, у него в cookie будет session_id, зная его и sender_id идём в сервис авторизации и убеждаемся, что Вася тот, за кого себя выдаёт.

3) Далее мы идём в Producer-service, в этом раз мы будем писать в базу синхронно, использовав в качестве ключа chat_id пишем в нужный шард.

4) Кладём сообщения в очередь, если получилось, то отдаем клиенту ок(в это время в чате отправителя у сообщения появляется вторая галочка).

5) Как только мы сказали клиенту, что сообщение в очереди, мы должны любыми способами достичь того, чтобы получатель его 100 % получил.

6) Читаем сообщения из Kafka и отправляем в Centrifugo, если получаем ок от Centrifugo, то помечаем сообщение прочитанным из очереди.

7) В Messages-service приходит сообщение из Centrifugo, которое он отдает клиенту.

<img alt="algorithms" src="https://github.com/marcussss1/Telegram/blob/main/images/4.jpg">

## 8. Технологии <a name="8"></a>

| Технология| Область применения | Мотивационная часть |
| :---: | :---: | :--- |
| TypeScript | Frontend | Фактически стандарт для веба сейчас, преимущество перед JS в наличие статической типизации.|
| GO | Backend | Быстро писать Backend, дает достаточно хорошую производительность на больших нагрузках. Имеет множество готовых библиотек, упрощающих жизнь разработчиков и ускоряющих процесс реализации, а также имеет встроенные горутины и каналы, которые позволяют эффективно распараллеливать производимые вычисления.  |
| Nginx | Веб-сервер, L7 балансировка, отдача статики | Быстрый асинхронный веб-сервер, легко настраивается. |
| PostgreSQL | База данных | Отлично подходит для большого объема данных, поддерживает репликацию, шардирование. |
| Tarantool | Кеш юзеров | In-memory база данных |
| Redis | Хранилище сессий | In-memory БД позволяет обеспечить высокую скорость на чтение и запись. |
| Kafka | Брокер сообщений | Ппредоставляет высокопроизводительную и устойчивую систему для хранения, передачи и обработки потоков данных. |
| Centrifugo | Шина данных | Предоставляет простой и гибкий способ реализации пуш-уведомлений, мгновенного обновления контента. |
| Amazon S3 | Хранение файлов | Удобное апи для загрузки/отдачи файлов, скорость работы, масштабируемость. |

## 9. Обеспечение надежности <a name="9"></a>

### Резервирование

- Наличие резервных серверов, дисков и другого оборудования, готового взять на себя нагрузку в случае сбоя основных компонентов.
- Резервирование ДЦ: наличие нескольких географически распределенных центров обработки данных, способных обслуживать систему в случае отказа основного центра.
- Резервирование БД - репликация: использование репликации баз данных позволяет создавать резервные копии данных и обеспечивать доступ к данным в случае отказа основной базы данных + распределение по шардам.
- Резервирование файлов в S3 хранилище.
- Расчеты пикового трафика и размера хранения с запасом.

### Failover policy

- Автоматическое выполнение повторных запросов к некорректным компонентам для повышения вероятности успешной обработки запроса.
- Уменьшение запросов на проблемный хост: в случае временной недоступности какого-либо сервиса, система автоматически уменьшает количество запросов на этот компонент, чтобы избежать возможной нагрузки и предотвратить негативное влияние на производительность системы.

### Graceful degradation
Обеспечивает то, что наше приложение будет продолжать корректно работать для пользователей, у которых отключены некоторые функциональные возможности или у которых использовано устаревшее оборудование или программное обеспечение.

### Graceful shutdown

Позволит системе закрыть все активные соединения, завершить обработку текущих задач и корректно освобождить ресурсы, прежде чем завершить работу.


### Асинхронные паттерны

- CQRS: используется для разделения операций записи (команд) и операций чтения (запросов) в системе.
- Event-driven архитектура: команды добавляются в брокер сообщений, после чего асинхронно обрабатываются.

## 10. Схема проекта <a name="10"></a>

<img alt="algorithms" src="https://github.com/marcussss1/Telegram/blob/main/images/56.jpg">

## 11. Расчет ресурсов <a name="11"></a>

### Справочная информация

| Технология | Характер сервиса      | RPS    | RAM    |
|------------|-----------------------|--------|--------|
| Go         | тяжелая бизнес-логика | 10     | 100 Mb |
| Go         | средняя бизнес-логика | 100    | 100 Mb |
| Go         | легкое JSON API       | 5000   | 10 Mb  |
| Nginx      | SSL handshake (CPS)   | 500    | 10 Mb  |

### Расчет ресурсов

| Сервис             | Характер сервиса      | Целевая пиковая нагрузка, RPS    | CPU  | RAM     |
|--------------------|-----------------------|----------------------------------|------|---------|
| Users              | легкое JSON API       | 14 000                           | 28   | 28  Gb  |
| Messages           | легкое JSON API       | 140 000                          | 280  | 280 Gb  |
| Chats              | среднее JSON API      | 42 000                           | 420  | 420 Gb  |
| Views              | легкое JSON API       | 14 000                           | 28   | 28  Gb  |
| Auth               | легкое JSON API       | 150 000                          | 300  | 300 Gb  |
| API-gateway        | легкое JSON API       | 160 000                          | 320  | 320 Gb  |
| Centrifugo         | Шина данных           | 130 000 RPS, 80 000 000 ws conns | 3200 | 2160 Gb |
| Redis              | In-memory             | 150 000 RPS                      | 250  | 240 Gb |
| Tarantool          | In-memory             | 14 000 RPS                       | 30   | 25 Gb   |

### Расчет ресурсов

| Сервис       | Хостинг | Конфигурация                        | Cores  | Cnt   | 
| ------------ | ------- | ----------------------------------  | ------ | ----- | 
| Users        | own     | 2x6430/4x8GB/1xNVMe256Gb/2x10Gb/s   | 2      | 4   | 
| Messages     | own     | 2x6430/8x16GB/1xNVMe256Gb/2x10Gb/s  | 18   | 36    | 
| Chats        | own     | 2x6430/8x64GB/1xNVMe256Gb/2x10Gb/s  | 28  | 54  | 
| Views        | own     | 2x6430/2x1GB/1xNVMe256Gb/2x10Gb/s   | 2   | 4   | 
| Auth         | own     | 2x6430/2x1GB/1xNVMe256Gb/2x10Gb/s   | 10  | 38  | 
| API-gateway  | own     | 2x6430/4x8GB/1xNVMe256Gb/2x10Gb/s   | 20     | 40    | 
| Centrifugo   | own     | 2x6430/4x8GB/1xNVMe256Gb/2x10Gb/s   | 200    | 270   | 
| Redis        | own     | 2x6430/4x8GB/1xNVMe256Gb/2x10Gb/s   | 18 | 30    | 
| Tarantool    | own     | 2x6430/4x8GB/1xNVMe256Gb/2x10Gb/s   | 2  | 4 |

## Список литературы
[^1]: [Аудитория телеграма по миру](https://www.bankmycell.com/blog/number-of-telegram-users)
[^2]: [Различные статистики телеграма](https://www.demandsage.com/telegram-statistics)
[^3]: [Новые пользователи в телеграм](https://www.ixbt.com/news/2023/07/19/telegram-2-5-twitter.html#:~:text=Павел%20Дуров%20сообщил%20о%20том,миллионов%20активных%20пользователей%20в%20месяц.)
[^4]: [Мессенджеры](https://www.tadviser.ru/index.php/Статья:Мессенджеры_(Instant_Messenger,_IM)#:~:text=Как%20выяснилось%2C%20в%20среднем%20пользователь,идут%20Viber%2C%20Telegram%20и%20Skype.)
