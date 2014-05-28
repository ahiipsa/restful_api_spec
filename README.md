## nextoneSpec

## RESTful API HTTP methods

resource | POST (create) | GET (read) | PUT (update) | DELETE (delete)
--- | ---
URI коллекции  | создает элемент в коллекции | возвращает список элементов | заменяет всю коллекцию на другую | удаляет всю колекцию
URI элемента   | not used | возвращает element | обновляет элемент | удаляет элемент

## Element

Обязательные параметры в элементе

- **id** идентификатор элемента
- **href** string uri элемента
- **createdAt** date || timestamp дата создания елемента

```json
{
    id: id,
    href: '/resources/1',
    createdAt: 123
}
```

### Create element:

`POST /resources`
`GET /resources/:elementId/resources`

### Get element and collection:

`GET /resources/:elementId`
`GET /resources/:elementId/resources2`

### Update element and collection:

`PUT /resources/:elementId`
`PUT /resources/:elementId/resources`

### Delete element and collection:

`DELETE /resources/:elementId`
`DELETE /resources/:elementId/resources`

## Collection data format

- **total** number количество элементов
- **limit** number лимит размер колекции default | limit
- **offset** number  смещение колекции
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


### Collection find element

find element where owner id in 12,13,14

`GET /resources?owner=12,13,14`


## Expand

Раскрытие **дочернего элемента**

- **expand** [string[,...]]

Так выглядит простой запрос за ресурсом и ответ:

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

## Order, Limit and Offset

### Order

Сортировка коллекции по свойствам

- **order** [string[:desc|asc]]

Example:

`GET /resources?order=createAt`
`GET /resources?order=createAt,id`
`GET /resources?order=createAt:desc`

### Limit, Offset

Лимитирование коллекции ресурсов

- **limit** number
- **offset** number

Example:

`GET /resources?limit=10&offset=0`
