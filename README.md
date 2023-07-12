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
