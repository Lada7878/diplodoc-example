## Описание REST API DAMASK 
### Введение

DAMASK это Средство Защиты Информации реализованное в формате доверенного высокопроизводительного программно-аппаратного комплекса (ДПАК), использующего метод динамической подмены данных (ДПД) для обеспечения безопасной обработки конфиденциальной данных в различных сценариях.s
Настройка и использование DAMASK может производиться путем вызова REST API. Вызывать API можно из командной строки с помощью утилиты сurl, или приложение Postman.
Для выполнения команд нужно аутентифицироваться и получить `jwt` токен, который использовать при следующих запросах.



### Запрос получения токена
#### POST /v1/auth
Метод аутентификации пользователя.
Пример запроса: 
``` json
{
    "username":"admin",
    "password":"admin"
}
```
Пример ответа: 
``` json
{
    "accessToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsInVpZCI6IjZhMjIyM2FiLTAyYzktNDcxZS1iMmNhLTIyOGRkYjkxZjYzZiIsIm5iZiI6MTcwNjA4MjMxNywiZXhwIjoxNzA2MTY4NzE3fQ.h74W49_holG9oiNlDVxb7YVwcKjODJsp62Cs8vobxPAFgBAwcJr-DGcJo-LQzRJxyCYwTjoed4wUSyQ8usnjeBtFgTZ1WmAagF1-bvR-66DpnP9gC6cWAn9izu7igIPQdc0daSUHqFC2obwAvhYfRt45FGmyI05s6D2Wb1ecQwT7muzqeVuLgxat_qgMfcxJhJQsN5ITH-VDq-X8cvLwIYHEW92onh2Q4KCf9dcy9DMyclvtwIIUC2Dv3ipywH7rqGhe7Kl7zR57U4kfT3u-O8cvzEZd9C8MvV497jff6kSRK6VuWwezF8tXAooZY4D2GlcEsmycM5nGyIi1i1ycdg",
    "tokenType":"Bearer"
}
```

### Работа с пользователям
#### GET /v1/user?limit=10&offset=10 

 Данный метод отдает всех пользователей DAMASK постранично, доступны параметры limit и offset (пример запроса - ?limit=10&offset=10)


Пример ответа:
``` json
{
    "items": [
        {
            "id": "3ec2dfeb-99ad-4b3e-88eb-e2613a463463",
            "username": "ivanovio",
            "display_name": "Ivanov IO",
            "email": "ivanovio@bssg.ru",
            "enabled": true,
            "roles": [
                "admin",
                "user"
            ]
        },
        {
            "id": "6a2223ab-02c9-471e-b2ca-228ddb91f63f",
            "username": "admin",
            "display_name": "admin",
            "description": "admin user",
            "email": "admin@bssg.ru",
            "enabled": true,
            "roles": [
                "admin",
                "user"
            ]
        }
    ],
    "limit": 10,
    "count": 2,
    "offset": 0
}
``` 


#### GET /v1/user/{user_id}

Данный метод отдает конкретного пользователя DAMASK по его `id`.

Пример ответа:
``` json
{
    "id": "3ec2dfeb-99ad-4b3e-88eb-e2613a463463",
    "username": "ivanovio",
    "display_name": "Ivanov IO",
    "email": "ivanovio@bssg.ru",
    "enabled": true,
    "roles": [
        "admin",
        "user"
    ]
}
```
#### POST /v1/user/

Данный метод создает нового пользователя в DAMASK. 
Пример тела POST запроса:
``` json
{
    "username":"ivanovio",
    "display_name":"Ivanov IO",
    "description":"Ivanov IO",
    "enabled":true,

    "email":"ivanovio@bssg.ru",
    "password":"xxxx",
    "roles":["admin","user"]
}
```
Обязательны к заполнению `username`, `display_name`, `email`, `password`. Массив `roles` может быть пустым, тогда ролей не будет, их можно потом добавить. Поле `enabled` если не указан, то по умолчанию будет `enabled=true`.
В ответ - бъект пользователя, аналогичный предыдущему методу.

