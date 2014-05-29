## NextoneSpec

- [API HTTP methods](#api-http-methods)
- [Data format](#data-format)
 - [Element](#element)
 - [Collection](#collection)
 - [Reference element, collection](#reference-element-collection)
- [Methods](#methods)
- [Expand reference element, collection](#expand-reference-element-collection)
- [Collection Order, Limit and Offset](#collection-order-limit-and-offset)
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
URI коллекции http://api.example.com/tasks <br/> http://api.example.com/tasks/:taskId/labels | создает элемент в коллекции | возвращает коллекцию элементов | заменяет всю коллекцию | удаляет всю коллекцию
URI элемента http://api.example.com/tasks/:taskId   | ошибка | возвращает элемент | обновляет элемент | удаляет элемент

## Data format

application/json
JSON

### Element

Обязательные параметры в элементе

- **id** идентификатор элемента
- **href** string uri элемента
- **createdAt** date || timestamp дата создания елемента

`GET //api.example.com/tasks/1`

```json
{
    "id": 1,
    "href": "//api.example.com/tasks/1",
    "createdAt": 123,
    "title": "titlestring",
    "owner": {
        "href": "//api.example.com/users/1"
    }
}
```

### Reference element, collection

- **referenceName** object 
 - **href** string resource uri 

Элемент может иметь ссылку на другой элемент(владельца/пользователя) или коллекцию (метки,связанные задачи, пользователи), в этом случае элемент должен содержать имя и указатель на ресурс **owner.href**,  **labels.href**

```json
{
    "id": 1,
    "href": "//api.example.com/tasks/1",
    "createdAt": 123,
    "title": "titlestring",
    "owner": {
        "href": "//api.example.com/users/1"
    },
    "labels": {
        "href": "//api.example.com/tasks/1/labels"
    }
}
```

### Collection

Обязательные параметры в коллекции

- **total** number количество элементов
- **limit** number лимит размер коллекции default | limit
- **offset** number  смещение коллекции
- **collection** array of element коллекция элементов

`GET //api.example.com/tasks?limit=30`

```json
{
    "total": 3,
    "limit": 30,
    "offset": 0,
    "collection": [
        {
            "id": 1,
            "href": "//api.example.com/tasks/1",
            "createdAt": 123,
            "title": "titlestring1",
            "owner": {
                "href": "//api.example.com/users/2"
            }
        },
        {
            "id": 2,
            "href": "//api.example.com/tasks/2",
            "createdAt": 123,
            "title": "titlestring2",
            "owner": {
                "href": "//api.example.com/users/1"
            }
        },
        {
            "id": 2,
            "href": "//api.example.com/tasks/3",
            "createdAt": 123,
            "title": "titlestring3",
            "owner": {
                "href": "//api.example.com/users/3"
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
    "collection": []
}
```

## Methods

### Create element:

`POST //api.example.comtasks`

`POST //api.example.comtasks/:taskId/labels`

### Get element, collection:

`GET //api.example.com/tasks/:taskId`

`GET //api.example.com/tasks/:taskId/labels`

### Update element, collection:

`PUT //api.example.comtasks/:taskId`

`PUT //api.example.comtasks/:taskId/labels`

### Delete element, collection:

`DELETE //api.example.comtasks/:taskId`

`DELETE //api.example.comtasks/:taskId/labels`

### Collection filter element

Фильтр коллекции по `owner.id` [12,13,14]

`GET //api.example.com/tasks?owner=12,13,14`


## Expand reference element, collection 

Раскрытие **ссылки** на **элемент** или **коллекцию**

- **expand** [string[,...]]

Так выглядит простой запрос за элементом и ответ:

`GET //api.example.com/tasks/1`

```json
{
    "id": 123,
    "href": "//api.example.com/tasks/1",
    "createdAt": 123,
    "title": "titlestring",
    "owner": {
        "href": "//api.example.com/users/1"
    },
    "labels": {
        "href": "//api.example.com/tasks/1/labels"
    }
}
```

Чтобы получить полную информацию **owner** и **labels**, выполняем запрос с **expand**=owner,labels в query params:

`GET //api.example.com/tasks/1?expand=owner`

```json
{
    "id": 123,
    "href": "//api.example.com/tasks/1",
    "created": 123,
    "title": "TodoTitle",
    "owner": {
        "id": 1,
        "href": "//api.example.com/users/1",
        "createdAt": 1234,
        "fullName": "Jon Doe",
        "nickname": "jondoe"
    },
    "labels": {
        "href": "//api.example.com/tasks/1/labels",
        "total": 1,
        "limit": 10,
        "offset": 0,
        "collection": [
            {
                "id": 1,
                "href": '//api.example.com',
                "created": 123,
                "name": 'specapi'
            }
        ]
    }
}
```

## Collection Order, Limit and Offset

### Order

Сортировка коллекции по свойствам

- **order** [string[:desc|asc]]

Example:

`GET //api.example.com/tasks?order=createAt`

`GET //api.example.com/tasks?order=createAt,id`

`GET //api.example.com/tasks?order=createAt:desc`

`GET //api.example.com/tasks?order=createAt:desc,id:asc`

### Limit, Offset

Лимитирование коллекции

- **limit** number
- **offset** number

Example:

`GET //api.example.com/tasks?limit=10&offset=0`


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

