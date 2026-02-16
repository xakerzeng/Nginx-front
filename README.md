# Nginx-front

Install Xray 

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

Install Nginx

```
apt install -y gnupg2 ca-certificates lsb-release ubuntu-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" > /etc/apt/sources.list.d/nginx.list && echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && echo -e "[Service]\nExecStartPost=/bin/sleep 0.1" > /etc/systemd/system/nginx.service.d/override.conf && systemctl daemon-reload
```

Uninstall Nginx

```
systemctl stop nginx && apt purge -y nginx && rm -r /etc/systemd/system/nginx.service.d/
```

project

program 	/usr/local/bin/xray

Configuration 	/usr/local/etc/xray/config.json

geoip 	/usr/local/share/xray/geoip.dat

geosite 	/usr/local/share/xray/geosite.dat

Restart 	systemctl restart xray

state 	systemctl status xray

View logs 	journalctl -u xray -o cat -e

Real-time logs 	journalctl -u xray -o cat -f



```
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",
            "port": 8001,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "chika",
                        "flow": "xtls-rprx-vision"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "dest": "8002",
                    "xver": 1,
                    "serverNames": [
                        "example.com"
                    ],
                    "privateKey": "",
                    "shortIds": [
                        ""
                    ]
                },
                "tcpSettings": {
                    "acceptProxyProtocol": true
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ]
            }
        },
        {
            "listen": "127.0.0.1",
            "port": 8003,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "chika",
                        "flow": "xtls-rprx-vision"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "dest": "8004",
                    "xver": 1, // 发送 PROXY protocol
                    "serverNames": [
                        "chika.example.com"
                    ],
                    "privateKey": "",
                    "shortIds": [
                        ""
                    ]
                },
                "tcpSettings": {
                    "acceptProxyProtocol": true
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ]
}
```

Эта конфигурация настраивает Xray сервер с протоколом VLESS + REALITY для обхода блокировок. Разберу каждый раздел:

1. Логирование
`"loglevel": "warning"`
Уровень детализации логов установлен на "warning" - будут записываться только предупреждения и ошибки, без избыточной информации.

2. Входящие подключения (inbounds)
У вас настроено два идентичных входящих подключения на разных портах:
Первый inbound (порт 8001):
Базовые параметры:

listen: 127.0.0.1 - сервер слушает только на локальном интерфейсе (не доступен извне напрямую)
port: 8001 - локальный порт для приёма соединений
protocol: vless - современный протокол без шифрования на уровне протокола (шифрование обеспечивает TLS/REALITY)

Настройки клиентов (settings):
`"clients": [{
    "id": "chika",
    "flow": "xtls-rprx-vision"
}]`

id: "chika" - UUID клиента (в реальной настройке должен быть настоящий UUID формата 123e4567-e89b-12d3-a456-426614174000)
flow: xtls-rprx-vision - использование XTLS Vision для дополнительной защиты от обнаружения
decryption: "none" - без дополнительного шифрования (использует REALITY)

REALITY настройки (realitySettings):

`"realitySettings": {
    "dest": "8002",
    "xver": 1,
    "serverNames": ["example.com"],
    "privateKey": "",
    "shortIds": [""]
}`
REALITY - это технология маскировки трафика под легитимный HTTPS:

dest: "8002" - порт, куда перенаправляются "настоящие" TLS соединения (не от VPN клиентов). Там должен работать реальный веб-сервер
xver: 1 - отправка PROXY protocol версии 1 на целевой сервер
serverNames: ["example.com"] - доменное имя, под которое маскируется соединение
privateKey: пустое (нужно сгенерировать приватный ключ командой xray x25519)
shortIds: пустое (короткие идентификаторы для аутентификации, можно оставить пустым или сгенерировать)

TCP настройки:
`"tcpSettings": {
    "acceptProxyProtocol": true
}`

Принимает PROXY protocol от вышестоящего прокси (например, HAProxy или Nginx)

Sniffing (определение протокола):
`"sniffing": {
    "enabled": true,
    "destOverride": ["http", "tls", "quic"]
}`

Автоматически определяет тип трафика (HTTP, TLS, QUIC) для правильной маршрутизации

Второй inbound (порт 8003):
Полностью идентичен первому, отличия:

port: 8003 (вместо 8001)
dest: "8004" (вместо 8002)
serverNames: ["chika.example.com"] (другой домен)


3. Исходящие подключения (outbounds)
Direct (прямое соединение):
`{
    "protocol": "freedom",
    "tag": "direct"
}`

Весь трафик идёт напрямую без дополнительной обработки

Blackhole (блокировка):
`{
    "protocol": "blackhole",
    "tag": "block"
}`

Для блокировки нежелательного трафика (в данной конфигурации не используется, но доступен)


🔧 Как это работает:

Клиент подключается к одному из портов (8001 или 8003) через внешний прокси
REALITY проверяет подлинность клиента по UUID и SNI (Server Name Indication)
Если клиент легитимный - трафик расшифровывается и пересылается через outbound
Если клиент не распознан - соединение перенаправляется на порт 8002/8004, где работает настоящий веб-сайт
Внешний наблюдатель видит обычный HTTPS трафик к example.com

⚠️ Что нужно исправить:

UUID клиента: замените "chika" на настоящий UUID
privateKey: сгенерируйте ключи командой xray x25519
На портах 8002 и 8004 должны работать реальные HTTPS сайты (Nginx, Caddy и т.д.)
Перед Xray обычно ставится reverse proxy (HAProxy/Nginx) для приёма внешних подключений

```
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log notice;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

stream {
    map $ssl_preread_server_name    $name {
        example.com                 backend1;
        chika.example.com           backend2;
        default                     default_backend;
    }

    upstream backend1 {
        server 127.0.0.1:8001;
    }

    upstream backend2 {
        server 127.0.0.1:8003;
    }

    upstream default_backend {
        server 127.0.0.1:8011;
    }

    server {
        listen            443;
        listen            [::]:443;
        proxy_pass        $name;
        ssl_preread       on;

        proxy_protocol    on;
    }
}

http {
    log_format main '[$time_local] $proxy_protocol_addr "$http_referer" "$http_user_agent"';
    access_log /var/log/nginx/access.log main;

    map $http_upgrade $connection_upgrade {
        default upgrade;
        ""      close;
    }

    map $proxy_protocol_addr $proxy_forwarded_elem {
        ~^[0-9.]+$        "for=$proxy_protocol_addr";
        ~^[0-9A-Fa-f:.]+$ "for=\"[$proxy_protocol_addr]\"";
        default           "for=unknown";
    }

    map $http_forwarded $proxy_add_forwarded {
        "~^(,[ \\t]*)*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*([ \\t]*,([ \\t]*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*)?)*$" "$http_forwarded, $proxy_forwarded_elem";
        default "$proxy_forwarded_elem";
    }

    server {
        listen 80;
        listen [::]:80;
        return 301 https://$host$request_uri;
    }

    server {
        listen                     127.0.0.1:8011 ssl proxy_protocol;

        ssl_reject_handshake       on;

        ssl_protocols              TLSv1.2 TLSv1.3;
    }

    server {
        listen                     127.0.0.1:8002 ssl proxy_protocol;
        http2                      on; # This directive appeared in version 1.25.1. Otherwise use it like this. "listen 127.0.0.1:8002 ssl http2 proxy_protocol;"

        set_real_ip_from           127.0.0.1;
        real_ip_header             proxy_protocol;

        ssl_certificate            /etc/ssl/private/example.com.cer;
        ssl_certificate_key        /etc/ssl/private/example.com.key;

        ssl_protocols              TLSv1.2 TLSv1.3;
        ssl_ciphers                TLS13_AES_128_GCM_SHA256:TLS13_AES_256_GCM_SHA384:TLS13_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305;
        ssl_prefer_server_ciphers  on;

        ssl_session_timeout        1h;
        ssl_session_cache          shared:SSL:10m;

        ssl_stapling               on;
        ssl_stapling_verify        on;
        resolver                   1.1.1.1 valid=60s;
        resolver_timeout           2s;

        location / {
            sub_filter                            $proxy_host $host;
            sub_filter_once                       off;

            set $website                          www.lovelive-anime.jp;
            proxy_pass                            https://$website;
            resolver                              1.1.1.1;

            proxy_set_header Host                 $proxy_host;

            proxy_http_version                    1.1;
            proxy_cache_bypass                    $http_upgrade;

            proxy_ssl_server_name                 on;

            proxy_set_header Upgrade              $http_upgrade;
            proxy_set_header Connection           $connection_upgrade;
            proxy_set_header X-Real-IP            $proxy_protocol_addr;
            proxy_set_header Forwarded            $proxy_add_forwarded;
            proxy_set_header X-Forwarded-For      $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto    $scheme;
            proxy_set_header X-Forwarded-Host     $host;
            proxy_set_header X-Forwarded-Port     $server_port;

            proxy_connect_timeout                 60s;
            proxy_send_timeout                    60s;
            proxy_read_timeout                    60s;
        }
    }

    server {
        listen                     127.0.0.1:8004 ssl proxy_protocol;
        http2                      on; # This directive appeared in version 1.25.1. Otherwise use it like this. "listen 127.0.0.1:8004 ssl http2 proxy_protocol;"

        set_real_ip_from           127.0.0.1;
        real_ip_header             proxy_protocol;

        ssl_certificate            /etc/ssl/private/chika.example.com.cer;
        ssl_certificate_key        /etc/ssl/private/chika.example.com.key;

        ssl_protocols              TLSv1.2 TLSv1.3;
        ssl_ciphers                TLS13_AES_128_GCM_SHA256:TLS13_AES_256_GCM_SHA384:TLS13_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305;
        ssl_prefer_server_ciphers  on;

        ssl_session_timeout        1h;
        ssl_session_cache          shared:SSL:10m;

        ssl_stapling               on;
        ssl_stapling_verify        on;
        resolver                   1.1.1.1 valid=60s;
        resolver_timeout           2s;

        location / {
            sub_filter                            $proxy_host $host;
            sub_filter_once                       off;

            set $website                          www.lovelive-anime.jp;
            proxy_pass                            https://$website;
            resolver                              1.1.1.1;

            proxy_set_header Host                 $proxy_host;

            proxy_http_version                    1.1;
            proxy_cache_bypass                    $http_upgrade;

            proxy_ssl_server_name                 on;

            proxy_set_header Upgrade              $http_upgrade;
            proxy_set_header Connection           $connection_upgrade;
            proxy_set_header X-Real-IP            $proxy_protocol_addr;
            proxy_set_header Forwarded            $proxy_add_forwarded;
            proxy_set_header X-Forwarded-For      $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto    $scheme;
            proxy_set_header X-Forwarded-Host     $host;
            proxy_set_header X-Forwarded-Port     $server_port;

            proxy_connect_timeout                 60s;
            proxy_send_timeout                    60s;
            proxy_read_timeout                    60s;
        }
    }
}
```