#### DELETE /v1/user/{user_id}

Данный метод удаляет пользователя из DAMASK вместе с привязкой его ролей
В ответ будут либо ошибки валидации, либо:
``` json
{
    "success": true
}
```
#### PATCH /v1/user/{user_id}

Данный метод обновляет данные пользователя по его `id`, обновлять можно отдельные поля, не затрагивая остальные. Если поле отсутствует в json, то оно не будет изменено. Если поле есть в json, но оно пустое – то будет проведена его валидация, и, если пустое разрешено (для `description`, например), то оно будет обновлено на пустое в БД DAMASK
Ответ – объект пользователя с новыми значениями:
``` json
{
    "username":"ivanovio",
    "display_name":"Ivanov IO",
    "description":"Ivanov IO",
    "enabled":true,

    "email":"ivanovio@bssg.ru",
    "password":"xxxx",
    "roles":["admin","user"]
}
```

#### POST /v1/user/{user_id}/role/{role_id}

Данный метод добавляет роль для пользователя. При прохождении валидации, если у пользователя такой роли нет, она будет добавлена. Если уже есть, то ничего не произойдет. 
Ответ:
``` json
{
    "success": true
}
```
#### DELETE /v1/user/{user_id}/role/{role_id}

Данный метод удаляет роль у пользователя. При прохождении валидации, если у пользователя есть такая роль, она будет удалена. Если роли нет – то будет ошибка валидации, что у пользователя нет такой роли.
Ответ:
``` json
{
    "success": true
}
```

### Работа с группами пользователей
#### GET /v1/user_group?limit=10&offset=10&sort_by=name&sort_order=ASC&query=name contains a 

 Данный метод отдает постранично группы пользователей DAMASK, доступны параметры:
 limit - количество записей на старнице ответа
 offset - смещение в страницах от начала 
 sort_by - опционально, можно указать поле для сортировки результатов запроса, поддерживаются поля ID, NAME, DESCRIPTION
 sort_order - опционально, порядок сортировки по полю, ASC или DESC. По умолчанию стоит ASC
 query - опционально, возможность указать условия фильтрации результата запроса в формате {поле} {операция} {параметр}. Для поля  можно использовать ID, NAME, DESCRIPTION. Для операции - contains, starts_with, ends_with и варианты с not (not contains, not starts_with, not ends_with). Параметр - искомые значения по условию. Пример - query=name starts_with a

Пример ответа:
``` json
{
    "items": [
        {
            "id": "7036bec7-a59a-4ac9-b867-b4c61fc8d209",
            "name": "Admins",
            "description": "Admins group"
        }
    ],
    "limit": 10,
    "count": 1,
    "offset": 0
}
``` 

#### GET /v1/user_group/{user_group_id}

Данный метод отдает конкретную группу пользователя DAMASK по ее `id`.

Пример ответа:
``` json
{
    "id": "7036bec7-a59a-4ac9-b867-b4c61fc8d209",
    "name": "Admins",
    "description": "Admins group"
}
```
#### POST /v1/user_group/

Данный метод создает новую группу пользователей в DAMASK. 
Пример тела POST запроса:
``` json
{
    "name":"Admins",
    "description":"Admins group"
}
```
Обязателно к заполнению `name`
В ответ - бъект группы пользователя, аналогичный предыдущему методу.

#### DELETE /v1/user_group/{user_group_id}

Данный метод удаляет группу пользователей из DAMASK. Если группа используется в разрешениях, то она не сможет быть удалена и в ответ придет ошибка.
Если ошибки нет, то в ответ будет:
``` json
{
    "success": true
}
```
#### PATCH /v1/user_group/{user_group_id}

Данный метод обновляет данные группы пользователей по ее `id`, обновлять можно отдельные поля, не затрагивая остальные. Если поле отсутствует в json, то оно не будет изменено. Если поле есть в json, но оно пустое – то будет проведена его валидация, и, если пустое разрешено (для `description`, например), то оно будет обновлено на пустое в БД DAMASK
Ответ – объект пользователя с новыми значениями:
``` json
{
    "name":"Admins1",
    "description":"Admins1 group"
}
```

