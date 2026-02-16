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

`user nginx;
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
}`

