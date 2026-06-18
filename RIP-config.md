# Настройка RIP на Cisco

## Что такое RIP

RIP (Routing Information Protocol) — динамический протокол маршрутизации типа Distance Vector.

RIP автоматически обменивается маршрутами между маршрутизаторами и позволяет строить таблицу маршрутизации без ручного добавления статических маршрутов.

RFC:

- RFC 1058 (RIP v1)
- RFC 2453 (RIP v2)

---

## Для чего нужен RIP

RIP позволяет:

- автоматически обмениваться маршрутами;
- уменьшить количество статических маршрутов;
- автоматически адаптироваться к изменениям топологии;
- обеспечивать резервирование маршрутов;
- упрощать настройку небольших сетей.

Без RIP:

```text
R1 ---- R2 ---- R3
```

На каждом маршрутизаторе необходимо вручную создавать статические маршруты.

С RIP:

```text
R1 <---- RIP ----> R2 <---- RIP ----> R3
```

Маршруты распространяются автоматически.

---

## Как работает RIP

RIP использует метрику:

### Hop Count

Количество маршрутизаторов до сети назначения.

Пример:

```text
R1 -> R2 -> R3 -> LAN
```

Метрика:

```text
3 Hops
```

Чем меньше значение, тем маршрут считается лучше.

---

## Ограничения RIP

| Параметр | Значение |
|-----------|-----------|
| Максимальная метрика | 15 Hops |
| Недоступная сеть | 16 Hops |

Поэтому RIP подходит только для небольших сетей.

---

## Версии RIP

### RIP Version 1

Особенности:

- Classful Routing
- Не передает маску подсети
- Не поддерживает VLSM
- Использует Broadcast

### RIP Version 2

Особенности:

- Поддержка VLSM
- Поддержка CIDR
- Передача маски подсети
- Поддержка аутентификации
- Multicast 224.0.0.9

Для современных сетей рекомендуется использовать только RIP Version 2.

---

## Обмен маршрутами

Каждые 30 секунд маршрутизатор отправляет соседям свою таблицу маршрутизации.

Процесс:

```text
Получение маршрута
        ↓
Увеличение метрики на 1
        ↓
Сравнение маршрутов
        ↓
Выбор лучшего маршрута
        ↓
Обновление таблицы маршрутизации
```

---

## Таймеры RIP

| Таймер | Значение |
|---------|-----------|
| Update Timer | 30 сек |
| Invalid Timer | 180 сек |
| Hold Down Timer | 180 сек |
| Flush Timer | 240 сек |

---

## Защита от петель маршрутизации

### Split Horizon

Маршрут не отправляется обратно через интерфейс, с которого был получен.

### Route Poisoning

Недоступный маршрут объявляется с метрикой:

```text
16
```

что означает:

```text
Network Unreachable
```

### Hold Down Timer

Временно запрещает принимать подозрительные обновления маршрутов.

---

## Особенность команды network

В отличие от OSPF, команда `network` в RIP работает по классовому принципу.

Пример:

Интерфейс:

```text
10.1.12.1/24
```

Конфигурация:

```cisco
R1(config)# router rip
R1(config-router)# network 10.1.12.0
```

Cisco интерпретирует это как:

```text
10.0.0.0
```

Нельзя использовать:

```cisco
network 10.1.12.0/24
```

или

```cisco
network 10.1.12.0 255.255.255.0
```

Проверка:

```cisco
R1# show ip protocols
```

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

# Настройка RIP Version 2

## R1

```cisco
R1> enable
R1# configure terminal

R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# no auto-summary
R1(config-router)# network 192.168.1.0
R1(config-router)# network 192.168.12.0
R1(config-router)# default-information originate

R1(config-router)# end
R1# copy running-config startup-config
```

## R2

```cisco
R2> enable
R2# configure terminal

R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# no auto-summary
R2(config-router)# network 192.168.12.0
R2(config-router)# network 192.168.23.0

R2(config-router)# end
R2# copy running-config startup-config
```