#### GET /v1/user_group/{user_group_id}/members?limit=10&offset=0&sort_by=name&sort_order=ASC

Данный метод отдает постранично идентификаторы пользоваетелй, входящих в группу пользователя DAMASK по ее `id`. Поддерживаются параметры limit и offset. Также можно указать значение параметров sort_by и sort_order для сортировки пользователей группы.

Пример ответа:
``` json
{
    "items": [
        {
            "id": "fe34ce9e-b31a-4936-8bd8-0ae251b7a5fe"
        }
    ],
    "limit": 10,
    "count": 1,
    "offset": 0
}
```

#### POST /v1/user_group/{user_group_id}/members

Данный метод добавляет пользователя (или несколько пользователей) в группу. При прохождении валидации, если пользователя еще нет в группе, то он будет добавлен. Если уже есть - то ничего не произойдет. В запросе заполняется массив user_identifiers
Пример тела POST запроса:
``` json
{
    "user_identifiers":["fe34ce9e-b31a-4936-8bd8-0ae251b7a5fe"]
}
```

Ответ:
``` json
{
    "success": true
}
```
#### DELETE /v1/user_group/{user_group_id}/members

Данный метод удалить пользователя (или нескольких пользователей) из группы пользователей. В теле запроса заполняется массив user_identifiers.
Пример тела POST запроса:
``` json
{
    "user_identifiers":["fe34ce9e-b31a-4936-8bd8-0ae251b7a5fe"]
}
```

Ответ:
``` json
{
    "success": true
}
```

###	Работа с шаблонами токенов

#### GET /v1/token_template

Данный метод получает все шаблоны токенов, которые есть сейчас в DAMASK. 
Ответ:
``` json
[
    {
        "id": "damask",
        "name": "Damask default",
        "description": null,
        "regex": ""
    },
    {
        "id": "uuid",
        "name": "Damask UUID",
        "description": null,
        "regex": ""
    },
    {
        "id": "dbmeta",
        "name": "Damask dbmeta",
        "description": null,
        "regex": ""
    }
]
```
#### GET /v1/token_template/{token_template_id}

Данный метод получает информацию о конкретном шаблоне токенов. 
В ответе:
``` json
{
    "id": "damask",
    "name": "Damask default",
    "description": null,
    "regex": ""
}
```
### Работа с токен-группами

#### GET /v1/token_group 

Данный метод отдает все токен группы постранично, доступны параметры limit и offset (пример ?limit=10&offset=10)
Пример ответа:
``` json
{
    "items": [
        {
            "id": "15d2e57d-97fd-4d28-95d2-637fee779acb",
            "name": "test_group1",
            "description": "Test group 1"
        },
        {
            "id": "866fbf8f-772a-46d9-856e-cecb1e2cbcc5",
            "name": "group11",
            "description": "group11"
        },
        {
            "id": "d3340c93-80d3-4e5b-b1e0-4dd04ff7a8e0",
            "name": "group21",
            "description": "group21"
        },
        {
            "id": "d4b25556-9f1d-4757-9a53-040f2e70d711",
            "name": "test_group2",
            "description": "Test group 2"
        },
        {
            "id": "e11d596b-37d7-4fbb-9189-41bbdac2f29e",
            "name": "group20",
            "description": "group20"
        },
        {
            "id": "faf4a2bd-aa85-4079-98e6-f516f615554b",
            "name": "group10",
            "description": "group10",
            "default_template_id": "uuid"
        }
    ],
    "limit": 10,
    "count": 6,
    "offset": 0
}
```
#### GET /v1/token_group/{token_group_id}?include_permissions=true

