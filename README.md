# SSH Bağlantısını Güvenli Hale Getirme

  Sunucu kurduktan sonra kurduğunuz sunucunun IP'si, kullanıcı adı(yüksek ihtimal root) ve şifresini öğrendikten sonra, SSH ile bağlantıyı:
```
ssh root@SUNUCU_IP
```
  yazarak sağlayabilirsiniz. Eğer **root** yetkisiyle giriş yapamıyorsanız, bu kurduğunuz sunucu servisinin **root** yetkisi ile SSH yapılmasını engellediğinden kaynaklanıyor olabilir _-ki bu da istediğimiz bir şey-_.

  Komutu "root" veya sunucu servisinin size belirttiği kullanıcı adıyla girdikten sonra sunucu servisinin belirttiği şifreyi girerek SSH ile bağlantıyı gerçekleştiriyoruz.

## Kullanıcı Oluşturmak

__EĞER _root_ YETKİSİYLE GİREBİLİYORSANIZ BU KISMI YAPMANIZ GEREKİR!__

Root ile giriş yaptığımız sunucumuza şu komutları girerek yeni kullanıcı oluşturuyoruz. Bu kullanıcı yönetici yetkisine **sudo** komutunu kullanarak sahip olur.

```
adduser kullanici_adi
```
- `adduser` komutu kullanıcı oluşturmayı sağlar
```
usermod -aG sudo kullanici_adi
```
- **kullanici_adi** adlı kullanıcıyı **sudo** grubuna eklemeye yarar. **sudo** grubunda olan kullanıcılar yönetici yetkilerine sahip olur.

Komutu girdikten sonra kullanıcıya ait şifre oluşturmak istenecek. İstediğiniz şifreyi yazın.

- `cd nano /etc/sudoers` komutunu girerek kullanıcı izinlerini görebilirsiniz. (Opsiyonel)

Kullanıcıya geçiş yapmak için şu komutu giriyoruz:
```
su kullanici_adi
```

## SSH anahtar çifti oluşturmak

Kendi local cihazınızın terminaline (Windowsta Powershell) şu komutları girmelisiniz:
```
cd .ssh
```
- .ssh dizinine gidersiniz.
```
ssh-keygen -t rsa -b 4096
```
- `ssh-keygen` ssh anahtar çifti oluşturmak için kullanılan komuttur.
- `-t rsa` anahtarın türü.
- `-b 4096` anahtarın bit cinsinden uzunluğu. SSH anahtar çifti oluştururken genelde 2048 ve 4096 bit kullanılır. bazı anahtar türlerinde bit değeri sabit olduğundan `-b` girilmez.

Komutu girdikten sonra anahatarın adı sorulacaktır. Burada **sshkey** yazıyorum. Siz ne yazmak isterseniz onu yazın. Türkçe karakterlerden kaçının.

Sonrasında passphrase soracak. Bu, anahtar çiftine ekelyeceğiniz ek bir şifre. Koymak zorunda değiliz fakat konumuz güvenlik olduğundan dolayı ekliyorum.

Oluşup oluşmadığını kontrol etmek için .ssh dizinindeyken "ls -l" yazarsanız .ssh dizininde olan dosyaları görebilirsiniz. daha farklı listeleme için lütfen "ls" komutunu araştırın.
- `.ssh` klasörü, Windows'ta C:\Users\kullanici_adi\ dizininde yer alır.

İki tane dosya oluşacak: birisi **.pub uzantılı**, diğeri ise uzantısız sadece bir dosya. Burada **sshkey.pub** dosyası bir **Public Key** yani **Açık Anahtar**dır. Paylaşılmasında sakınca yoktur ve bağlanılacak sisteme (bizim durumumuzda sunucuya) aktarılır.

Diğeri ise **sshkey** dosyası, **Private Key** yani **Özel Anahtar**dır ve paylaşılmaması gerekir. Kimlik doğrulama yapılırken açık anahtar, bağlanan cihaza bir _soru_ gönderir ve bu sorunun cevabı sadece ama sadece özel anahtarda mevcuttur. Böylelikle kimlik doğrulanır.

