# Tonnel Api Documentation

Это неофициальная документация по апи [тонеля](https://tonnel-gift.vercel.app), самой популярной площадке для покупки и продаже телеграм подарков на данный момент.

###### для ленивых рабочий код на питоне [**тут**](#рабочий-код-на-питоне)

### При заходе на сайт, совершается пост запрос по этому юрл:

<code>https://gifts2.tonnel.network/api/pageGifts</code>

### По нему загружаются подарки, выставленные на тонеле!

#### Но обычного пост запроса не хватит, нужно передать данные для фильтрации подарков в json формате:

```python
json_data = {
    'page': 1,
    'limit': 1,
    'sort': '{"message_post_time":-1,"gift_id":-1}',
    'filter': '{"price":{"$exists":true},"refunded":{"$ne":true},"buyer":{"$exists":false},"export_at":{"$exists":true},"gift_name":"Desk Calendar","model":"Deadline (0.2%)","asset":"TON"}',
    'price_range': None,
    'user_auth': '',
}
```

### Разберём по порядку:

**page** — Страница с подарками, которую вы хотите увидеть в ответе  
**limit** — Максимальное количество подарков, которое может быть на 1 странице. При значении выше 30 будет возвращать ошибку `{'error': 'limit is too big'}`  
**sort** — Тип сортировки всех этих подарков (их всего 10) 👇

> !!! Для каждого типа сортировки нужно ставить значение -1 либо 1. -1 означает что результат будет в убывании, 1 в возрастании

- **message_post_time** — По времени выставления
- **export_at** — По времени выставления **!! message_post_time со значением 1 даст тот же результат, что и export_at со значением -1, и наоборот !!**
- **price** — По стоимости
- **gift_num** — По номеру подарка (plushpepe-**1**)
- **modelRarity | backdropRarity | symbolRarity** — По процентной редкости аттрибута (модели/фона/узора)
- **rarity** — Неизвестно
- **gift_id** — Неизвестно

**filter** — Фильтр для поиска конкретных подарков\
Подстроку `"price":{"$exists":true},"refunded":{"$ne":true},"buyer":{"$exists":false},"export_at":{"$exists":true},` оставляете как есть, после нее добавляете **необязательные параметры для поиска нужного подарка**:

- gift_name - Название
- model - Модель
- backdrop - Фон
- pattern - Узор

1. Для модели, фона и узора значение нужно указывать в формате regex, чтобы дополнительно не искать процентную редкость. Пример: `"gift_name":"Lunar Snake", "model": {"$regex": f"^Albino \\("}`
2. Если же вам лень писать фильтр для регекса или у вас откуда-то есть процентная редкость аттрибута, то указываете: `"model": "Albino (1.5%)"`
3. Для поиска по нескольким объектам одного аттрибута(хотите увидеть 5 моделей/фонов), нужно указывать значение в формате списка: `"model":{'$in':["Synthwave (0.3%)", "Albino (1.5%)"]}`\
- Альтернативный вариант для regex: `"model": {'$regex':f"^({'|'.join(["Synthwave", "Albino"])})"}`\
   В самом конце фильтра указываете последний фильтр: `"asset":"TON"`, соответственно отвечающий за валюту в которой будут выданы подарки. Может быть TONNEL и USDT, но смысла от такого фильтра я не вижу, результаты будут милипиздрические.

**price_range** - Диапозон цены в формате списка: `'price_range': [1,1000]`. Оставьте None для отключения фильтра по цене.  
**user_auth** - Обязательное поле, в значении пустая строка означает что вы никак не залогинились на сайте и просто просматриваете подарки.

### И самое главное - библиотека для запросов

> Код будет на питоне, кому надо на другом яп сами подгоняйте

Поскольку тонель делал не коля из 3Б, обычная либка по типу requests/httpx не подойдет. Отлично работает с _curl_cffi_

### Рабочий код на питоне:

```python
import json
from curl_cffi import requests


sort_data = {
    'message_post_time': -1,
    'gift_id': -1
}

filter_data = {
    "price":{"$exists": True},
    "refunded":{"$ne":True},
    "buyer":{"$exists": False},
    "export_at":{"$exists": True},
    "gift_name":"Lunar Snake",

    "model": "Albino (1.5%)",
    # Либо юзаем regex если нет возможности взять % редкость
    # "model":{"$regex": "^Albino \\("},

    # Запрос нескольких моделей:

    # "model":{'$in':["Synthwave (0.3%)", "Albino (1.5%)"]},
    # Либо юзаем regex если нет возможности взять % редкость
    # "model": {'$regex':f"^({'|'.join(["Synthwave", "Albino"])})"},
    "asset":"TON"
}

json_data = {
    'page': 1,
    'limit': 5,
    'sort': json.dumps(sort_data),
    'filter': json.dumps(filter_data),
    'price_range': None,
    'user_auth': '',
}



response = requests.post('https://gifts2.tonnel.network/api/pageGifts', json=json_data, impersonate="chrome")
print(response.json())
# Код выведет первую страницу с 5 самыми новыми подарками с именем Lunar Snake, с моделью Albino


# Хедеры необязательны, но на всякий случай:
# headers = {
#     'origin': 'https://market.tonnel.network',
#     'referer': 'https://market.tonnel.network/',
#     'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36',
# }
```

В общем как-то так, за последтсвия использования информации вами из этой статьи ответственность на мне не несётся.\
тг для связи - [@bxxst](t.me/bxxst)


P. S. Можете глянуть [этот репозиторий](https://github.com/bleach-hub/tonnelmp) уже с полноценной либкой по апи тоннеля (не моей)
