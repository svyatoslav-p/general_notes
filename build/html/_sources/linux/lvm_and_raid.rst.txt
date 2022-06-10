.. |br| raw:: html

   <br />

LVM and RAID
############

Создание аппаратного RAID
*************************

Если материнская плата поддерживает создание аппаратного RAID, создаем такой средствами
BIOS. В ОС Linux (например ubuntu) установим утилиту для работы с RAID ``sudo apt install mdadm``.
Если после установки и перезагрузки системы RAID не опознался (в данном случае это ``sdd`` и ``sde``)

.. code-block:: console

   gratia@gratia-uonmap:~$ lsblk
   NAME                MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT

   sdd                   8:48   0   1,8T  0 disk 
   sde                   8:64   0   1,8T  0 disk 

то выполним ``sudo IMSM_NO_PLATFORM=1 mdadm --assemble --scan --verbose`` после проверим результат.

.. code-block:: console

   gratia@gratia-uonmap:~$ lsblk
   NAME                MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
   
   sdd                   8:48   0   1,8T  0 disk  
   └─md126               9:126  0   1,7T  0 raid1 
     └─md126p1         259:0    0   1,7T  0 part  
   sde                   8:64   0   1,8T  0 disk  
   └─md126               9:126  0   1,7T  0 raid1 
     └─md126p1         259:0    0   1,7T  0 part 

в выводе можно увидеть что диски ``sdd`` и ``sde`` находятся в массиве ``md126`` далее создаем файловую систему.
к самому RAID можно обратиться через ``/dev/md126``

Монтирование диска из RAID1
***************************

Ниже пример когда из зеркального RAID1 извлечен один из дисков и его необходимо примонтировать
как обычный диск. В примере ниже сам диск размечен через LVM

.. code-block:: bash
   
   #Находим необходимый диск
   >lsblk
   sdb                          8:16   0   1,8T  0 disk 
   └─sdb1                       8:17   0   1,8T  0 part 
     └─md127                    9:127  0     0B  0 md  
  
   #Останавливаем массив raid
   >sudo mdadm --stop /dev/md127
   #Создаем новый raid
   >sudo mdadm -A -R /dev/md0 /dev/sdb1
   #Теперь разделы должны быть доступны
   >lsblk
   sdb                                8:16   0   1,8T  0 disk  
   └─sdb1                             8:17   0   1,8T  0 part  
     └─md0                            9:0    0   1,8T  0 raid1 
       ├─Pandora--raid1-ubuntu10.04 254:2    0   100G  0 lvm   
       ├─Pandora--raid1-nfs_share   254:3    0   450G  0 lvm   
       ├─Pandora--raid1-backup      254:4    0   400G  0 lvm   
       ├─Pandora--raid1-share       254:5    0   150G  0 lvm   
       ├─Pandora--raid1-docker      254:6    0    60G  0 lvm   
       ├─Pandora--raid1-docker_lib  254:7    0    70G  0 lvm   
       ├─Pandora--raid1-var_www     254:8    0    20G  0 lvm   
       ├─Pandora--raid1-mysql--lib  254:9    0   400G  0 lvm   
       ├─Pandora--raid1-var_log     254:10   0    30G  0 lvm   
       └─Pandora--raid1-centos7.0   254:11   0    30G  0 lvm 
   #Монтируем как обычно
   >mount /dev/Pandora-raid1/backup /mnt/raid/1
