# Настройка DHCP Snooping на Cisco Switch

## Что такое DHCP Snooping

DHCP Snooping — это механизм безопасности коммутаторов Cisco, предназначенный для защиты сети от несанкционированных DHCP-серверов (Rogue DHCP Server).

Функция анализирует DHCP-трафик и разрешает передачу DHCP-ответов только через доверенные (Trusted) порты.

---

# Для чего нужен DHCP Snooping

Без DHCP Snooping любой пользователь может подключить собственный DHCP-сервер.

Например:

```text
Легальный DHCP-сервер
IP: 192.168.1.1

Злоумышленник запускает DHCP-сервер
IP: 192.168.1.254
```

Если клиент получит настройки от злоумышленника, он может:

* потерять доступ к сети;
* использовать неверный шлюз;
* использовать поддельный DNS-сервер;
* передавать весь трафик через атакующего.

---

# Как работает DHCP Snooping

Коммутатор разделяет порты на два типа.

### Trusted

Доверенные порты.

Через них разрешены все DHCP-сообщения:

* Discover
* Offer
* Request
* ACK

Обычно Trusted применяется:

* к порту DHCP-сервера;
* к аплинку на маршрутизатор;
* к транковым каналам между коммутаторами.

> Если DHCP-сервер находится за другим коммутатором, все транковые порты на пути DHCP-трафика должны быть настроены как Trusted. Иначе DHCP Offer и DHCP ACK будут блокироваться.

---

### Untrusted

Недоверенные порты.

По умолчанию все порты являются Untrusted.

С таких портов разрешены только клиентские запросы:

* Discover
* Request

DHCP-ответы:

* Offer
* ACK

будут автоматически блокироваться.

---

# Схема работы

```text
                DHCP Server
                      |
                      |
               Gi0/1 (Trusted)
                      |
                +-----------+
                |  Switch   |
                +-----------+
                 |         |
                 |         |
      Fa0/1      |         |      Fa0/2
   (Untrusted)   |         |   (Untrusted)

               PC1       PC2
```

Только порт Gi0/1 имеет право передавать DHCP-ответы.

---

# Включение DHCP Snooping

Переходим в режим глобальной настройки:

```bash
SW1# configure terminal
```

Включаем DHCP Snooping:

```bash
SW1(config)# ip dhcp snooping
```

---

# Что делает ip dhcp snooping

Команда:

```bash
ip dhcp snooping
```

Активирует механизм DHCP Snooping на коммутаторе.

Без этой команды остальные настройки работать не будут.

---

# Включение DHCP Snooping для VLAN

DHCP Snooping работает только в указанных VLAN.

Например:

```bash
SW1(config)# ip dhcp snooping vlan 10
```

Для нескольких VLAN:

```bash
SW1(config)# ip dhcp snooping vlan 10,20,30
```

---

# Что делает ip dhcp snooping vlan

Команда:

```bash
ip dhcp snooping vlan 10
```

Сообщает коммутатору анализировать DHCP-трафик внутри VLAN 10.

---

# Настройка Option 82

По умолчанию многие коммутаторы Cisco добавляют в DHCP-пакеты специальное поле Option 82.

Некоторые DHCP-серверы могут отклонять такие запросы.

Для максимальной совместимости рекомендуется отключить добавление Option 82:

```bash
SW1(config)# no ip dhcp snooping information option
```

---

# Что делает no ip dhcp snooping information option

Команда:

```bash
no ip dhcp snooping information option
```

Запрещает коммутатору автоматически добавлять Option 82 в DHCP-пакеты.

Это помогает избежать проблем совместимости с DHCP-серверами сторонних производителей.

---

# Настройка Trusted-порта

Предположим, DHCP-сервер подключен к интерфейсу GigabitEthernet0/1.

Переходим на интерфейс:

```bash
SW1(config)# interface gigabitethernet 0/1
```

Назначаем доверенным:

```bash
SW1(config-if)# ip dhcp snooping trust
```

---

# Что делает ip dhcp snooping trust

Команда:

```bash
ip dhcp snooping trust
```

Разрешает прохождение DHCP-ответов через данный интерфейс.

Обычно применяется:

* к порту DHCP-сервера;
* к транковым соединениям;
* к аплинкам на маршрутизаторы.

---

# Настройка пользовательских портов

Пользовательские порты оставляют в состоянии Untrusted.

Например:

```bash
SW1(config)# interface range fastethernet 0/1 - 24
```

Дополнительных команд не требуется.

По умолчанию они уже являются Untrusted.

---

# Ограничение DHCP-пакетов

Для защиты от DHCP Starvation и DHCP Flood можно ограничить количество DHCP-пакетов.

Настроим порт Fa0/5:

```bash
SW1(config)# interface fastethernet 0/5

SW1(config-if)# ip dhcp snooping limit rate 10
```

---

# Что делает limit rate

Команда:

```bash
ip dhcp snooping limit rate 10
```

Разрешает не более 10 DHCP-пакетов в секунду.

При превышении лимита интерфейс может перейти в состояние err-disabled.

---

# DHCP Snooping Binding Table

DHCP Snooping создает таблицу соответствий:

```text
MAC Address
IP Address
VLAN
Interface
Lease Time
```

Пример:

```text
00D0.BA11.1234
192.168.1.100
VLAN 10
Fa0/5
86300 sec
```

Эта таблица используется другими механизмами безопасности Cisco:

* Dynamic ARP Inspection (DAI)
* IP Source Guard

---

# Проверка DHCP Snooping

Просмотр состояния DHCP Snooping:

```bash
SW1# show ip dhcp snooping
```

Просмотр таблицы привязок:

```bash
SW1# show ip dhcp snooping binding
```

Просмотр статистики:

```bash
SW1# show ip dhcp snooping statistics
```

---

# Полный пример настройки

Предположим:

```text
DHCP Server -> Gi0/1

Clients -> Fa0/1-Fa0/24

Рабочая VLAN -> VLAN 10
```

Конфигурация:

```bash
enable

configure terminal

ip dhcp snooping

ip dhcp snooping vlan 10

no ip dhcp snooping information option

interface gigabitethernet 0/1
 ip dhcp snooping trust

interface range fastethernet 0/1 - 24
 ip dhcp snooping limit rate 10

end

copy running-config startup-config
```

---

# Основные команды проверки

```bash
show ip dhcp snooping

show ip dhcp snooping binding

show ip dhcp snooping statistics

show running-config
```

---

# Типичные ошибки

### DHCP-сервер не работает

Причина:

Trusted-порт не настроен.

Проверить:

```bash
show ip dhcp snooping
```

---

### Клиенты не получают адрес

Причина:

DHCP Snooping включен не для той VLAN.

Проверить:

```bash
show ip dhcp snooping
```

---

### Клиенты не получают адрес после включения DHCP Snooping

Причина:

DHCP-сервер отклоняет пакеты с Option 82.

Проверить:

```bash
show ip dhcp snooping
```

Если отображается:

```text
DHCP snooping information option insertion is enabled
```

отключить:

```bash
configure terminal

no ip dhcp snooping information option
```

---

### Порт перешел в err-disabled

Причина:

Превышен лимит DHCP-пакетов.

Проверить:

```bash
show interfaces status
```

---

# Итог

После выполнения данной инструкции коммутатор сможет:

✅ Блокировать несанкционированные DHCP-серверы

✅ Разделять порты на Trusted и Untrusted

✅ Защищать сеть от Rogue DHCP

✅ Ограничивать DHCP Flood и DHCP Starvation атаки

✅ Формировать DHCP Snooping Binding Table

✅ Поддерживать работу Dynamic ARP Inspection

✅ Поддерживать работу IP Source Guard

✅ Повышать безопасность локальной сети

✅ Сохранять настройки после перезагрузки