Данный метод получает токен группу по `id`.
Параметр `include_permissions` определяет отдавать ли список разрешений на данную токен группу.
Пример ответа:
``` json
{
    "id": "15d2e57d-97fd-4d28-95d2-637fee779acb",
    "name": "test_group1",
    "description": "Test group 1",
    "permissions": [
        {
            "permission": "tokengroup_detokenize",
            "object_id": "6a2223ab-02c9-471e-b2ca-228ddb91f63f",
            "object_type": "user"
        },
        {
            "permission": "tokengroup_tokenize",
            "object_id": "6a2223ab-02c9-471e-b2ca-228ddb91f63f",
            "object_type": "user"
        }
    ]
}
```
#### POST /v1/token_group/

Данный метод создает новую токен группу.
Пример тела POST запроса:
``` json
{
    "name":"demo_group1",
    "description":"Demo group 1",
    "default_template_id":"uuid",
    "tokenize_permitted_users":["3ec2dfeb-99ad-4b3e-88eb-e2613a463463"],
    "detokenize_permitted_users":["3ec2dfeb-99ad-4b3e-88eb-e2613a463463"]
}
```
Name – обязательно, description необязательно, шаблон обязателен.
Поля tokenize_permitted_users и detokinze_permitted_users – массивы пользователей, которым сразу будет разрешен доступ на токенизацию и детокенизацию в этой группе.


#### PATCH /v1/token_group/{token_group_id}

Данный метод обновляет отдельные поля конкретной токен группы.
Пример тела POST запроса:
``` json
{
"description":"Demo group 1",
    "tokenize_permitted_users":["3ec2dfeb-99ad-4b3e-88eb-e2613a463463"],
    "detokenize_permitted_users":["3ec2dfeb-99ad-4b3e-88eb-e2613a463463"],
    "search_permitted_users":["3ec2dfeb-99ad-4b3e-88eb-e2613a463463"],
    "detokenize_permitted_users_with_masking_rules":[
        {"user_id":
        masking_rule_id}
    ]
}
```
Обновляет только те поля, которые указаны в запросе. Если поле отсутствует в `json`, то оно не будет изменено. Если поле есть в `json`, но оно пустое – то будет проведена его валидация, и, если пустое разрешено (для description, например), то оно будет обновлено на пустое в БД DAMASK
В ответ – объект пользователя с новыми значениями

#### POST /v1/token_group/{token_group_id}/permissions

Данный метод добавляет разрешения на работу с токен группой массиву объектов. 
Пример:
``` json
[{
"permission":"tokengroup_tokenize",
"object_type":"user",
"object_id":"3ec2dfeb-99ad-4b3e-88eb-e2613a463463"
},
{
"permission":"tokengroup_detokenize",
"object_type":"user",
"object_id":"3ec2dfeb-99ad-4b3e-88eb-e2613a463463"
}]
```

Виды разрешений только `tokengroup_search`, `tokengroup_tokenize` и `tokengroup_detokenize`.
В ответ:
``` json
{
    "success": true
}
```

#### DELETE /v1/token_group/{token_group_id}/permissions

Данный метод удаляет одно или несколько разрешений на работу с токен группой
Запрос:
``` json
[{
"permission":"tokengroup_tokenize",
"object_type":"user",
"object_id":"3ec2dfeb-99ad-4b3e-88eb-e2613a463463"
},
{
"permission":"tokengroup_detokenize",
"object_type":"user",
"object_id":"3ec2dfeb-99ad-4b3e-88eb-e2613a463463"
}]
```
Ответ:
``` json
{
    "success": true
}
```
### Ошибки валидации или ошибки в системе

Все ошибки валидации, как и ошибки в системе возвращаются в следующем виде:
``` json
{
  "trace": "e1a281fd-17f5-4244-a611-ffe71faeea1b",
  "errors": [
    {
      "code": "not_found",
      "id": "DME0003",
      "message": "User with id 3ec2dfeb-99ad-4b3e-88eb-e2613a4634631 not found"
    }
  ]
}
```
`Trace` – параметр для службы поддержки DAMASK, используется для поиска ошибки в логах
`Errors` – массив ошибок, где `id` – идентификатор ошибки в DAMASK, `message` – описание на выбранном языке

