# Fdisk_Part TP {Auth : BenABdejlilOmar} 
CCDAD.1
# fdisk_partitions
> pour aficher les partions :
```bash
lsblk
# or sudo fdisk -l 
```
> 1/ Creation of the `primary partiotion` : 
```bash
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 
Using default value 1
First sector (2048-4194303, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-4194303, default 4194303): +1G
```
<hr>

> 2/ partition Extended : 
```bash
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): e
Partition number (1-4, default 2): 
Using default value 2
First sector (2099200-4194303, default 2099200): 
Using default value 2099200
Last sector, +sectors or +size{K,M,G} (2099200-4194303, default 4194303): +10M
```
<hr>

> 3/ partions logic : 
```bash
Command (m for help): n
Partition type:
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 5
First sector (2101248-4196351, default 2101248): 
Using default value 2101248
Last sector, +sectors or +size{K,M,G} (2101248-4196351, default 4196351): +500M
```
> see the result : 
```bash
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
useradm@serveur:~$ lsblk
NAME                          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                             8:0    0     8G  0 disk 
|-sda1                          8:1    0   243M  0 part /boot
|-sda2                          8:2    0     1K  0 part 
`-sda5                          8:5    0   7.8G  0 part 
  |-serveur--vg-root (dm-0)   252:0    0   7.2G  0 lvm  /
  `-serveur--vg-swap_1 (dm-1) 252:1    0   508M  0 lvm  [SWAP]
sdb                             8:16   0     3G  0 disk 
|-sdb1                          8:17   0     1G  0 part 
|-sdb2                          8:18   0     1K  0 part 
|-sdb5                          8:21   0   500M  0 part 
`-sdb6                          8:22   0   522M  0 part 
sr0                            11:0    1  1024M  0 rom  
```
> change The Last `Partition System` With `swap` :
```bash
Command (m for help): t
Partition number (1-6): 6
Hex code (type L to list codes): 82
Changed system type of partition 6 to 82 (Linux swap / Solaris)

Command (m for help): p

Disk /dev/sdb: 3221 MB, 3221225472 bytes
255 heads, 63 sectors/track, 391 cylinders, total 6291456 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x84664141

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200     4196351     1048576    5  Extended
/dev/sdb5         2101248     3125247      512000   83  Linux
/dev/sdb6         3127296     4196351      534528   82  Linux swap / Solaris
```
<hr>

> formater les nouvelles partitions avec le type de système de fichiers (ext4, ext3, swap), on insère aussi le label de chaque partition :
```bash
useradm@serveur:~$ sudo mkfs.ext4 -L "DATA" /dev/sdb1
mke2fs 1.42.9 (4-Feb-2014)
Étiquette de système de fichiers=DATA
Type de système d'exploitation : Linux
Taille de bloc=4096 (log=2)
Taille de fragment=4096 (log=2)
« Stride » = 0 blocs, « Stripe width » = 0 blocs
65536 i-noeuds, 262144 blocs
13107 blocs (5.00%) réservés pour le super utilisateur
Premier bloc de données=0
Nombre maximum de blocs du système de fichiers=268435456
8 groupes de blocs
32768 blocs par groupe, 32768 fragments par groupe
8192 i-noeuds par groupe
Superblocs de secours stockés sur les blocs : 
	32768, 98304, 163840, 229376

Allocation des tables de groupe : complété                        
Écriture des tables d'i-noeuds : complété                        
Création du journal (8192 blocs) : complété
Écriture des superblocs et de l'information de comptabilité du système de
fichiers : complété
```
> `Logs` Label  :
```bash
sudo mkfs.ext3 -L "LOGS" /dev/sdb
```
> `Swap` Label : 
```bash
sudo mkswap -L "SWAP" /dev/sdb6
Setting up swapspace version 1, size = 534524 KiB
LABEL=SWAP, UUID=d12277be-17a3-4a4a-a2c8-41b640b4482c
```
<hr>

> création des répertoires et faire les montages de deux partitions `data` et `Logs` :<br>
`sudo mkdir /mnt/data /mnt/logs `
> monter sur ces partitions : 
```bash
sudo mount /dev/sdb1 /mnt/data/ ; sudo mount /dev/sdb5 /mnt/logs/
```
> Simulation :
```bash
useradm@serveur:~$ lsblk -f
NAME                          FSTYPE LABEL MOUNTPOINT
sda                                        
|-sda1                                     /boot
|-sda2                                     
`-sda5                                     
  |-serveur--vg-root (dm-0)                /
  `-serveur--vg-swap_1 (dm-1)              [SWAP]
sdb                                        
|-sdb1                                     /mnt/data
|-sdb2                                     
|-sdb5                                     /mnt/logs
`-sdb6                                     
```
> Pour afficher tous les `UUID` tapez : `sudo blkid /dev/sdb*`
```bash
useradm@serveur:~$ sudo blkid /dev/sdb*
/dev/sdb1: LABEL="DATA" UUID="5d01c12e-1d4d-40dd-a0f0-e4a5f9ed1a6d" TYPE="ext4" 
/dev/sdb5: LABEL="LOGS" UUID="6d848778-946a-495e-9266-fc3ca4f0a40c" TYPE="ext3" 
/dev/sdb6: LABEL="SWAP" UUID="d12277be-17a3-4a4a-a2c8-41b640b4482c" TYPE="swap" 
```
> Ajouter ces lignes dans le fichier `/etc/fstab` pour assurer un `montage automatique` des partitions a chaque démarrage de la machine : 
```bash
UUID=5d01c12e-1d4d-40dd-a0f0-e4a5f9ed1a6d       /mnt/data       ext4    default  0   0
UUID=6d848778-946a-495e-9266-fc3ca4f0a40c       /mnt/logs       ext3    default  0   0
UUID=d12277be-17a3-4a4a-a2c8-41b640b4482c       none            swap    default  0   0
```
> Une présentation présenté dans ce Tableau : 

|*|Partition|Systeme de fichier|Taille|Label|Point de montage|Péripherique|
|---|---|---|---|---|---|---|
|1|primaire|ext4|1GB|DATA|/mnt/data|/dev/sdb1|
|2|logique|ext3|500Mb|LOGS|/mnt/logs|/dev/sdb5|
|3|logique|swap|500MB|SWAP|------|/dev/sdb6|
