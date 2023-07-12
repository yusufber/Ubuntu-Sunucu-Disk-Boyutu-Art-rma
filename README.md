# Ubuntu-Sunucu-Disk-Boyutu-Art-rma
Linux İşletim Sistemi kurulumu tamamlandıktan sonra kullanım amacına bağlı olarak mevcut disk boyutunun artırılması gerekebilir. Özellikle bazı kullanım senaryolarında disk boyutunun yetersiz olmasından dolayı işletim sisteminin performansında yavaşlık görülmektedir. Bunun için 2 alternatif yöntem kullanılmaktadır. Birinci yöntem olan mevcut diski büyütme ile sanal makine yazılımı üzerinde Linux İşletim Sisteminin mevcuttaki 25 GB olan diski artırılarak 40 GB olarak ayarlanacaktır. İkinci yöntemde ise sanal makineye disk ekleme yöntemiyle disk boyutu artırılabilmektedir. Bu kılavuzda birinci yöntem olan mevcut sanal makine diskini büyütme işlemi ile disk boyutunu artırma süreci anlatılmaktadır.

## Mevcut Disk Boyutunu Artırarak Diski Büyütme
Sanal makine yöneticisinde çalışan Linux İşletim Sisteminde mevcut diskin boyutunu artırmak için öncelikle Linux İşletim Sisteminin çalıştığı makine komut satırı arayüzünde sudo shutdown -h now komutu ile kapatılmalıdır. Makine kapatıldıktan sonra VirtualBox sanal ortam yöneticisinden disk boyutunu artırmak istediğimiz makinenin diski 40 GB olarak ayarlanmalıdır. 

İşlem tamamlandıktan sonra sanal makinemizi açıyoruz. Makine açıldıktan sonra terminal ekranından sunucuya aşağıdaki örnekte belirtildiği gibi ssh protolü ile bağlanıyoruz:
```ruby
ssh ubuntu@192.168.56.112
```

Disk boyutunu artırma işlemi süper kullanıcısı olmanın da ötesinde doğrudan root kullanıcısı olarak bir dizi komut satırı işlemi gerektirdiğinden öncelikle mevcut kullanıcıyı root kullanıcısı olarak değiştirmek gerekecektir. Bunun için `sudo -i` komutu kullanılacaktır. Bu komut süper kullanıcı parolası gerektirecektir.
```ruby
ubuntu@ubuntu-local:~$ sudo -i
[sudo] password for ubuntu:
root@ubuntu-local:~#
```

Sunucu disk bloklarını listelemek `lsblk` komutu çalıştırılacaktır.
```ruby
root@ubuntu-local:~# lsblk
```

lsblk komutunun verdiği bilgilere göre de makinenin diski sda’da sda1, sda2 ve sda3 olmak üzere 3bölümden oluşmaktadır. sda disk bölümü 60 GB’tır. sda3’ün altında pembe yazı rengi ile gösterilen satırda 38 GB’lık disk alanı kök dizin (/) tarafından kullanılmaktadır.

Buradan görüleceği üzere kök (/) dizini 38 GB’lık disk alanına sahip olup sanal makine yönetici panelinde20 GB’lık disk alanı işletim sisteminin kullanımına henüz sunulmamıştır. Bununla birlikte, boyutuartırılan diskin sda bölümüne eklendiği görülmektedir. Öncelikle `cfdisk /dev/sda`komutu ile bu eklenenyeni disk bölümlendirilecektir

cfdisk /dev/sda komutu sonrasında terminal ekranında yukarıdaki satırlar görünecektir. Burada oktuşları ile boş alan (Free space) satırına inildiğinde sanal makine yönetici panelinde eklenen 20 GB’lıkdiskin boş ve biçimlendirilmemiş olarak durduğu görülmektedir. Type sütunu altında 20 GB’lık diskiçin herhangi bir dosya sistemi bulunmamaktadır. Öncelikle sol altta bulunan New üzerine ok tuşları ilegelerek klavyede enter tuşuna basılmalıdır. Sol altta Partition size: 20G ifadesi görünecektir. Bu ifadegörüldüğünde klavyede enter tuşuna basılarak bölümlendirme boyutunun 20 GB olduğu onaylanacaktır.Enter tuşuna basıldıktan sonra son durum aşağıdaki gibi olacaktır. Bu aşamada değişikliğin etkin olmasıiçin ok tuşlarıyla Yaz (Write) üzerine gelinip klavyede enter tuşuna basılmalıdır. Sol altta belirecek Areyou sure you want to write the partition table to disk? ifadesinin karşısına klavyede yesyazıp enter tuşuna basılarak işlem tamamlanacaktır. Yes tuşuna basıldıktan sonra ok tuşlarıyla Quit üzerinegelinip enter tuşuna basılarak disk bölümlendirme ekranından çıkılacaktır.

Disk bölümlendirmenin etkin olabilmesi için işletim sisteminin yeniden başlatılması istenmiyorsa `partprobe /dev/sda` komutu kullanılmalıdır. Bu komut kullanılmayacaksa disk bölümlendirmeninsistem tarafından algılanabilmesi için reboot ya da shutdown -r now komutlarından biri ile yenidenbaşlatılması gerekmektedir.
```ruby
root@ubuntu-local:~# partprobe /dev/sda
```

