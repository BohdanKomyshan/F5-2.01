<!--
Бланк звіту до лабораторної роботи «TLS та сертифікати».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота 2. TLS та сертифікати

| Поле | Значення |
|---|---|
| ПІБ |Комишан Богдан Васильович |
| Група |2.01 F5 |
| Ідентифікатор |cs-201-11 |
| Дата виконання | |
| Середовище | GitHub Codespaces|
| Версія OpenSSL | OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024) |

## Параметри варіанта

<!-- Переписати з картки stand/cards/<id>.md. -->

| Параметр | Значення |
|---|---|
| Віддалений сервер |cdn.cs-201-11.lab |
| Імена для сертифіката |www.cs-201-11.lab, status.cs-201-11.lab |
| Ваш сервер (IP) |10.53.1.1 |

<!-- Якщо стенд перезапускався під час роботи, вказати, після якого завдання. -->

## Завдання 1. HTTP і HTTPS у дампі трафіку

Вивід кроку 1.1:

```text
[1] 37
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
server: web
Host: cdn.cs-201-11.lab
SNI: 
protocol: HTTP/1.1 
client: 10.53.1.1
10 packets captured
10 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 80' > t1-http.txt
07:02:26.497255 IP 10.53.1.1.45306 > 10.53.0.80.80: Flags [P.], seq 1:81, ack 1, win 502, options [nop,nop,TS val 2740104880 ecr 4095892667], length 80: HTTP: GET / HTTP/1.1
.R..."\.GET / HTTP/1.1
Host: cdn.cs-201-11.lab
server: web
Host: cdn.cs-201-11.lab
```

Вивід кроку 1.2:

```text
[1] 43
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
server: web
Host: cdn.cs-201-11.lab
SNI: cdn.cs-201-11.lab
protocol: HTTP/2.0 TLSv1.3
client: 10.53.1.1
21 packets captured
21 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t1-https.txt
0
1
```

| Що шукали в дампі | HTTP | HTTPS |
|---|---|---|
| рядок запиту `GET /` |GET / HTTP/1.1 | 0 |
| заголовок `Host` |Host: cdn.cs-201-11.lab | 0 |
| тіло відповіді (`server: web`) |server: web | 0|
| ім'я віддаленого сервера (кількість входжень) |1 |1 |

