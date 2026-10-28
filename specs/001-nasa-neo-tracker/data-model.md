# Data Model: NASA NEO Tracker

## DateQuery

| Field | Type | Source/rules |
|---|---|---|
| `value` | `date` | Получается только из строки `YYYY-MM-DD`; дата должна существовать. |
| `text` | `str` | Исходное значение поля; пустое и неверное значение отклоняется до HTTP. |

## AsteroidApproach

| Field | Type | Source/API unit | Rules |
|---|---|---|---|
| `name` | `str` | `name` | Обязательное непустое название. |
| `diameter_min_m` | `float \| None` | `estimated_diameter.meters.estimated_diameter_min`, m | При отсутствии — «Нет данных»; ноль не подставляется. |
| `diameter_max_m` | `float \| None` | `estimated_diameter.meters.estimated_diameter_max`, m | При отсутствии — «Нет данных»; ноль не подставляется. |
| `velocity_km_s` | `float \| None` | `relative_velocity.kilometers_per_second`, km/s | При отсутствии — «Нет данных»; ноль не подставляется. |
| `distance_km` | `float \| None` | `miss_distance.kilometers`, km | При отсутствии — «Нет данных»; ноль не подставляется. |
| `is_potentially_hazardous` | `bool \| None` | `is_potentially_hazardous_asteroid` | `Да`/`Нет`; при отсутствии — «Неизвестно». |
| `approach_date` | `date` | `close_approach_date` | Должна совпадать с `DateQuery`. |

## Dataset

| Field | Type | Rules |
|---|---|---|
| `data_date` | `date` | Дата ключа `near_earth_objects` и выбранного дня сближения. |
| `received_at` | `datetime` | Время успешного получения и разбора, ISO 8601 с локальным offset. |
| `source` | `str` | Всегда `NASA NeoWs`. |
| `asteroids` | list[AsteroidApproach] | Непустой список для пригодного набора. |
| `schema_version` | `str` | Версия формата кэша, например `1`. |

## ResultState

| State | Meaning | UI |
|---|---|---|
| `idle` | Запрос не выполняется | Кнопка доступна только при валидной дате. |
| `loading` | Идёт worker-запрос | Кнопка отключена, виден индикатор. |
| `fresh` | Показан свежий непустой NASA-ответ | Источник и время ответа. |
| `cache` | Показан последний пригодный кэш | Явное предупреждение и раздельные даты. |
| `error_no_data` | Ошибка/пустой ответ без кэша | Понятное сообщение, пустая таблица. |
| `invalid_input` | Дата не прошла проверку | Сообщение, сетевой вызов отсутствует. |

## State transitions

`idle -> invalid_input` при изменении неверной даты; `idle -> loading` после
нажатия с валидной датой; `loading -> fresh` при успешном непустом разборе;
`loading -> cache` при ошибке/пустом ответе и пригодном кэше;
`loading -> error_no_data` при ошибке/пустом ответе без кэша.

## Cache envelope

```text
{
  "schema_version": "1",
  "data_date": "YYYY-MM-DD",
  "received_at": "ISO-8601 datetime",
  "source": "NASA NeoWs",
  "asteroids": [normalized asteroid records]
}
```

Чтение принимает только известную версию, корректную дату/время, непустой
массив и валидные записи. Записи без названия или сближения за дату отбрасываются
с предупреждением; если после этого список пуст, набор непригоден. Числовые
пропуски и отсутствующий hazard-флаг сохраняются как `None` и отображаются как
«Нет данных» и «Неизвестно». JSON, структура или значения, не соответствующие
модели, дают ошибку чтения кэша; пригодный предыдущий файл не перезаписывается.