📡 STREAM блок (TCP/TLS проксирование)
Это ключевая часть для работы REALITY - здесь происходит маршрутизация на основе SNI (Server Name Indication):
SNI маппинг
```
map $ssl_preread_server_name $name {
    example.com                 backend1;
    chika.example.com           backend2;
    default                     default_backend;
}
```
Что делает:

Читает SNI из TLS handshake (НЕ расшифровывая трафик!)
Если клиент запрашивает example.com → направляет на backend1
Если chika.example.com → на backend2
Всё остальное → на default_backend

Upstream бэкенды
```
upstream backend1 {
    server 127.0.0.1:8001;  # → Xray inbound #1
}

upstream backend2 {
    server 127.0.0.1:8003;  # → Xray inbound #2
}

upstream default_backend {
    server 127.0.0.1:8011;  # → Nginx заглушка (отклоняет соединение)
}
```
TCP прокси сервер
```
server {
    listen 443;                 # Слушает на порту 443 (внешний HTTPS)
    listen [::]:443;            # IPv6
    proxy_pass $name;           # Передаёт на выбранный бэкенд
    ssl_preread on;             # Читает SNI БЕЗ расшифровки
    proxy_protocol on;          # Отправляет PROXY protocol header
}
```
Как это работает:

Клиент подключается на порт 443
Nginx читает SNI из TLS ClientHello
В зависимости от домена направляет на:

8001 (Xray для example.com)
8003 (Xray для chika.example.com)
8011 (заглушка для неизвестных доменов)


Передаёт PROXY protocol, чтобы Xray знал реальный IP клиента


