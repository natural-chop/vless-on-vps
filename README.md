# Автоматическая настройка VPS и развертывание Xray (VLESS-REALITY)

Данный Ansible-плейбук предназначен для первоначальной настройки «чистого» сервера и быстрого развертывания прокси-сервера Xray с протоколом **VLESS + REALITY**.

---

## Требования

### 1. На целевом сервере (VPS):

* **Операционная система:** Ubuntu (22.04 / 24.04) или Debian (требуется пакетный менеджер `apt`).
* **Доступ:** Доступ к пользователю `root` по паролю через SSH.

### 2. На локальной машине (где запускается Ansible):

* **Ansible** (версия 2.12 или выше).

---

## Конфигурация переменные (`vars.yml`)

Перед запуском создайте или отредактируйте файл с переменными (`vars.yml`):

```yaml
# Сетевые и учетные данные VPS
ansible_host: "1.2.3.4"            # IP-адрес вашего VPS
root_password: "old-pass"           # Временный пароль root, выданный провайдером
new_user: "new-login"              # Имя создаваемого sudo-пользователя
new_user_password: "new-pass"       # Пароль для нового sudo-пользователя

# Параметры Xray (VLESS-REALITY)
xray_port: 443                      # Входящий порт Xray (рекомендуется 443)
xray_sni: "dl.google.com"           # Домен для маскировки (SNI)
xray_dest: "dl.google.com:443"      # Целевой адрес маскируемого сайта (DEST)
xray_private_key: "private-key"     # Приватный ключ REALITY
xray_public_key: "public-key"       # Публичный ключ REALITY (для клиентов)
xray_short_id: "1a2b3c4d"          # Short ID REALITY (hex-строка)

# Список клиентов Xray
xray_clients:
  - name: "client1"
    uuid: "uuid1"                  # UUID первого клиента
  - name: "client2"
    uuid: "uuid2"                  # UUID второго клиента

```

---

## Генерация ключей Xray и Short ID

Для работы протокола VLESS-REALITY вам необходимо сгенерировать пару ключей (Private Key / Public Key), валидный Short ID и уникальные UUID для каждого клиента.

---

### 1. Пара ключей REALITY (`xray_private_key` и `xray_public_key`)

Пара ключей задействует эллиптическую кривую Curve25519 (`x25519`).

#### Вариант А: Через Docker (без установки Xray на свой ПК)
Если у вас установлен Docker, можно сгенерировать ключи одной командой:
```bash
docker run --rm ghcr.io/xtls/xray-core xray x25519

```

#### Вариант Б: Через утилиту `xray` (если Xray уже установлен на VPS или локально)

```bash
xray x25519

```

*Пример вывода:*

```text
Private key: eFA1... (вставьте в vars.yml как xray_private_key)
Public key:  3kB9... (вставьте в vars.yml как xray_public_key)

```

#### Вариант В: Онлайн-ресурсы

Если под рукой нет терминала или Docker, можно воспользоваться открытыми онлайн-генераторами, например:

* Встроенными веб-инструментами генерации REALITY-ключей в веб-панелях (3X-UI, Marzban и др.).
* Сторонними веб-сервисами генерации ключей Xray X25519 (используйте с осторожностью и только для тестовых стендов).

---

### 2. Генерация Short ID (`xray_short_id`)

Short ID представляет собой шестнадцатеричную (hex) строку четной длины от 2 до 16 символов (от 1 до 8 байт).

#### Через консоль Linux:

```bash
# Сгенерировать 16-символьный (8 байт) hex-код с помощью OpenSSL:
openssl rand -hex 8

# Или с помощью системного генератора случайных чисел:
head -c 8 /dev/urandom | xxd -p

```

---

### 3. Генерация UUID для клиентов (`uuid`)

Каждому клиенту из списка `xray_clients` требуется уникальный идентификатор в формате UUID v4.

#### Через консоль Linux:

```bash
# Использование утилиты uuidgen:
uuidgen

# Альтернативный способ без установки утилит:
cat /proc/sys/kernel/random/uuid

```

#### Онлайн-ресурсы:

* Официальный веб-сервис: [uuidgenerator.net](https://www.uuidgenerator.net/?utm_source=gemini)

```


## Использование

Запускать плейбуки по очереди, отлаживая ошибки, если будут.

```bash
ansible-playbook 1-init.yml
ansible-playbook 2-config.yml
ansible-playbook 3-xray.yml

```

---
