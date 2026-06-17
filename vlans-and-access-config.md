# Настройка VLAN и Trunk-портов на Cisco Switch

## Что такое VLAN

VLAN (Virtual Local Area Network) — технология логического разделения одной физической сети на несколько независимых сетей.

Например, один коммутатор может одновременно обслуживать несколько отделов организации:

| VLAN    | Назначение   |
| ------- | ------------ |
| VLAN 10 | Бухгалтерия  |
| VLAN 20 | Отдел продаж |
| VLAN 30 | IT-отдел     |

Устройства из разных VLAN находятся в разных широковещательных доменах и не могут обмениваться данными напрямую без маршрутизации.

---

# Что такое Access-порт

Access-порт используется для подключения конечных устройств:

* компьютеров;
* принтеров;
* IP-телефонов;
* серверов.

Access-порт принадлежит только одной VLAN.

Пример:

```text id="jlwmk6"
PC1 ---- Fa0/1 [Switch]
```

Если порт находится во VLAN 10, весь трафик через него будет относиться к VLAN 10.

---

# Что такое Trunk-порт

Trunk-порт используется для передачи трафика нескольких VLAN через одно физическое соединение.

Обычно транковые каналы используются между:

* двумя коммутаторами;
* коммутатором и маршрутизатором;
* коммутатором и гипервизором (VMware, Proxmox, Hyper-V).

Пример:

```text id="upw7s8"
Switch1 ===== Gi0/1 ===== Switch2
                Trunk
```

Через один кабель одновременно могут передаваться VLAN 10, VLAN 20, VLAN 30 и другие.

---

# Как работает Trunk

Для передачи нескольких VLAN через один интерфейс используется стандарт IEEE 802.1Q.

Перед отправкой кадра коммутатор добавляет специальный VLAN-тег.

Пример:

```text id="58xv0q"
VLAN 10 → тег VLAN 10
VLAN 20 → тег VLAN 20
VLAN 30 → тег VLAN 30
```

При получении кадра коммутатор анализирует тег и определяет, к какой VLAN относится трафик.

---

# Создание VLAN

Создадим несколько VLAN:

```bash id="6i89ib"
Switch(config)# vlan 10
Switch(config-vlan)# name OFFICE

Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name SERVERS

Switch(config-vlan)# exit

Switch(config)# vlan 999
Switch(config-vlan)# name NATIVE
```

Проверим результат:

```bash id="nq8r2m"
Switch# show vlan brief
```

Пример вывода:

```text id="okyyb2"
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3, Fa0/4, Fa0/5
10   OFFICE                           active
20   SERVERS                          active
999  NATIVE                           active
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                    active
```

---

# Настройка Access-порта

Назначим интерфейс FastEthernet0/1 во VLAN 10.

```bash id="7bxqdc"
Switch(config)# interface fastethernet 0/1

Switch(config-if)# switchport mode access

Switch(config-if)# switchport access vlan 10
```

### Что делают команды

| Команда                   | Назначение                    |
| ------------------------- | ----------------------------- |
| switchport mode access    | Переводит порт в режим Access |
| switchport access vlan 10 | Назначает VLAN 10             |

Теперь все устройства, подключенные к порту Fa0/1, будут находиться во VLAN 10.

---

# Настройка второго Access-порта

Назначим интерфейс FastEthernet0/2 во VLAN 20.

```bash id="z64n1j"
Switch(config)# interface fastethernet 0/2

Switch(config-if)# switchport mode access

Switch(config-if)# switchport access vlan 20
```

Получаем:

```text id="v6g5x5"
Fa0/1 → VLAN 10
Fa0/2 → VLAN 20
```

Теперь устройства подключены к разным логическим сетям.

---

# Настройка Trunk-порта

Предположим, что интерфейс GigabitEthernet0/1 соединяет два коммутатора.

На некоторых моделях Cisco перед переводом порта в режим Trunk необходимо указать тип инкапсуляции.

```bash id="3yqjpm"
Switch(config)# interface gigabitethernet 0/1

Switch(config-if)# switchport trunk encapsulation dot1q

Switch(config-if)# switchport mode trunk
```

> На многих современных коммутаторах Cisco (например, Catalyst 2960 и Packet Tracer) команда `switchport trunk encapsulation dot1q` отсутствует, поскольку устройство поддерживает только стандарт IEEE 802.1Q. В этом случае используется только команда `switchport mode trunk`.

---

# Ограничение VLAN на транке

По умолчанию через транк передаются все VLAN.

Для повышения безопасности рекомендуется разрешать только необходимые VLAN.

Например:

```bash id="1ojsva"
Switch(config-if)# switchport trunk allowed vlan 10,20
```

Теперь через транк будут передаваться только VLAN 10 и VLAN 20.

---

# Настройка Native VLAN

Native VLAN используется для кадров, передаваемых без тега 802.1Q.

По умолчанию Cisco использует:

```text id="s13v7m"
VLAN 1
```

В целях безопасности рекомендуется изменить Native VLAN.

Например:

```bash id="h0p9ns"
Switch(config-if)# switchport trunk native vlan 999
```

Важно:

Native VLAN должна совпадать на обоих концах транкового соединения.

Иначе коммутатор может вывести предупреждение:

```text id="xx8q7h"
%CDP-4-NATIVE_VLAN_MISMATCH
```

---

# Проверка настроек Trunk

Проверим состояние транковых интерфейсов:

```bash id="hqj4tw"
Switch# show interfaces trunk
```

Пример вывода:

```text id="4o02k5"
Port        Mode         Encapsulation  Status      Native vlan
Gi0/1       on           802.1q         trunking    999

Port        Vlans allowed on trunk
Gi0/1       10,20

Port        Vlans active in management domain
Gi0/1       10,20,999
```

---

# Проверка настроек VLAN

Просмотр VLAN:

```bash id="e5smcq"
Switch# show vlan brief
```

Просмотр транковых интерфейсов:

```bash id="x3mr9j"
Switch# show interfaces trunk
```

Просмотр параметров конкретного порта:

```bash id="oq8e7u"
Switch# show interfaces gigabitethernet 0/1 switchport
```

---

# Полный пример настройки

Создание VLAN:

```bash id="t68cl4"
vlan 10
 name OFFICE

vlan 20
 name SERVERS

vlan 999
 name NATIVE
```

Настройка Access-портов:

```bash id="h9ym1h"
interface fastethernet 0/1
 switchport mode access
 switchport access vlan 10

interface fastethernet 0/2
 switchport mode access
 switchport access vlan 20
```

Настройка Trunk-порта:

```bash id="xxq5e0"
interface gigabitethernet 0/1

 switchport trunk encapsulation dot1q
 switchport mode trunk

 switchport trunk allowed vlan 10,20

 switchport trunk native vlan 999
```

Сохранение конфигурации:

```bash id="wmyx4z"
copy running-config startup-config
```

---

# Итог

После выполнения данной инструкции коммутатор будет:

✅ Иметь несколько VLAN

✅ Разделять устройства на независимые логические сети

✅ Использовать Access-порты для подключения конечных устройств

✅ Использовать Trunk-порт для передачи нескольких VLAN

✅ Передавать только разрешенные VLAN через транк

✅ Использовать отдельную Native VLAN

✅ Поддерживать стандарт IEEE 802.1Q

✅ Сохранять настройки после перезагрузки