1. У якому повідомленні і в якому полі передається ім'я сервера, яке видно в дампі HTTPS?
Ім'я сервера передається у першому повідомленні рукостискання TLS — ClientHello (яке ще не зашифроване), у розширенні SNI (Server Name Indication), поле server_name.
2. Чому рядок `SNI:` у відповіді сервера порожній для HTTP і заповнений для HTTPS?
У звичайному HTTP протокол TLS не використовується, тому рукостискання не відбувається і розширення SNI відсутнє (ім'я сервера передається лише в HTTP-заголовку Host:). У HTTPS перед обміном HTTP-даними встановлюється TLS-з'єднання, під час якого клієнт передає ім'я хоста в розширенні SNI, щоб сервер знав, який саме сертифікат надати.
3. Рядок `protocol:` показує HTTP/1.1 для запиту по HTTP і HTTP/2.0 для запиту по HTTPS, хоча обидва запити виконано однією командою `curl` без додаткових ключів. Що визначило версію HTTP у другому випадку?
Версію протоколу визначило розширення ALPN (Application-Layer Protocol Negotiation) під час рукостискання TLS. Клієнт (curl) у ClientHello запропонував список підтримуваних протоколів (h2, http/1.1), а сервер обрав і підтвердив використання h2 (HTTP/2.0) (у звичайному HTTP без TLS узгодження через ALPN неможливе, тому використовується HTTP/1.1).
## Завдання 2. Рукостискання по записах

Вивід кроку 2.1 (TLS 1.3):

```text
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=328
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=118
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    EncryptedExtensions, Length=15
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Certificate, Length=983
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    CertificateVerify, Length=74
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Alert (21)
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
ALPN protocol: h2
Verify return code: 0 (ok)
```

Вивід кроку 2.2 (TLS 1.2):

```text
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=223
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=74
Received Record
  Content Type = Handshake (22)
    Certificate, Length=978
Received Record
  Content Type = Handshake (22)
    ServerKeyExchange, Length=111
Received Record
  Content Type = Handshake (22)
    ServerHelloDone, Length=0
Sent Record
  Content Type = Handshake (22)
    ClientKeyExchange, Length=33
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = Handshake (22)
    NewSessionTicket, Length=214
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = Handshake (22)
    Finished, Length=12
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES256-GCM-SHA384
ALPN protocol: h2
Received Record
  Content Type = ApplicationData (23)
Sent Record
  Content Type = Alert (21)
```

<!-- Один рядок на кожен запис з виводу 2.1, у тому самому порядку. Напрям: К→С або С→К. -->

| № | Напрям | `Content Type` | `Inner Content Type` | Повідомлення | `Length` |
| --- | --- | --- | --- | --- | --- |
| 1 | К→С | Handshake (22) | — | ClientHello | 328 |
| 2 | С→К | Handshake (22) | — | ServerHello | 118 |
| 3 | С→К | ChangeCipherSpec (20) | — | — | — |
| 4 | С→К | ApplicationData (23) | Handshake (22) | EncryptedExtensions | 15 |
| 5 | С→К | ApplicationData (23) | Handshake (22) | Certificate | 983 |
| 6 | С→К | ApplicationData (23) | Handshake (22) | CertificateVerify | 74 |
| 7 | С→К | ApplicationData (23) | Handshake (22) | Finished | 48 |
| 8 | К→С | ChangeCipherSpec (20) | — | — | — |
| 9 | К→С | ApplicationData (23) | Handshake (22) | Finished | 48 |
| 10 | К→С | ApplicationData (23) | Alert (21) | — | — |

Вивід кроків 2.3 і 2.4:

```text
2.3
 extension_type=server_name(0), length=22
          0000 - 00 14 00 00 11 63 64 6e-2e 63 73 2d 32 30 31   .....cdn.cs-201
          000f - 2d 31 31 2e 6c 61 62                           -11.lab
        extension_type=ec_point_formats(11), length=4
[1] 65
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22 packets captured
22 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 4 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-1.3.txt
TLS 1.3: 0
[1] 71
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21 packets captured
21 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 4 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-1.2.txt
TLS 1.2: 2


```

| Показник| TLS 1.3 | TLS 1.2 |
| ----- | ----- | ----- |
| Довжина ClientHello, байт | 328 | 223 |
| Номер першого запису, тип якого спостерігач не бачить | 4 | 12 |
| Кількість RTT від ClientHello до першого HTTP-запиту | 1 | 2 |
| Чи видно сертифікат у дампі (результат 2.4) | ні (0) | так (2) |
| Обраний протокол ALPN | h2 | h2 |

1. За якими рядками виводу 2.1 визначено, з якого запису починається шифрування? Що бачить спостерігач замість типу кожного наступного запису?
Шифрування починається з 4-го запису (після ServerHello та ChangeCipherSpec), що визначається за появою у виводі 2.1 пари рядків Content Type = ApplicationData (23) та Inner Content Type = Handshake (22) (повідомлення EncryptedExtensions). Замість справжнього типу кожного наступного запису (Handshake або Alert) зовнішній спостерігач бачить лише ApplicationData (23).

2. Чому ClientHello у TLS 1.3 довший, ніж у TLS 1.2? Назвіть розширення, яке за рахунок цього скорочує рукостискання на один RTT.
У TLS 1.3 повідомлення ClientHello довше (328 байт проти 223 байт у TLS 1.2), оскільки клієнт одразу передає в ньому свої ефемерні відкриті ключі для обміну ключами, не чекаючи відповіді від сервера. Це забезпечує розширення key_share, завдяки якому обидві сторони обчислюють спільний секрет уже після першого обміну повідомленнями та скорочують рукостискання на 1 RTT.

3. Довжина розширення `server_name` у виводі 2.3 становить 17 байт для імені з 12 символів. З яких частин складається це значення і яким воно буде для вашого імені?
Довжина розширення server_name складається з 5 службових байтів  та байтів ASCII-рядка імені сервера. Для імені варіанта cdn.cs-201-11.lab, яке містить 17 символів , загальна довжина розширення становить 5 + 17 = 22 байти (що підтверджується рядком extension_type=server_name(0), length=22 у виводі 2.3).
4. Навіщо в TLS 1.3 передається запис ChangeCipherSpec, якщо в рукостисканні він нічого не змінює?
Запис ChangeCipherSpec передається виключно для зворотної сумісності зі старими проміжними мережевими пристроями, які очікують побачити ChangeCipherSpec перед появою записів ApplicationData і можуть розірвати з'єднання за його відсутності.
## Завдання 3. Сертифікат для власного сервера

Вивід кроків 3.1 і 3.2:

```text
збережено /lab/tls/acme-root.crt
subject=CN = Pebble Root CA 20f890
notBefore=Nov 17 17:21:41 2026 GMT
notAfter=Nov 17 17:21:41 2056 GMT
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

Вивід кроку 3.3 (останні рядки `lego` і рядки журналу nginx):

```text
026/11/17 18:16:35 [INFO] [status.cs-201-11.lab] acme: Trying to solve HTTP-01
2026/11/17 18:16:39 [INFO] [status.cs-201-11.lab] The server validated our request
2026/11/17 18:16:39 [INFO] [www.cs-201-11.lab] acme: Trying to solve HTTP-01
2026/11/17 18:16:45 [INFO] [www.cs-201-11.lab] The server validated our request
2026/11/17 18:16:45 [INFO] [www.cs-201-11.lab, status.cs-201-11.lab] acme: Validations succeeded; requesting certificates
2026/11/17 18:16:45 [INFO] Wait for certificate [timeout: 30s, interval: 500ms]
2026/11/17 18:16:45 [INFO] [www.cs-201-11.lab] Server responded with a certificate.
10.53.0.10 - - [17/Nov/2026:18:16:35 +0000] "GET /.well-known/acme-challenge/JNJLk_glyqJ4paHqltcEpF4MEJiIvjaPNnW7B4aSs9A HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [17/Nov/2026:18:16:35 +0000] "GET /.well-known/acme-challenge/JNJLk_glyqJ4paHqltcEpF4MEJiIvjaPNnW7B4aSs9A HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [17/Nov/2026:18:16:39 +0000] "GET /.well-known/acme-challenge/JiPZ4Uq5UujEoDl7XsASGiuwzHGDeSB99eHRT4o3NQY HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [17/Nov/2026:18:16:39 +0000] "GET /.well-known/acme-challenge/JiPZ4Uq5UujEoDl7XsASGiuwzHGDeSB99eHRT4o3NQY HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [17/Nov/2026:18:16:39 +0000] "GET /.well-known/acme-challenge/JiPZ4Uq5UujEoDl7XsASGiuwzHGDeSB99eHRT4o3NQY HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
```

Вивід кроку 3.4:

```text
subject=CN = www.cs-201-11.lab
issuer=CN = Pebble Intermediate CA 593abd
notBefore=Nov 17 18:16:45 2026 GMT
notAfter=Nov 17 18:16:44 2031 GMT
serial=6DFA2850787BD2DE
X509v3 Subject Alternative Name: 
    DNS:www.cs-201-11.lab, DNS:status.cs-201-11.lab
subject=CN = Pebble Intermediate CA 593abd
issuer=CN = Pebble Root CA 20f890
```

Вивід кроку 3.5:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/17 18:44:52 [notice] 221#221: signal process started
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
Certificate chain
 0 s:CN = www.cs-201-11.lab
   i:CN = Pebble Intermediate CA 593abd
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 17 18:16:45 2026 GMT; NotAfter: Nov 17 18:16:44 2031 GMT
---
Verify return code: 21 (unable to verify the first certificate)
```

Вивід кроку 3.6:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/17 18:48:41 [notice] 243#243: signal process started
server: cs-201-11 (https)
Host: www.cs-201-11.lab
SNI: www.cs-201-11.lab
protocol: TLSv1.3
Certificate chain
 0 s:CN = www.cs-201-11.lab
   i:CN = Pebble Intermediate CA 593abd
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 17 18:16:45 2026 GMT; NotAfter: Nov 17 18:16:44 2031 GMT
 1 s:CN = Pebble Intermediate CA 593abd
   i:CN = Pebble Root CA 20f890
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 17 17:21:41 2026 GMT; NotAfter: Nov 17 17:21:41 2056 GMT
---
Verify return code: 0 (ok)
server: cs-201-11 (https)
Host: status.cs-201-11.lab
SNI: status.cs-201-11.lab
protocol: TLSv1.3
```

| Параметр | Значення |
|---|---|
| Видавець сертифіката сервера | |
| Видавець сертифіката видавця | |
| Імена в SAN | |
| Строк дії (від – до) | |
| Тип ключа сервера і алгоритм підпису | |
| Кількість сертифікатів, які надсилає сервер, у кроці 3.5 і в кроці 3.6 | |

1. Хто і як перевірив, що ви контролюєте обидва імена? Вкажіть рядки виводу, які це підтверджують.

2. Чому в кроці 3.5 перевірка не пройшла, хоча корінь центру сертифікації вказано в `--cacert`, а сертифікат виданий правильно?

3. У виводі ланцюжка для сертифіката сервера вказано `id-ecPublicKey` і `RSA-SHA256`. Що означає кожне з цих значень і чому вони не суперечать одне одному?

4. Що станеться з перевіркою вашого сервера після перезапуску стенда і чому? Що в реальній інфраструктурі відповідає цій ситуації?

## Завдання 4. Діагностика несправностей

Вивід кроку 4.1:

```text

```

| Сервер | Помилка `curl` | `Verify return code` | Яка перевірка не пройшла | Рядок виводу, що це доводить | Що має змінити власник сервера |
|---|---|---|---|---|---|
| f1 | | | | | |
| f2 | | | | | |
| f3 | | | | | |
| f4 | | | | | |
| f5 | | | | | |

1. Для сервера з помилкою `unable to get local issuer certificate` визначте, яка це несправність: сервер не надсилає проміжний сертифікат чи сертифікат виданий центром, якому клієнт не довіряє. Чим ці дві несправності відрізняються у виводі `openssl s_client`?

2. Якщо серед ваших серверів є сервер з невідповідним ім'ям, чому `openssl s_client` повертає для нього `Verify return code: 0 (ok)`, а `curl` повідомляє про помилку? Якщо такого сервера немає, дайте відповідь для загального випадку.

3. Які з п'яти серверів відповіли б на запит `curl -k`? Що при цьому втрачається порівняно з `--cacert`?

## Завдання 5. Ціна нового з'єднання

**Прогноз** (записати до вимірювання). Скільки RTT мине до завершення кожного етапу:

| Етап | TLS 1.3, нове з'єднання | TLS 1.2, нове з'єднання | Повторне використання з'єднання |
|---|---|---|---|
| `time_connect` | | | |
| `time_appconnect` | | | |
| `time_starttransfer` | | | |

Вивід кроків 5.2 і 5.3:

```text

```

<!-- Значення в мілісекундах. Для нових з'єднань — медіана трьох вимірювань. -->

| Запит | `namelookup` | `connect` | `appconnect` | `starttransfer` | `total` | `num_connects` |
|---|---|---|---|---|---|---|
| TLS 1.3, нове | | | | | | |
| TLS 1.2, нове | | | | | | |
| 5.3, перший запит | | | | | | |
| 5.3, другий запит | | | | | | |
| 5.3, третій запит | | | | | | |

RTT, визначений вимірюванням, і з якого значення його визначено:

1. Виразіть кожне виміряне значення через RTT. Де вимірювання відрізняється від цілого числа RTT і чим це пояснюється?

2. Чому `time_namelookup` не близький до нуля, хоча резолвер уже має ім'я в кеші? За яких умов він був би приблизно вдвічі більшим?

3. Сервіс виконує 200 послідовних запитів до цього сервера. Розрахуйте, скільки часу займе лише очікування мережі, якщо кожен запит встановлює нове з'єднання TLS 1.3, нове з'єднання TLS 1.2 і якщо всі запити йдуть через одне з'єднання.

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено і що з'ясувати не вдалося:

## Декларація використання ШІ

Використовувалися інструменти ШІ: <!-- так / ні -->

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | | | |
