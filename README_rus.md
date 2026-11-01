# AmneziaWG для FriendlyElec NanoPi R3S

Резервный установочный комплект AmneziaWG для **FriendlyWrt 25.12.5** на **FriendlyElec NanoPi R3S LTS**.

> **Важно:** комплект подготовлен для ядра `6.1.141` и архитектуры `aarch64`. Модуль ядра нельзя считать совместимым с другой версией ядра.

## Характеристики

| Параметр | Значение |
|---|---|
| Устройство | FriendlyElec NanoPi R3S LTS |
| Прошивка | FriendlyWrt 25.12.5 |
| Архитектура | aarch64 |
| Версия ядра | 6.1.141 |
| Версия AWG | 3.1.20260812 |

## Состав комплекта

```text
AmneziaWG-NanoPi-R3S/
├── kernel-module/
│   └── amneziawg.ko
├── awg-tools/
│   └── awg
├── luci-integration/
│   ├── lib/netifd/proto/amneziawg.sh
│   ├── usr/bin/amneziawg_watchdog
│   ├── usr/share/rpcd/
│   └── www/luci-static/resources/
└── build-info/
    └── build-info.txt
```

- `amneziawg.ko` — модуль ядра.
- `awg` — утилита AmneziaWG для ARM64.
- `luci-integration` — интеграция с netifd, rpcd и LuCI.
- `build-info` — сведения о сборке.

## Установка

### 1. Проверка ядра

Подключитесь к роутеру:

```sh
ssh root@192.168.1.1
```

Проверьте версию ядра:

```sh
uname -r
```

Она должна быть `6.1.141`. Если версия отличается, не устанавливайте приложенный модуль.

### 2. Передача файлов

Распакуйте архив на компьютере. В PowerShell выполните:

```powershell
scp -r .\AmneziaWG-NanoPi-R3S root@192.168.1.1:/tmp/
```

### 3. Установка модуля ядра

Подключитесь к роутеру:

```sh
ssh root@192.168.1.1
cd /tmp/AmneziaWG-NanoPi-R3S
```

Установите модуль:

```sh
mkdir -p /lib/modules/$(uname -r)/extra
cp kernel-module/amneziawg.ko /lib/modules/$(uname -r)/extra/
depmod -a
```

Если модуль ещё не загружен:

```sh
insmod /lib/modules/$(uname -r)/extra/amneziawg.ko
```

Добавьте его в автозагрузку:

```sh
mkdir -p /etc/modules.d
echo amneziawg > /etc/modules.d/amneziawg
```

### 4. Установка утилиты

```sh
cp awg-tools/awg /usr/bin/awg
chmod 755 /usr/bin/awg
```

### 5. Установка LuCI/netifd

```sh
cp -a luci-integration/. /
chmod 755 /usr/bin/amneziawg_watchdog
chmod 755 /lib/netifd/proto/amneziawg.sh
```

### 6. Перезапуск служб

```sh
/etc/init.d/rpcd restart
/etc/init.d/network restart
```

Перезапуск сети может оборвать SSH-соединение. При необходимости подключитесь повторно.

## Проверка

Проверьте загрузку модуля:

```sh
lsmod | grep amneziawg
```

Проверьте утилиту:

```sh
awg --version
```

Проверьте регистрацию протокола:

```sh
ubus call network get_proto_handlers
```

В списке должен присутствовать `amneziawg`.

## Важные замечания

- Не используйте `opkg --force`.
- Не устанавливайте модуль на ядро другой версии.
- После обновления FriendlyWrt проверяйте совместимость модуля.
- Комплект предназначен для ручной установки и не содержит готовой настройки VPN-туннеля, ключей, пиров или firewall.
- Не публикуйте приватные ключи и личные VPN-конфигурации в GitHub.

## Лицензии

При публикации соблюдайте лицензии исходных проектов AmneziaWG, OpenWrt/FriendlyWrt и компонентов интеграции. Добавьте необходимые лицензионные уведомления и ссылки на исходные проекты.