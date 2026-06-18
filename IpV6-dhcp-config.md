# Настройка DHCPv6 на Cisco Router

## Что такое DHCPv6

DHCPv6 (Dynamic Host Configuration Protocol for IPv6) — это протокол автоматической выдачи сетевых параметров устройствам в IPv6-сетях.

С помощью DHCPv6 клиент может получить:

* IPv6-адрес;
* DNS-сервер;
* доменное имя;
* дополнительные параметры сети.

В IPv6 DHCP работает совместно с механизмом Router Advertisement (RA), который позволяет маршрутизатору сообщать клиентам способ получения сетевых настроек.

---

# Типы настройки IPv6-клиентов

Существует три основных способа автоматической настройки IPv6.

### SLAAC

SLAAC (Stateless Address Autoconfiguration) позволяет клиенту самостоятельно сформировать IPv6-адрес на основе префикса, полученного от маршрутизатора.

```text id="slaac1"
Router → Prefix 2001:DB8:1::/64

PC → 2001:DB8:1::A12F:4BFF:FE89:1234
```

DHCPv6 при этом не используется.

---

### Stateless DHCPv6

Клиент самостоятельно формирует IPv6-адрес через SLAAC.

DHCPv6 используется только для получения дополнительных параметров:

* DNS-серверов;
* доменного имени;
* других сетевых опций.

---

### Stateful DHCPv6

DHCPv6-сервер выдает клиенту полный IPv6-адрес и остальные сетевые параметры.

Работает аналогично DHCP в IPv4.

---

# Включение IPv6-маршрутизации

Перед настройкой DHCPv6 необходимо включить поддержку IPv6.

```bash id="ipv6route1"
R1(config)# ipv6 unicast-routing
```

Без этой команды маршрутизатор не будет отправлять Router Advertisement.

---

# Настройка IPv6-адреса интерфейса

Назначим IPv6-адрес интерфейсу GigabitEthernet0/0.

```bash id="intipv61"
R1(config)# interface gigabitethernet 0/0

R1(config-if)# ipv6 address 2001:DB8:1::1/64

R1(config-if)# no shutdown
```

Проверка:

```bash id="showipv61"
R1# show ipv6 interface brief
```

Пример вывода:

```text id="ipv6brief1"
GigabitEthernet0/0     [up/up]
    FE80::1
    2001:DB8:1::1
```

Каждый IPv6-интерфейс автоматически получает Link-Local адрес из диапазона FE80::/10.

---

# Настройка Stateless DHCPv6

Создаем DHCPv6-пул:

```bash id="pool1"
R1(config)# ipv6 dhcp pool IPV6-POOL
```

Указываем DNS-сервер:

```bash id="dns1"
R1(config-dhcpv6)# dns-server 2001:4860:4860::8888
```

Указываем доменное имя:

```bash id="domain1"
R1(config-dhcpv6)# domain-name lab.local
```

Выходим из режима настройки пула:

```bash id="exit1"
R1(config-dhcpv6)# exit
```

Привязываем пул к интерфейсу:

```bash id="bind1"
R1(config)# interface gigabitethernet 0/0

R1(config-if)# ipv6 dhcp server IPV6-POOL
```

Указываем клиентам получать дополнительные параметры через DHCPv6:

```bash id="otherflag1"
R1(config-if)# ipv6 nd other-config-flag
```

---

# Что делает other-config-flag

Команда:

```bash id="otherflag2"
ipv6 nd other-config-flag
```

Сообщает клиентам:

> IPv6-адрес формируется через SLAAC, а дополнительные параметры необходимо получить через DHCPv6.

---

# Настройка Stateful DHCPv6

Создаем DHCPv6-пул:

```bash id="statepool1"
R1(config)# ipv6 dhcp pool IPV6-STATEFUL
```

Указываем префикс и время аренды:

```bash id="prefix1"
R1(config-dhcpv6)# address prefix 2001:DB8:10::/64 lifetime infinite infinite
```

