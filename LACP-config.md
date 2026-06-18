# Настройка LACP на Cisco Switch

## Что такое LACP

LACP (Link Aggregation Control Protocol) — это протокол объединения нескольких физических каналов в один логический канал.

LACP является стандартом IEEE 802.3ad (позже IEEE 802.1AX).

На Cisco объединенный интерфейс называется Port-Channel.

---

## Для чего нужен LACP

LACP позволяет:

* увеличивать пропускную способность;
* обеспечивать резервирование каналов;
* автоматически обнаруживать ошибки подключения;
* балансировать нагрузку между несколькими линиями.

Пример без LACP:

```text
SW1 -------- 1 Gbps -------- SW2
```

Пропускная способность: 1 Gbps

Пример с LACP:

```text
SW1 ======== 4 x 1 Gbps ======== SW2
```

Общий Port-Channel: 4 Gbps

---

## Как работает LACP

LACP объединяет несколько физических интерфейсов в одну логическую группу.

Коммутаторы обмениваются специальными LACP-пакетами и согласовывают параметры канала.

Если один из физических линков выйдет из строя, остальные продолжат работать.

---

## Балансировка нагрузки в EtherChannel

Важно понимать, что EtherChannel не делит один TCP-сеанс одновременно по нескольким физическим каналам.

Балансировка выполняется путем вычисления хеша на основе определенных полей пакета.

В зависимости от модели коммутатора могут использоваться:

* MAC-адреса;
* IP-адреса;
* TCP/UDP-порты;
* комбинации нескольких параметров.

Пример:

```text
Поток 1 → Gi0/1
Поток 2 → Gi0/2
Поток 3 → Gi0/3
Поток 4 → Gi0/4
```

Один конкретный поток всегда использует только один физический линк.

### Просмотр текущего алгоритма балансировки

```cisco
SW1# show etherchannel load-balance
```

Пример вывода:

```text
EtherChannel Load-Balancing Configuration:

src-mac
```

### Изменение алгоритма балансировки

Для более эффективного распределения нагрузки рекомендуется использовать IP-адреса:

```cisco
SW1(config)# port-channel load-balance src-dst-ip
```

На некоторых платформах доступны дополнительные варианты:

```cisco
SW1(config)# port-channel load-balance src-dst-port

SW1(config)# port-channel load-balance src-dst-mixed-ip-port
```

Доступные варианты зависят от модели коммутатора и версии IOS.

### Почему это важно

Если используется балансировка:

```text
src-mac
```

и весь трафик идет через один шлюз, то большинство пакетов может попадать только на один физический интерфейс.

Пример:

```text
Gi0/1 = 1 Gbps
Gi0/2 = почти 0
Gi0/3 = почти 0
Gi0/4 = почти 0
```

Балансировка по IP-адресам или TCP/UDP-портам обычно обеспечивает более равномерное распределение нагрузки.

---

## Режимы работы LACP

### Active

Активный режим.

Интерфейс самостоятельно инициирует создание LACP-соединения.

Пример:

```cisco
channel-group 1 mode active
```

### Passive

Пассивный режим.

Интерфейс ожидает инициативу от другой стороны.

Пример:

```cisco
channel-group 1 mode passive
```

---

## Сочетания режимов

### Active + Active

Работает.

```text
SW1 (Active) <----> SW2 (Active)
```

### Active + Passive

Работает.

```text
SW1 (Active) <----> SW2 (Passive)
```

### Passive + Passive

Не работает.

Ни одна сторона не инициирует создание канала.

```text
SW1 (Passive) <----> SW2 (Passive)
```

Port-Channel не поднимется.

---

## Схема работы

```text
          Port-Channel 1

      +--------+      +--------+
      |  SW1   |======|  SW2   |
      +--------+      +--------+

       Gi0/1 ========= Gi0/1
       Gi0/2 ========= Gi0/2
       Gi0/3 ========= Gi0/3
       Gi0/4 ========= Gi0/4
```

Четыре физических интерфейса образуют один логический интерфейс Port-Channel1.

---

## Проверка совместимости интерфейсов

Перед объединением интерфейсов необходимо убедиться, что параметры совпадают:

* скорость (Speed);
* Duplex;
* MTU;
* режим Access или Trunk;
* список разрешенных VLAN;
* Native VLAN.

Несовпадение параметров может привести к состояниям:

```text
suspended
individual
err-disabled
```

и исключению интерфейса из Port-Channel.

---

# Настройка LACP (Best Practice)

## Шаг 1. Очистка физических интерфейсов

Перед созданием EtherChannel рекомендуется сбросить конфигурацию физических портов.

```cisco
SW1(config)# default interface range gigabitethernet 0/1 - 4
```

Для старых версий IOS:

```cisco
SW1(config)# interface range gigabitethernet 0/1 - 4

SW1(config-if-range)# shutdown

SW1(config-if-range)# no switchport access vlan

SW1(config-if-range)# no switchport trunk allowed vlan

SW1(config-if-range)# no switchport trunk native vlan
```

---

## Шаг 2. Создание интерфейса Port-Channel

```cisco
SW1(config)# interface port-channel 1
```

---

## Шаг 3. Настройка Port-Channel

### Вариант Access

Для передачи только одной VLAN:

```cisco
SW1(config-if)# switchport mode access

SW1(config-if)# switchport access vlan 10
```

### Что делает Access Port-Channel

Все физические интерфейсы будут работать как один access-порт VLAN 10.

---

### Вариант Trunk

Для передачи нескольких VLAN:

```cisco
SW1(config-if)# switchport mode trunk

SW1(config-if)# switchport trunk allowed vlan 10,20,30
```

