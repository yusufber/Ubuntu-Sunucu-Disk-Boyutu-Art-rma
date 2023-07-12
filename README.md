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

Buradan görüleceği üzere kök (/) dizini 38 GB’lık disk alanına sahip olup sanal makine yönetici panelinde20 GB’lık disk alanı işletim sisteminin kullanımına henüz sunulmamıştır. Bununla birlikte, boyutuartırılan diskin sda bölümüne eklendiği görülmektedir. Öncelikle `cfdisk /dev/sda komutu` ile bu eklenenyeni disk bölümlendirilecektir