Указываем DNS-сервер:

```bash id="dns2"
R1(config-dhcpv6)# dns-server 2001:4860:4860::8888
```

Указываем доменное имя:

```bash id="domain2"
R1(config-dhcpv6)# domain-name lab.local
```

Выходим из режима настройки пула:

```bash id="exit2"
R1(config-dhcpv6)# exit
```

Назначаем интерфейсу IPv6-адрес:

```bash id="intstate1"
R1(config)# interface gigabitethernet 0/0

R1(config-if)# ipv6 address 2001:DB8:10::1/64
```

Отключаем SLAAC для данного префикса:

```bash id="noautoconfig1"
R1(config-if)# ipv6 nd prefix 2001:DB8:10::/64 no-autoconfig
```

Привязываем DHCPv6-пул:

```bash id="dhcpbind1"
R1(config-if)# ipv6 dhcp server IPV6-STATEFUL
```

Сообщаем клиентам использовать DHCPv6 для получения адресов:

```bash id="managed1"
R1(config-if)# ipv6 nd managed-config-flag
```

---

# Что делает managed-config-flag

Команда:

```bash id="managed2"
ipv6 nd managed-config-flag
```

Сообщает клиентам:

> IPv6-адрес необходимо получать через DHCPv6-сервер.

---

# Что делает no-autoconfig

Команда:

```bash id="noauto2"
ipv6 nd prefix 2001:DB8:10::/64 no-autoconfig
```

Запрещает клиентам использовать SLAAC для данного префикса.

Это предотвращает появление одновременно SLAAC-адреса и DHCPv6-адреса на одном устройстве.

---

# Проверка DHCPv6

Просмотр пулов:

```bash id="verify1"
R1# show ipv6 dhcp pool
```

Просмотр клиентов:

```bash id="verify2"
R1# show ipv6 dhcp binding
```

Просмотр параметров интерфейса:

```bash id="verify3"
R1# show ipv6 interface gigabitethernet 0/0
```

Просмотр Router Advertisement:

```bash id="verify4"
R1# show ipv6 interface
```

---

# Полный пример Stateless DHCPv6

```bash id="fullstateless"
enable
configure terminal

ipv6 unicast-routing

ipv6 dhcp pool IPV6-POOL
 dns-server 2001:4860:4860::8888
 domain-name lab.local

interface gigabitethernet 0/0
 ipv6 address 2001:DB8:1::1/64
 ipv6 dhcp server IPV6-POOL
 ipv6 nd other-config-flag
 no shutdown

end

copy running-config startup-config
```

---

# Полный пример Stateful DHCPv6

```bash id="fullstateful"
enable
configure terminal

ipv6 unicast-routing

ipv6 dhcp pool IPV6-STATEFUL
 address prefix 2001:DB8:10::/64 lifetime infinite infinite
 dns-server 2001:4860:4860::8888
 domain-name lab.local

interface gigabitethernet 0/0
 ipv6 address 2001:DB8:10::1/64
 ipv6 nd prefix 2001:DB8:10::/64 no-autoconfig
 ipv6 dhcp server IPV6-STATEFUL
 ipv6 nd managed-config-flag
 no shutdown

end

copy running-config startup-config
```

---

# Основные команды проверки

```bash id="verifyall"
show ipv6 interface brief

show ipv6 dhcp pool

show ipv6 dhcp binding

show ipv6 interface

show running-config
```

---

# Итог

После выполнения данной инструкции маршрутизатор сможет:

✅ Работать с IPv6

✅ Отправлять Router Advertisement

✅ Выдавать DNS-серверы через DHCPv6

✅ Поддерживать Stateless DHCPv6

✅ Выдавать IPv6-адреса через Stateful DHCPv6

✅ Исключать дублирование адресов при использовании Stateful DHCPv6

✅ Автоматически настраивать IPv6-клиентов

✅ Сохранять настройки после перезагрузки
