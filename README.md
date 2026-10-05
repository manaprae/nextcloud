# Nextcloud
```
git clone https://github.com/manaprae/nextcloud.git
cd nextcloud
```
แก้ไข docker-compose.yml

### Hdd ลูกเดียว
   ในส่วนของ  
```
   volumes: 
	   - ./nextcloud_data:/var/www/html
```
### Hdd 2 ลูก
ต้อง ทำการ format ลูกที่ 2 และทำการ mount disk ให้เรียบร้อยก่อน
```
lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0   40G  0 disk 
├─sda1               8:1    0  600M  0 part /boot/efi
├─sda2               8:2    0    2G  0 part /boot
└─sda3               8:3    0 37.4G  0 part 
  ├─almalinux-root 253:0    0 35.4G  0 lvm  /
  └─almalinux-swap 253:1    0    2G  0 lvm  [SWAP]
sdb                  8:16   0  200G  0 disk 
sr0                 11:0    1  1.5G  0 rom 
```
จะเห็นว่า hdd แสดงสถานะดังนี้
ลูกที่ 1 แสดงสถานะ sda
ลูกที่ 2 แสดงสถานะ sdb
ถ้ามี ลูกที่ 3 แสดงสถานะ sdc

ทำการ format Hdd 
```
mkfs.ext4 /dev/sdb

blkid /dev/sdb
/dev/sdb: UUID="7fb10af2-b28d-4746-b9e0-27014f055d60" BLOCK_SIZE="4096" TYPE="ext4"
```
Format เสร็จ
แล้ว mount disk
```
mkdir -p /mnt/nextcloud-data
mount /dev/sdb /mnt/nextcloud-data

vim /etc/fstab
UUID=7fb10af2-b28d-4746-b9e0-27014f055d60 /mnt/nextcloud-data  ext4  defaults  0  2
```
แก้ไข docker-compose.yml 
```
	volumes:
	  - /mnt/nextcloud-data:/var/www/html
```
แก้ไข docker-compose.yml ก็สามารถสั่ง run docker compose up ได้เลย
```
docker compose up -d
[+] up 38/38
 ✔ Image mariadb:10.11       Pulled                                                   48.1s
 ✔ Image nextcloud:latest    Pulled                                                   99.4s
 ✔ Network nextcloud_default Created                                                   0.2s
 ✔ Container nextcloud-db    Started                                                   4.5s
 ✔ Container nextcloud-app   Started                                                   2.2s
```
เข้าใช้งานผ่าน
http://<ipaddress>:8080
ได้เลย

```
docker compose down -v
```
ทำการ add HDD แล้วแก้ docker-compose.yml
ถ้าต้องการใช้งานจากภายนอก ผ่าน cloudflared
เมื่อทำการติดตั้ง cloudflared tunnel เสร็จเรียบร้อย
ต้องเข้าไปแก้ไฟล์ config.php
path ของ ไฟล์จะอยู่ที่นี้
/var/www/html/config/config.php
กรณีนี้ เราจะเข้าไปแก้ผ่าน map volumes ที่เรา map มาที่เครื่องเราแล้ว
vim path-mapvolume-nextcloud/config/config.php
```
vim config/config.php
'trusted_domains' =>
  array (
    0 => 'localhost:8080',
    1 => '<newdomain>',
  ),
```


