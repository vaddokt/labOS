# Лабораторная работа №2
## Задание 1. Установка ОС и настройка LVM и RAID.
Создаем новую виртуальную машину с 1 GB RAM, одним cpu, и двумя виртуальными дисками ssd1 и ssd2 равного размера. При этом у SATA-контроллера 4 порта.
![pg101](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg101.png)
Начинам установку Linux, доходим до выбора жестких дисков и вручную настраиваем их разделы.
У обоих дисков создаем таблицу разделов размером 512 MB, первой присваиваем mount point: /boot, второй none, поскольку нельзя одновременно монтировать /boot 2 раза.
![pg102](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg102.png)
Далее настраиваем RAID:
На обоих дисках выбираем свободное пространство и настраиваем в качестве типа раздела physical volume for RAID.
![pg103](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg103.png)
Настраиваем Software RAID и выбираем разделы под RAID на обоих дисках.
![pg104](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg104.png)
Далее настраиваем LVM. Создаем volume group с названием system и три раздела в ней:
![pg105](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg105.png)
После завершения настройки LVM видим:
![pg106](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg106.png)
Далее размечаем отделы выбрав соответствующие точки монтирования, после чего завершаем настройку разделов.
![pg107](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg107.png)
Устанавливаем GRUB на первое устройство (sda) и загружаем систему. После этого копируем содержимое раздела /boot с первого диска (ssd1) на второй (ssd2).
Просматриваем диски в системе и устанавливаем GRUB на второе устройство (sdb):
![pg108](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg108.png)
Видим, что у нас имеются два устройства sda и sdb, где sda1 и sdb1 ¬– это разделы под /boot. Разделы sda2 и sdb2 отведены под RAID.
md0 – physical volume for RAID;
На обоих дисках имеется volume group с именем system и три раздела:
- system-root – корневой раздел;
- system-var – каталог для динамических данных;
- system-log – раздел, где хранятся все log-файлы.
Команда cat /proc/mdstat показывает нам текущее состояние RAID-массива, где md0 – имя RAID-устройства, raid1 sda2[0] sdb2[1] – уровень избыточности и из каких дисков собран, 3691520 blocks – размер массива, [2/2] [UU] – кол-во юнитов, которые на данный момент используются.
Посмотрим вывод команд:
- pvs – вывод информации о physical volume;
- vgs – вывод информации о volume group;
- lvs – вывод информации о logical volume;
![pg109](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg109.png)
Посмотрим информацию о смонтированных файловых системах (ФС), используя команду mount. Результат содержит все ФС, в том числе виртуальные, такие как cgroup, sysfc и т.д. Каждая строка содержит информацию об имени устройства, директории, в которой оно смонтировано, типе и опциях монтирования в следующей форме:
```
имя_устройства on директория type тип_файловой_системы (опции).
```
Например /dev/sda1 смонтировано в директории /boot, где файловая система имеет тип ext4.
![pg110](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg110.png)
Таким образом, мы установили ОС Debian с двумя дисками равного размера и настроили в памяти дисков разделы под RAID и LVM, а также просмотрели информацию об их структуре и состоянии.
## Задание 2. Эмуляция отказа одного из дисков.
Представим, что у нас отказал 1-ый диск (ssd1).
![pg201](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg201.png)
Тем не менее, после перезагрузки ВМ все еще работает благодаря 2-ому диску. Проверяем статус RAID-массива.
![pg202](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg202.png)
На замену 1-ому диску добавим новый диск такого же размера. Назовем его ssd33 (в моем случае).
![pg203](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg203.png)
Новый диск был добавлен в систему. Теперь копируем таблицу разделов со старого на новый диск.
![pg204](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg204.png)
Добавляем в рейд массив новый диск командой:
```
mdadm --manage /dev/md0 --add /dev/sdb2
```
![pg205](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg205.png)
Видим, что началась синхронизация. Далее вручную синхронизируем отделы, не входящие в RAID. Выполняем команду:
```
dd if=/dev/sda1 of=/dev/sdb1.
```
![pg206](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg206.png)
После синхронизации устанавливаем GRUB на новый диск и перезагружаем систему, чтобы убедиться, что все работает.
![pg207](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg207.png)
После перезагрузки смотрим состояние RAID-массивов.
![pg208](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg208.png)
Таким образом, на замену отключенного диска мы поставили новый, куда скопировали таблицу разделов, рейд массив и остальные данные со 2-го диска(ssd2). В результате в системе снова имеются два рабочих диска(ssd2 и ssd33).
## Задание 3. Добавление новых дисков и перенос раздела.
Внезапно отказывает диск ssd2. Перезагружаем систему и смотрим текущее состояние дисков и RAID.
![pg301](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg301.png)
![pg302](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg302.png)
![pg303](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg303.png)
Добавляем новый диск ssd4. Смотрим, что он был подключен.
![pg304](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg304.png)
Теперь мы переносим данные со старого диска на новый ssd4 (в системе отображается как sdb). Копируем файловую таблицу командой:
```
sfdisk -d /dev/sda | sfdisk /dev/sdb
```
![pg305](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg305.png)
Теперь на новом диске появились аналогичные sda1 и sda2 разделы. Далее копируем данные раздела /boot на новый диск командой:
```
dd if=/dev/sda1 of=/sdb1
```
![pg306](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg306.png)
Перемонтируем /boot на новый диск и установим на нем GRUB. Далее создадим новый RAID-массив, но включим туда только новый диск.
![pg307](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg307.png)
Проверяем состояние командой cat /proc/mdstat и видим, что появилось новое RAID-устройство, к которому подключен новый диск. Устройство находится в разделе sdb2.
![pg308](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg308.png)
Настраиваем LVM. Создаем новый физический том и включаем в него новый RAID-массив:
```
pvcreate /dev/md63
```
Теперь тип файловой системы md63 отображается как LVM2_member. Команда pvs теперь отображает новое рейд-устройство.
![pg309](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg309.png)
Увеличиваем размер Volume Group system:
```
vgextend system /dev/md63
```
Видим, что сейчас LV var, log, root находятся на старом устройстве md0. Теперь нужно переместить эти данные на новый диск:
```
pvmove -i 10 -n /dev/system/root /dev/md0 /dev/md63
```
Аналогично для var и log.
Теперь видим, что данные перемещены на устройство md63.
![pg310](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg310.png)
Удаляем из VG диск md0:
```
vgreduce system /dev/md0
```
VG system больше не отображается в md0.
![pg311](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg311.png)
Проверяем, что /boot перемонтирован на новый диск, и раздел не пустой:
```
ls /boot
```
Раздел /boot содержит все файлы, связанные с загрузчиком системы. Это ядро vmlinuz, образ initrd, файлы загрузчика в /boot/grub, файлы config (опции ядра). System.map – это файл, в котором находится символьная таблица адресов функций и процедур, используемых ядром операционной системы. В lost+found содержатся файлы, которые были отсоединены, но все же были открыты каким-то процессом, когда система внезапно остановилась.
![pg312](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg312.png)
Теперь удаляем диск ssd33 и подсоединяем три новых (ssd5, hdd1, hdd2). В результате у нас имеется два ssd и два hdd диска.
![pg313](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg313.png)
![pg314](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg314.png)
Новые диски отображаются в системе (ssd5 – sda, hdd1 – sdc, hdd2 – sdd). Имя рейд-устройства изменилось на md127.
Копируем таблицу разделов на новый ssd5:
```
sfdisk -d /dev/sdb | sfdisk /dev/sda
```
После чего копируем загрузочный раздел на новый диск:
```
dd if=/dev/sdb1 of=/dev/sda1
```
И устанавливаем grub:
```
grub-install /dev/sda
```
Так как новый размер не использует весь объем диска, то делаем следующее:
Открываем утилиту для работы с разметкой дисков:
```
fdisk /dev/sda
```
Заново создаем раздел под RAID, указываем тип Linux raid auto и сохраняем изменения.
![pg315](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg315.png)
Добавляем новый диск к существующему рейд-массиву и расширим количество дисков в массиве до 2 штук.
![pg316](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg316.png)
Однако оба раздела, входящие в массив имеют разные размеры. Поэтому, увеличиваем размер раздела на ssd4. При помощи утилиты `fdisk` заново создаем 2-ой раздел и в конце разметки оставляем сигнатуру принадлежности раздела к массиву как есть.
![pg317](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg317.png)
Теперь разделы одинакового размера. Далее расширяем размер raid:
```
mdadm --grow /dev/md127 --size=max
```
Теперь raid занимает все отведенное для него пространство. Однако размеры VG root, var, log остались прежними.
Расширяем размер PV:
```
pvresize /dev/md127
```
Расширяем размер root и var используя `lvextend`.
![pg318](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg318.png)
Основной массив теперь перенесен на новые диски. Теперь нужно переместить /var/log. Создадим новый массив и lvm на hdd дисках.
Создадим raid массив:
```
mdadm --create /dev/md63 --level=1 --raid-devices=2 /dev/sdc /dev/sdd
```
Создаем новый PV:
```
pvcreate data /dev/md63
```
Создаем в нем PV группу с названием data:
```
vgcreate data /dev/md63
```
Создаем логический том размером всего свободного пространства с именем var_log:
```
lvcreate -l 100%FREE -n var_log data
```
Смотрим результат.
![pg319](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg319.png)
Теперь отформатируем раздел в ext4:
```
mkfs.ext4 /dev/mapper/data-var_log
```
![pg320](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg320.png)
Теперь переносим данные со старого раздела на новый.
Примонтируем новое временное хранилище логов:
```
mount /dev/mapper/data-var_log /mnt
```
Синхронизируем разделы при помощи rsync:
```
apt install rsync
rsync -avzr /var/log/ /mnt/
```
Узнаем какие процессы работают с /var/log в данный момент и останавливаем их:
```
apt install lsof
lsof | grep '/var/log'
systemctl stop rsyslog.service syslog.socket
```
И еще раз выполняем синхронизацию разделов:
```
rsync -avzr /var/log/ /mnt/
```
![pg321](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg321.png)
Меняем разделы местами:
```
umount /mnt
umount /var/log
mount /dev/mapper/data-var_log /var/log
```
![pg322](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg322.png)
Теперь нужно отредактировать /etc/fstab. В fstab указаны правила монтирования разделов при загрузке. Изменяем 'system-log' на 'data-var_log'.
Изменяем тип файловой системы командой `resize2fs`.
![pg323](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg323.png)
Теперь перезагружаем систему и проверяем результат.
![pg324](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg324.png)
![pg325](https://github.com/vaddokt/labOS/blob/master/lab2/images/pg325.png)
В результате проведения лабораторной работы мы переместили данные системы со старых дисков на новые более вместительные диски.
