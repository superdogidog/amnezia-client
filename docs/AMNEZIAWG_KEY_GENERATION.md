# Генерация ключей AmneziaWG через Linux терминал

## О протоколе AmneziaWG

**AmneziaWG** - это **новый основной протокол** в Amnezia VPN (ранее назывался просто "WireGuard" или "WireGuard с обфускацией"). Это обфусцированная версия WireGuard с механизмами обхода Deep Packet Inspection (DPI) для обхода блокировок.

> ⚠️ **Важно:** Это документация для **нового** протокола AmneziaWG, который сейчас является стандартным. Старый WireGuard без обфускации теперь называется "Amnezia Legacy" или "amnezia-wg".

---

## Содержание

- [Формат ключей](#формат-ключей)
- [Генерация ключей через терминал Linux](#генерация-ключей-через-терминал-linux)
- [Установка инструментов AmneziaWG](#установка-инструментов-amneziawg)
- [Создание конфигурации клиента](#создание-конфигурации-клиента)
- [Конвертация в vpn:// ссылку](#конвертация-в-vpn-ссылку)
- [Формат конфигурации AmneziaWG](#формат-конфигурации-amneziawg)

---

## Формат ключей

AmneziaWG использует те же криптографические примитивы, что и WireGuard:
- **Алгоритм:** Curve25519 для обмена ключами
- **Кодирование:** Base64
- **Длина ключа:** 32 байта (256 бит)
- **Длина в Base64:** 44 символа (включая завершающий `=`)

**Примеры ключей:**
```
Приватный ключ: yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=
Публичный ключ:  HIgo9xNzJMWLKASShiTqIybxZ0U3wGLiUeHV6U2v220=
Preshared ключ:  FBqnfJNKcIE8VVf+UJxKLCvFPZ4IJKJYyaJKfH6pJAE=
```

Ключи являются криптографически случайными данными, закодированными в Base64.

---

## Генерация ключей через терминал Linux

### Основные команды

AmneziaWG использует утилиту `awg` (AmneziaWG) вместо стандартной `wg` (WireGuard).

#### 1. Генерация приватного ключа
```bash
awg genkey
```
**Вывод:** `yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=`

#### 2. Генерация публичного ключа из приватного
```bash
echo "yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=" | awg pubkey
```
**Вывод:** `HIgo9xNzJMWLKASShiTqIybxZ0U3wGLiUeHV6U2v220=`

#### 3. Генерация пары ключей (приватный + публичный)
```bash
# Вариант 1: С сохранением в файлы
awg genkey | tee privatekey | awg pubkey > publickey

# Вариант 2: С выводом в переменные
PRIVATE_KEY=$(awg genkey)
PUBLIC_KEY=$(echo "$PRIVATE_KEY" | awg pubkey)

echo "Приватный ключ: $PRIVATE_KEY"
echo "Публичный ключ: $PUBLIC_KEY"
```

#### 4. Генерация preshared ключа (дополнительная безопасность)
```bash
awg genpsk
```
**Вывод:** `FBqnfJNKcIE8VVf+UJxKLCvFPZ4IJKJYyaJKfH6pJAE=`

---

### Полный скрипт генерации ключей для клиента

```bash
#!/bin/bash
# Скрипт для генерации полного набора ключей для нового клиента

CLIENT_NAME=$1

if [ -z "$CLIENT_NAME" ]; then
    echo "Использование: $0 <имя_клиента>"
    echo "Пример: $0 alice"
    exit 1
fi

echo "Генерация ключей для клиента: $CLIENT_NAME"
echo "============================================"

# Создать временную директорию
mkdir -p /tmp/awg-keys/$CLIENT_NAME
cd /tmp/awg-keys/$CLIENT_NAME

# Генерация ключей
echo "1. Генерация приватного и публичного ключей..."
awg genkey | tee ${CLIENT_NAME}_private.key | awg pubkey > ${CLIENT_NAME}_public.key

echo "2. Генерация preshared ключа..."
awg genpsk > ${CLIENT_NAME}_preshared.key

# Вывод результатов
echo ""
echo "✓ Ключи успешно сгенерированы!"
echo "============================================"
echo "Приватный ключ клиента:"
cat ${CLIENT_NAME}_private.key
echo ""
echo "Публичный ключ клиента:"
cat ${CLIENT_NAME}_public.key
echo ""
echo "Preshared ключ:"
cat ${CLIENT_NAME}_preshared.key
echo "============================================"
echo ""
echo "Файлы сохранены в: /tmp/awg-keys/$CLIENT_NAME/"
echo ""
echo "⚠️  ВАЖНО: Сохраните приватный ключ в безопасном месте!"
echo "⚠️  Не передавайте приватный ключ никому!"
```

**Использование:**
```bash
chmod +x generate_awg_keys.sh
./generate_awg_keys.sh alice
```

---

## Установка инструментов AmneziaWG

### Внутри Docker контейнера AmneziaWG

Если у вас уже запущен сервер AmneziaWG в Docker:
```bash
# Войти в контейнер
docker exec -it amnezia-awg bash

# Инструменты awg уже установлены и готовы к использованию
awg version
awg genkey
```

### На хост-системе Linux

Для установки AmneziaWG на хост-системе:

#### Ubuntu/Debian
```bash
# Скачать пакет из релизов
wget https://github.com/amnezia-vpn/amneziawg-linux-kernel-module/releases/latest/download/amneziawg-tools.deb

# Установить
sudo dpkg -i amneziawg-tools.deb

# Или собрать из исходников
git clone https://github.com/amnezia-vpn/amneziawg-tools
cd amneziawg-tools/src
make
sudo make install
```

#### CentOS/RHEL/Fedora
```bash
# Собрать из исходников
git clone https://github.com/amnezia-vpn/amneziawg-tools
cd amneziawg-tools/src
make
sudo make install
```

#### Arch Linux
```bash
# Из AUR
yay -S amneziawg-tools

# Или собрать из исходников
git clone https://github.com/amnezia-vpn/amneziawg-tools
cd amneziawg-tools/src
make
sudo make install
```

---

## Создание конфигурации клиента

### Формат конфигурации AmneziaWG

Конфигурация AmneziaWG выглядит как конфигурация WireGuard, но с дополнительными параметрами обфускации:

```ini
[Interface]
# Приватный ключ клиента (НЕ ПЕРЕДАВАТЬ НИКОМУ!)
PrivateKey = yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=

# IP-адрес клиента в VPN сети
Address = 10.8.1.10/32

# DNS серверы
DNS = 1.1.1.1, 1.0.0.1

# === Параметры обфускации AmneziaWG ===
# Эти параметры ДОЛЖНЫ совпадать с параметрами на сервере!

# Junk packets - добавление мусорных пакетов
Jc = 5          # Количество junk-пакетов (3-7 рекомендуется)
Jmin = 10       # Минимальный размер junk (10-50)
Jmax = 50       # Максимальный размер junk (50-1000)

# Размеры junk для разных типов пакетов
S1 = 85         # Init packet junk (15-150) - КРИТИЧЕН для обхода DPI!
S2 = 142        # Response packet junk (15-150) - должен отличаться от S1
S3 = 28         # Empty packet junk (0-64)
S4 = 15         # Data packet junk (0-20) - влияет на скорость

# Magic headers - имитация других протоколов
H1 = 1234567-9876543
H2 = 2345678-8765432
H3 = 3456789-7654321
H4 = 4567890-6543210

# Special patterns (обычно 0 для стандартной конфигурации)
I1 = 0
I2 = 0
I3 = 0
I4 = 0
I5 = 0

[Peer]
# Публичный ключ сервера
PublicKey = 2bXVEJz1v5DkZ9w7r9NQpPHb0Q3C7M8iO6VexLLG0XE=

# Preshared ключ для дополнительной безопасности
PresharedKey = FBqnfJNKcIE8VVf+UJxKLCvFPZ4IJKJYyaJKfH6pJAE=

# IP-адрес и порт сервера
Endpoint = 89.125.213.14:51820

# Маршрутизировать весь трафик через VPN
AllowedIPs = 0.0.0.0/0, ::/0

# Keepalive для поддержания соединения через NAT
PersistentKeepalive = 25
```

### Автоматическое создание конфигурации

Скрипт для автоматического создания клиентской конфигурации на сервере:

```bash
#!/bin/bash
# Выполнять внутри контейнера Docker: docker exec -it amnezia-awg bash

CLIENT_NAME=$1
CLIENT_IP=$2

if [ -z "$CLIENT_NAME" ] || [ -z "$CLIENT_IP" ]; then
    echo "Использование: $0 <имя> <ip>"
    echo "Пример: $0 alice 10.8.1.10"
    exit 1
fi

# Генерация ключей
cd /tmp
awg genkey | tee ${CLIENT_NAME}_private.key | awg pubkey > ${CLIENT_NAME}_public.key
awg genpsk > ${CLIENT_NAME}_preshared.key

CLIENT_PRIVATE=$(cat ${CLIENT_NAME}_private.key)
CLIENT_PUBLIC=$(cat ${CLIENT_NAME}_public.key)
CLIENT_PSK=$(cat ${CLIENT_NAME}_preshared.key)

# Получение параметров сервера
SERVER_PUBLIC=$(cat /opt/amnezia/awg/wireguard_server_public_key.key)
SERVER_IP=$(hostname -I | awk '{print $1}')
AWG_PORT=$(grep "ListenPort" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)

# Получение параметров обфускации
JC=$(grep "^Jc" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
JMIN=$(grep "^Jmin" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
JMAX=$(grep "^Jmax" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
S1=$(grep "^S1" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
S2=$(grep "^S2" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
S3=$(grep "^S3" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
S4=$(grep "^S4" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
H1=$(grep "^H1" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
H2=$(grep "^H2" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
H3=$(grep "^H3" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)
H4=$(grep "^H4" /opt/amnezia/awg/awg0.conf | cut -d'=' -f2 | xargs)

# Добавление peer на сервер
cat >> /opt/amnezia/awg/awg0.conf << EOF

[Peer]
# ${CLIENT_NAME}
PublicKey = ${CLIENT_PUBLIC}
PresharedKey = ${CLIENT_PSK}
AllowedIPs = ${CLIENT_IP}/32
EOF

# Применение конфигурации
awg syncconf awg0 <(awg-quick strip /opt/amnezia/awg/awg0.conf)

# Создание клиентской конфигурации
cat > /tmp/${CLIENT_NAME}_amneziawg.conf << EOF
[Interface]
PrivateKey = ${CLIENT_PRIVATE}
Address = ${CLIENT_IP}/32
DNS = 1.1.1.1, 1.0.0.1
Jc = ${JC}
Jmin = ${JMIN}
Jmax = ${JMAX}
S1 = ${S1}
S2 = ${S2}
S3 = ${S3}
S4 = ${S4}
H1 = ${H1}
H2 = ${H2}
H3 = ${H3}
H4 = ${H4}
I1 = 0
I2 = 0
I3 = 0
I4 = 0
I5 = 0

[Peer]
PublicKey = ${SERVER_PUBLIC}
PresharedKey = ${CLIENT_PSK}
Endpoint = ${SERVER_IP}:${AWG_PORT}
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
EOF

echo "✓ Клиент ${CLIENT_NAME} успешно создан!"
echo "Конфигурация: /tmp/${CLIENT_NAME}_amneziawg.conf"
echo ""
echo "Скачать конфигурацию:"
echo "docker cp amnezia-awg:/tmp/${CLIENT_NAME}_amneziawg.conf ./"
```

---

## Конвертация в vpn:// ссылку

### Что такое vpn:// ссылка?

`vpn://` ссылка - это специальный формат Amnezia Client для обмена конфигурациями VPN. Это **не просто Base64**, а:

1. **JSON структура** с метаданными сервера и контейнера
2. **zlib сжатие** (уровень 8)
3. **URL-safe Base64 кодирование** без padding символов `=`
4. **Префикс vpn://**

**Формат:** `vpn://eNplUF1PgzAU_S99nTJKCxQSHwhx...`

### Процесс создания vpn:// ссылки

```
Конфигурация WireGuard
        ↓
1. Преобразование в JSON структуру Amnezia
        ↓
2. Сжатие с помощью zlib (уровень 8)
        ↓
3. Кодирование в URL-safe Base64 без padding
        ↓
4. Добавление префикса "vpn://"
        ↓
    vpn://eNplUF1Pg...
```

### Использование Python скрипта (рекомендуется)

В папке `docs/` есть готовый скрипт `wireguard_to_vpn_link.py`:

```bash
# Конвертировать конфигурацию в vpn:// ссылку
python3 docs/wireguard_to_vpn_link.py --config /path/to/config.conf

# Декодировать vpn:// ссылку обратно
python3 docs/wireguard_to_vpn_link.py --decode 'vpn://eNplUF1Pg...'
```

**Пример вывода:**
```
WireGuard конфигурация успешно преобразована в vpn:// ссылку!

vpn:// ссылка:
vpn://eNplUF1PgzAU_S99nTJKCxQSHwhxbirTAWZ-xCwIFWF8jRZ0LPvvtjjjg7nJ7b3n3nNyew7go2Z8GZUU2IBYCtR0RYNIgRicgYS-R13B3briUVbRVqxEZUWHLDqPPlO5UDEoQKiM8QNoUkgZQwDxL5cB--Xw1_6Tktk-gKZuuZhhg0CpFxcZrfimabN-s6V7MVkt4GXQcmc9pHhheKQnbe37ya5IfLdmuW8MTTpr4MyvvIs_gayRZ6riKKhoU01-jtG2p-2m6d5Oyo7vhN46yXnGczT17gh3d2bpOebjLcFBFz_sJuVNHC9ZyKRyw7Yn4j56uiqmqQf7ZVmoQTjx11tnVcVoQl0re7bC-_2A8r5bzGJJvI6BbYinzCpgQ1VW0RewdVEFwk2oIVEIFyESW_PRXxUjIlrdEPS5NBgiCxPVNNGIIGm5riJimLopASwAE2LLULEFwfH4evwG1xCNOA

Вы можете использовать эту ссылку для:
1. Импорта в Amnezia Client
2. Генерации QR кода
3. Обмена конфигурацией с другими пользователями
```

### Ручная конвертация через Python

Если нужно встроить в свой скрипт:

```python
#!/usr/bin/env python3
import json
import zlib
import base64

def create_vpn_link(private_key, client_ip, server_pubkey, preshared_key, 
                    server_ip, server_port, dns1="1.1.1.1", dns2="8.8.8.8",
                    jc=5, jmin=10, jmax=50, s1=85, s2=142, s3=28, s4=15,
                    h1="1234567-9876543", h2="2345678-8765432", 
                    h3="3456789-7654321", h4="4567890-6543210"):
    """
    Создать vpn:// ссылку из параметров AmneziaWG.
    """
    
    # Создать JSON структуру Amnezia
    config = {
        "hostName": server_ip,
        "defaultContainer": "amnezia-awg",
        "dns1": dns1,
        "dns2": dns2,
        "containers": [
            {
                "container": "amnezia-awg",
                "awg": {
                    "port": str(server_port),
                    "client_priv_key": private_key,
                    "client_ip": client_ip,
                    "psk_key": preshared_key,
                    "server_pub_key": server_pubkey,
                    "Jc": jc,
                    "Jmin": jmin,
                    "Jmax": jmax,
                    "S1": s1,
                    "S2": s2,
                    "H1": h1,
                    "H2": h2,
                    "H3": h3,
                    "H4": h4
                }
            }
        ]
    }
    
    # Сериализация в JSON (компактный формат)
    json_data = json.dumps(config, separators=(',', ':')).encode('utf-8')
    
    # Сжатие zlib (уровень 8)
    compressed = zlib.compress(json_data, 8)
    
    # URL-safe Base64 кодирование без padding
    encoded = base64.urlsafe_b64encode(compressed).decode('ascii').rstrip('=')
    
    return f"vpn://{encoded}"

# Пример использования
vpn_link = create_vpn_link(
    private_key="yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=",
    client_ip="10.8.1.10/32",
    server_pubkey="2bXVEJz1v5DkZ9w7r9NQpPHb0Q3C7M8iO6VexLLG0XE=",
    preshared_key="FBqnfJNKcIE8VVf+UJxKLCvFPZ4IJKJYyaJKfH6pJAE=",
    server_ip="89.125.213.14",
    server_port=51820
)

print(vpn_link)
```

### Конвертация через командную строку (Bash)

Простой однострочник для конвертации:

```bash
#!/bin/bash
# Требует установленного Python 3

create_vpn_link() {
    local PRIVATE_KEY=$1
    local CLIENT_IP=$2
    local SERVER_PUBKEY=$3
    local PRESHARED_KEY=$4
    local SERVER_IP=$5
    local SERVER_PORT=$6
    
    python3 -c "
import json, zlib, base64
config = {
    'hostName': '$SERVER_IP',
    'defaultContainer': 'amnezia-awg',
    'dns1': '1.1.1.1',
    'dns2': '8.8.8.8',
    'containers': [{
        'container': 'amnezia-awg',
        'awg': {
            'port': '$SERVER_PORT',
            'client_priv_key': '$PRIVATE_KEY',
            'client_ip': '$CLIENT_IP',
            'psk_key': '$PRESHARED_KEY',
            'server_pub_key': '$SERVER_PUBKEY',
            'Jc': 5, 'Jmin': 10, 'Jmax': 50,
            'S1': 85, 'S2': 142, 'S3': 28, 'S4': 15,
            'H1': '1234567-9876543', 'H2': '2345678-8765432',
            'H3': '3456789-7654321', 'H4': '4567890-6543210'
        }
    }]
}
data = json.dumps(config, separators=(',',':')).encode()
compressed = zlib.compress(data, 8)
encoded = base64.urlsafe_b64encode(compressed).decode().rstrip('=')
print(f'vpn://{encoded}')
"
}

# Использование
VPN_LINK=$(create_vpn_link \
    "yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=" \
    "10.8.1.10/32" \
    "2bXVEJz1v5DkZ9w7r9NQpPHb0Q3C7M8iO6VexLLG0XE=" \
    "FBqnfJNKcIE8VVf+UJxKLCvFPZ4IJKJYyaJKfH6pJAE=" \
    "89.125.213.14" \
    "51820"
)

echo $VPN_LINK
```

---

## Формат конфигурации AmneziaWG

### Структура JSON для vpn:// ссылки

```json
{
  "hostName": "89.125.213.14",
  "defaultContainer": "amnezia-awg",
  "dns1": "1.1.1.1",
  "dns2": "8.8.8.8",
  "containers": [
    {
      "container": "amnezia-awg",
      "awg": {
        "port": "51820",
        "client_priv_key": "yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=",
        "client_ip": "10.8.1.10/32",
        "psk_key": "FBqnfJNKcIE8VVf+UJxKLCvFPZ4IJKJYyaJKfH6pJAE=",
        "server_pub_key": "2bXVEJz1v5DkZ9w7r9NQpPHb0Q3C7M8iO6VexLLG0XE=",
        "Jc": 5,
        "Jmin": 10,
        "Jmax": 50,
        "S1": 85,
        "S2": 142,
        "H1": "1234567-9876543",
        "H2": "2345678-8765432",
        "H3": "3456789-7654321",
        "H4": "4567890-6543210"
      }
    }
  ]
}
```

### Параметры обфускации

| Параметр | Описание | Рекомендуемое значение |
|----------|----------|------------------------|
| **Jc** | Количество junk-пакетов | 3-7 (5 рекомендуется) |
| **Jmin** | Минимальный размер junk | 10-50 (10 рекомендуется) |
| **Jmax** | Максимальный размер junk | 50-1000 (50 рекомендуется) |
| **S1** | Init packet junk | 15-150 (**КРИТИЧЕН!** 85 рекомендуется) |
| **S2** | Response packet junk | 15-150 (142 рекомендуется, отличается от S1) |
| **S3** | Empty packet junk | 0-64 (28 рекомендуется) |
| **S4** | Data packet junk | 0-20 (10-15 рекомендуется, влияет на скорость) |
| **H1-H4** | Magic headers | Диапазоны чисел для имитации протоколов |

**⚠️ ВАЖНО:** Параметры обфускации должны быть **ИДЕНТИЧНЫ** на сервере и всех клиентах!

---

## Дополнительные ресурсы

### Документация в этом репозитории

1. **[AMNEZIAWG_QUICK_REFERENCE.md](AMNEZIAWG_QUICK_REFERENCE.md)** - Быстрая справка с командами
2. **[AMNEZIAWG_MANUAL_MANAGEMENT.md](AMNEZIAWG_MANUAL_MANAGEMENT.md)** - Полное пошаговое руководство
3. **[AMNEZIAWG_ADVANCED_CONFIGURATION.md](AMNEZIAWG_ADVANCED_CONFIGURATION.md)** - Детальное описание параметров обфускации
4. **[VPN_LINK_GENERATION_PROCESS.md](VPN_LINK_GENERATION_PROCESS.md)** - Процесс генерации vpn:// ссылок
5. **[wireguard_to_vpn_link.py](wireguard_to_vpn_link.py)** - Python скрипт для конвертации

### Внешние ссылки

- [Официальная документация Amnezia](https://docs.amnezia.org/)
- [AmneziaWG на GitHub](https://github.com/amnezia-vpn/amneziawg-linux-kernel-module)
- [AmneziaWG Tools](https://github.com/amnezia-vpn/amneziawg-tools)
- [WireGuard Documentation](https://www.wireguard.com/)

---

## Часто задаваемые вопросы (FAQ)

### В чем разница между AmneziaWG и WireGuard?

**AmneziaWG** = WireGuard + обфускация для обхода блокировок DPI

- Добавляет junk-данные к пакетам (изменяет размер и паттерн)
- Использует magic headers для имитации других протоколов
- Поддерживает special patterns для маскировки под DNS/HTTPS
- Полностью совместим с WireGuard на уровне протокола

### Можно ли использовать обычный wg вместо awg?

**Нет.** Для AmneziaWG нужно использовать инструменты `awg`, так как они поддерживают дополнительные параметры обфускации. Обычный `wg` не понимает параметры `Jc`, `S1-S4`, `H1-H4` и т.д.

### Где взять публичный ключ сервера?

```bash
# Внутри контейнера
docker exec amnezia-awg cat /opt/amnezia/awg/wireguard_server_public_key.key

# Или из конфигурации
docker exec amnezia-awg grep "PublicKey" /opt/amnezia/awg/awg0.conf | head -1
```

### Как проверить, что ключ валиден?

Валидный ключ AmneziaWG/WireGuard:
- Имеет длину ровно 44 символа
- Состоит из символов Base64: `A-Z`, `a-z`, `0-9`, `+`, `/`, `=`
- Заканчивается на `=` (обычно)
- Является случайной последовательностью (нет очевидных паттернов)

```bash
# Проверка длины
echo "yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=" | wc -c
# Должно быть 45 (44 символа + символ новой строки)
```

### Безопасно ли использовать один и тот же ключ повторно?

**НЕТ!** Каждый клиент должен иметь уникальную пару ключей. Никогда не используйте один и тот же приватный ключ для нескольких клиентов.

### Что делать, если ключ скомпрометирован?

1. Немедленно удалить peer с этим публичным ключом с сервера
2. Сгенерировать новую пару ключей
3. Создать новую конфигурацию с новыми ключами
4. Обновить конфигурацию на клиенте

---

**Версия:** 1.0  
**Дата:** 2025-01-26  
**Автор:** Документация Amnezia Client
