# Домашнє завдання 2

## Завдання 1

Команда:

```bash
ss -tuln
```

Результат:

```
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:Port
udp   UNCONN 0      0         127.0.0.54:53         0.0.0.0:*
udp   UNCONN 0      0      127.0.0.53%lo:53         0.0.0.0:*
udp   UNCONN 0      0          127.0.0.1:323        0.0.0.0:*
udp   UNCONN 0      0          127.0.0.1:323        0.0.0.0:*
udp   UNCONN 0      0              [::1]:323           [::]:*
udp   UNCONN 0      0              [::1]:323           [::]:*
tcp   LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*
tcp   LISTEN 0      4096       127.0.0.1:35097      0.0.0.0:*
tcp   LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*
tcp   LISTEN 0      4096      127.0.0.54:53         0.0.0.0:*
tcp   LISTEN 0      4096            [::]:22            [::]:*
```

Ключі: `-t` - TCP, `-u` - UDP, `-l` - порти в режимі очікування, `-n` - номери портів.

| Протокол | Порт | Стан | Сервіс |
|---|---|---|---|
| TCP | 22 | LISTEN | SSH, доступний з мережі |
| TCP/UDP | 53 | LISTEN/UNCONN | Локальний DNS |
| UDP | 323 | UNCONN | Синхронізація часу |
| TCP | 35097 | LISTEN | `containerd`, лише локально |

`LISTEN` - TCP чекає на з'єднання `UNCONN` - UDP без з'єднання. `127.x.x.x` локальний доступ.

---

## Завдання 2

Команда з `sudo` - шоб було видно процеси:

```bash
sudo ss -tulpn
```

Результат:

```
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
udp   UNCONN 0      0         127.0.0.54:53         0.0.0.0:*    users:(("systemd-resolve",pid=81,fd=18))
udp   UNCONN 0      0      127.0.0.53%lo:53         0.0.0.0:*    users:(("systemd-resolve",pid=81,fd=16))
udp   UNCONN 0      0          127.0.0.1:323        0.0.0.0:*    users:(("chronyd",pid=294,fd=4))
udp   UNCONN 0      0          127.0.0.1:323        0.0.0.0:*
udp   UNCONN 0      0              [::1]:323           [::]:*
udp   UNCONN 0      0              [::1]:323           [::]:*    users:(("chronyd",pid=294,fd=5))
tcp   LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*    users:(("systemd-resolve",pid=81,fd=17))
tcp   LISTEN 0      4096       127.0.0.1:35097      0.0.0.0:*    users:(("containerd",pid=270,fd=15))
tcp   LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*    users:(("systemd",pid=1,fd=80))
tcp   LISTEN 0      4096      127.0.0.54:53         0.0.0.0:*    users:(("systemd-resolve",pid=81,fd=19))
tcp   LISTEN 0      4096            [::]:22            [::]:*    users:(("systemd",pid=1,fd=81))
```

`-p` процес

| Процес | PID | Порт | Для чого |
|---|---:|---|---|
| `systemd-resolve` | 81 | 53 TCP/UDP | DNS |
| `chronyd` | 294 | 323 UDP | Час |
| `containerd` | 270 | 35097 TCP | Контейнери |
| `systemd` | 1 | 22 TCP | Запускає SSH після підключення |

---

## Завдання 3

Команда:

```bash
nc -vz google.com 80
```

Результат:

```
Connection to google.com (142.251.98.100) 80 port [tcp/http] succeeded!
```

Успішне по TCP з портом 80 http. `-z` порт, `-v` результат.

---

## Завдання 4

Команди і результати:

```bash
nc -vz 8.8.8.8 53
```

```
Connection to 8.8.8.8 53 port [tcp/domain] succeeded!
```

```bash
nc -u -vz 8.8.8.8 53
```

```
Connection to 8.8.8.8 53 port [udp/domain] succeeded!
```

Для TCP `succeeded` це з'єднання встановлено. Для UDP це пакет надіслано тіко.

DNS зазвичай використовує UDP, а TCP для великих відповідей

---

## Завдання 5

У WSL не було `nslookup` і `dig`, тому:

```bash
sudo apt install dnsutils
```

Команда:

```bash
nslookup github.com
```

Результат:

```
;; Got recursion not available from 172.20.112.1
Server:		172.20.112.1
Address:	172.20.112.1#53

Non-authoritative answer:
Name:	github.com
Address: 140.82.121.4
```

через `dig`:

```bash
dig github.com
```

```
;; ANSWER SECTION:
github.com.		0	IN	A	140.82.121.4

;; Query time: 4 msec
;; SERVER: 172.20.112.1#53(172.20.112.1) (UDP)
```

Домен `github.com` має IP `140.82.121.4`. Запит DNS-сервер `172.20.112.1` , шлюз WSL2.

`Non-authoritative answer` - відповідь із кешу
Запис `A` - IPv4-адреса.