lsblk komutu tekrar çalıştırıldığında aşağıda mavi yazı rengi ile görüldüğü üzere 20 GB’lık diskin sda4olarak disk bloğuna eklenmiş olarak karşımıza çıkacaktır:
```ruby
root@ubuntu-local:~# lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
loop0 7:0 0 62M 1 loop /snap/core20/1611
loop1 7:1 0 79.9M 1 loop /snap/lxd/22923
loop2 7:2 0 103M 1 loop /snap/lxd/23541
loop3 7:3 0 44.7M 1 loop /snap/snapd/15534
loop4 7:4 0 63.2M 1 loop /snap/core20/1623
loop5 7:5 0 47M 1 loop /snap/snapd/16292
sda 8:0 0 60G 0 disk
├─sda1 8:1 0 1M 0 part
├─sda2 8:2 0 2G 0 part /boot
├─sda3 8:3 0 38G 0 part
│ └─ubuntu--vg-ubuntu--lv 253:0 0 38G 0 lvm /
└─sda4 8:4 0 20G 0 part
sr0 11:0 1 1024M 0 rom
```

Oluşturulan bir disk bölümünün Mantıksal Bölüm Yönetimi (LVM) tarafından algılanabilmesi içinöncelikle bu bölümün fiziksel bölüm (pyhsical volume) hâline getirilmesi gereklidir. sda4 fiziksel taraftanbakıldığında disk bölümü olmakla birlikte Mantıksal Bölüm Yönetimi tarafından bakıldığında bir fizikselbölümdür. Öncelikle `pvcreate /dev/sda4` komutu ile sda4 adında bir fiziksel bölüm oluşturulmalıdır
```ruby
root@ubuntu-local:~# pvcreate /dev/sda4
Physical volume “/dev/sda4” successfully created.
```
Sonrasında vgdisplay komutu ile diskin ekleneceği bölüm grubu adı (ubuntu-vg) öğrenilecektir. Bölüm
grubu adı bir sonraki aşamada bölümü büyütme komutu olan vgextend ile kullanılacaktır.
```ruby
root@ubuntu-local:~# vgdisplay
--- Volume group ---
VG Name ubuntu-vg
System ID
Format lvm2
Metadata Areas 1
Metadata Sequence No 4
VG Access read/write
VG Status resizable
VG Size <38.00 GiB
PE Size 4.00 MiB
Total PE 9727
Alloc PE / Size 9727 / <38.00 GiB
Free PE / Size 0 / 0
VG UUID H5kXUE-YF5c-tbl2-Qnyo-oYz1-m9m5-1QQdIH
```

vgextend ve ardından disk grubu_adı (ubuntu-vg) ve yeni disk bloğu (/dev/sad4) parametreleri aşağıdaki
gibi girilerek ubuntu-vg disk grubu genişletilecektir


Sonraki aşamada mantıksal bölüm disk yolunu öğrenilecektir. Mantıksal bölüm disk yolu mantıksal
bölümü genişletmek için kullanılan lvextend komutu ile eklenen diskteki tüm boş alanın mevcut mantıksal
bölüme eklenmesinde kullanılacaktır. lvdisplay komutu ile mantıksal bölüm disk yolu öğrenilecektir

root@ubuntu-local:~# lvdisplay
--- Logical volume ---
LV Path /dev/ubuntu-vg/ubuntu-lv
LV Name ubuntu-lv
VG Name ubuntu-vg
LV UUID MXO2ud-hQ0P-1h13-arTd-Mo6Y-uh4V-Y7YEvV
LV Write Access read/write
LV Creation host, time ubuntu-server, 2022-08-19 06:02:58 +0000
LV Status available
# open 1
LV Size <38.00 GiB
Current LE 9727
Segments 1
Allocation inherit
Read ahead sectors auto
- currently set to 256
Block device 253:0

Sonraki aşamada lvextend komutu aşağıdaki gibi kullanılarak mantıksal bölüm tüm boş alan kullanılacak
şekilde genişletilecektir:
root@ubuntu-local:~# lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
Size of logical volume ubuntu-vg/ubuntu-lv changed from <38.00 GiB (9727 extents)
to 57.99 GiB (14846 extents).
Logical volume ubuntu-vg/ubuntu-lv successfully resized.
Son adımda yeni oluşturulan boş alan kök (/) dizinine eklenerek dosya sistemi yeniden boyutlandırılacaktır.
Bunun için resize2fs komutu mantıksal bölüm konumu ile birlikte kullanılacaktır:
root@ubuntu-local:~# resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 5, new_desc_blocks = 8
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 15202304 (4k) blocks long.
Artık df -h komutu ile sanal makinenin mevcut disk durumu görüntülenecektir. Görüleceği üzere disk
eklenmeden önce 38 GB olan kök dizinin boyutu disk ekleme sürecinden sonra 57 GB olarak artırılmıştır.
root@ubuntu-local:~# df -h
Filesystem Size Used Avail Use% Mounted on
tmpfs 393M 1.1M 392M 1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv 57G 7.6G 47G 14% /
tmpfs 2.0G 0 2.0G 0% /dev/shm
tmpfs 5.0M 0 5.0M 0% /run/lock
/dev/sda2 2.0G 247M 1.6G 14% /boot
tmpfs 393M
