# Bulk PDF Certificate Issuer
> SVG şablonlarından kimlik ve QR doğrulama içeren kişiselleştirilmiş PDF sertifikaları üreten Python uygulaması.

## Açıklama

Katılımcı bilgilerini TSV dosyasından okuyarak kişiselleştirilmiş PDF sertifikaları toplu biçimde üreten bir konsol uygulamasıdır. Sertifika tasarımındaki ve mesaj şablonundaki değişkenleri her katılımcı için doldurur, benzersiz bir sertifika kimliği oluşturur ve hazırlanan SVG dosyasını Inkscape üzerinden PDF biçimine dönüştürür.

Her sertifika için doğrulama bağlantısını içeren bir QR kod oluşturulur. Üretilen sertifikaların kimlikleri kayıt altında tutulur ve gerektiğinde kimlikleri üzerinden iptal edilebilir. Her çalıştırmada, yeni oluşturulan sertifikalar için [Personalized Bulk Email Sender](https://github.com/seymenkonuk/personalized-bulk-email-sender) ile uyumlu bir TSV dosyası hazırlanır. Oluşturulan bu dosya ve ilgili araç kullanılarak sertifikalar ilgili kişilere hızlı ve toplu biçimde e-posta yoluyla gönderilebilir.

## Özellikler

- TSV dosyasından katılımcı listesi okuma
- Her katılımcı için kişiselleştirilmiş sertifika oluşturma
- Özelleştirilebilir SVG sertifika tasarımları
- Her sertifika için benzersiz kimlik oluşturma
- Doğrulama bağlantısı içeren QR kod üretme
- SVG dosyalarını PDF biçimine dönüştürme
- PDF içindeki metinleri yola dönüştürme
- Üretilen sertifikaların kimliklerini kayıt altında tutma
- Sertifikaları kimlikleri üzerinden iptal etme
- [Personalized Bulk Email Sender](https://github.com/seymenkonuk/personalized-bulk-email-sender) ile uyumlu TSV dosyası oluşturma
- Docker ile çalıştırma

## Kurulum

Proje Python 3.9 ile çalışacak şekilde hazırlanmıştır.

Gerekli yazılımlar:

- Python 3.9 veya üzeri
- Inkscape
- Docker ve Docker Compose (isteğe bağlı)

> 💡 **Not:** Programın Docker üzerinden çalıştırılması tavsiye edilir.

Projeyi klonlayın:

```bash
git clone https://github.com/seymenkonuk/bulk-pdf-certificate-issuer.git
cd bulk-pdf-certificate-issuer
```

Python bağımlılıklarını yükleyin:

```bash
pip install -r requirements.txt
```

> 💡 **Not:** QR kodlar çevrim içi bir servis üzerinden oluşturulduğu için uygulama çalışırken internet bağlantısı gereklidir.

## Yapılandırma

### Genel Ayarlar

`settings/settings.conf` dosyasını düzenleyin:

```ini
[General]
certificate_template_svg=assets/templates/certificate/basic.svg
certificate_verification_base_url=https://example.com/certificates
certificate_verification_include_file_extension=true
certificate_id_length=10
text_to_path=true

[Message]
certificate_message_template=assets/templates/message/example1
certificate_message_x_coordinate=144.28885
certificate_message_y_coordinate=121.16418
certificate_message_row_height=10
certificate_message_row_max_length=70
```

- `certificate_template_svg`: Kullanılacak SVG sertifika şablonunun yolu
- `certificate_verification_base_url`: QR kodun yönlendireceği temel doğrulama adresi
- `certificate_verification_include_file_extension`: Doğrulama bağlantısına `.pdf` uzantısının eklenip eklenmeyeceği
- `certificate_id_length`: Oluşturulacak sertifika kimliğinin karakter sayısı
- `text_to_path`: PDF oluşturulurken metinlerin yola dönüştürülüp dönüştürülmeyeceği
- `certificate_message_template`: Kullanılacak mesaj şablonu dizininin yolu
- `certificate_message_x_coordinate`: Mesajın SVG üzerindeki yatay başlangıç konumu
- `certificate_message_y_coordinate`: Mesajın SVG üzerindeki dikey başlangıç konumu
- `certificate_message_row_height`: Mesaj satırları arasındaki dikey mesafe
- `certificate_message_row_max_length`: Bir satırda bulunabilecek yaklaşık en fazla karakter sayısı

> 💡 **Not:** Mesaj konumu ve satır ayarları kullanılan SVG şablonuna göre düzenlenmelidir.

> 💡 **Not:** `text_to_path` değeri `true` olduğunda sertifikadaki metinler PDF oluşturulurken yola dönüştürülür. Bu işlem, kullanılan yazı tiplerinin alıcı cihazda kurulu olmadığı durumlarda görünümün korunmasını sağlar.

### Mesaj Şablonu

Yeni bir mesaj şablonu oluşturunuz veya varolan mesaj şablonunu (`assets/templates/message/example1`) düzenleyiniz:

> 💡 **Not:** Her mesaj şablonu, `message.conf` ve `message.txt` dosyalarını içeren bir dizindir. 

`message.conf` dosyasında tüm sertifikalar için ortak sabitleri tanımlayın:

```ini
[Constant]
EVENT_NAME=Example Event
START_DATE=01.01.2026
END_DATE=30.01.2026
```

> 💡 **Not:** `message.conf` dosyasında ortak sabit kullanılmasa bile `[Constant]` bölümü bulunmalıdır.

`message.txt` dosyasında ortak sabitleri `{{#DEGISKEN#}}`, katılımcıya özel değerleri ise `{{$DEGISKEN$}}` biçiminde kullanın:

```text
{{#START_DATE#}} ile {{#END_DATE#}} tarihleri arasında gerçekleştirilen
{{#EVENT_NAME#}} eğitimine katılımınızdan dolayı
bu belgeyi almaya hak kazandınız.

Katılımcı: {{$NAME$}}
Başarı Puanı: {{$SCORE$}}
```

> 💡 **Not:** Değişken adları büyük-küçük harfe duyarlıdır.

> 💡 **Not:** Katılımcı değişkenleri `new_participants.tsv` dosyasında tanımlanmalıdır.

> 💡 **Not:** Katılımcı değişkenlerinin adları `new_participants.tsv` dosyasındaki sütunlarla aynı olmalıdır.

> 💡 **Not:** Mesaj satırları boşluk karakterleri esas alınarak bölünür. Belirlenen satır uzunluğunu aşan kesintisiz ifadelerden kaçınılmalıdır.

### Sertifika Şablonu

Yeni bir sertifika tasarımı oluşturunuz veya var olan SVG şablonlarından birini düzenleyiniz.

Sertifika şablonunda mesaj sabitleri ve katılımcı değişkenlerinin yanında aşağıdaki özel değişkenler de kullanılabilir:

- `{{CERTIFICATE_ID}}`: Oluşturulan benzersiz sertifika kimliği
- `{{QR_CODE}}`: Oluşturulan QR kod görselinin mutlak dosya yolu
- `{{MESSAGE}}`: Mesaj şablonundan hazırlanan ve satırlara ayrılan sertifika metni

> 💡 **Not:** SVG şablonundaki `{{MESSAGE}}` alanının metin özellikleri ve konumu, genel ayarlardaki koordinatlarla uyumlu olmalıdır.

### Katılımcı Listesi

Sertifika oluşturulacak kişileri ve kişiye özel verileri `inputs/new_participants.tsv` dosyasına sekmeyle (tab) ayrılmış biçimde ekleyin:

```text
NAME	EMAIL
Recep	recep@example.com
Seymen	seymen@example.com
```

Zorunlu sütunlar:

- `NAME`: Katılımcının adı
- `EMAIL`: Katılımcının e-posta adresi

TSV dosyasına istenildiği kadar sütun eklenebilir. Eklenen sütunlarda bulunan değerler mesaj ve sertifika şablonlarında katılımcı değişkeni olarak kullanılabilir:

```text
NAME	EMAIL	SCORE
Recep	recep@example.com	100
Seymen	seymen@example.com	90
```


> 💡 **Not:** Microsoft Excel'den kopyalanan tablo verileri sekmeyle ayrılmış olarak yapıştırılabilir.

> 💡 **Not:** Uygulama çalıştırıldıktan sonra `new_participants.tsv` dosyası otomatik olarak temizlenir.

### İptal Edilecek Sertifikalar

İptal edilecek sertifikaların kimliklerini `inputs/revoked_certificates.lst` dosyasına her satıra bir kimlik gelecek biçimde ekleyin:

```text
Ab12Cd34Ef
Xy98Zt76Qr
```

Uygulama çalıştırıldığında bu kimliklere ait:

- PDF sertifika dosyaları,
- QR kod görselleri,
- `all_issued_certificates.lst` içindeki kayıtlar

silinir.

> 💡 **Not:** Uygulama çalıştırıldığında önce `revoked_certificates.lst` dosyasındaki sertifikalar iptal edilir, daha sonra `new_participants.tsv` dosyasındaki sertifikalar üretilir.

> 💡 **Not:** Uygulama çalıştırıldıktan sonra `revoked_certificates.lst` dosyası otomatik olarak temizlenir.

## Çalıştırma

### Docker

Önce Docker imajını oluşturun:

```bash
docker compose build
```

Uygulamayı çalıştırın:

```bash
docker compose run --rm bulk-pdf-certificate-issuer 
```

Bu komut proje dizinini container içindeki `/app` dizinine bağlar. Oluşturulan sertifikalar ve diğer çıktılar yerel `outputs/` dizinine kaydedilir.

> 💡 **Not:** Sisteminizde Docker ve Docker Compose kurulu olmalıdır.

### Python

Uygulamayı Docker kullanmadan proje kök dizininden doğrudan çalıştırabilirsiniz:

```bash
python src/main.py
```

> 💡 **Not:** Sisteminizde Python ve Inkscape kurulu olmalıdır.

> 💡 **Not:** Inkscape komutunun sistem yolundan erişilebilir olması gerekir.

> 💡 **Not:** Doğrudan Python ile çalıştırırken sertifika şablonunda kullanılan yazı tiplerinin sistemde kurulu olması gerekir. Docker imajı, `assets/fonts/` dizinindeki yazı tiplerini otomatik olarak yükler.

## Çıktılar

Uygulama çalıştırıldığında aşağıdaki dosyalar oluşturulur veya güncellenir:

```text
outputs/
├── certificates/
│   ├── <certificate_id>.pdf
│   ├── all_issued_certificates.lst
│   └── latest_issued_certificates.lst
├── qrcodes/
│   └── <certificate_id>.png
└── mailsend.tsv
```

- `certificates/<certificate_id>.pdf`: Katılımcı için oluşturulan PDF sertifikası
- `qrcodes/<certificate_id>.png`: Sertifika doğrulama bağlantısını içeren QR kod
- `all_issued_certificates.lst`: İptal edilmemiş tüm sertifika kimlikleri
- `latest_issued_certificates.lst`: Son çalıştırmada oluşturulan sertifika kimlikleri
- `mailsend.tsv`: Son çalıştırmada oluşturulan sertifikaların e-posta gönderim listesi

> 💡 **Not:** `latest_issued_certificates.lst` ve `mailsend.tsv` her çalıştırmada sıfırlanarak yeniden oluşturulur.

> 💡 **Not:** `all_issued_certificates.lst` önceki çalıştırmalarda oluşturulan ve iptal edilmemiş sertifika kimliklerini korur.

## Galeri

### Sertifika Oluşturma

![Sertifika oluşturma](assets/demo/running.gif)

### Yapılandırma

![Yapılandırma](assets/demo/config.gif)

## Lisans

Bu proje [MIT Lisansı](https://github.com/seymenkonuk/bulk-pdf-certificate-issuer/blob/main/LICENSE) ile lisanslanmıştır.