## R3

```cisco
R3> enable
R3# configure terminal

R3(config)# router rip
R3(config-router)# version 2
R3(config-router)# no auto-summary
R3(config-router)# network 192.168.23.0
R3(config-router)# network 192.168.3.0

R3(config-router)# end
R3# copy running-config startup-config
```

---

## Default Route

На R1:

```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

R1(config)# router rip
R1(config-router)# default-information originate
```

После этого R2 и R3 автоматически получат маршрут:

```text
0.0.0.0/0
```

---

## Passive Interface

```cisco
R1(config)# router rip
R1(config-router)# passive-interface GigabitEthernet0/0
```

Маршрут будет объявляться, но RIP-пакеты отправляться не будут.

---

## Аутентификация RIP

Создание ключа:

```cisco
R1(config)# key chain RIP-KEYS
R1(config-keychain)# key 1
R1(config-keychain-key)# key-string StrongPassword123
```

Настройка интерфейса:

```cisco
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip rip authentication mode md5
R1(config-if)# ip rip authentication key-chain RIP-KEYS
```

Настройка должна присутствовать на обеих сторонах канала.

---

## Administrative Distance

| Источник | AD |
|-----------|----|
| Connected | 0 |
| Static | 1 |
| EIGRP | 90 |
| OSPF | 110 |
| RIP | 120 |
| Unknown | 255 |

Пример:

```text
R 192.168.3.0/24 [120/2]
```

Расшифровка:

- R — маршрут RIP
- 120 — Administrative Distance
- 2 — Hop Count

---

## Проверка RIP

Таблица маршрутизации:

```cisco
R1# show ip route
```

Информация о протоколе:

```cisco
R1# show ip protocols
```

База маршрутов RIP:

```cisco
R1# show ip rip database
```

Отладка:

```cisco
R1# debug ip rip
```

Отключение:

```cisco
R1# undebug all
```

---

## Типичные ошибки

### Маршруты не появляются

Проверить:

```cisco
R1# show ip protocols
```

### Не работает VLSM

Используется RIP Version 1.

Решение:

```cisco
version 2
no auto-summary
```

### Не работает аутентификация

Разные MD5-ключи на соседних устройствах.

### Не распространяется маршрут по умолчанию

Отсутствует:

```cisco
default-information originate
```

или отсутствует сам маршрут:

```cisco
ip route 0.0.0.0 0.0.0.0 ...
```

### Медленная сходимость

Особенность RIP.

Для быстрой сходимости рекомендуется использовать OSPF или EIGRP.

### Более 15 маршрутизаторов

Ограничение RIP по Hop Count.

---

## Best Practices

Рекомендуется:

✅ Использовать только RIP Version 2

✅ Всегда использовать no auto-summary

✅ Настраивать passive-interface для клиентских сетей

✅ Использовать MD5 Authentication

✅ Распространять default route через default-information originate

✅ Проверять активные сети через show ip protocols

✅ Понимать влияние Administrative Distance

Не рекомендуется:

❌ Использовать RIP Version 1

❌ Использовать автоматическую суммаризацию

❌ Использовать RIP в крупных сетях

❌ Использовать RIP там, где требуется быстрая сходимость

❌ Игнорировать AD при совместной работе с OSPF или EIGRP

---

## Итог

После выполнения данной инструкции маршрутизаторы смогут:

✅ Автоматически обмениваться маршрутами

✅ Использовать RIP Version 2

✅ Поддерживать VLSM и CIDR

✅ Использовать MD5-аутентификацию

✅ Автоматически распространять маршрут по умолчанию

✅ Автоматически обнаруживать новые сети

✅ Автоматически удалять недоступные маршруты

✅ Корректно работать с современными схемами адресации

✅ Сохранять настройки после перезагрузки

✅ Обеспечивать динамическую маршрутизацию в небольших сетях и лабораторных стендах
