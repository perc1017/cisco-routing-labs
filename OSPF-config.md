# Настройка OSPF на Cisco

## Что такое OSPF

OSPF (Open Shortest Path First) — динамический протокол маршрутизации типа Link-State.

В отличие от RIP, OSPF строит полную карту сети и рассчитывает кратчайший путь до каждой сети с помощью алгоритма SPF (Shortest Path First), также известного как алгоритм Дейкстры.

RFC:

- RFC 2328 (OSPFv2 для IPv4)
- RFC 5340 (OSPFv3 для IPv6)

---

## Для чего нужен OSPF

OSPF позволяет:

- автоматически обмениваться маршрутами;
- быстро реагировать на изменения топологии;
- поддерживать большие сети;
- использовать VLSM и CIDR;
- выполнять балансировку нагрузки;
- строить иерархическую маршрутизацию через области (Area).

Без динамической маршрутизации:

```text
R1 ---- R2 ---- R3
```

Необходимо создавать статические маршруты.

С OSPF:

```text
R1 <---- OSPF ----> R2 <---- OSPF ----> R3
```

Маршруты распространяются автоматически.

---

## Как работает OSPF

OSPF является протоколом типа Link-State.

Каждый маршрутизатор:

1. Обнаруживает соседей.
2. Формирует соседство (Adjacency).
3. Обменивается информацией о состоянии каналов.
4. Создает LSDB (Link-State Database).
5. Запускает алгоритм SPF.
6. Формирует таблицу маршрутизации.

---

## Метрика OSPF

OSPF использует метрику:

### Cost

Формула:

```text
Cost = Reference Bandwidth / Interface Bandwidth
```

По умолчанию Cisco использует:

```text
100 Mbps
```

Пример:

| Скорость | Cost |
|-----------|------|
| 10 Mbps | 10 |
| 100 Mbps | 1 |
| 1 Gbps | 1 |
| 10 Gbps | 1 |

Из-за этого OSPF не различает современные каналы.

---

## Настройка Reference Bandwidth

### Почему это важно

По умолчанию все интерфейсы быстрее 100 Mbps получают одинаковый Cost = 1.

Это может привести к неправильному выбору маршрутов.

### Настройка

Для сети до 10 Gbps:

```cisco
R1(config)# router ospf 1
R1(config-router)# auto-cost reference-bandwidth 10000
```

Для сети до 40 Gbps:

```cisco
R1(config-router)# auto-cost reference-bandwidth 40000
```

Для сети до 100 Gbps:

```cisco
R1(config-router)# auto-cost reference-bandwidth 100000
```

### Важно

Команда должна быть настроена на всех маршрутизаторах OSPF-домена.

Проверка:

```cisco
R1# show ip ospf
```

---

## Преимущества OSPF

- Быстрая сходимость
- Поддержка больших сетей
- Поддержка VLSM
- Поддержка CIDR
- Балансировка нагрузки
- Поддержка многоуровневой архитектуры
- Открытый стандарт

---

## Типы пакетов OSPF

| Тип | Назначение |
|-------|------------|
| Hello | Поиск соседей |
| DBD | Database Description |
| LSR | Link State Request |
| LSU | Link State Update |
| LSAck | Link State Acknowledgment |

---

## OSPF Areas

OSPF использует области (Area).

Основная область:

```text
Area 0
```

Она называется Backbone Area.

Все остальные области должны иметь связь с Area 0.

Пример:

```text
          Area 1
             |
             |
Area 0 ------R2------ Area 2
```

---

## Router ID

Каждый маршрутизатор должен иметь уникальный Router ID.

Пример:

```text
1.1.1.1
2.2.2.2
3.3.3.3
```

Приоритет выбора:

1. router-id
2. Loopback интерфейс
3. Максимальный IP активного интерфейса

Рекомендуется всегда задавать Router ID вручную.

---

## Схема сети

```text
LAN1                          LAN3
192.168.1.0/24          192.168.3.0/24

      |                      |
      |                      |
     R1 ------- R2 ------- R3
        /30         /30

   192.168.12.0   192.168.23.0
```

---

# Настройка OSPF

## R1

```cisco
R1> enable
R1# configure terminal

R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1

R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 192.168.12.0 0.0.0.3 area 0

R1(config-router)# end
R1# copy running-config startup-config
```

## R2

```cisco
R2> enable
R2# configure terminal

R2(config)# router ospf 1
R2(config-router)# router-id 2.2.2.2

R2(config-router)# network 192.168.12.0 0.0.0.3 area 0
R2(config-router)# network 192.168.23.0 0.0.0.3 area 0

R2(config-router)# end
R2# copy running-config startup-config
```

## R3

```cisco
R3> enable
R3# configure terminal

R3(config)# router ospf 1
R3(config-router)# router-id 3.3.3.3

R3(config-router)# network 192.168.23.0 0.0.0.3 area 0
R3(config-router)# network 192.168.3.0 0.0.0.255 area 0

R3(config-router)# end
R3# copy running-config startup-config
```

---

## Современный способ включения OSPF

Рекомендуемый способ настройки.

