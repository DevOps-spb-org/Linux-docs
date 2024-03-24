## Изменение размера root lvm раздела

### Подготовим временный раздел 

```bash
pvcreate /dev/sdb
vgcreate vg_root /dev/sdb
lvcreate -n lv_root -l +100%FREE /dev/vg_root
```

>  Physical volume "/dev/sdb" successfully created.  
>  Volume group "vg_root" successfully created  
>  Logical volume "lv_root" created.    

### Создадим на нем файловую систему и смонтируем его, чтобы перенести туда данные

```bash
mkfs.xfs /dev/vg_root/lv_root
mount /dev/vg_root/lv_root /mnt
```

### Этой командой скопируем все данные с / раздела в /mnt

```bash
xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
```

> xfsrestore: Restore Status: SUCCESS

### Затем переконфигурируем grub для того, чтобы при старте перейти в новый /

Сымитируем текущий root -> сделаем в него chroot и обновим grub:

```bash
for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
chroot /mnt/
grub2-mkconfig -o /boot/grub2/grub.cfg
```

> Generating grub configuration file ...                                          
> Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64               
> Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img     
> done

### Обновим образ initrd. 

```bash
cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
s/.img//g"` --force; done
```

> *** Creating image file ***              
> *** Creating image file done ***               
> *** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done *** 

Ну и для того, чтобы при загрузке был смонтирован нужны root нужно в файле
**/boot/grub2/grub.cfg** заменить `rd.lvm.lv=VolGroup00/LogVol00` на `rd.lvm.lv=vg_root/lv_root`

### Перезагружаемся успешно с новым рут томом.

### Убедиться в этом можно посмотрев вывод lsblk:

```bash
 lsblk
```

> NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT             
> sda 8:0 0 40G 0 disk                           
> |-sda1 8:1 0 1M 0 part                           
> |-sda2 8:2 0 1G 0 part /boot                           
> \`-sda3 8:3 0 39G 0 part                           
>  |-VolGroup00-LogVol01 253:1 0 1.5G 0 lvm [SWAP]                           
>  \`-VolGroup00-LogVol00 253:2 0 37.5G 0 lvm                           
> sdb 8:16 0 10G 0 disk                           
> \`-vg_root-lv_root 253:0 0 10G 0 lvm /                           
> sdc 8:32 0 2G 0 disk                           
> sdd 8:48 0 1G 0 disk                           
> sde 8:64 0 1G 0 disk                           

### Теперь нам нужно изменить размер старой VG и вернуть на него рут. Для этого удаляем старый LV размеров в 40G и создаем новый на 8G:

```bash
 lvremove /dev/VolGroup00/LogVol00
```
 
> Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y     
>  Logical volume "LogVol00" successfully removed            
 
```bash
 lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
```

> WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y            
>  Wiping xfs signature on /dev/VolGroup00/LogVol00.            
>  Logical volume "LogVol00" created.                

### Проделываем на нем те же операции, что и в первый раз:

```bash
mkfs.xfs /dev/VolGroup00/LogVol00
mount /dev/VolGroup00/LogVol00 /mnt
xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
```

> xfsdump: Dump Status: SUCCESS                        
> xfsrestore: restore complete: 37 seconds elapsed                    
> xfsrestore: Restore Status: SUCCESS                    

### Так же как в первый раз переконфигурируем grub, за исключением правки /etc/grub2/grub.cfg

```bash
 for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
 chroot /mnt/
 grub2-mkconfig -o /boot/grub2/grub.cfg
```

> Generating grub configuration file ...                        
> Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64                        
> Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img                
> done                

```bash
cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
s/.img//g"` --force; done
```

> *** Creating image file ***                        
> *** Creating image file done ***                        
> *** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***      

**********
[LVM](/tags/LVM.md)
[root](/tags/root.md)
