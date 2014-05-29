## nextoneSpec

- [API HTTP methods](#api-http-methods)
- [Data format](#data-format)
 - [Element](#element)
 - [Collection](#collection)
- [Methods](#methods)
- [Expand child element](#expand-child-element)
- [Collection Order, Limit and Offset](#collection-order-limit-and-offset)

## API HTTP methods

RESTful

Resource | POST (create) | GET (read) | PUT (update) | DELETE (delete)
--- | --- | --- | --- | ---
URI коллекции  | создает элемент в коллекции | возвращает список элементов | заменяет всю коллекцию на другую | удаляет всю коллекцию
URI элемента   | not used | возвращает element | обновляет элемент | удаляет элемент

## Data format

application/json
JSON

### Element

Обязательные параметры в элементе

- **id** идентификатор элемента
- **href** string uri элемента
- **createdAt** date || timestamp дата создания елемента

`GET /resources/1`

```json
{
    id: 1,
    href: '/resources/1',
    createdAt: 123
}
```

### Collection

Обязательные параметры в коллекции

- **total** number количество элементов
- **limit** number лимит размер коллекции default | limit
- **offset** number  смещение коллекции
- **collection** array коллекция элементов

`GET /resources?limit=30`

```json
{
    total: 3,
    limit: 30,
    offset: 0,
    collection: [
        {
            id: 1,
            href: '/resource/1',
            title: 'titlestring'
        },
        {
            id: 2,
            href: '/resource/2',
            title: 'titlestring'
        },
        {
            id: 2,
            href: '/resource/3',
            title: 'titlestring'
        }
    ]
}
```

## Methods

### Create element:

`POST /resources`

`POST /resources/:elementId/resources2`

### Get element, collection:

`GET /resources/:elementId`

`GET /resources/:elementId/resources2`

### Update element, collection:

`PUT /resources/:elementId`

`PUT /resources/:elementId/resources`

### Delete element, collection:

`DELETE /resources/:elementId`

`DELETE /resources/:elementId/resources`

### Collection filter element

Фильтр коллекции по `owner.id` [12,13,14]

`GET /resources?owner=12,13,14`


## Expand child element 

Раскрытие **дочернего элемента**

- **expand** [string[,...]]

Так выглядит простой запрос за элементом и ответ:

`GET /todos/1`

```json
{
    id: 123,
    title: 'TodoTitle',
    owner: {
        href: '/users/1'
    }
}
```

Чтобы получить полную информацию **owner** выполняем запрос с **expand**:

`GET /todos/1?expand=owner`

```json
{
    id: 123,
    title: 'TodoTitle',
    owner: {
        id: 1,
        href: '/users/1',
        createdAt: 1234,
        updatedAt: 1234,
        fullName: 'Jon Doe',
        nickname: 'jondoe'
    }
}
```

## Collection Order, Limit and Offset

### Order

Сортировка коллекции по свойствам

- **order** [string[:desc|asc]]

Example:

`GET /resources?order=createAt`

`GET /resources?order=createAt,id`

`GET /resources?order=createAt:desc`

`GET /resources?order=createAt:desc,id:asc`

### Limit, Offset

Лимитирование коллекции

- **limit** number
- **offset** number

Example:

`GET /resources?limit=10&offset=0`
