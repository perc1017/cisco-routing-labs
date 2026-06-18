# Настройка ACL на Cisco

## Что такое ACL

ACL (Access Control List) — механизм фильтрации трафика на маршрутизаторах и коммутаторах Cisco.

ACL позволяет:

- разрешать трафик;
- запрещать трафик;
- ограничивать доступ к сетям и сервисам;
- защищать сетевую инфраструктуру;
- контролировать маршрутизацию и NAT;
- фильтровать управляющий доступ (SSH, Telnet, SNMP).

ACL проверяются сверху вниз.

Как только найдено совпадение, дальнейшая проверка прекращается.

---

## Принцип работы ACL

Логика обработки:

```text
Пакет поступает
       ↓
Проверка первой строки ACL
       ↓
Совпадение найдено?
      / \
    Да   Нет
    ↓      ↓
 Выполнить Следующая строка
 действие
```

После последней строки автоматически применяется скрытое правило:

```text
deny ip any any
```

Это называется:

```text
Implicit Deny
```

---

## Типы ACL

### Standard ACL

Фильтрация только по IP-адресу источника.

Можно определить:

- кто отправил пакет;
- нельзя определить получателя;
- нельзя определить протокол.

---

### Extended ACL

Фильтрация по:

- IP источника;
- IP назначения;
- протоколу;
- TCP-порту;
- UDP-порту;
- ICMP.

Позволяет создавать гибкие политики безопасности.

---

## Номера ACL

### Standard ACL

| Диапазон |
|-----------|
| 1-99 |
| 1300-1999 |

### Extended ACL

| Диапазон |
|-----------|
| 100-199 |
| 2000-2699 |

---

## Именованные ACL

Вместо номеров рекомендуется использовать имена.

Пример:

```cisco
R1(config)# ip access-list extended INTERNET-FILTER
```

Преимущества:

- проще читать;
- проще сопровождать;
- удобнее изменять.

---

## Wildcard Mask

ACL используют Wildcard Mask.

Это инвертированная маска подсети.

| Маска | Wildcard |
|--------|----------|
| /24 | 0.0.0.255 |
| /25 | 0.0.0.127 |
| /26 | 0.0.0.63 |
| /27 | 0.0.0.31 |
| /28 | 0.0.0.15 |
| /29 | 0.0.0.7 |
| /30 | 0.0.0.3 |

---

## Специальные ключевые слова

### host

Указывает конкретный IP-адрес.

Вместо:

```cisco
access-list 10 permit 192.168.1.10 0.0.0.0
```

Можно использовать:

```cisco
access-list 10 permit host 192.168.1.10
```

---

### any

Совпадает с любым адресом.

Вместо:

```cisco
0.0.0.0 255.255.255.255
```

Используется:

```cisco
any
```

---

# Standard ACL

## Разрешить сеть

Разрешить сеть:

```text
192.168.1.0/24
```

```cisco
R1(config)# access-list 10 permit 192.168.1.0 0.0.0.255
```

Все остальные адреса будут заблокированы скрытым правилом:

```text
deny ip any any
```

---

## Разрешить один хост

```cisco
R1(config)# access-list 10 permit host 192.168.1.100
```

---

## Применение ACL

На интерфейс:

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip access-group 10 in
```

---

## Направления ACL

### Inbound

Пакет фильтруется до обработки маршрутизатором.

```cisco
ip access-group 10 in
```

### Outbound

Пакет фильтруется перед отправкой.

```cisco
ip access-group 10 out
```

---

# Extended ACL

## Общий синтаксис

```cisco
access-list NUMBER permit|deny protocol source destination
```

---

## Разрешить HTTP

```cisco
R1(config)# access-list 100 permit tcp any any eq 80
```

---

## Разрешить HTTPS

```cisco
R1(config)# access-list 100 permit tcp any any eq 443
```

---

## Разрешить ICMP

```cisco
R1(config)# access-list 100 permit icmp any any
```

---

## Полный пример

Разрешить:

- HTTP
- HTTPS
- ICMP

Запретить остальное.

```cisco
R1(config)# access-list 100 permit tcp any any eq 80
R1(config)# access-list 100 permit tcp any any eq 443
R1(config)# access-list 100 permit icmp any any
```

После последней строки автоматически действует:

```text
deny ip any any
```

---

# Named ACL

## Создание ACL

```cisco
R1(config)# ip access-list extended INTERNET-FILTER
```

---

## Добавление правил

```cisco
R1(config-ext-nacl)# permit tcp any any eq 80
R1(config-ext-nacl)# permit tcp any any eq 443
R1(config-ext-nacl)# deny ip any any
```

---

## Применение

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip access-group INTERNET-FILTER in
```

---

# Sequence Numbers

Именованные ACL используют номера строк.

По умолчанию Cisco создает записи с шагом:

```text
10
```

Пример:

```text
10 permit tcp any any eq 80
20 permit tcp any any eq 443
30 deny ip any any
```

Это позволяет вставлять новые правила между существующими.

Пример:

