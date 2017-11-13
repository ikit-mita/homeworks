# Kittens micro-service

## Введение
_<Вставьте сюда трогательную историю о дружбе и взаимопомощи, с отсылками к первой домашней работе>_

Поэтому _<имя персонажа>_ теперь предстоит написать микро-сервис для редактирования списка котиков.

## Общее описание задачи
Необходимо реализовать микро-сервис для управления сущностями в соответствии с требованиями.

|  Код   |                             Название                             |
|--------|------------------------------------------------------------------|
| OP-001 | [Получение токена](#op-001-Получение-токена)                     |
| OP-002 | [Создание котика](#op-002-Создание-котика)                       |
| OP-003 | [Получение списка котиков](#op-003-Получение-списка-котиков)     |
| AU-001 | [Заголовок Authorization](#au-001-Заголовок-authorization)       |
| AU-002 | [Параметры валидации токена](#au-002-Параметры-валидации-токена) |
| NF-001 | [Структура приложения](#nf-001-Структура-приложения)             |
| NF-002 | [База данных](#nf-002-База-данных)                               |

## Описание операций
### OP-001 Получение токена
`POST /token`
```js
{
    "username": "user",
    "password": "user",
    "expiresIn": 60
}
```
**OP-001.1** Поля тела запроса:
  + `username` - имя пользователя, строка, обязательное, не больше 32 символов
  + `password` - пароль, строка, обязательное, не больше 32 символов
  + `expiresIn` - через сколько секунд истечет выданный токен, целое число, от 10 до 300 включительно

**OP-001.2** Ответ:
  + `400` - `{ "error": "" }`, если входная модель невалидная
  + `200` - `{ "token": "eyJOiJIUz...6fMPiagT" }`

**OP-001.3** Валидной считается модель, в которой все поля не нарушают ограничения, а так же `username == password`.

**OP-001.4** Для валидной модели генерируется JWT токен, в котором есть информация об имени пользователя.

**OP-001.5** Токен должен стать невалидным через `expiresIn` секунд.

**OP-001.6** Если `username` начинается с `"admin"`, то в токен должна быть добавлена роль `admin`. Для остальных случаев роли быть не должно.

### OP-002 Создание котика
`POST /kittens`
```js
{
    "name": "Markus",
    "color": "cosmic blue"
}
```

**OP-002.1** Поля тела запроса:
  + `name` - имя котика, строка, обязательное, не больше 32 символов
  + `color` - цвет котика, строка, обязательное, не больше 32 символов

**OP-002.2** Ответ:
  + `400` - `{ "error": "" }`, если входная модель невалидная
  + `401` - если пользователь не авторизован
  + `403` - если пользователь не админ
  + `204` - если успешно добавили котика

**OP-002.3** Валидной считается модель, в которой все поля не нарушают ограничения.

**OP-002.4** Только пользователь в роли `admin` может добавить котика. Неавторизованные пользователи отклоняются с кодом `401`. Авторизованные запросы без нужной роли отклоняются с кодом `403`.

**OP-002.5** Проверка на принадлежность роли должна осуществляться методом `ctx.User.IsInRole("admin")`.

**OP-002.6** Валидная модель должна сохраняться в базу данных, включая имя пользователя, добавившего запись.

### OP-003 Получение списка котиков
`GET /kittens`

**OP-003.1** Ответ 
  + `401` - если пользователь не авторизован
  + `200`

**OP-003.2** Тело ответа `200` - это массив котиков. Каждый котик представлен именем, цветом и именем владельца. Владелец - это пользователь, который добавил котика в базу.
```js
[{
    "name": "Markus",
    "color": "cosmic blue",
    "owner": "adminKate"
}]
```

**OP-003.3** Список котиков загружается из базы данных. Ответ содержит всех котиков. Если котиков нету - возвращается пустой массив.

## Требования к авторизации
### AU-001 Заголовок Authorization
Запросы должны авторизовываться при помощи JWT токена в заголовке `Authorization`

### AU-002 Параметры валидации токена
В задании параметры валидации не описываются прямо. Разработчик должен определить их самостоятельно, на основе требований ниже и анализа предоставленных токенов. Для анализа токенов удобно использовать [JWT debugger (jwt.io)](https://jwt.io/#debugger-io)

**AU-002.1** Следующий токен должен авторизовать запрос с ролью `admin`:
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiaGFja2VyIiwiaHR0cDovL3NjaGVtYXMubWljcm9zb2Z0LmNvbS93cy8yMDA4LzA2L2lkZW50aXR5L2NsYWltcy9yb2xlIjoiYWRtaW4iLCJleHAiOjE4OTM0MzA4MDAsImlzcyI6ImlraXQtbWl0YSIsImF1ZCI6ImlraXQtbWl0YSJ9.lf31q2msiNO6mGQUuRvMRbOb7ITUOk9bu26fMPiagTY`

Этот токен подписан ключом `ikitmita-micro-service`.

**AU-002.2** Следующие токены должны быть отклонены:
  + `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiYWRtaW4iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJhZG1pbiIsImV4cCI6MTg5MzQzMDgwMCwiaXNzIjoiaWtpdC1taXRhIn0.JcxnCDcKZ--6Mn2RzxFYa71KJkMwhpyg8M1yzwrNMco`
  + `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiaGFja2VyIiwiaHR0cDovL3NjaGVtYXMubWljcm9zb2Z0LmNvbS93cy8yMDA4LzA2L2lkZW50aXR5L2NsYWltcy9yb2xlIjoiYWRtaW4iLCJpc3MiOiJpa2l0LW1pdGEiLCJhdWQiOiJpa2l0LW1pdGEifQ.HakhJ1UUHMg2kYkdQsNUWeJgcHtC-O3K18czYVzjdgA`
  + `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiaGFja2VyIiwiaHR0cDovL3NjaGVtYXMubWljcm9zb2Z0LmNvbS93cy8yMDA4LzA2L2lkZW50aXR5L2NsYWltcy9yb2xlIjoiYWRtaW4iLCJleHAiOjE4OTM0MzA4MDAsImlzcyI6ImlraXQtbWl0YSIsImF1ZCI6ImlraXQtbWl0YSJ9.citwAIqKdaMuu83LGV7F4hL0qP0jlNE1juhTc3ZwODU`
  + `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiaGFja2VyIiwiaHR0cDovL3NjaGVtYXMubWljcm9zb2Z0LmNvbS93cy8yMDA4LzA2L2lkZW50aXR5L2NsYWltcy9yb2xlIjoiYWRtaW4iLCJleHAiOjE4OTM0MzA4MDAsImF1ZCI6ImlraXQtbWl0YSJ9.vyoAypERM6L6lxRy1HKL9y6WDE8rte1QxPgSt3-l83Y`

**AU-002.3** Токены, полученные операцией [`POST /token`](#OP-001-Получение-токена), должны правильно авторизовывать запросы.

## Нефункциональные требования
### NF-001 Структура приложения

**NF-001.1** Организовать обработку запроса как цепочку middlewares.

**NF-001.2** Разветвление потока обработки организовать при помощи методов-расширений `.Map` (стандартный) и `.MapMethod` (написан на практике).

### NF-002 База данных

**NF-002.1** БД содержит одну таблицу для хранения котиков.

**NF-002.2** Логика доступа к данным должна быть инкапсулирована (repository, command/query, etc.).

**NF-002.3** Строка подключения к БД должна браться из конфигурации приложения. Предусмотреть возможность определения строки подключения в переменной окружения.

**NF-002.4** При старте приложения или при первом обращении к данным должна быть создана БД с правильной структурой, если она еще не создана на этот момент.

## Дополнительные материалы

  + [JWT debugger (jwt.io)](https://jwt.io/#debugger-io)
  + [JWT-токены на metanit.com](https://metanit.com/sharp/aspnet5/23.7.php)
  + [JWT Validation and Authorization in ASP.NET Core](https://blogs.msdn.microsoft.com/webdev/2017/04/06/jwt-validation-and-authorization-in-asp-net-core/)
  + [ASP.NET Core 2.0 Bearer Authentication](https://www.codeproject.com/Articles/1205160/ASP-NET-Core-Bearer-Authentication)
  + [Configure an ASP.NET Core App](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration?tabs=basicconfiguration)
