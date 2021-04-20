# Description
К примеру, имеем работающий на localhost:9200 Elasticsearch (В моем случае, я его установил через Homebrew)

Создаем новый индекс, к примеру для товаров каталога (используем explicit mapping, чтобы случайно не проиндексировать что попало):
'''
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
'''
'''
curl -X PUT "localhost:9200/product/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Testcase product one",
  "price": 2500,
  "category": "Equipment"
}
'
'''
curl -X PUT "localhost:9200/product/_doc/2?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Testcase product two",
  "price": 1750,
  "category": "Garden"
}
'
'''

Проверяем, что все на месте
curl -X GET "localhost:9200/product/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "price": "asc" }
  ]
}
'


При необходимости добавляем новые поля в наш mapping, например описание товара:

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

Смотрим, изменился ли mapping:

curl -X GET "localhost:9200/product/_mapping?pretty"

Добавляем продукт, уже можно с новым полем

curl -X PUT "localhost:9200/product/_doc/3?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Testcase product tree",
  "price": 3250,
  "category": "Garden",
  "description": "It is totaly fuckin awesome product ever seen!"
}
'

Теперь пишем разные запросы по выборке/агрегации:

Например ищем по точной фразе в названии:

curl -X GET "localhost:9200/product/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "name": "product tree" } }
}
'

Или узнаем все цены товаров:
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