🌐 HTTP блок
Логирование
```
nginxlog_format main '[$time_local] $proxy_protocol_addr "$http_referer" "$http_user_agent"';
```

Логирует реальный IP клиента из PROXY protocol

HTTP → HTTPS редирект
```
server {
    listen 80;
    listen [::]:80;
    return 301 https://$host$request_uri;
}
```

Все HTTP запросы перенаправляются на HTTPS


Заглушка (порт 8011)
```
server {
    listen 127.0.0.1:8011 ssl proxy_protocol;
    ssl_reject_handshake on;
    ssl_protocols TLSv1.2 TLSv1.3;
}
```
Назначение:

Принимает соединения с неизвестными SNI
Сразу отклоняет TLS handshake
Защита от сканирования сервера


Маскирующий сайт #1 (порт 8002)
```
server {
    listen 127.0.0.1:8002 ssl proxy_protocol;
    http2 on;
```
SSL настройки:
```
nginxssl_certificate            /etc/ssl/private/example.com.cer;
ssl_certificate_key        /etc/ssl/private/example.com.key;
ssl_protocols              TLSv1.2 TLSv1.3;
ssl_stapling               on;  # OCSP stapling для безопасности
```
Получение реального IP:
```
nginxset_real_ip_from           127.0.0.1;
real_ip_header             proxy_protocol;
```

Извлекает реальный IP клиента из PROXY protocol

Проксирование контента:
```
location / {
    set $website www.lovelive-anime.jp;
    proxy_pass https://$website;
    
    sub_filter $proxy_host $host;  # Подмена доменов в HTML
    sub_filter_once off;
    
    proxy_set_header Host $proxy_host;
    proxy_ssl_server_name on;
}
```

**Что происходит:**
1. Если Xray НЕ распознал клиента как VPN (нет правильного UUID)
2. Соединение перенаправляется сюда (на порт 8002)
3. Nginx **проксирует реальный сайт** www.lovelive-anime.jp
4. Внешний наблюдатель видит легитимный HTTPS трафик к аниме-сайту!
5. Подменяет домены в ответах (`www.lovelive-anime.jp` → `example.com`)

---

### **Маскирующий сайт #2 (порт 8004)**

Полностью идентичен 8002, но для домена `chika.example.com`:
- Свой SSL сертификат
- Также проксирует www.lovelive-anime.jp

---

## 🔄 **Как работает вся схема вместе:**

### **Сценарий 1: VPN клиент (с правильным UUID)**
```
1. Клиент → Nginx:443 (SNI: example.com)
2. Nginx читает SNI → направляет на 127.0.0.1:8001
3. Xray проверяет UUID → ОК!
4. Xray расшифровывает REALITY → туннелирует трафик
5. Клиент получает доступ в интернет через VPN
```

### **Сценарий 2: Обычный браузер / сканер**
```
1. Браузер → Nginx:443 (SNI: example.com)
2. Nginx читает SNI → направляет на 127.0.0.1:8001
3. Xray проверяет → UUID отсутствует или неверный
4. Xray перенаправляет соединение на 127.0.0.1:8002 (dest)
5. Nginx:8002 проксирует реальный сайт www.lovelive-anime.jp
6. Браузер видит обычный аниме-сайт!
```

### **Сценарий 3: Случайный SNI**

1. Клиент → Nginx:443 (SNI: randomdomain.com)
2. Nginx → default_backend (127.0.0.1:8011)
3. Nginx:8011 отклоняет TLS handshake
4. Соединение закрывается

🎯 Ключевые особенности:
PROXY Protocol

Nginx передаёт реальный IP клиента в Xray
Xray передаёт его дальше в маскирующий Nginx
Логи показывают настоящие IP адреса

ssl_preread

Nginx читает SNI не расшифровывая TLS
Xray самостоятельно выполняет REALITY handshake
Маскировка остаётся незаметной

Двойная защита:

SNI фильтрация - только известные домены попадают в Xray
UUID проверка - Xray дополнительно проверяет клиента
Fallback сайт - неудачные попытки видят реальный сайт


⚙️ Что нужно для работы:

DNS записи:

example.com → IP сервера
chika.example.com → IP сервера


SSL сертификаты:

