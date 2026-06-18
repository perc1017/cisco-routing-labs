# Настройка HSRP на Cisco

## Что такое HSRP

HSRP (Hot Standby Router Protocol) — это протокол отказоустойчивости первого перехода (First Hop Redundancy Protocol, FHRP), разработанный Cisco.

HSRP позволяет нескольким маршрутизаторам или L3-коммутаторам предоставлять клиентам единый виртуальный шлюз по умолчанию.

Если основное устройство выходит из строя, резервное автоматически принимает его роль без необходимости изменять настройки на клиентах.

---

## Для чего нужен HSRP

HSRP позволяет:

* обеспечить отказоустойчивость шлюза по умолчанию;
* минимизировать простой сети при отказе оборудования;
* автоматически переключать трафик на резервное устройство;
* обеспечить высокую доступность сервисов;
* выполнять обслуживание оборудования без изменения настроек клиентов.

Без HSRP:

```text
PC ---- SW1 (Gateway)

Если SW1 выйдет из строя,
клиенты потеряют доступ к сети.
```

С HSRP:

```text
               Virtual Gateway

                    |
                    |
          +---------+---------+
          |                   |
       Active              Standby

        SW1                 SW2
```

При отказе SW1 роль шлюза автоматически перейдет SW2.

---

## Как работает HSRP

Каждое устройство имеет собственный IP-адрес.

Пример:

```text
SW1 = 192.168.10.1

SW2 = 192.168.10.2
```

Дополнительно создается виртуальный IP-адрес:

```text
192.168.10.254
```

Клиенты используют только этот адрес:

```text
Default Gateway = 192.168.10.254
```

При отказе Active-устройства резервный узел начинает отвечать за этот IP.

Для клиентов процесс переключения остается незаметным.

---

## Роли устройств в HSRP

### Active

Основное устройство.

Отвечает за обработку пользовательского трафика.

---

### Standby

Резервное устройство.

Отслеживает состояние Active и готово принять его роль.

---

### Listen

Участвует в группе HSRP, но не является ни Active, ни Standby.

---

## HSRP Version 2 (Рекомендуется)

### Почему HSRPv1 считается устаревшим

HSRP Version 1 имеет ограничения:

* поддерживает только группы 0–255;
* ограничена масштабируемость;
* отсутствует полноценная поддержка IPv6;
* используется устаревший диапазон виртуальных MAC-адресов.

Пример MAC HSRPv1:

```text
0000.0C07.AC01
```

---

### Преимущества HSRPv2

HSRP Version 2 поддерживает:

* до 4096 групп;
* IPv6;
* улучшенную масштабируемость;
* новый диапазон виртуальных MAC-адресов.

Диапазон MAC-адресов HSRPv2:

```text
0000.0C9F.F000
—
0000.0C9F.FFFF
```

В современных сетях рекомендуется всегда использовать HSRPv2.

---

## Виртуальный IP и Virtual MAC

HSRP создает:

```text
Virtual IP

Virtual MAC
```

Пример:

```text
Virtual IP  = 192.168.10.254

Virtual MAC = 0000.0C9F.F00A
```

Для VLAN 10 и группы 10.

Все клиенты отправляют трафик на виртуальный MAC.

При смене Active-устройства виртуальный MAC остается неизменным.

---

## Схема работы

```text
                 HSRP Group 10

            Virtual IP .254

                    |
                    |
        +-----------+-----------+
        |                       |
     Active                 Standby

   SW1 (.1)               SW2 (.2)

        |                       |
        +-----------+-----------+
                    |
                 Clients
```

---

## Выбор Active-устройства

По умолчанию используется Priority.

Стандартное значение:

```text
100
```

Пример:

```text
SW1 Priority 110

SW2 Priority 100
```

Результат:

```text
SW1 = Active

SW2 = Standby
```

Побеждает устройство с наибольшим приоритетом.

---

## Что такое Preempt

Без Preempt устройство не вернет роль Active после восстановления.

Пример:

```text
SW1 Active

SW2 Standby
```

После отказа SW1:

```text
SW2 становится Active
```

После восстановления SW1:

```text
SW2 остается Active
```

Чтобы SW1 автоматически вернул себе роль:

```cisco
standby 10 preempt
```

---

## Настройка таймеров HSRP

### Таймеры по умолчанию

Стандартные значения:

```text
Hello = 3 секунды

Hold = 10 секунд
```

Это означает, что переключение может занять до 10 секунд.

Для корпоративных сетей это слишком долго.

---

### Ускоренная сходимость

Рекомендуемые значения:

```cisco
standby 10 timers msec 200 msec 600
```

Расшифровка:

```text
Hello = 200 мс

Hold = 600 мс
```

Переключение происходит менее чем за секунду.

---

## Object Tracking

### Для чего нужен Tracking

Иногда устройство остается доступным, но теряет uplink к ядру сети или Интернету.

Без Tracking такое устройство продолжит быть Active.

В результате клиенты потеряют связь.

---

### Создание объекта отслеживания

SW1:

```cisco
track 10 interface GigabitEthernet1/0/24 line-protocol
```

SW2:

```cisco
track 20 interface GigabitEthernet1/0/23 line-protocol
```