### Что делает Trunk Port-Channel

Коммутатор будет передавать VLAN 10, 20 и 30 через агрегированный канал.

---

## Шаг 4. Добавление физических интерфейсов в группу

```cisco
SW1(config)# interface range gigabitethernet 0/1 - 4

SW1(config-if-range)# channel-group 1 mode active

SW1(config-if-range)# no shutdown
```

### Что делает команда

```cisco
channel-group 1 mode active
```

Добавляет интерфейс в Port-Channel 1 и включает LACP в активном режиме.

---

## Почему важен такой порядок

После выполнения команды:

```cisco
channel-group 1 mode active
```

Cisco немедленно создает интерфейс Port-Channel и наследует текущие настройки физических интерфейсов.

Если порты имеют неподходящие настройки, могут возникнуть состояния:

```text
suspended
err-disabled
```

Поэтому рекомендуется:

* сначала очистить интерфейсы;
* настроить Port-Channel;
* только потом добавлять физические интерфейсы в группу.

После объединения физических интерфейсов все настройки L2 должны выполняться только на Port-Channel.

Правильно:

```cisco
interface port-channel 1

 switchport mode trunk

 switchport trunk allowed vlan 10,20,30
```

Неправильно:

```cisco
interface gigabitethernet 0/1

 switchport mode trunk
```

---

## Настройка второй стороны

На втором коммутаторе выполняется аналогичная настройка.

Пример:

```cisco
SW2(config)# default interface range gigabitethernet 0/1 - 4

SW2(config)# interface port-channel 1

SW2(config-if)# switchport mode trunk

SW2(config-if)# switchport trunk allowed vlan 10,20,30

SW2(config)# interface range gigabitethernet 0/1 - 4

SW2(config-if-range)# channel-group 1 mode active
```

Либо:

```cisco
SW2(config-if-range)# channel-group 1 mode passive
```

---

# Проверка LACP

## Проверка состояния EtherChannel

```cisco
SW1# show etherchannel summary
```

Пример:

```text
Group  Port-channel  Protocol  Ports
------+-------------+---------+----------------
1      Po1(SU)      LACP      Gi0/1(P)
                               Gi0/2(P)
                               Gi0/3(P)
                               Gi0/4(P)
```

Расшифровка:

```text
S = Layer2
R = Layer3
U = In Use
P = Bundled in Port-Channel
```

---

## Проверка LACP-соседа

```cisco
SW1# show lacp neighbor
```

---

## Проверка Port-Channel

```cisco
SW1# show interfaces port-channel 1
```

---

## Проверка конфигурации

```cisco
SW1# show running-config
```

---

# Полный пример настройки

## Схема

```text
SW1 Gi0/1-Gi0/4
        ||
        ||
SW2 Gi0/1-Gi0/4

VLAN 10,20,30
```

## Настройка SW1

```cisco
enable

configure terminal

port-channel load-balance src-dst-ip

default interface range gigabitethernet 0/1 - 4

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

interface range gigabitethernet 0/1 - 4
 channel-group 1 mode active
 no shutdown

end

copy running-config startup-config
```

## Настройка SW2

```cisco
enable

configure terminal

port-channel load-balance src-dst-ip

default interface range gigabitethernet 0/1 - 4

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

interface range gigabitethernet 0/1 - 4
 channel-group 1 mode active
 no shutdown

end

copy running-config startup-config
```

---

# Основные команды проверки

```cisco
show etherchannel summary

show etherchannel load-balance

show etherchannel port-channel

show lacp neighbor

show interfaces port-channel 1

show running-config
```

---

# Типичные ошибки

## Port-Channel не поднимается

Причина:

Обе стороны настроены в режиме Passive.

Проверка:

```cisco
show etherchannel summary
```

Решение:

```cisco
channel-group 1 mode active
```

Хотя бы на одной стороне.

---

## Интерфейс не входит в Port-Channel

Причина:

Различаются настройки интерфейсов.

Проверка:

```cisco
show run interface gi0/1

show run interface gi0/2
```

Проверить:

* Speed;
* Duplex;
* MTU;
* VLAN;
* Trunk;
* Native VLAN.

---

## Часть линков не используется

Причина:

Интерфейсы находятся в состоянии Suspended.

Проверка:

```cisco
show etherchannel summary
```

Решение:

Проверить:

* VLAN;
* Trunk;
* MTU;
* Speed;
* Duplex;
* алгоритм балансировки.

---

## Весь трафик идет по одному кабелю

Причина:

Используется неподходящий алгоритм балансировки.

Проверка:

```cisco
show etherchannel load-balance
```

Решение:

```cisco
conf t

port-channel load-balance src-dst-ip
```

или другой алгоритм, подходящий для вашей платформы.

---

## Потеря связи после настройки

Причина:

Настройки были выполнены на физических интерфейсах вместо Port-Channel.

Правильно:

```cisco
interface port-channel 1

 switchport mode trunk
```

Неправильно:

```cisco
interface gi0/1

 switchport mode trunk
```

---

# Итог

После выполнения данной инструкции коммутатор сможет:

✅ Объединять несколько физических каналов в один логический канал

✅ Увеличивать пропускную способность между устройствами

✅ Обеспечивать отказоустойчивость соединения

✅ Автоматически обнаруживать ошибки линков

✅ Балансировать нагрузку между интерфейсами

✅ Использовать оптимальный алгоритм распределения трафика

✅ Передавать несколько VLAN через агрегированный канал

✅ Поддерживать стандартизированный протокол LACP

✅ Сохранять работоспособность при отказе одного из каналов

✅ Сохранять настройки после перезагрузки
