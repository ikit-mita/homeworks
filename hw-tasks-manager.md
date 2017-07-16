# Tasks Manager

## Введение
_<Вставьте сюда трогательную историю о дружбе и взаимопомощи>_

Поэтому _<имя персонажа>_ теперь предстоит написать web API для управления задачами.

## Общее описание задачи
Необходимо реализовать web API для управления сущностями.

## Описание сущностей
### Заметки
- Свойство "только для чтения" - его нельзя задать напрямую при создании и редактировании

### 1. Проекты

#### Свойства
- Название
  - строка
  - обязательное
  - ограничение на длинну: 64 символа
- Описание
  - строка
  - ограничение на длинну: 2048 символа
- Количество открытых задач (которые не в статусе Completed)
  - число
  - только для чтения

#### Операции
- Просмотр списка (GET /api/projects)
  - Постраничный вывод (paging) и сортировка по любому свойству (sorting)
  - Фильтрация
    - Название - "начинается с"
    - Количество открытых задач - "от ... до"
- Создание (POST /api/projects)
- Просмотр (GET /api/projects/{projectId})
- Обновление (PUT /api/projects/{projectId})
- Удаление (DELETE /api/projects/{projectId})
  - Запрещено удалять проекты, в которых есть задачи

### 2. Задачи

#### Свойства
- Название
  - строка
  - обязательное
  - ограничение на длинну: 64 символа
- Описание
  - строка
  - ограничение на длинну: 2048 символа
- Крайний срок (due date)
  - Дата
- Статус
  - перечисление
    - Создана (Created)
    - Выполняется (InProgress)
    - Отложена (Posponded)
    - Завершена (Completed)
  - обязательное
- Тэги
  - список строк
- Проект
  - ссылка на проект
  - обязательное
- Дата создания
  - дата
  - обязательное
  - только для чтения
- Дата завершения
  - дата
  - только для чтения

#### Операции
- Просмотр списка (GET /api/tasks)
  - Постраничный вывод (paging) и сортировка по любому свойству (sorting)
  - Фильтрация
    - Название - "начинается с"
    - Крайний срок - "от ... до"
    - Дата создания - "от ... до"
    - Дата завершения - "от ... до"
    - Статус - "равен"
    - Проект - "равен"
    - Тэг - "содержит", т.е. задачи, у которых есть переданный тэг
    - Есть Крайний срок - "да/нет"
- Создание (POST /api/tasks)
  - установить статус Created
  - установить свойство Дата создания
- Просмотр (GET /api/tasks/{taskId})
- Обновление (PUT /api/tasks/{taskId})
  - При изменении статуса на Completed выставить свойство Дата завершения
- Удаление (DELETE /api/tasks/{taskId})
- Добавить тэг (PUT /api/tasks/{taskId}/tags/{tag})
  - Если такого тега нету в системе - создать его
  - Если такой тэг уже есть в задаче - ничего не делать
- Удалить тэг (DELETE /api/tasks/{taskId}/tags/{tag})