Каждое устройство отслеживает свой собственный uplink.

---

### Подключение Tracking к HSRP

SW1:

```cisco
standby 10 track 10 decrement 20
```

SW2:

```cisco
standby 10 track 20 decrement 20
```

---

### Как это работает

Исходное значение:

```text
Priority = 110
```

При отказе uplink:

```text
110 - 20 = 90
```

После этого резервное устройство становится Active.

---

## Аутентификация HSRP

### Почему это важно

По умолчанию HSRP Hello-пакеты не защищены.

Злоумышленник может отправить поддельные HSRP-пакеты и объявить себя Active-устройством.

Это позволяет выполнить атаку типа:

```text
Man-in-the-Middle
```

---

### Использование MD5 Authentication

Создаем Key Chain:

```cisco
key chain HSRP-KEYS

 key 1

  key-string StrongPassword123
```

Подключаем к HSRP:

```cisco
standby 10 authentication md5 key-chain HSRP-KEYS
```

Это рекомендуемый вариант для production-сетей.

---

## Настройка HSRP

### Исходные данные

```text
VLAN 10

SW1 = 192.168.10.1

SW2 = 192.168.10.2

Virtual IP = 192.168.10.254
```

В качестве Best Practice номер HSRP-группы соответствует номеру VLAN.

---

## Настройка SW1

```cisco
enable

configure terminal

track 10 interface GigabitEthernet1/0/24 line-protocol

key chain HSRP-KEYS
 key 1
  key-string StrongPassword123

interface vlan 10

 ip address 192.168.10.1 255.255.255.0

 standby version 2

 standby 10 ip 192.168.10.254

 standby 10 priority 110

 standby 10 preempt

 standby 10 timers msec 200 msec 600

 standby 10 authentication md5 key-chain HSRP-KEYS

 standby 10 track 10 decrement 20

 no shutdown

end

copy running-config startup-config
```

---

## Настройка SW2

```cisco
enable

configure terminal

track 20 interface GigabitEthernet1/0/23 line-protocol

key chain HSRP-KEYS
 key 1
  key-string StrongPassword123

interface vlan 10

 ip address 192.168.10.2 255.255.255.0

 standby version 2

 standby 10 ip 192.168.10.254

 standby 10 priority 100

 standby 10 preempt

 standby 10 timers msec 200 msec 600

 standby 10 authentication md5 key-chain HSRP-KEYS

 standby 10 track 20 decrement 20

 no shutdown

end

copy running-config startup-config
```

---

## Настройка нескольких VLAN

### VLAN 10

```cisco
interface vlan 10

 standby 10 ip 192.168.10.254
```

---

### VLAN 20

```cisco
interface vlan 20

 standby 20 ip 192.168.20.254
```

---

### VLAN 30

```cisco
interface vlan 30

 standby 30 ip 192.168.30.254
```

---

## Балансировка Active/Standby

Рекомендуется распределять нагрузку между коммутаторами.

Пример:

```text
VLAN10

SW1 Active

SW2 Standby
```

```text
VLAN20

SW2 Active

SW1 Standby
```

Таким образом используются ресурсы обоих устройств.

---

# Проверка HSRP

## Краткий статус

```cisco
show standby brief
```

Пример:

```text
Interface Grp Pri State Active Standby

Vl10      10 110 Active local 192.168.10.2
```

---

## Подробная информация

```cisco
show standby
```

---

## Проверка Tracking

```cisco
show track
```

---

## Проверка интерфейсов

```cisco
show ip interface brief
```

---

## Проверка конфигурации

```cisco
show running-config
```

---

# Основные команды проверки

```cisco
show standby

show standby brief

show track

show ip interface brief

show running-config
```

---

# Типичные ошибки

## Не происходит возврат Active

Причина:

Не настроен Preempt.

Решение:

```cisco
standby 10 preempt
```

---

## Устройства не видят друг друга

Причина:

Разные номера групп.

Проверить:

```cisco
show standby brief
```

Номер группы должен совпадать на обоих устройствах.

---

## Не работает Tracking

Причина:

Не создан объект Track.

Проверить:

```cisco
show track
```

---

## Не работает аутентификация

Причина:

Разные ключи MD5.

Проверить:

```cisco
show standby
```

---

## Медленное переключение

Причина:

Используются таймеры по умолчанию.

Решение:

```cisco
standby 10 timers msec 200 msec 600
```

---

## Клиенты теряют связь

Причина:

На клиентах указан физический адрес коммутатора вместо виртуального IP.

Правильно:

```text
192.168.10.254
```

---

# Итог

После выполнения данной инструкции сеть сможет:

✅ Использовать отказоустойчивый шлюз по умолчанию

✅ Автоматически переключаться на резервное устройство

✅ Использовать HSRP Version 2

✅ Поддерживать до 4096 групп HSRP

✅ Использовать безопасную MD5-аутентификацию

✅ Контролировать состояние uplink через Object Tracking

✅ Выполнять быстрое переключение за сотни миллисекунд

✅ Балансировать нагрузку между несколькими VLAN

✅ Минимизировать простой сети

✅ Сохранять настройки после перезагрузки
