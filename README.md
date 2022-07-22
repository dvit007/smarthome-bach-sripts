# Smarthome scripts
Скрипты на bach и других языках, которые я использую для автоматизации в Умном доме 

Здесь собраны скрипты, которые используются на серверах:
- master
- ubuntu16srv

Роутеры wifi:
 - Tomato (OpenWrt)
 - Mikrotik (Router OS 6)

Для каждого сервера, скрипты храняться в соответствующих папках. 
Они повторяют структуру каталогов на устройстве.

## Master сервер
### Скрипты для упраления виртуальными машинами QEMU
Расположение скриптов:
`etc/libvrt/hooks/`

**qemu**<br/>
Скрипт обеспечивает передачу информации в OH2 о состоняии гостевой ОС<br/>
и автоматическое ее отключение, по прошествии времени заданного в durationwork

Он вызывается автоматически службой libvirt.

Ему передаются параметры:
* `${1}` имя гостевой ОС
* `${2}` команда для этой виртуальки    

**shutdown_guest.sh**<br/>
Скрипт выключает гостевую ОС.

Он вызывается автоматически службой Cron. Задание Cron созадется в скрипте *qemu*

Ему передаются параметры:                                                        
* `${1}` имя гостевой ОС    

### Скрипты для управления UPS
Сервер подключен к ups Ippon. Скрипты автоматизируют выключение сервера при отключении электроэнерегии, и его автоматическому включению при возобновлении электроснабжения.<br/>
Перед отключением корректно завершается работа всех виртуальных машин. Что позволяет избежать потери данных из-за аварийного отключения.

Расположение настроек программы NUT для управления UPS:
`etc/nut/`

**nut.conf**,**ups.conf**,**upsd.conf**,**upsd.users**,**upsmon.conf**,**upssched.conf**

Расположение скриптов для управления UPS и сервером
`home/master/ups/`

**kvm_guest_shutdown.sh**<br/>
Скрипт реализующий корректное выключение гостевых ОС.<br/>
Он вызывается из скриптов службы NUT.

Ему передаются параметры:
* `--shutdown` отключение госетвых машин и выключение Хоста
* `--reboot` отключение гоствых машин и перезагрузка Хоста

**ups_host_start.sh**<br/>
Скрипт получает текущее состояние UPS  и отправляет его в OH2.<br/>
Вызывается из системеного скрипта *rc.local* при включении сервера.

Параметров передавать не надо.

**upssched-script.sh**<br/>
Скипт дублирует статус UPS в OH2. Меняет состояние прокси-items через Rest API

**/etc/rc.local**<br/>
Системный скрипт, который запускается при включении сервера. 
Обновлеяет статус UPS через 2 минуты, после старта.

### Скрипты для управления USB ключами
Для предоставления безопасного доступа к ключам USB, используется специальная виртуальная машина *ubuntu16srv*.<br/>
Она по команде через Telegram включается и автоматически отключается через 30 минут.

**manage_usb.sh**<br/>
Скрипт реализующий доступ к USB устройствам в зависимости от ролей пользователей.<br/>

Вызывается из сервера OpenHab. 

Ему надо передать один параметр - Роль пользовеля:
* Роль `1` - для всех неавторизванных пользователей
* Роль `2` - Бухгалтер 
* Роль `3` - Администратор ему доступны все устройства

Роли настраиваюся в правилах OpenHab                           

## Роутер Mikrotik
Скрипты для роутера Mikrotik.<br/>
Версия Router OS 6

### Библиотека для скриптов
В ней собраны функции, которые облегчают разработку других скриптов.

**library-ros.rsc**<br/>
За основу взята идеия из репозитария<br/>
https://github.com/AleksovAnry/Edelweiss-ROS

Добавлены свои функции, связанные с определением подключения устройств к определенным серверам. 

Эту библиотеку надо сохранить на роутере в папке `flash/` чтобы она не удалилась после перезагрузки устройства.

Для автоматической загрузки бибилотеки надо добавить задание в System->Scheduler<br/>
`/system scheduler add comment="Load library for MikroTik Router OS" name=StartScriptLibrary \
    on-event="#The file must first be uploaded via File->Upload
    /import flash/library-ros.rsc"
    policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon \
    start-time=startup`

**netflixunlock.rsc**<br/>
Скрипт разблоиркует доступ к Netflix через VPN для любых устройств без установки клиентов VPN.<br/>
В том числе и для любых Smart TV

Его надо сохранить в роутере, и настроить задание в System->Scheduler<br/>
`/system scheduler add comment="Smart TV scripts" interval=1m name=NetflixUnlcok on-event="\
    /system script run NetflixUnlock_LGTV1
    /system script run NetflixUnlock_LGTV1,5"
    policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon`
    


