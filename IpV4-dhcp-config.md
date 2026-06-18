# Настройка DHCPv4 на Cisco Router

## Что такое DHCP

DHCP (Dynamic Host Configuration Protocol) — это протокол автоматической выдачи сетевых параметров устройствам в IPv4-сетях.

С помощью DHCP клиент может получить:

* IPv4-адрес;
* маску подсети;
* шлюз по умолчанию;
* DNS-сервер;
* доменное имя;
* дополнительные сетевые параметры.

DHCP позволяет автоматически настраивать сетевые устройства без ручного ввода параметров.

---

# Принцип работы DHCP

При получении настроек клиент и сервер обмениваются сообщениями по схеме DORA:

1. Discover — поиск DHCP-сервера.
2. Offer — предложение параметров.
3. Request — запрос выбранных параметров.
4. Acknowledgment (ACK) — подтверждение выдачи адреса.

```text
+---------+                               +-------------+
| Client  |                               | DHCP Server |
+---------+                               +-------------+

     | -------- DHCP Discover --------> |
     | <--------- DHCP Offer ---------- |
     | -------- DHCP Request ---------> |
     | <---------- DHCP ACK ----------- |
```

---

# Настройка IPv4-адреса интерфейса

Назначим IPv4-адрес интерфейсу GigabitEthernet0/0.

```bash
R1(config)# interface gigabitethernet 0/0

R1(config-if)# ip address 192.168.1.1 255.255.255.0

R1(config-if)# no shutdown
```

Проверка:

```bash
R1# show ip interface brief
```

Пример вывода:

```text
Interface              IP-Address      OK? Method Status                Protocol

GigabitEthernet0/0     192.168.1.1     YES manual up                    up
```

---

# Исключение адресов из выдачи

Перед созданием DHCP-пула рекомендуется исключить адреса, которые будут использоваться вручную.

Например:

```bash
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.20
```

Данный диапазон не будет выдаваться клиентам.

---

# Настройка DHCP-пула

Создаем DHCP-пул:

```bash
R1(config)# ip dhcp pool LAN-POOL
```

Указываем сеть:

```bash
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
```

Указываем шлюз по умолчанию:

```bash
R1(dhcp-config)# default-router 192.168.1.1
```

Указываем DNS-сервер:

```bash
R1(dhcp-config)# dns-server 8.8.8.8
```

Указываем доменное имя:

```bash
R1(dhcp-config)# domain-name lab.local
```

Выходим из режима настройки пула:

```bash
R1(dhcp-config)# exit
```

---

# Что делает network

Команда:

```bash
network 192.168.1.0 255.255.255.0
```

Определяет диапазон адресов, которые DHCP может выдавать клиентам.

---

# Что делает default-router

Команда:

```bash
default-router 192.168.1.1
```

Передает клиентам адрес шлюза по умолчанию.

Через этот адрес осуществляется доступ к другим сетям и Интернету.

---

# Что делает dns-server

Команда:

```bash
dns-server 8.8.8.8
```

Передает клиентам адрес DNS-сервера для преобразования доменных имен в IP-адреса.

---

# Что делает domain-name

Команда:

```bash
domain-name lab.local
```

Передает клиентам DNS-домен, который может использоваться для поиска ресурсов внутри сети.

---

# Настройка времени аренды

По умолчанию Cisco использует аренду 1 день.

При необходимости можно изменить:

```bash
R1(config)# ip dhcp pool LAN-POOL

R1(dhcp-config)# lease 7 0 0
```

Формат команды:

```text
lease <дни> <часы> <минуты>
```

Пример:

```bash
lease 7 0 0
```

означает аренду адреса на 7 дней.

Также можно указать:

```bash
lease infinite
```

для бессрочной аренды (поддерживается не всеми версиями IOS).

---

# Проверка DHCP

Просмотр настроенных пулов:

```bash
R1# show ip dhcp pool
```

Просмотр выданных адресов:

```bash
R1# show ip dhcp binding
```

Просмотр статистики DHCP:

```bash
R1# show ip dhcp server statistics
```

---

# Полный пример DHCPv4

```bash
enable

configure terminal

interface gigabitethernet 0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

ip dhcp excluded-address 192.168.1.1 192.168.1.20

ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 domain-name lab.local
 lease 7 0 0

end

copy running-config startup-config
```

---

# Основные команды проверки

```bash
show ip interface brief

show ip dhcp pool

show ip dhcp binding

show ip dhcp server statistics

show running-config
```

---

# Итог

После выполнения данной инструкции маршрутизатор сможет:

✅ Работать как DHCP-сервер IPv4

✅ Автоматически выдавать IPv4-адреса клиентам

✅ Передавать маску подсети

✅ Передавать шлюз по умолчанию

✅ Передавать DNS-серверы

✅ Передавать доменное имя

✅ Контролировать время аренды адресов

✅ Исключать определенные адреса из выдачи

✅ Сохранять настройки после перезагрузки
