## NextoneSpec

- [API HTTP methods](#api-http-methods)
- [Data format](#data-format)
 - [Element](#element)
 - [Collection](#collection)
 - [Reference element, collection](#reference-element-collection)
- [Methods](#methods)
- [Expand reference element, collection](#expand-reference-element-collection)
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

Resource | POST (create) | GET (read) | PUT (update) | DELETE (delete)
--- | --- | --- | --- | ---
URI коллекции http://<domain>
<domain>/tasks <br/> http://<domain>
<domain>/tasks/:taskId/labels | создает элемент в коллекции | возвращает коллекцию элементов | заменяет всю коллекцию | удаляет всю коллекцию
URI элемента http://<domain>
<domain>/tasks/:taskId   | ошибка | возвращает элемент | обновляет элемент | удаляет элемент

## Data format
Only JSON

### Element

Обязательные параметры в элементе

- **id** идентификатор элемента
- **href** string uri элемента
- **createdAt** date || timestamp дата создания елемента

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

#### Elements

`GET http://<domain>/tasks/<taskId, ...>,`

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

### Reference element, collection

- **referenceName** object
- **href** string resource uri 

Элемент может иметь ссылку на другой элемент(владельца/пользователя) или коллекцию (метки,связанные задачи, пользователи), в этом случае элемент должен содержать имя и указатель на ресурс **owner.href**,  **labels.href**

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

- **total** number количество элементов
- **limit** number лимит размер коллекции default | limit
- **offset** number  смещение коллекции
- **collection** array of element коллекция элементов

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

### Empty Collection 

Если коллеция пустая, приходит ответ 200 с данными

```json
{
    "total": 0,
    "limit": 30,
    "offset": 0,
    "rows": []
}
```

## Methods

### Create element:

`POST http://<domain>/tasks`

`POST http://<domain>/tasks/:taskId/labels`

### Get element, collection:

`GET http://<domain>/tasks/:taskId`

`GET http://<domain>/tasks/:taskId/labels`

### Update element, collection:

`PUT http://<domain>/tasks/:taskId`

`PUT http://<domain>/tasks/:taskId/labels`

### Delete element, collection:

`DELETE http://<domain>/tasks/:taskId`

`DELETE http://<domain>/tasks/:taskId/labels`

### Collection filter element

Фильтр коллекции по `owner.id` [12,13,14]

`GET http://<domain>/tasks?owner=12,13,14`

## Expand reference element, collection 

Раскрытие **ссылки** на **элемент** или **коллекцию**

- **expand** [string[,...]]

Так выглядит простой запрос за элементом и ответ:

`GET http://<domain>/tasks/1`

```json
{
    "id": 123,
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

Чтобы получить полную информацию **owner** и **labels**, выполняем запрос с **expand**=owner,labels в query params:

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

Example:

`GET http://<domain>/tasks?order=createAt`

`GET http://<domain>/tasks?order=createAt,id`

`GET http://<domain>/tasks?order=createAt:desc`

`GET http://<domain>/tasks?order=createAt:desc,id:asc`

### Paging, Limit, Offset

Лимитирование коллекции

- **limit** number
- **offset** number

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

