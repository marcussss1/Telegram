# Telegram

## Содержание

* ### [Тема и целевая аудитория](#1)
* ### [Расчет нагрузки](#2)

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

### Подсчет RPS по метрикам

Регистрация новых пользователей в день: *17.4 RPS*
> 1 500 000 / (24 * 60 * 60) = 17.4

Авторизация пользователей в день: *1736 RPS*
> 150 000 000 / (24 * 60 * 60) = 1736   

Количество сообщений в день на одного пользователя: *80 RPS*
> 12 000 000 000 / 150 000 000 = 80

Отправка сообщения в день: *138 889 RPS*
> 150 000 000 * 80 / (24 * 60 * 60) = 138 889    

Получение списка чатов в день: *41 166 RPS*
> 150 000 000 * 24 / (24 * 60 * 60) = 41 166

Получение истории сообщений в день: *13 722 RPS*
> 150 000 000 * 8 / (24 * 60 * 60) = 13 722

### Технические метрики

## 1. Расчет нагрузки на отправку сообщений

Средняя длинна сообщения в мессенджере 70 символов.
Возьмем кодировку UTF-8, где 1 символ равен 1 байту.
Считать нагрузку будем среднюю за сутки и в часы наибольшей активности аудитории с 9 до 23 итого 14 часов.

Среднее за сутки: *0.07 Гбит/с*
> 70 * 12 000 000 000 / (24 * 60 * 60) = 0.07 Гбит/с

В период наибольшей активности: *0.12 Гбит/с*
> 70 * 12 000 000 000 / (14 * 60 * 60) = 0.12 Гбит/с

Учтем еще то, что часть сообщений будет отправлена в групповые чаты, следовательно и всем пользователям в этом чате. Возьмем, что 1/3 всех сообщений отправляется в чаты и среднее количество участников в чате 20 человек.

Среднее за сутки:
> 70 * 8 000 000 000 / (24 * 60 * 60) + 70 * 4 000 000 000 * 20 / (24 * 60 * 60) = 0.4 Гбит/с
    
В период наибольшей активности:
> 70 * 8 000 000 000 / (14 * 60 * 60) + 70 * 4 000 000 000 * 20 / (14 * 60 * 60) = 0.7 Гбит/с
    
Зная, что одно сообщение в среднем весит 70 байт, то нам нужно следующего размера хранилище на пользователей в месяц:
> 70 * 12 000 000 000 * 30 = 22.9 Тбайт

## 2. Рассчет нагрузки на отправку фото

Возьмем, что ежедневно отправляется 175 миллионов фото.
Возьмем вес одной картинки 500 Кбайт получаем.
Считать нагрузку будем среднюю за сутки и в часы наибольшей активности аудитории с 9 до 23 итого 14 часов.

### Расчет RPS

Среднее за сутки:
> 175 000 000 / (24 * 60 * 60) = 2 026 RPS
    
В период наибольшей активности:
> 175 000 000 / (24 * 60 * 60) = 3 472 RPS

### Расчет нагрузки
    
Среднее за сутки:
> 500 * 175 000 000 / (24 * 60 * 60) = 7.7 Гбит/с
    
В период наибольшей активности:
> 500 * 175 000 000 / (24 * 60 * 60) = 13.3 Гбит/с

### Расчет нагрузки отправки фотографий

Среднее за сутки:
> 500 * 116.666.667 / (24 * 60 * 60) + 500 * 58.333.333 * 20 / (24 * 60 * 60) = 41.2 Гбит/с
    
В период наибольшей активности:
> 500 * 116.666.667 / (14 * 60 * 60) + 500 * 58.333.333 * 20 / (14 * 60 * 60) = 70.3 Гбит/с
    
Тогда аналогично расчету выше посчитаем размер хранилища на всех пользователей в месяц:
> 500 * 175 000 000 * 30 = 2.39 Тбайт

## 3. Расчет авторизации пользователей

Возьмем, что в день пользователь авторизуется 0.25 раз за день.

### Расчет RPS

Среднее за сутки:
> 150 000 000 * 0.25 / (24 * 60 * 60) = 434 RPS
    
В период наибольшей активности:
> 150 000 000 * 0.25 / (14 * 60 * 60) = 744 RPS

Возьмем, что в день пользователи авторизуются по логину и паролю, такой запрос не будет превышать 128 байт.

Среднее за сутки:
> 0.128 * 150 000 000 * 0.25 / (24 * 60 * 60) = 0.0013 Гбит/с

В период наибольшей активности:
> 0.128 * 150 000 000 * 0.25 / (14 * 60 * 60) = 0.0022 Гбит/с

В качестве базы для хранения сессий возьмем NoSQL базу, в которой будет хранится ключ + значение (логин + пароль)
Для хранения будем использовать NoSQL базу данных, в которой будет хранится ключ + значение (логин + пароль)
В таком базе поле не будет превышать 128 байт. В этом случае для хранения базы всех пользователей нам потребуется:
> 550 000 000 * 0.128 = 0.067 Тбайт

## Список литературы
[^1]: [Аудитория телеграма по миру](https://www.bankmycell.com/blog/number-of-telegram-users)
[^2]: [Различные статистики телеграма](https://www.demandsage.com/telegram-statistics)
[^3]: [Новые пользователи в телеграм](https://www.ixbt.com/news/2023/07/19/telegram-2-5-twitter.html#:~:text=Павел%20Дуров%20сообщил%20о%20том,миллионов%20активных%20пользователей%20в%20месяц.)
[^4]: [Мессенджеры](https://www.tadviser.ru/index.php/Статья:Мессенджеры_(Instant_Messenger,_IM)#:~:text=Как%20выяснилось%2C%20в%20среднем%20пользователь,идут%20Viber%2C%20Telegram%20и%20Skype.)
