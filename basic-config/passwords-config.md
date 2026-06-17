# Базовая настройка паролей Cisco Router через Console

## Вход в Privileged EXEC Mode

После подключения к роутеру через консольный порт (USB или RJ-45 Console) необходимо перейти в режим **Privileged EXEC**, используя команду:

```bash
Router> enable
```

Данный режим предназначен для базового управления устройством и предоставляет доступ к таким командам, как:

* `show`
* `copy`
* `ping`
* `traceroute`
* `reload`
* `clock set`

Также из этого режима можно перейти в режим глобальной конфигурации:

```bash
Router# configure terminal
```

Пример:

```bash
Router> enable
Router# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)# hostname R1
R1(config)#
```

Команда `hostname` изменяет имя устройства.

---

# Настройка Console Line

Для защиты локального доступа рекомендуется установить пароль на консольную линию.

```bash
R1(config)# line console 0
R1(config-line)# password 123
R1(config-line)# login
```

### Описание команд

| Команда          | Назначение                                |
| ---------------- | ----------------------------------------- |
| `line console 0` | Переход к настройке консольной линии      |
| `password 123`   | Установка пароля                          |
| `login`          | Включение проверки пароля при подключении |

> Пароль может быть любым, но не должен содержать пробелов.

---

# Настройка пароля Privileged EXEC Mode

После настройки консоли рекомендуется защитить режим Privileged EXEC отдельным паролем.

Выходим из режима настройки консольной линии:

```bash
R1(config-line)# exit
```

Устанавливаем пароль:

```bash
R1(config)# enable secret 12345
```

Команда `enable secret` создает зашифрованный пароль для доступа к Privileged EXEC Mode.

---

# Настройка VTY-линий (Telnet/SSH)

VTY-линии используются для удаленного управления устройством через протоколы:

* Telnet
* SSH

Настроим пароль для удаленного доступа:

```bash
R1(config)# line vty 0 5
R1(config-line)# password 12345
R1(config-line)# login
R1(config-line)# exit
```

### Описание

| Команда        | Назначение                   |
| -------------- | ---------------------------- |
| `line vty 0 5` | Настройка VTY-линий с 0 по 5 |
| `password`     | Установка пароля             |
| `login`        | Включение проверки пароля    |

Например:

* `line vty 0 4` → 5 линий
* `line vty 0 15` → 16 линий

---

# Шифрование паролей

На данный момент все пароли (кроме `enable secret`) хранятся в конфигурации в открытом виде.

Для их шифрования используйте:

```bash
R1(config)# service password-encryption
```

После выполнения команды все пароли, настроенные через `password`, будут сохранены в зашифрованном виде.

---

# Сохранение конфигурации

По умолчанию все изменения сохраняются только в оперативной памяти (**RAM**).

После перезагрузки устройства настройки будут потеряны.

Чтобы сохранить текущую конфигурацию в энергонезависимую память (**NVRAM**), выполните:

```bash
R1(config)# do copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

### Что происходит

* `running-config` — текущая конфигурация в RAM.
* `startup-config` — конфигурация, которая загружается при включении устройства.
* Команда копирует текущие настройки в постоянную память.

Поскольку команда `copy` относится к режиму **Privileged EXEC**, в режиме конфигурации перед ней используется ключевое слово `do`.

---

# Итоговая последовательность команд

```bash
enable
configure terminal

hostname R1

line console 0
 password 123
 login
 exit

enable secret 12345

line vty 0 5
 password 12345
 login
 exit

service password-encryption

do copy running-config startup-config
```

После выполнения данных действий роутер будет:

✅ Иметь собственное имя
✅ Требовать пароль при подключении через Console
✅ Требовать пароль для входа в Privileged EXEC Mode
✅ Требовать пароль для удаленного подключения (Telnet/SSH)
✅ Хранить пароли в зашифрованном виде
✅ Сохранять настройки после перезагрузки
