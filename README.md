## NextoneSpec

- [API HTTP methods](#api-http-methods)
- [Data format](#data-format)
 - [Element](#element)
 - [Collection](#collection)
 - [Reference resource, collection](#reference-resource-collection)
- [Methods](#methods)
- [Expand reference resource, collection](#expand-reference-resource-collection)
- [Collection Order, Limit and Offset](#collection-order-limit-and-offset)
- [Restrict Fields](#restrict-fields)
- [Errors](#errors)
 - [400 Bad request](#400-bad-request)
 - [401 Unauthorized](#401-unauthorized)
 - [403 Permission denied](#403-permission-denied)
 - [409 Conflict](#409-conflict)
 - [500 Internal error](#500-internal-error)

## API HTTP methods

RESTful

| HTTP      | Method    |
| :--       | :--       |
| POST      | Create    |
| GET       | Read      |
| PUT       | Update    |
| DELETE    | Delete    |


| Resource | POST | GET | PUT | DELETE|
| :--- | --- | --- | --- | --- |
| `http://<domain>/<resources>`| Create | Read | Update | Delete |
| `http://<domain>/<resources>/<resourceId>` | Update | Read | Update | Delete |

## Data format
Only JSON

### Resource

Обязательные параметры ресурса

- **id** идентификатор ресурса
- **href** string uri элемента
- **createdAt** date || timestamp дата создания ресурса

`GET http://<domain>/tasks/1`

```json
{
    "id": 1,
    "href": "http://<domain>/tasks/1",
    "createdAt": 571352400000,
    "title": "Task name",
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
            "title": "Task name1",
            "owner": {
                "href": "http://<domain>/users/2"
            }
        },
        {
            "id": 2,
            "href": "http://<domain>/tasks/2",
            "createdAt": 571352400000,
            "title": "Task name2",
            "owner": {
                "href": "http://<domain>/users/1"
            }
        }
    ]
}
```

### Reference resource, collection

- **referenceName** object
- **href** string resource uri 

Ресурс может иметь ссылку на другой элемент(владельца/пользователя) или коллекцию (метки,связанные задачи, пользователи), в этом случае элемент должен содержать имя и указатель на ресурс **owner.href**,  **labels.href**

```json
{
    "id": 1,
    "href": "http://<domain>/tasks/1",
    "createdAt": 571352400000,
    "title": "Task name",
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

- **total** number общее количество ресурсов в коллекции
- **limit** number лимит на размер возвращаемой коллекции default | limit
- **offset** number смещение
- **rows** array of resource массив ресурсов

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
            "title": "Task name1",
            "owner": {
                "href": "http://<domain>/users/2"
            }
        },
        {
            "id": 2,
            "href": "http://<domain>/tasks/2",
            "createdAt": 571352400000,
            "title": "Task name2",
            "owner": {
                "href": "http://<domain>/users/1"
            }
        },
        {
            "id": 2,
            "href": "http://<domain>/tasks/3",
            "createdAt": 571352400000,
            "title": "Task name3",
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

Фильтр коллекции по `owner.id` [12,13,14]

`GET http://<domain>/<resources>?<filter>=<filterValue,...>`

Example:

`GET http://<domain>/tasks?owner=12,13,14`

## Expand reference resource, collection 

Раскрытие **ссылки** на **ресурс** или **коллекцию**

- **expand** [string[,...]]

`GET http://<domain>/<resources>/<resourceId>?<expand>=<expandValue,...>`

Чтобы получить полную информацию **owner** и **labels**, выполняем запрос с **expand**=owner,labels:

`GET http://<domain>/tasks/1?expand=owner,labels`

```json
{
    "id": 123,
    "href": "http://<domain>/tasks/1",
    "createdAt": 571352400000,
    "title": "Task name",
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

- **fields** [string[,...]]

`GET http://<domain>/<resources>?fields=<field1,field2,...>`

`GET http://<domain>/<resources>/<resourceId>?fields=<field1,field2,...>`

Например нам необходимы id, название и дата создания задачи

`GET http://<domain>/tasks/1?fields=id,title,createdAt`

```json
{
    "id": 1,
    "createdAt": 571352400000,
    "title": "Task name"
    "href": "http://<domain>/tasks/1",
}
```

`GET http://<domain>/tasks?fields=id,title,createdAt`

```json
{
    "total": 3,
    "limit": 25,
    "offset": 0,
    "rows": [
        {
            "id": 1,
            "createdAt": 571352400000,
            "href": "http://<domain>/tasks/1",
            "title": "Task name1"
        },
        {
            "id": 2,
            "createdAt": 571352400000,
            "href": "http://<domain>/tasks/2",
            "title": "Task name2"
        },
        {
            "id": 3,
            "createdAt": 571352400000,
            "href": "http://<domain>/tasks/3",
            "title": "Task name3"
        }
    ]
}

```

## Collection Order, Limit and Offset

### Order

Сортировка коллекции по свойствам

- **order** [string[:desc|asc]]

`GET http://<domain>/<resources>?order=<field[:desc|:asc], ...>`

Example:

`GET http://<domain>/tasks?order=createAt`

`GET http://<domain>/tasks?order=createAt,id`

`GET http://<domain>/tasks?order=createAt:desc`

`GET http://<domain>/tasks?order=createAt:desc,id:asc`

### Paging, Limit, Offset

Лимитирование коллекции

- **limit** number
- **offset** number

`GET http://<domain>/<resources>?limit=<int>&offset=<int>`

Example:

`GET http://<domain>/tasks?limit=10&offset=0`


## Errors

Обязательный параметры

- **message** string сообщение об ошибке

```json
{
    "message": "Some error message"
}
```

### 400 Bad request

```json
{
    "message": "Bad request"
}
```

```json
{
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

```json
{
    "message": "Unauthorized"
}
```

### 403 Permission Denied
```json
{
    "message": "Permission Denied"
}
```

### 404 Not fount

```json
{
    "message": "Not found"
}
```

### 409 Conflict

- dublicate entry
- ...

```json
{
    "message": "Conflict"
}
```

### 500 Internal error

```json
{
    "message": "Internal error"
}
```

