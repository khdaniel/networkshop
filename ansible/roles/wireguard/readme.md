# Ansible Role: WireGuard

Эта роль настраивает сервер WireGuard на целевой машине с динамическим рендерингом конфигурации на основе списка клиентов.

## Поддерживаемые ОС
Ubuntu Server:
- 20.04
- 22.04
- 24.04

## Описание

Роль выполняет следующие действия:

1. **Установка WireGuard**: Устанавливает WireGuard и необходимые зависимости на целевой хост.
2. **Включение IP-маршрутизации**: Включает пересылку пакетов (IP forwarding) на целевой машине для поддержки маршрутизации трафика через VPN.
3. **Копирование ключей**: Копирует приватный и публичный ключи сервера, которые хранятся в папке `files`, в целевую систему.
4. **Динамическое рендеринг конфигурации**: Шаблон конфигурации WireGuard рендерится с учётом списка клиентов, который передаётся через переменные.
5. **Перезапуск службы WireGuard**: После изменения конфигурации сервер WireGuard автоматически перезапускается.


## Переменные

В файле `vars/main.yml` должен быть описан список клиентов, их публичные ключи и IP-адреса для назначения:
- `wireguard_port`: Порт, который использует сервер WireGuard.
- `wireguard_address`: IP-адрес и подсеть для WireGuard сервера.
- `wireguard_clients`: Список клиентов с их публичными ключами и IP-адресами.

Пример:

```yaml
wireguard_port: 51820
wireguard_address: 10.0.0.1/24

# Список клиентов
wireguard_clients:
  - name: client1
    public_key: "PUBLIC_KEY_CLIENT1"
    ip_address: "10.0.0.2/32"
  - name: client2
    public_key: "PUBLIC_KEY_CLIENT2"
    ip_address: "10.0.0.3/32"
```

## Файлы
Следующие файлы должны быть размещены в директории files роли:
- `server_private.key`: Приватный ключ сервера WireGuard.
- `server_public.key`: Публичный ключ сервера WireGuard.

## Шаблоны
 - `wg0.conf.j2`: Шаблон конфигурации сервера WireGuard, который динамически рендерится на основе переменных и списка клиентов.

Пример шаблона конфигурации:
```jinja
[Interface]
Address = {{ wireguard_address }}
ListenPort = {{ wireguard_port }}
PrivateKey = {{ lookup('file', 'files/server_private.key') }}

{% for client in wireguard_clients %}
[Peer]
PublicKey = {{ client.public_key }}
AllowedIPs = {{ client.ip_address }}
{% endfor %}
```

## Пример использования
Пример playbook для использования роли:
```yaml
---
- hosts: wireguard_servers
  become: yes
  roles:
    - wireguard
  vars:
    wireguard_port: 51820
    wireguard_address: 10.0.0.1/24
    wireguard_clients:
      - name: client1
        public_key: "PUBLIC_KEY_CLIENT1"
        ip_address: "10.0.0.2/32"
      - name: client2
        public_key: "PUBLIC_KEY_CLIENT2"
        ip_address: "10.0.0.3/32"
```

## Зависимости
Ansible 2.9 или выше.

WireGuard должен поддерживаться целевой операционной системой (например, Ubuntu).
## Лицензия
MIT