Oluşan anahtarı sunucumuza kopyalamak için `scp` komutunu kullanacağız (Unutmayın, hala local cihazdayız).
```
scp sshkey.pub kullanici_adi@SUNUCU_IP:
```
- `scp` **Secure Copy Protocol** anlamına gelir. Bu komutla dosyaları kopyalayabilriz.
- `sshkey.pub` kısmına oluşturduğunuz SSH anahtar çiftinin açık anahtarını (.pub) giriyoruz.
- `kullanici_adi@SUNUCU_IP` kısmında ise oluşturduğumuz kullanıcı adı ve bağlanacağımız sunucunun IPv4'ünü yazıyorsunuz.
- sondaki `:` karakteri ise kopyalanılacak dosyanın, sunucuda hangi dizine kopyalanacağını belirtir. Eğer bir şey yazmasaydık `/home/kullanici_adi` dizininde aktaracaktı. Fakat `:` karakteri ise kullanıcının ana dizinine kopyalamasını sağayacak (`~`).

Şifre sorduğu zaman sunucunun şifresini girin ve kopyalama gerçekleşecek.

## Anahtar çiftini tanımlama ve SSH yapılandırması

### Anahtar çiftini tanımlama
Ana dizine gitmemiz gerekiyor
```
cd ~
```
- `cd` ana dizine gidersiniz

Ana dizindeki dosyaları listeler
```
ls
```
- `ls` olduğunuz dizindeki dosyaları listeler

Listelenen dosyalarda `sshkey.pub` görüyorsanız doğru yoldasınız. SCP başarıyla gerçekleşmiş. Şimdi bu **Public Key**'i işletim sisteminin SSH ayarlarının yapıldığı dosyada yazan dizine aktarmamız gerek. Bu dizin çoğu Linux dağıtımalarında aynıdır (`.ssh/authorized_keys`).
Olduğunuz dizinin `~` olduğuna emin olun. Bunları ana dizinde gerçekleştiriyoruz. Aşağıdaki komut, `~` dizininde `.ssh` dizinini (klasörünü) oluşturur.
```
mkdir .ssh
```
- `mkdir` komutu klasör oluşturmak için kullanılır.
Şimdi ise .ssh klasörüne authorized_keys dosyasını oluşturalım. Bu dosya, **Public Key**'lerin içeriğine sahip olan dosyadır.
```
touch .ssh/authorized_keys
```
- `touch` komutu dosya oluşturmak için kullanılır.
Şimdi ise `sshkey.pub` içeriğini `authorized_keys` dosyasına aktarmamız gerek.
```
cat sshkey.pub >> .ssh/authorized_keys
```
- `cat` komutu belirli dosyaların içeriğini belirli dosyalara yazılmasını sağlar.
- `>>` operatörü solundaki dosyanın içeriğini sağındaki dosyaya _ekler_. Eğer `>` kullansaydık mevcut içerik silinip yerine yazılırdı.
Eğer burada iki tane **Public Key** tanımlamak isteseydik, **Public Key**'leri konumlarıyla birlikte sırasıyla yazmamız gerekirdi. Aşağıya örneğini ekliyorum. Bu örnek, `>>` kullanıldığından bir içeriği başka bir içeriğin yerine yazmaz, ekleme yapar.
```
cat ~/sshkey1.pub ~/sshkey2.pub >> .ssh/authorized_keys
```
- Ana dizinde olduğumuz için `~/` eklememiz gerekmiyor, örnek olduğundan dolayı değinmek istedim.

Artık `sshkey.pub` dosyasına gerek kalmadı çünkü içeriğini `authorized_keys` dosyasına _aktardık_. Silmek için: 
```
rm sshkey.pub
```
- `rm` komutu silme işlemini gerçekleştirir.