Вместо команды network:

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip ospf 1 area 0
```

Для линка между маршрутизаторами:

```cisco
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip ospf 1 area 0
```

Преимущества:

- отсутствуют Wildcard Mask;
- меньше ошибок;
- проще сопровождение;
- лучше читается конфигурация.

---

## Wildcard Mask

При использовании команды network применяется Wildcard Mask.

| Маска | Wildcard |
|---------|-----------|
| /24 | 0.0.0.255 |
| /25 | 0.0.0.127 |
| /26 | 0.0.0.63 |
| /27 | 0.0.0.31 |
| /28 | 0.0.0.15 |
| /29 | 0.0.0.7 |
| /30 | 0.0.0.3 |

Пример:

```cisco
network 192.168.1.0 0.0.0.255 area 0
```

---

## Passive Interface

Для клиентских сетей:

```cisco
R1(config)# router ospf 1
R1(config-router)# passive-interface GigabitEthernet0/0
```

Маршрут будет распространяться, но Hello-пакеты отправляться не будут.

---

## Default Route

На граничном маршрутизаторе:

```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

Распространение:

```cisco
R1(config)# router ospf 1
R1(config-router)# default-information originate
```

---

## Аутентификация OSPF

MD5-аутентификация:

```cisco
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip ospf authentication message-digest
R1(config-if)# ip ospf message-digest-key 1 md5 StrongPassword123
```

Настройка должна совпадать на обеих сторонах.

---

## DR и BDR

На Ethernet-сетях OSPF выбирает:

- DR (Designated Router)
- BDR (Backup Designated Router)

Проверка:

```cisco
R1# show ip ospf neighbor
```

---

## Administrative Distance

| Источник | AD |
|-----------|----|
| Connected | 0 |
| Static | 1 |
| EIGRP | 90 |
| OSPF | 110 |
| RIP | 120 |

Если маршрут получен одновременно через RIP и OSPF, будет выбран OSPF.

---

## Типы маршрутов OSPF

Проверка:

```cisco
R1# show ip route ospf
```

### O — Intra-Area

Маршрут внутри одной Area.

```text
O 192.168.10.0/24 [110/20]
```

---

### O IA — Inter-Area

Маршрут получен из другой Area через ABR.

```text
O IA 192.168.20.0/24 [110/30]
```

---

### O E1 — External Type 1

Учитывается:

```text
External Cost + Internal Cost
```

---

### O E2 — External Type 2

Учитывается только внешний Cost.

Наиболее распространенный тип внешних маршрутов.

---

### O N1 — NSSA External Type 1

Маршрут из NSSA Area.

---

### O N2 — NSSA External Type 2

Маршрут из NSSA Area.

---

## Таблица обозначений

| Код | Значение |
|------|-----------|
| O | Intra-Area |
| O IA | Inter-Area |
| O E1 | External Type 1 |
| O E2 | External Type 2 |
| O N1 | NSSA External Type 1 |
| O N2 | NSSA External Type 2 |

---

## Проверка OSPF

Соседи:

```cisco
R1# show ip ospf neighbor
```

Интерфейсы:

```cisco
R1# show ip ospf interface brief
```

База LSDB:

```cisco
R1# show ip ospf database
```

Маршруты:

```cisco
R1# show ip route ospf
```

Параметры процесса:

```cisco
R1# show ip protocols
```

---

## Отладка

Соседство:

```cisco
R1# debug ip ospf adj
```

Пакеты:

```cisco
R1# debug ip ospf packet
```

Отключение:

```cisco
R1# undebug all
```

---

## Типичные ошибки

### Нет соседства

Проверить:

- Area
- Hello Timer
- Dead Timer
- Маски
- Аутентификацию

Проверка:

```cisco
show ip ospf neighbor
```

---

### Неверный Router ID

После изменения:

```cisco
R1# clear ip ospf process
```

---

### Ошибка Wildcard Mask

Неверно:

```cisco
network 192.168.1.0 255.255.255.0 area 0
```

Правильно:

```cisco
network 192.168.1.0 0.0.0.255 area 0
```

---

### Не распространяется Default Route

Отсутствует:

```cisco
default-information originate
```

или отсутствует маршрут по умолчанию.

---

### Разные Area

Соседство не сформируется.

---

## Best Practices

Рекомендуется:

✅ Настраивать Router ID вручную

✅ Использовать Interface-Level OSPF

✅ Настраивать auto-cost reference-bandwidth

✅ Использовать Area 0 как Backbone

✅ Настраивать Loopback-интерфейсы

✅ Использовать OSPF Authentication

✅ Настраивать passive-interface для клиентских сетей

✅ Проверять соседство через show ip ospf neighbor

Не рекомендуется:

❌ Использовать случайные Router ID

❌ Оставлять стандартный Reference Bandwidth

❌ Использовать неправильные Wildcard Mask

❌ Игнорировать DR/BDR

❌ Отключать аутентификацию в производственной сети

---

## Итог

После выполнения данной инструкции маршрутизаторы смогут:

✅ Автоматически обмениваться маршрутами

✅ Использовать алгоритм SPF

✅ Поддерживать VLSM и CIDR

✅ Работать с Area

✅ Использовать аутентификацию

✅ Распространять маршрут по умолчанию

✅ Автоматически реагировать на изменения топологии

✅ Поддерживать средние и крупные сети

✅ Сохранять настройки после перезагрузки
