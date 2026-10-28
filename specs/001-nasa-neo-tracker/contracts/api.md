# NASA NeoWs API Contract

## Request

```text
GET https://api.nasa.gov/neo/rest/v1/feed
    ?start_date=YYYY-MM-DD
    &end_date=YYYY-MM-DD
    &api_key=<from NASA_API_KEY or DEMO_KEY>
```

`start_date` и `end_date` MUST иметь одинаковое значение выбранной даты.
Ключ не записывается в исходный код, fixtures, кэш или документацию с
секретами. При использовании `DEMO_KEY` интерфейс показывает ровно уведомление:
«Используется демонстрационный ключ NASA с ограниченной квотой. Для регулярной
работы настройте NASA_API_KEY». Реальное значение ключа не показывается.

## Expected response shape

```text
{
  "near_earth_objects": {
    "YYYY-MM-DD": [
      {
        "name": "...",
        "estimated_diameter": {
          "meters": {
            "estimated_diameter_min": number,
            "estimated_diameter_max": number
          }
        },
        "is_potentially_hazardous_asteroid": boolean,
        "close_approach_data": [
          {
            "close_approach_date": "YYYY-MM-DD",
            "relative_velocity": {
              "kilometers_per_second": "number"
            },
            "miss_distance": {
              "kilometers": "number"
            }
          }
        ]
      }
    ]
  }
}
```

## Mapping to UI

| UI column | JSON path | Unit/display |
|---|---|---|
| Название | `name` | Text |
| Мин. диаметр | `estimated_diameter.meters.estimated_diameter_min` | m |
| Макс. диаметр | `estimated_diameter.meters.estimated_diameter_max` | m |
| Скорость | `close_approach_data[].relative_velocity.kilometers_per_second` | km/s |
| Расстояние | `close_approach_data[].miss_distance.kilometers` | km |
| Потенциальная опасность | `is_potentially_hazardous_asteroid` | `Да` / `Нет` |

Парсер берёт только массив по ключу выбранной даты и элемент
`close_approach_data`, у которого `close_approach_date` совпадает с ней.
Отсутствующий совпадающий элемент не заменяется записью другой даты.

## Error normalization

- HTTP 401/403: неверный или недоступный ключ.
- HTTP 429: превышен лимит API; UI показывает понятное сообщение о превышении
  квоты, использует кэш согласно спецификации и оставляет повторный запрос
  доступным.
- HTTP 4xx/5xx: ошибка NASA API с кодом ответа.
- `URLError`: сеть недоступна или соединение не установлено.
- `TimeoutError`/socket timeout: истёк конечный тайм-аут.
- `JSONDecodeError`: повреждённый JSON.
- `KeyError`, `TypeError`, неверные числа/даты: некорректная структура.

Все ошибки преобразуются в результат операции для UI; исключение не должно
достигать Tkinter callback как необработанное. Тайм-аут сетевой операции —
15 секунд.