### SSH Yapılandırması
Şimdi ise SSH yapılandırmasını ayarlamamız gerek.
```
sudo nano /etc/ssh/sshd_config
```
- `nano` komutu metin düzenleme komutu. belirlenen dizindeki dosyayı düzenlemeye yarar. `sshd_config` dosyası, SSH ayarlarının yapıldığı dosyadır.
- Dosya düzenlemesinden `nano` komutu haricinde `vim` de kullanılıyor. Kullandığınız komutun detayları için lütfen araştırın.

Başında **#** yazan satırlar yorum satırlarıdır. Başındaki bu işareti kaldırarak yorum satırı olmamasını sağlayabilirsiniz. Yorum satırı olan her satır, varsayılan SSH yapılandırmasına aittir. Değiştirmek istediğiniz ayarların yorum işaretini değiştirerek düzenleyebilirsiniz. Aşağıda yazılanların yorum işaretlerini silip yanındaki değeriyle düzenleyin:
```
PermitRootLogin no 
PubkeyAuthentication yes 
Password Authentication no 
AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2
```
Satırlar sırasıyla şunları temsil eder:
- SSH bağlantısını root olarak bağlanılmasını engeller.
- Public key doğrulamasını aktif eder.
- Kullanıcı şifresiyle doğrulamayı kapatır.
- Public key'lerin hangi dizinde ve hangi dosyada olduğunu belirtir. Bu dizini ve dosyayı biz yukarıda oluşturmuştuk. Varsayılan olarak (Başında # işareti varsa) bizim oluşturduğumuz dizin ve dosyaya sahipse değiştirmenize gerek yok.

Yukarıdaki dördü değişmesi/düzenlenmesi gereken en önemli olanlardı. **Alttakiler ise varsayılan olarak geliyor. Değiştirmenize gerek yok, referans olsun diye ekliyorum.** (Kullandığım dağıtım Ubuntu 24.04 LTS. Siz farklı bir dağıtım kullanıyorsanız alttaki seçenekler farklı olabilir)

```
Include /etc/ssh/sshd_config.d/*.conf
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
UsePam yes
X11Forwarding yes
PrintMotd no 
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server
```

Yukarıdaki gibi düzenledikten sonra **CTRL+X** tuşlarına basarak çıkıyoruz. Çıkarken "Save modified buffer?" diye soracak, **Y** tuşuna basıyoruz ve sonrasında düzenlenen dosyanın adını ve dizinini düzenlemek için bir kısım açılıyor. Direkt **Enter** tuşuna tıklarsanız adını ve dizini değiştirmeden kaydedecektir.

SSH hizmetini yeniden başlatmamız gerekiyor.
```
sudo systemctl restart sshd
```
- SSH hizmetini yeniden başlatır. buradaki `systemctl` ise servis ve sistem bileşenlerini yönetir.

Şifre sorarsa kullanıcı adının şifresini giriniz.
```
exit
```
root'tan kullanıcı adına `su kullanici_adi` ile geçiş yaptığımız için `exit` dediğimizde root durumuna geri döndük.
```
exit
```
Tekrardan `exit` diyerek root durumundan çıkış yapıyoruz.

## SSH Bağlantısı Kurma
SSH bağlantısını `ssh kullanici_adi@SUNUCU_IP` yazarak bağlanıyorduk fakat şimdi alttaki komutu yazarak bağlanacağız.
```
ssh -i PRIVATE_KEY_DOSYA_YOLU kullanici_adi@SUNUCU_IP
```
- **PRIVATE_KEY_DOSYA_YOLU** kısmına özel anahtarınızın dosya yolunu yazmanız lazım.
    - Windows'ta özel anahtara sağ tıklayın ve özellikler'e basın. Dosya yolunu kopyalayın. Örnek olarak: `C:\Users\kullanici_adi\.ssh`. Burada yazan dosya yoludur. Anahtarın kendsini de belirtmemiz gerekiyor. Bunun için `C:\Users\kullanici_adi\.ssh\sshkey` yazmamız gerekiyor.
Passphrase eklediyseniz eğer size passphrase soracaktır. Yazdıktan sonra "Enter"a tıklayın ve başarıyla ssh bağlantınızı gerçekleştirin.
