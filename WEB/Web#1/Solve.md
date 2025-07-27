### Уязвимый сайт:

http://platform_ctf_1.caplag.su:33212

![](./screens/screen1.PNG)

### Попробуем пофазить:

```
ffuf -w /usr/share/wordlists/dirb/big.txt \
-u "http://platform_ctf_1.caplag.su:33212/FUZZ" \
-t 50 \
-mc all \
-fc 400,404 \
-recursion \
-recursion-depth 10
```

Единственное, что мы нашли - это страницу /flag , но доступа к ней у нас очевидно нет.

### Задание намекает нам на "Графы"

Загуглим название задания в поиске подсказки:

![](./screens/screen2.PNG)

### Далее еще один запрос в гугл и найдем эндпоинт:

http://platform_ctf_1.caplag.su:33212/graphql

![](./screens/screen7.PNG)

Запросим схему:

```
query {
  __schema {
    types {
      name
      fields {
        name
        type {
          name
        }
      }
    }
  }
}
```

Ответ:

```json
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "JSON",
          "fields": null
        },
        {
          "name": "UserStats",
          "fields": [
            {
              "name": "_id",
              "type": {
                "name": "ID"
              }
            },
```

<details>
<summary>Полный ответ</summary>

```json

            {
              "name": "count",
              "type": {
                "name": "Int"
              }
            }
          ]
        },
        {
          "name": "ID",
          "fields": null
        },
        {
          "name": "Int",
          "fields": null
        },
        {
          "name": "Query",
          "fields": [
            {
              "name": "getUserStats",
              "type": {
                "name": null
              }
            }
          ]
        },
        {
          "name": "Boolean",
          "fields": null
        },
        {
          "name": "String",
          "fields": null
        },
        {
          "name": "__Schema",
          "fields": [
            {
              "name": "description",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "types",
              "type": {
                "name": null
              }
            },
            {
              "name": "queryType",
              "type": {
                "name": null
              }
            },
            {
              "name": "mutationType",
              "type": {
                "name": "__Type"
              }
            },
            {
              "name": "subscriptionType",
              "type": {
                "name": "__Type"
              }
            },
            {
              "name": "directives",
              "type": {
                "name": null
              }
            }
          ]
        },
        {
          "name": "__Type",
          "fields": [
            {
              "name": "kind",
              "type": {
                "name": null
              }
            },
            {
              "name": "name",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "description",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "specifiedByUrl",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "fields",
              "type": {
                "name": null
              }
            },
            {
              "name": "interfaces",
              "type": {
                "name": null
              }
            },
            {
              "name": "possibleTypes",
              "type": {
                "name": null
              }
            },
            {
              "name": "enumValues",
              "type": {
                "name": null
              }
            },
            {
              "name": "inputFields",
              "type": {
                "name": null
              }
            },
            {
              "name": "ofType",
              "type": {
                "name": "__Type"
              }
            }
          ]
        },
        {
          "name": "__TypeKind",
          "fields": null
        },
        {
          "name": "__Field",
          "fields": [
            {
              "name": "name",
              "type": {
                "name": null
              }
            },
            {
              "name": "description",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "args",
              "type": {
                "name": null
              }
            },
            {
              "name": "type",
              "type": {
                "name": null
              }
            },
            {
              "name": "isDeprecated",
              "type": {
                "name": null
              }
            },
            {
              "name": "deprecationReason",
              "type": {
                "name": "String"
              }
            }
          ]
        },
        {
          "name": "__InputValue",
          "fields": [
            {
              "name": "name",
              "type": {
                "name": null
              }
            },
            {
              "name": "description",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "type",
              "type": {
                "name": null
              }
            },
            {
              "name": "defaultValue",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "isDeprecated",
              "type": {
                "name": null
              }
            },
            {
              "name": "deprecationReason",
              "type": {
                "name": "String"
              }
            }
          ]
        },
        {
          "name": "__EnumValue",
          "fields": [
            {
              "name": "name",
              "type": {
                "name": null
              }
            },
            {
              "name": "description",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "isDeprecated",
              "type": {
                "name": null
              }
            },
            {
              "name": "deprecationReason",
              "type": {
                "name": "String"
              }
            }
          ]
        },
        {
          "name": "__Directive",
          "fields": [
            {
              "name": "name",
              "type": {
                "name": null
              }
            },
            {
              "name": "description",
              "type": {
                "name": "String"
              }
            },
            {
              "name": "isRepeatable",
              "type": {
                "name": null
              }
            },
            {
              "name": "locations",
              "type": {
                "name": null
              }
            },
            {
              "name": "args",
              "type": {
                "name": null
              }
            }
          ]
        },
        {
          "name": "__DirectiveLocation",
          "fields": null
        }
      ]
    }
  }
}
```
</details>


### Скорее всего, сервер использует MongoDB.

1. **`_id` вместо `id`**  
   → Стандартно для MongoDB (`ObjectId`), в SQL — просто `id`.
2. **Нет SQL-фильтров**  
   → Отсутствуют `WHERE`, `JOIN`, `_eq` (Hasura).
3. **Тип `JSON` в схеме**  
   → MongoDB работает с BSON, PostgreSQL — `JSONB`.
4. **Плоская структура запросов**  
   → Минимум вложенности, нет сложных связей.

### Перехватываем запрос POST к /login:
![](./screens/screen3.PNG)

В задании есть намек, нам нужно найти пароль который начинается с "admin"

NoSQL-инъекция работает, только если сервер принимает JSON и передаёт его "как есть" в БД. 

### Поэтому меняем

```
Content-Type: application/x-www-form-urlencoded
```

на

```
Content-Type: application/json
```

### А также добавляем регулярку с помощью которой найдем пароль который начинается с "admin"

![](./screens/screen4.PNG)

### Отправляем запрос и получаем перенаправление на эндпоинт flag, который был найден ранее:

![](./screens/screen5.PNG)

### Отправляем этот запрос с полученными куками:

![](./screens/screen6.PNG)

