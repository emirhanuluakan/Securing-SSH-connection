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

İki tane dosya oluşacak: birisi **.pub uzantılı**, diğeri ise uzantısız sadece bir dosya. Burada **sshkey.pub** dosyası bir **Public Key** yani **Açık Anahtar**dır. Paylaşılmasında sakınca yoktur ve bağlanılacak sisteme (bizim durumumuzda sunucuya) aktarılır.

Diğeri ise **sshkey** dosyası ile **Private Key** yani **Özel Anahtar**dır ve paylaşılmaması gerekir. Kimlik doğrulama yapılırken açık anahtar, bağlanan cihaza bir _soru_ gönderir ve bu sorunun cevabı sadece ama sadece özel anahtarda mevcuttur. Böylelikle kimlik doğrulanır.

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
```
cd ~
```
- Ana dizine gidersiniz
```
ls
```
- Olduğunuz dizindeki dosyaları listeler

Listelenen dosyalarda `sshkey.pub` görüyorsanız doğru yoldasınız. SCP başarıyla gerçekleşmiş. Şimdi bu **Public Key**'i işletim sisteminin SSH ayarlarının yapıldığı dosyada yazan dizine aktarmamız gerek. Bu dizin çoğu Linux işletim sisteminde aynıdır (`.ssh/authorized_keys`).
```
mkdir .ssh
```
- Olduğunuz dizinin `~` olduğuna emin olun. Bunları ana dizinde gerçekleştiriyoruz. Bu komut, `~` dizininde `.ssh` dizinini (klasörünü) oluşturur.
```
touch .ssh/authorized_keys
```
- `touch` komutu dosya oluşturmak için kullanılır.
- `mkdir` komutu klasör oluşturmak için kullanılır.