/etc/ssl/private/example.com.cer
/etc/ssl/private/chika.example.com.cer
Можно получить через Let's Encrypt


Порты:

443 - открыт в фаерволе (внешний)
80 - открыт для получения сертификатов
8001, 8002, 8003, 8004, 8011 - только локальные




🛡️ Защита от обнаружения:

Сервер неотличим от обычного HTTPS сайта
При сканировании показывает легитимный контент
REALITY шифрование выглядит как обычный TLS
Даже активное зондирование не выявит VPN

Это очень умная и надёжная схема обхода блокировок! 🚀


--------------------------------
**Тоесть получатся в сценарии, если я открываю сайт example.com, я вижу сайт lovelive-anime.jp, хотя, по факту, нахожусь на сайте example.com?**
--------------------------------

Да, абсолютно верно! 🎯
Вот что происходит:
Обычный браузер (без VPN клиента):

Вы вводите в браузере: https://example.com
Браузер подключается к серверу, отправляет SNI: example.com
Nginx направляет на Xray (порт 8001)
Xray проверяет: "Есть ли у тебя правильный UUID?" → НЕТ
Xray: "Тогда пошли на порт 8002" (параметр "dest": "8002")
Nginx на порту 8002:

Принимает соединение
Проксирует контент с www.lovelive-anime.jp
Заменяет в HTML все www.lovelive-anime.jp на example.com (директива sub_filter)


В браузере вы видите:

Адресная строка: https://example.com ✅
SSL сертификат: для example.com ✅
Содержимое: аниме-сайт www.lovelive-anime.jp 🎌




Визуально:
Вы думаете:     example.com → example.com (обычный сайт)
На самом деле:  example.com → www.lovelive-anime.jp (проксируется)

Зачем это нужно?
Это гениальная маскировка:

🕵️ Цензор видит: "О, это просто обычный сайт про аниме, всё легально"
🔒 На самом деле: За этим же доменом скрывается VPN сервер
🎭 Результат: Невозможно отличить VPN-сервер от обычного сайта


Для VPN клиента (с правильным UUID):

VPN клиент подключается с SNI: example.com + UUID: chika
Xray: "О, UUID правильный!" → открывает VPN туннель
Вы получаете доступ в интернет через VPN
Никакого www.lovelive-anime.jp не показывается

Итог: Один и тот же домен example.com работает и как обычный сайт, и как VPN сервер одновременно! Просто для разных людей он показывает разное. Это и есть суть технологии REALITY. 🚀

**там в конфиге ещё используется домен chika.example.com, как демаешь, для чего?**

Причины использования двух доменов:
1. Резервирование (Failover)
Если один домен заблокируют - второй продолжит работать:

example.com заблокирован → переключаетесь на chika.example.com
Это как запасной выход из здания 🚪

2. Разделение пользователей
Хотя UUID одинаковый ("chika"), можно:

Раздавать разным группам пользователей разные домены
Балансировать нагрузку
Отслеживать, какой домен больше используется

3. Дополнительный уровень маскировки
Наличие поддомена делает всё более реалистичным:
example.com          → основной сайт компании
chika.example.com    → например, "раздел про персонажа Чику"
Выглядит как обычная структура сайта! 🎭
4. Защита от сканирования

Один домен можно афишировать публично
Второй держать в секрете для "своих"
Если первый попадёт в базы блокировок, второй останется незамеченным

Как это работает:
Клиент A → example.com       → Nginx:443 → Xray:8001 → VPN
Клиент B → chika.example.com → Nginx:443 → Xray:8003 → VPN
Оба получают доступ (UUID одинаковый), но через разные "входы".

Интересная деталь:
Заметьте, что оба маскирующих сайта (8002 и 8004) проксируют один и тот же сайт - www.lovelive-anime.jp. Это не случайно!
Автор явно фанат аниме "Love Live!" и использует имя персонажа Тайками Чики (Takami Chika) 🎤 в конфигурации:

UUID: "chika"
Поддомен: chika.example.com
Маскировка: сайт Love Live

Практическая польза:
В реальном использовании вы можете:

Дать друзьям разные домены:

Группа 1: example.com
Группа 2: chika.example.com

При блокировке быстро переключиться:

Отправить всем новый конфиг с другим доменом

Мониторинг:

Смотреть логи отдельно для каждого домена

Разные маскировки (можно настроить):

example.com → показывает один сайт
chika.example.com → показывает другой сайт

Вывод: Это стратегия выживаемости и гибкости. Два независимых входа = выше надёжность и сложнее заблокировать! 🛡️

**Ты можешь изменить оба конфига, чтобы остался только example.com?**

Конфигурация Xray (упрощённая)
```
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",
            "port": 8001,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "chika",
                        "flow": "xtls-rprx-vision"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "dest": "8002",
                    "xver": 1,
                    "serverNames": [
                        "example.com"
                    ],
                    "privateKey": "",
                    "shortIds": [
                        ""
                    ]
                },
                "tcpSettings": {
                    "acceptProxyProtocol": true
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ]
}
```

Конфигурация Nginx (упрощённая)

```
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log notice;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

stream {
    map $ssl_preread_server_name $name {
        example.com                 backend1;
        default                     default_backend;
    }

    upstream backend1 {
        server 127.0.0.1:8001;
    }

    upstream default_backend {
        server 127.0.0.1:8011;
    }

    server {
        listen            443;
        listen            [::]:443;
        proxy_pass        $name;
        ssl_preread       on;

        proxy_protocol    on;
    }
}

http {
    log_format main '[$time_local] $proxy_protocol_addr "$http_referer" "$http_user_agent"';
    access_log /var/log/nginx/access.log main;

    map $http_upgrade $connection_upgrade {
        default upgrade;
        ""      close;
    }

    map $proxy_protocol_addr $proxy_forwarded_elem {
        ~^[0-9.]+$        "for=$proxy_protocol_addr";
        ~^[0-9A-Fa-f:.]+$ "for=\"[$proxy_protocol_addr]\"";
        default           "for=unknown";
    }

    map $http_forwarded $proxy_add_forwarded {
        "~^(,[ \\t]*)*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*([ \\t]*,([ \\t]*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*)?)*$" "$http_forwarded, $proxy_forwarded_elem";
        default "$proxy_forwarded_elem";
    }

    server {
        listen 80;
        listen [::]:80;
        return 301 https://$host$request_uri;
    }

    server {
        listen                     127.0.0.1:8011 ssl proxy_protocol;

        ssl_reject_handshake       on;

        ssl_protocols              TLSv1.2 TLSv1.3;
    }

    server {
        listen                     127.0.0.1:8002 ssl proxy_protocol;
        http2                      on;

        set_real_ip_from           127.0.0.1;
        real_ip_header             proxy_protocol;

        ssl_certificate            /etc/ssl/private/example.com.cer;
        ssl_certificate_key        /etc/ssl/private/example.com.key;

        ssl_protocols              TLSv1.2 TLSv1.3;
        ssl_ciphers                TLS13_AES_128_GCM_SHA256:TLS13_AES_256_GCM_SHA384:TLS13_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305;
        ssl_prefer_server_ciphers  on;

        ssl_session_timeout        1h;
        ssl_session_cache          shared:SSL:10m;

        ssl_stapling               on;
        ssl_stapling_verify        on;
        resolver                   1.1.1.1 valid=60s;
        resolver_timeout           2s;

        location / {
            sub_filter                            $proxy_host $host;
            sub_filter_once                       off;

            set $website                          www.lovelive-anime.jp;
            proxy_pass                            https://$website;
            resolver                              1.1.1.1;

            proxy_set_header Host                 $proxy_host;

            proxy_http_version                    1.1;
            proxy_cache_bypass                    $http_upgrade;

            proxy_ssl_server_name                 on;

            proxy_set_header Upgrade              $http_upgrade;
            proxy_set_header Connection           $connection_upgrade;
            proxy_set_header X-Real-IP            $proxy_protocol_addr;
            proxy_set_header Forwarded            $proxy_add_forwarded;
            proxy_set_header X-Forwarded-For      $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto    $scheme;
            proxy_set_header X-Forwarded-Host     $host;
            proxy_set_header X-Forwarded-Port     $server_port;

            proxy_connect_timeout                 60s;
            proxy_send_timeout                    60s;
            proxy_read_timeout                    60s;
        }
    }
}
```

