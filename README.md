# Задание

> Создать пример схемы для 2-3 полей поискового движка ( ElasticSearch ) для хранения атрибутов и значений атрибутов продукта.
> Атрибутов и их значений может быть неограниченное количество, изменяющееся в процессе работы приложения.
> При поиске продуктов поисковый движок возвращает найденные продукты, соответствующие атрибуты и их значения, список всех доступных атрибутов Приложить пример curl запроса получения данных продукта

# Реализация

К примеру, имеем работающий на localhost:9200 Elasticsearch (В моем случае, я его установил через Homebrew)

Создаем новый индекс, к примеру для товаров каталога (используем explicit mapping, чтобы случайно не проиндексировать что попало):
```bash
curl -X PUT "localhost:9200/product?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "name":   { "type": "text"  },     
      "price":    { "type": "integer" },  
      "category":  { "type": "keyword"  }
    }
  }
}
'
```

И заполняем парой первоначальных значений
```bash
curl -X PUT "localhost:9200/product/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Testcase product one",
  "price": 2500,
  "category": "Equipment"
}
'
curl -X PUT "localhost:9200/product/_doc/2?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Testcase product two",
  "price": 1750,
  "category": "Garden"
}
'
```

Проверяем, что все на месте
```bash
curl -X GET "localhost:9200/product/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "price": "asc" }
  ]
}
'
```

При необходимости добавляем новые поля в наш mapping, например описание товара:
```bash
curl -X PUT "localhost:9200/product/_mapping" -H 'Content-Type: application/json' -d'
{
  "properties": {
    "description": {
      "type": "text",
      "index": true
    }
  }
}
'
```
Смотрим, изменился ли mapping:
```bash
curl -X GET "localhost:9200/product/_mapping?pretty"
```

Добавляем продукт, уже можно с новым полем
```bash
curl -X PUT "localhost:9200/product/_doc/3?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Testcase product tree",
  "price": 3250,
  "category": "Garden",
  "description": "It is totaly fuckin awesome product ever seen!"
}
'
```

Теперь пишем разные запросы по выборке/агрегации:

Например ищем по точной фразе в названии:
```bash
curl -X GET "localhost:9200/product/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "name": "product tree" } }
}
'
```

Или пример аггрегации:
```bash
curl -X GET "localhost:9200/product/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_category": {
      "terms": {
        "field": "category.keyword"
      },
      "aggs": {
        "average_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
'
```
