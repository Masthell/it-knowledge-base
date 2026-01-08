### VoIP: Asterisk/FreePBX - виртуальная АТС

Что такое VoIP и зачем это нужно?

VoIP (Voice over IP) — передача голоса по IP-сетям. Современная замена традиционной телефонии.

Компоненты VoIP-системы:

IP-АТС (Asterisk/FreePBX) — "мозг" системы

SIP-телефоны — аппаратные или программные

SIP-провайдер — подключение к городской/междугородной связи

SIP-транк — канал связи с провайдером

Asterisk — ядро АТС, настройка через конфиг-файлы

туториал https://youtu.be/0Q53Q8KUXoA?si=dbZ1KMNahrUKjgTY

#### Архитектура домашней VoIP-лаборатории
Вариант 1: Полностью виртуальный 
```
[FreePBX VM] ←→ [SIP-клиенты на Windows/Android]
    ↓
[Виртуальная сеть]
    ↓
[OPNsense VPN] ←→ [Удалённые SIP-клиенты]
Вариант 2: Гибридный (с реальным оборудованием)
text
[FreePBX на Raspberry Pi] ←→ [Физический SIP-телефон]
    ↓
[Домашний роутер] ←→ [SIP-провайдер (Zadarma/Telphin)]
```

#### Пошаговый план настройки (виртуальный вариант)
Шаг 1: Установка FreePBX
Скачать ISO с freepbx.org виртуалка

Шаг 2: Базовая настройка через веб-интерфейс
Открыть http://[IP-адрес-FreePBX]

Войти с логином admin и паролем из установки

Обновить систему через Module Admin

Шаг 3: Создание внутренних номеров (Extensions)
```
Applications → Extensions → Add Extension
- Extension: 101
- Display Name: User 1
- Secret: [пароль]
- Context: from-internal
```

Шаг 4: Настройка SIP-клиентов

Для Windows:

Установить MicroSIP (microsip.org)

Настройка аккаунта:

Сервер: IP FreePBX

Логин: 101

Пароль: [из настроек extension]

Для Android:

Установить CSipSimple или Zoiper

Аналогичные настройки аккаунта

Шаг 5: Настройка Dialplan (правила набора)
```
Connectivity → Inbound Routes → Add
- Description: Internal Calls
- DID Number: 101
- Destination: Extension 101
```


