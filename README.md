## RESTful API Specification 

- [API HTTP methods](#api-http-methods)
- [Data format](#data-format)
 - [Resource](#resource)
 - [Collection](#collection)
 - [Reference resource, collection](#reference-resource-collection)
- [Methods](#methods)
- [Expand reference resource, collection](#expand-reference-resource-collection)
- [Collection Sort, Limit and Offset](#collection-sort-limit-and-offset)
- [Restrict Fields](#restrict-fields)
- [Errors](#errors)
 - [400 Bad request](#400-bad-request)
 - [401 Unauthorized](#401-unauthorized)
 - [403 Permission denied](#403-permission-denied)
 - [409 Conflict](#409-conflict)
 - [500 Internal error](#500-internal-error)

## API HTTP methods

| HTTP      | Method    |
| :--       | :--       |
| POST      | Create    |
| GET       | Read      |
| PUT       | Update    |
| DELETE    | Delete    |

| Resource | POST | GET | PUT | DELETE|
| :--- | --- | --- | --- | --- |
| `http://<domain>/<resources>`| create | read | update | delete |
| `http://<domain>/<resources>/<resourceId>` | update | read | update | delete |

## Data format
JSON

### Resource

Обязательные параметры ресурса

- **id** [`integer`|`string`] идентификатор ресурса
- **href** `string` uri ресурса
- **createdAt** [`integer`] дата создания ресурса timestamp

`GET http://<domain>/tasks/1`

```json
{
    "id": 1,
    "href": "http://<domain>/tasks/1",
    "createdAt": 571352400000,
    "name": "Task name",
    "owner": {
        "href": "http://<domain>/users/1"
    }
}
```

#### Resources

`GET http://<domain>/<resources>/<resourceId1, resourceId2, ...>`

Example:

`GET http://<domain>/tasks/1,2`

```json
{
    "rows": [
        {
            "id": 1,
            "href": "http://<domain>/tasks/1",
            "createdAt": 571352400000,
            "name": "Task name1",
            "owner": {
                "href": "http://<domain>/users/2"
            }
        },
        {
            "id": 2,
            "href": "http://<domain>/tasks/2",
            "createdAt": 571352400000,
            "name": "Task name2",
            "owner": {
                "href": "http://<domain>/users/1"
            }
        }
    ]
}
```

### Reference resource, collection

- **resourceName** `object` имя ресурса
  - **resourceName.href** `string` uri ресурса

Ресурс может иметь ссылку на другой ресурс (пользователя, событие) или коллекцию (метки, связанные задачи, пользователи), в этом случае ресурс должен содержать имя и указатель на ресурс **owner.href**,  **labels.href**

```json
{
    "id": 1,
    "href": "http://<domain>/tasks/1",
    "createdAt": 571352400000,
    "name": "Task name",
    "owner": {
        "href": "http://<domain>/users/1"
    },
    "labels": {
        "href": "http://<domain>/tasks/1/labels"
    }
}
```

### Collection

Обязательные параметры в коллекции

- **total** `integer` общее количество ресурсов в коллекции
- **limit** `integer` лимит на размер возвращаемой коллекции (значение установленное по умолчанию или переденнное через query params)
- **offset** `integer` смещение (значение установленное по умолчанию или переденнное через query params)
- **rows** `array` массив ресурсов

`GET http://<domain>/<resources>?limit=30`

`GET http://<domain>/tasks?limit=30`

```json
{
    "total": 3,
    "limit": 30,
    "offset": 0,
    "rows": [
        {
            "id": 1,
            "href": "http://<domain>/tasks/1",
            "createdAt": 571352400000,
            "name": "Task name1",
            "owner": {
                "href": "http://<domain>/users/2"
            }
        },
        {
            "id": 2,
            "href": "http://<domain>/tasks/2",
            "createdAt": 571352400000,
            "name": "Task name2",
            "owner": {
                "href": "http://<domain>/users/1"
            }
        },
        {
            "id": 2,
            "href": "http://<domain>/tasks/3",
            "createdAt": 571352400000,
            "name": "Task name3",
            "owner": {
                "href": "http://<domain>/users/3"
            }
        }
    ]
}
```

Если коллеция пустая:

```json
{
    "total": 0,
    "limit": 30,
    "offset": 0,
    "rows": []
}
```

## Methods

### Create resource:

`POST http://<domain>/<resources>`

Example:

`POST http://<domain>/tasks`

### Get resource or collection:

`GET http://<domain>/<resources>`

`GET http://<domain>/<resources>/<resourceId>`

Example:

`GET http://<domain>/tasks/1`

`GET http://<domain>/tasks/1/labels`

### Update resource, collection:

`PUT http://<domain>/<resources>/<resourceId>`

`PUT http://<domain>/<resources>/<resourceId>/<resources>`

Example:

`PUT http://<domain>/tasks/1`

`PUT http://<domain>/tasks/1/labels`

### Delete resource, collection:

`DELETE http://<domain>/<resources>/<resourceId>`

`DELETE http://<domain>/<resources>/<resourceId>/<resources>`

Example:

`DELETE http://<domain>/tasks/1`

`DELETE http://<domain>/tasks/1/labels`

### Collection filter resources

Фильтр коллекции по **owner.id** [12,13,14]

`GET http://<domain>/<resources>?<field>=<value,...>`

Example:

`GET http://<domain>/tasks?owner=12,13,14`

## Expand reference resource, collection 

Раскрытие **ссылки** на **ресурс** или **коллекцию**

- **expand** [`string`[, ...]] имя ресурса или коллекции

`GET http://<domain>/<resources>/<resourceId>?expand=<field1,...>`

Чтобы получить полную информацию **owner** и **labels**, выполняем запрос с **expand**=owner,labels:

`GET http://<domain>/tasks/1?expand=owner,labels`

```json
{
    "id": 123,
    "href": "http://<domain>/tasks/1",
    "createdAt": 571352400000,
    "name": "Task name",
    "owner": {
        "id": 1,
        "href": "http://<domain>/users/1",
        "createdAt": 571352400000,
        "fullName": "Jon Doe",
        "nickname": "jondoe"
    },
    "labels": {
        "href": "http://<domain>/tasks/1/labels",
        "total": 1,
        "limit": 10,
        "offset": 0,
        "rows": [
            {
                "id": 1,
                "href": "//<domain>/labels/1",
                "createdAt": 571352400000,
                "name": "specapi"
            }
        ]
    }
}
```

## Restrict Fields

Ограничение списка возвращаемых параметров

- **fields** [`string`[, ...]] имя параметра

`GET http://<domain>/<resources>?fields=<field1,field2,...>`

`GET http://<domain>/<resources>/<resourceId>?fields=<field1,field2,...>`

Например нам необходимы `id`, `название` задачи

`GET http://<domain>/tasks/1?fields=id,name`

```json
{
    "id": 1,
    "name": "Task name"
}
```

`GET http://<domain>/tasks?fields=id,name`

```json
{
    "total": 3,
    "limit": 25,
    "offset": 0,
    "rows": [
        {
            "id": 1,
            "name": "Task name 1"
        },
        {
            "id": 2,
            "name": "Task name 2"
        },
        {
            "id": 3,
            "name": "Task name 3"
        }
    ]
}

```

## Collection Sort, Limit and Offset

### Paging, Limit, Offset

Лимитирование коллекции ресурсов

- **limit** `integer` размер возвращаемой коллекции
- **offset** `integer` смещение

`GET http://<domain>/<resources>?limit=<int>&offset=<int>`

Example:

`GET http://<domain>/tasks?limit=10`

`GET http://<domain>/tasks?offset=10`

`GET http://<domain>/tasks?limit=10&offset=10`


### Sort

Сортировка коллекции по параметрам 

- **sort** [`string`[:desc|asc]]

`GET http://<domain>/<resources>?sort=<field[:desc|:asc], ...>`

Example:

`GET http://<domain>/tasks?sort=createAt`

`GET http://<domain>/tasks?sort=createAt:desc`

`GET http://<domain>/tasks?sort=createAt:asc`

`GET http://<domain>/tasks?sort=createAt,id`

`GET http://<domain>/tasks?sort=createAt:desc,name`

`GET http://<domain>/tasks?sort=createAt:asc,name:desc`


## Errors

Обязательный параметры

- **statusCode** `integer` статус код
- **errorCode** `string` код ошибки
- **message** `string` сообщение об ошибке


```json
{
    "statusCode": 400,
    "errorCode": "MESSAGE_CODE",
    "message": "Some error message"
}
```

### 400 Bad request

Не валидный запрос

```json
{
    "statusCode": 400,
    "errorCode": "BAD_REQUEST",
    "message": "Bad request"
}
```

```json
{
    "statusCode": 400,
    "errorCode": "BAD_REQUEST",
    "message": "Bad request",
    "errors": [
        {
            "property": "data.email",
            "value": "example@@invalid.com",
            "message": "does not conform to the 'email' format"
        }
    ]
}
```


### 401 Unauthorized

Необходима авторазация

```json
{
    "statusCode": 401,
    "errorCode": "UNAUTHORIZED",
    "message": "Unauthorized"
}
```

### 403 Permission Denied

Доступ рапрещен

```json
{
    "statusCode": 403,
    "errorCode": "PERMISSION_DENIED",
    "message": "Permission Denied"
}
```

### 404 Not found

Ресурс не найден

```json
{
    "statusCode": 404,
    "errorCode": "NOT_FOUND_RESOURCE",
    "message": "Not found resource"
}
```

Не существующий путь

```json
{
    "statusCode": 404,
    "errorCode": "NOT_FOUND_ROUTE",
    "message": "Not found route"
}
```

### 409 Conflict

- пользователь с таким именен существует
- ...

```json
{
    "statusCode": 409,
    "errorCode": "CONFLICT_ERROR",
    "message": "Conflict error"
}
```

### 500 Internal error

Что то пошло не так

```json
{
    "statusCode": 500,
    "errorCode": "INTERNAL_SERVER_ERROR",
    "message": "Internal server error"
}
```

