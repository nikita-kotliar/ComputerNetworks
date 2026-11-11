# Домашнє завдання 3

## Завдання 1
```bash
curl -I http://example.com
```

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Server: cloudflare
...
```

Статус-код: `200 OK` - сторінка віддалась нормально
Location: немає, бо немає редірект з HTTP на HTTPS
Протокол: `HTTP/1.1`

Щоб редірект був, перевірив на github.com:

```bash
curl -I http://github.com
```

```
HTTP/1.1 301 Moved Permanently
Content-Length: 0
Location: https://github.com/
```

Статус-код: `301 Moved Permanently` - сторінка назавжди переїхала
Location: `https://github.com/` -  відправляє на HTTPS
Протокол: `HTTP/1.1`


## Завдання 2

 маленьке тестове API на Python і на порту 3000. GET - список постів, POST -новий

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

Результат: `201 Created` - створився, сервер повернув його з `id`.

Різниця між GET і POST:

| | GET | POST |
|---|---|---|
| Для чого | Отримати дані | відправити/створити дані |
| Де дані | В URL (`?id=1`) | В тілі запиту |
| Кешування | браузером і проксі | не кешується |
| Повтор запиту | Ннічого не змінює | кожен раз новий запис |


## Завдання 3

```bash
openssl s_client -connect github.com:443
```

```
subject=CN=github.com
issuer=C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication CA DV E36
New, TLSv1.3, Cipher is TLS_AES_128_GCM_SHA256
Protocol: TLSv1.3
```

Issuer: сертифікації з британії
Протокол: `TLSv1.3`
Шифр: `TLS_AES_128_GCM_SHA256` - дані шифруються AES з ключем у режимі GCM, SHA256  для перевірки цілісності


## Завдання 4

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
| `_gh_sess` | так | так | Lax |
| `_octo`  | ні | так | Lax |
| `logged_in` | так | так | Lax |

**HttpOnly** захищає від XSS. Якщо зловмисник вставить на сторінку свій Js, він не зможе прочитати cookie  і вкрасти сесію. сесійна `_gh_sess` має HttpOnly, а `_octo` ні, бо її читає скрипт аналітики

**Secure** означає, що cookie відправляється тільки по https. по звичайному http її можна перехопити в тій же мережі

`SameSite=Lax` додатково захищає від CSRF: з чужого сайту cookie не відправляться, тільки но звичайного переходу за посиланням



## Завдання 5

**Сценарій А: графіки цін у браузері - WebSocket.**

Ціни міняються кожну секунду тому сервер має сам відправляти їх клієнту. якщо робити це через REST, браузер мав би питати щось нове, і більшість запитів були б без сенсу. WebSocket робить одне з'єднання, яке живе весь час, і сервер просто кидає нові ціни, коли вони з'являються . мінімальна затримка

**Сценарій Б: білінг <-> облік акцій - gRPC.**

Тут дві сторони - сервіси, браузер не треба. gRPC працює через HTTP/2 і передає дані в бінарному форматі, а це менше і швидше, ніж JSON в REST. Ще структура  описується в `.proto` файлі, тому обидва сервіси точно знають формат даних і помилки ловляться ще при компіляції
