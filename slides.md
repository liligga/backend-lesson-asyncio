---
theme: seriph
background: https://cover.sli.dev
title: Асинхронное программирование на Python
highlighter: shiki
transition: slide-left
mdc: true
---

## Асинхронное программирование на Python

---

### Асинхронность

Существуют задачи, которые связаны с вводом-выводом, иногда
такие задачи останавливают(блокируют) выполнение остального кода программы.

<v-click>ввод-вывод:</v-click>

<v-clicks>

- работа с БД
- работа с сетью
- работа с файлами 

</v-clicks>

---
layout: center
---

```python
import requests
from time import perf_counter

URL = 'https://www.mashina.kg/search/all/?page='
start = perf_counter()
for i in range(1, 11):
    response = requests.get(URL + str(i))
    print("Page: ", i, "Status: ", response.status_code)

print("Time: ", perf_counter() - start)
```


---
layout: center
---

### Асинхронность

```mermaid {scale: 0.8}
sequenceDiagram
    participant Браузер
    participant Сервер
    Браузер ->> Сервер: Запрос: покажи мне страницу с объявлениями #1
    Сервер -->> Сервер: Обработка запроса
    Сервер ->> Браузер: Ответ
    Браузер ->> Сервер: Запрос: покажи мне страницу с объявлениями #2
    Сервер -->> Сервер: Обработка запроса
    Сервер ->> Браузер: Ответ
```

вместо браузера можно поставить скрипт, который делает запрос

---
layout: center
---

### Асинхронность

```mermaid {scale: 1}
sequenceDiagram
    participant Функция
    participant База данных
    Функция ->>+ База данных: Запрос: дай мне список товаров из таблицы
    База данных -->>- База данных: Обработка запроса
    База данных ->> Функция: Ответ
```

---
layout: center
---

### Минусы:

<v-clicks>

- обработка запроса(на сервере или в БД) может занять неопределенное время
- во время ожидания ответа наша программа просто ничего не делает

</v-clicks>

---
layout: center
---

```python
import asyncio
from time import perf_counter
import httpx

URL = 'https://www.mashina.kg/search/all/?page='

async def get_page(url: str, client: httpx.AsyncClient) -> tuple[int, str]:
    response = await client.get(url)
    return response.status_code, url

async def parse_data():
    async with httpx.AsyncClient() as client:
        tasks = []
        for i in range(1, 11):
            tasks.append(
                asyncio.create_task(
                    get_page(URL + str(i), client)
                )
            )

        results = await asyncio.gather(*tasks)
        for status_code, url in results:
            print("Page: ", url, "Status: ", status_code)

if __name__ == '__main__':
    start = perf_counter()
    asyncio.run(parse_data())
    print("Time: ", perf_counter() - start)
```

