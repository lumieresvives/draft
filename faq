# Часто задаваемые вопросы

<details><summary><h2>Не активируется Numa vServer</summary>

1. Уточнить количество физических процессоров и проверить соответствие в выданной лицензии.
2. Проверить время на сервере пользователя, если сильный разбег с фактическим, выполнить:
    - При наличии ntp сервера, ntpdate -u <адрес ntp сервера>
    - При отсутствии ntp сервера:
        - date + hwclock -w
        - Настроить правильное время в bios
</details>

<details><summary><h2>Как включить высокую доступность</summary>

Включение высокой доступности в пуле выполняется из CLI Numa vServer.

Для включения нужно выполнить команду:

`xe pool-ha-enable heartbeat-sr-uuids=<UUID общего хранилища>`

UUID общего хранилища можно узнать используя команду: `xe sr-list` или в Numa Collider, нажав на соответствующее хранилище (см. под именем хранилища).

> Примечание: рекомендуемое минимальное количество серверов для реализации высокой доступности 3 или больше. При меньшем количестве нужно установить параметр pool-ha-compute-max-host-failures-to-tolerate равным 1.

`xe pool-param-set ha-host-failures-to-tolerate=1 uuid=<UUID пула>`

UUID пула можно узнать с помощью команды: `xe pool-list`

После включение HA в пуле, можно выставить приоритеты для ВМ машин, в Numa Collider они доступны в расширенных настройках ВМ (пункт высокая доступность).

Выключение высокой доступности выполняется командой:

`xe pool-ha-disable` 

В случае, если попытка выключения высокой доступности оканчивается сообщением:

<code> The uuid you supplied was invalid. <br>
type: VDI <br>
uuid: 'uuid' </code>

Высокая доступность выключается следующей последовательностью команд:

`xe host-emergency-ha-disable --force`

`xe-toolstack-restart`

`xe pool-ha-disable`
</details>

<details><summary><h2>Настройка интерфейса управления на VLAN интерфейсе</summary>

Выполните в консоли следующие действия:

1. Получите UUID всех физических сетевых интерфейсов 

`xe pif-list params=uuid,device physical=true`

зафиксируйте UUID интерфейса (далее - PIF-UUID), где будет планируется настроить VLAN.

2. Создайте сеть, имя может быть произвольным, например MGT. Зафиксируйте UUID новой сети (далее NET-UUID). 

`xe network-create name-label=MGT`

3. Создайте VLAN, используя PIF-UUID и NET-UUID, после создания зафиксируйте UUID нового интерфейса (далее - VLAN-UUID)

`xe vlan-create pif-uuid=PIF-UUID network-uuid=NET-UUID vlan=vlan_тег` 

4. Подключите виртуальный интерфейс

`xe pif-plug uuid=VLAN-UUID`

5. Перенастройте менеджмент интерфейс:
    - Для работы в режиме DHCP: 
    
    `xe pif-reconfigure-ip uuid=VLAN-UUID mode=dhcp`

    - Для ввода статических настроек выполните: 
    
    `xe pif-reconfigure-ip uuid=VLAN-UUID mode=static IP=192.168.1.1 gateway=192.168.1.254 netmask=255.255.255.0 DNS=8.8.4.4`

6. Назначьте интерфейс управляющим: 

`xe host-management-reconfigure pif-uuid=VLAN-UUID`

Подсказка: можно использовать для автодополнения клавишу `Tab`
</details>

<details><summary><h2>Переименование сетевых интерфейсов</summary>

Переименование сетевых интерфейсов осуществляется через скрипт `interface_rename.py` (`/nix/store/fgsdwjhnb7fy9a9cac9k90jkzmv8m62g-interface-rename/libexec/interface_rename.py`).

1. Просмотр актуального перечня сетевых интерфейсов: 

`/nix/store/fgsdwjhnb7fy9a9cac9k90jkzmv8m62g-interface-rename/libexec/interface_rename.py -l`

2. Переименование интерфейса: 

`/nix/store/fgsdwjhnb7fy9a9cac9k90jkzmv8m62g-interface-rename/libexec/interface_rename.py -u <eth_name>=MAC|PCI`

3. Для применения изменений необходимо выполнить перезагрузку сервера: 

`reboot`

Пример: 

```[root@vserver-icnwkawg:~]# /nix/store/
fgsdwjhnb7fy9a9cac9k90jkzmv8m62g-interface-rename/libexec/interface_rename.py -l
Name   MAC                PCI              ethN  Phys  SMBios  Driver  Version  Firmware  
eth0   00:50:b6:5b:ca:6a  0000:02:00.0[0]  eth0                igb     5.4.0-k  3.25, 0x800005cc
eth1   00:50:b6:5b:ca:7a  0000:03:00.0[0]  eth1                igb     5.4.0-k  3.25, 0x800005cc
eth2   00:50:b6:5b:ca:8a  0000:04:00.0[0]  eth2                igb     5.4.0-k  3.25, 0x800005d0
eth3   00:50:b6:5b:ca:8b  0000:05:00.0[0]  eth3                igb     5.4.0-k  3.25, 0x800005d0
 
[root@vserver-icnwkawg:~]# /nix/store/
fgsdwjhnb7fy9a9cac9k90jkzmv8m62g-interface-rename/libexec/interface_rename.py -u eth0=0000:03:00.0[0] eth1=0000:02:00.0[0]
INFO     [2023-08-04 17:31:55] Performing manual update of rules.  Not actually renaming interfaces
INFO     [2023-08-04 17:31:55] All done
 
[root@vserver-icnwkawg:~]# reboot
```

> Примечание: Недопустимо использование двумя интерфейсами одного MAC или PCI-ID. Переименовывать необходимо парами либо всем списком.
</details>

<details><summary><h2>Разворачивание Numa vServer на программном RAID1</summary>

> Для версии ПО 1.1.87f2fd8a5 (от 09.08.2023) и выше 

Для установки Numa vServer на RAID необходимо:

1. Загрузить Numa vServer с USB-накопителя.
2. На экране загрузки выбрать пункт загрузки "Консоль инсталлятора".
3. В консоли создать массив RAID1, например, из блочных устройств `/dev/sda` и `/dev/sdb`:

`mdadm -C /dev/md0 -l1 -n2 /dev/sda /dev/sdb --metadata=0.90`

4. Запустить установщик Numa vServer: 

`tui-installer`

5. Дальнейшая установка должна осуществляться согласно инструкции (Серверная доверенная виртуальная среда функционирования программных
средств Numa vServer. Руководство администратора. Установка, настройка Numa vServer. 643.АМБН.00021-01 32 01), в качестве целевого устройства для установки необходимо выбрать созданный ранее RAID массив. В текущем примере `/dev/md0`
</details>

<details><summary><h2>RAIDIX Multipath</summary>

Ввиду того, что RAIDIX использует нестандартный SCSI ID, для работы MPIO необходимо модифицировать `multipath.conf`

```
device {
vendor "Raidix"
product ".*"
path_grouping_policy "group_by_prio"
    path_selector "round-robin 0"
    path_checker        "tur"
    prio "alua"
    failback immediate
    rr_min_io 100
    rr_weight "priorities"
    no_path_retry 12
    features "1 queue_if_no_path"
    hardware_handler "1 alua"
    getuid_callout "/nix/store/8kgyilcahwfql5q2lnkdbkbck50v13da-systemd-245/lib/udev/scsi_id --whitelisted --replace-whitespace --device=/dev/%n"
}
```

Ключевой параметр и команда содержится в строке `getuid_callout`
</details>

