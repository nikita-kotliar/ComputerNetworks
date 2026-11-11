# Домашнє завдання 3: Веб-протоколи та прикладний рівень

Команди виконував у WSL2 (Ubuntu).

## Завдання 1. HTTP-заголовки та статус-коди

```bash
curl -I http://example.com
```

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Server: cloudflare
...
```

- Статус-код: `200 OK` - сторінка віддалась нормально.
- Location: немає, бо example.com не робить редирект з HTTP на HTTPS.
- Протокол: `HTTP/1.1`.

Щоб побачити редирект, перевірив ще github.com:

```bash
curl -I http://github.com
```

```
HTTP/1.1 301 Moved Permanently
Content-Length: 0
Location: https://github.com/
```

- Статус-код: `301 Moved Permanently` - сторінка назавжди переїхала.
- Location: `https://github.com/` - GitHub не працює по звичайному HTTP і відправляє на HTTPS.
- Протокол: `HTTP/1.1`.

---

## Завдання 2. Методи HTTP

Зробив маленьке тестове API на Python і запустив локально на порту 3000. GET повертає список постів, POST додає новий:

```python
def do_GET(self):
    self.send(200, posts)

def do_POST(self):
    item = json.loads(self.rfile.read(n))
    item["id"] = len(posts) + 1
    posts.append(item)
    self.send(201, item)
```

POST:

```bash
curl -i -X POST -H "Content-Type: application/json" -d '{"title":"test","author":"nikita"}' http://127.0.0.1:3000/posts
```

```
HTTP/1.0 201 Created
Content-Type: application/json

{"title": "test", "author": "nikita", "id": 2}
```

GET:

```bash
curl -i http://127.0.0.1:3000/posts
```

```
HTTP/1.0 200 OK

[{"id": 1, "title": "hello"}, {"title": "test", "author": "nikita", "id": 2}]
```

Результат: `201 Created` - ресурс створився, сервер повернув його разом з новим `id`.

Різниця між GET і POST:

| | GET | POST |
|---|---|---|
| Для чого | Отримати дані | Відправити/створити дані |
| Де дані | В URL (`?id=1`) | В тілі запиту |
| Кешування | Кешується браузером і проксі | Зазвичай не кешується |
| Повтор запиту | Нічого не змінює | Кожен раз створює новий запис |

---

## Завдання 3. TLS-сертифікат

```bash
openssl s_client -connect github.com:443
```

```
subject=CN=github.com
issuer=C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication CA DV E36
New, TLSv1.3, Cipher is TLS_AES_128_GCM_SHA256
Protocol: TLSv1.3
```

- Issuer: Sectigo Limited (центр сертифікації з Великобританії).
- Протокол: `TLSv1.3`.
- Шифр: `TLS_AES_128_GCM_SHA256` - дані шифруються AES з 128-бітним ключем у режимі GCM, а SHA256 використовується для перевірки цілісності.

---

## Завдання 4. Безпека Cookies

```bash
curl -I https://github.com
```

```
set-cookie: _gh_sess=4kocfjh1x...; path=/; HttpOnly; secure; SameSite=Lax
set-cookie: _octo=GH1.1.1275965893.1794418921; domain=.github.com; path=/; secure; SameSite=Lax
set-cookie: logged_in=no; domain=.github.com; path=/; HttpOnly; secure; SameSite=Lax
```

| Cookie | HttpOnly | Secure | SameSite |
|---|---|---|---|
| `_gh_sess` (сесія) | так | так | Lax |
| `_octo` (аналітика) | ні | так | Lax |
| `logged_in` | так | так | Lax |

**HttpOnly** захищає від XSS. Якщо зловмисник вставить на сторінку свій JavaScript, він не зможе прочитати cookie через `document.cookie` і вкрасти сесію. Тому сесійна `_gh_sess` має HttpOnly, а `_octo` - ні, бо її читає скрипт аналітики.

**Secure** означає, що cookie відправляється тільки по HTTPS. По звичайному HTTP її можна перехопити в тій же Wi-Fi мережі, а так вона завжди йде зашифрованою.

`SameSite=Lax` додатково захищає від CSRF: з чужого сайту cookie не відправляться, крім звичайного переходу за посиланням.

---

## Завдання 5. Архітектура для онлайн-біржі

**Сценарій А: графіки цін у браузері - WebSocket.**

Ціни міняються щосекунди, тому сервер має сам відправляти їх клієнту. Якщо робити це через REST, браузер мусив би постійно питати "що нового?", і більшість запитів були б даремні. WebSocket відкриває одне з'єднання, яке живе весь час, і сервер просто штовхає нові ціни, коли вони з'являються. Затримка мінімальна, і браузери це підтримують з коробки.

**Сценарій Б: білінг <-> облік акцій - gRPC.**

Тут обидві сторони - наші сервіси, браузер не потрібен. gRPC працює через HTTP/2 і передає дані в бінарному форматі Protobuf, а це менше і швидше, ніж JSON у REST. Ще структура повідомлень описується в `.proto` файлі, тому обидва сервіси точно знають формат даних і помилки ловляться ще при компіляції. Плюс gRPC підтримує стрімінг, якщо треба буде передавати потік операцій.