```cisco
R1(config)# ip access-list extended INTERNET-FILTER
R1(config-ext-nacl)# 15 permit icmp any any
```

Результат:

```text
10 permit tcp any any eq 80
15 permit icmp any any
20 permit tcp any any eq 443
30 deny ip any any
```

Проверка:

```cisco
R1# show access-lists
```

---

# Рекомендации по размещению ACL

### Standard ACL

Размещать ближе к назначению.

```text
Источник ---- сеть ---- ACL ---- Получатель
```

### Extended ACL

Размещать ближе к источнику.

```text
Источник ---- ACL ---- сеть ---- Получатель
```

---

# ACL для SSH

Разрешить SSH только с одного компьютера.

Создание ACL:

```cisco
R1(config)# access-list 50 permit host 192.168.1.10
```

Применение к VTY:

```cisco
R1(config)# line vty 0 4
R1(config-line)# access-class 50 in
```

---

# ACL для Telnet

Запретить Telnet:

```cisco
R1(config)# access-list 101 deny tcp any any eq 23
R1(config)# access-list 101 permit ip any any
```

Результат:

- Telnet запрещен;
- остальной IP-трафик разрешен.

---

# ACL для ICMP

Запретить Ping:

```cisco
R1(config)# access-list 102 deny icmp any any
R1(config)# access-list 102 permit ip any any
```

Результат:

- Ping запрещен;
- остальной IP-трафик разрешен.

---

# Time-Based ACL

Разрешить доступ только в рабочее время.

Создание временного диапазона:

```cisco
R1(config)# time-range WORK-HOURS
R1(config-time-range)# periodic weekdays 8:00 to 18:00
```

ACL:

```cisco
R1(config)# ip access-list extended INTERNET
R1(config-ext-nacl)# permit tcp any any eq 80 time-range WORK-HOURS
R1(config-ext-nacl)# permit ip any any
```

---

# Счетчики совпадений (Matches)

Cisco ведет статистику срабатывания правил.

Проверка:

```cisco
R1# show access-lists
```

Пример:

```text
Extended IP access list INTERNET-FILTER

10 permit tcp any any eq 80 (125 matches)

20 permit tcp any any eq 443 (87 matches)

30 deny tcp any any eq 23 (15 matches)

40 permit ip any any (542 matches)
```

Расшифровка:

- 125 HTTP-соединений разрешено;
- 87 HTTPS-соединений разрешено;
- 15 Telnet-соединений заблокировано;
- 542 других пакета разрешены.

---

## Сброс счетчиков

```cisco
R1# clear access-list counters
```

---

# Проверка ACL

Просмотр ACL:

```cisco
R1# show access-lists
```

---

Просмотр интерфейсов:

```cisco
R1# show ip interface
```

---

Проверка привязки ACL:

```cisco
R1# show running-config
```

---

# Отладка

Использовать осторожно.

```cisco
R1# debug ip packet
```

Отключение:

```cisco
R1# undebug all
```

---

# Типичные ошибки

## ACL не привязана к интерфейсу

ACL создана, но не применяется.

Проверка:

```cisco
R1# show ip interface
```

---

## Неправильное направление

Настроено:

```cisco
ip access-group 100 out
```

Вместо:

```cisco
ip access-group 100 in
```

---

## Ошибка Wildcard Mask

Неверно:

```cisco
access-list 10 permit 192.168.1.0 255.255.255.0
```

Правильно:

```cisco
access-list 10 permit 192.168.1.0 0.0.0.255
```

---

## Неверный порядок правил

ACL проверяется сверху вниз.

Неверно:

```cisco
deny ip any any
permit tcp any any eq 80
```

HTTP никогда не будет разрешен.

---

## Забыли про Implicit Deny

Пример:

```cisco
access-list 101 deny tcp any any eq 23
```

В результате будет заблокирован весь остальной трафик.

Правильно:

```cisco
access-list 101 deny tcp any any eq 23
access-list 101 permit ip any any
```

---

# Best Practices

Рекомендуется:

✅ Использовать Named ACL

✅ Использовать Sequence Numbers

✅ Документировать назначение ACL

✅ Проверять порядок правил

✅ Использовать show access-lists для контроля счетчиков

✅ Размещать Extended ACL ближе к источнику

✅ Размещать Standard ACL ближе к назначению

✅ Ограничивать доступ к VTY через ACL

✅ Тестировать ACL до внедрения

Не рекомендуется:

❌ Использовать только номерные ACL

❌ Размещать deny ip any any в начале списка

❌ Игнорировать implicit deny

❌ Применять ACL без тестирования

❌ Использовать debug ip packet в загруженной сети

---

## Итог

После выполнения данной инструкции администратор сможет:

✅ Создавать Standard ACL

✅ Создавать Extended ACL

✅ Использовать Named ACL

✅ Фильтровать трафик по IP, протоколам и портам

✅ Ограничивать доступ к устройствам

✅ Защищать SSH и VTY-доступ

✅ Использовать временные политики доступа

✅ Анализировать статистику ACL

✅ Диагностировать проблемы фильтрации

✅ Безопасно применять ACL в корпоративной сети