Что изменилось:
В Xray:

❌ Удалён второй inbound (порт 8003)
✅ Остался только один вход на порту 8001

В Nginx:

❌ Удалён backend2 (chika.example.com)
❌ Удалён server на порту 8004
❌ Убран маппинг для chika.example.com
✅ Теперь только example.com → 127.0.0.1:8001

Теперь нужны только:

SSL сертификат:

/etc/ssl/private/example.com.cer
/etc/ssl/private/example.com.key

Порты:
443 - внешний (открыт в firewall)
80 - для редиректа на HTTPS
8001 - Xray (локальный)
8002 - маскирующий сайт (локальный)
8011 - заглушка (локальный)

DNS запись:

example.com → IP вашего сервера

Конфигурация стала проще и понятнее! 🎯

**а что за заглушка 8011?**

Отличный вопрос! Давайте разберём, зачем нужна эта "заглушка" на порту 8011:

```
server {
    listen                     127.0.0.1:8011 ssl proxy_protocol;
    ssl_reject_handshake       on;  # ← Ключевая директива
    ssl_protocols              TLSv1.2 TLSv1.3;
}
```
Это специальный сервер, который отклоняет TLS соединение на этапе handshake.

Зачем это нужно:
Защита от сканирования сервера
Представьте ситуацию:
Сценарий 1: Злоумышленник сканирует ваш сервер

Хакер пытается подключиться с случайным SNI
openssl s_client -connect yourserver.com:443 -servername random-domain.com

Что происходит:

Nginx получает SNI: random-domain.com
Проверяет маппинг:

```
map $ssl_preread_server_name $name {
       example.com    backend1;
       default        default_backend;  # ← Не совпало!
   }
```

3. Направляет на `default_backend` (127.0.0.1:8011)
4. **Сервер на 8011 ОТКЛОНЯЕТ соединение**
5. Хакер получает ошибку: `SSL handshake failed`

---

## **Без заглушки vs С заглушкой:**

### **❌ Без заглушки (плохой вариант):**

Хакер → SNI: unknown.com → ??? → Что показать?

Варианты:
- Показать ошибку → "Тут что-то скрывают"
- Показать дефолтный сайт → "Странно, зачем фильтруют SNI?"

### **✅ С заглушкой (хороший вариант):**

Хакер → SNI: unknown.com → 8011 → SSL handshake rejected

Результат:
- Выглядит как **неправильно настроенный сервер** или **строгая безопасность**
- Нет подозрительной активности
- Не раскрывает структуру сервера

---

## **Реальный пример:**

### **Сценарий A: Нормальный пользователь**

Браузер → https://example.com → Nginx → Xray:8001 → 
  → UUID правильный? → ДА → VPN работает ✅
  → UUID неправильный? → НЕТ → Показать www.lovelive-anime.jp ✅


### **Сценарий B: Сканер безопасности**

Сканер → https://123.45.67.89 (прямой IP без SNI)
  → Nginx → default_backend → 8011 → SSL rejected ❌
  
Сканер → https://random.com (SNI к вашему IP)
  → Nginx → default_backend → 8011 → SSL rejected ❌

Почему не показывать маскирующий сайт всем?
Можно было бы настроить так:

```
map $ssl_preread_server_name $name {
    example.com    backend1;
    default        backend1;  # Все неизвестные SNI тоже на Xray
}
```
Но это плохо, потому что:

Увеличенная нагрузка - все сканеры будут проксироваться через Xray и www.lovelive-anime.jp
Подозрительно - зачем серверу отвечать на ЛЮБОЙ SNI одинаково?
Следы в логах - больше мусорного трафика
Расход ресурсов - проксирование требует CPU/память

Можно ли убрать заглушку?
Технически - да, но не рекомендуется.
Если убрать:
```
map $ssl_preread_server_name $name {
    example.com    backend1;
    # default отсутствует
}
```
Nginx будет выдавать ошибку для неизвестных SNI, что может выглядеть подозрительно.

Вывод:
Заглушка на порту 8011 - это "сторожевой пёс" 🐕:

Отпугивает любопытных сканеров
Защищает от разведки структуры сервера
Минимизирует следы в логах
Экономит ресурсы сервера

Это best practice для REALITY конфигурации! 
