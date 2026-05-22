# Turkish News RoI Dataset Guidelines

### 1. Veri Kumesi 

Veri kumesi, farkli DOM yapilarini ve sablon cesitliligini temsil etmek amaciyla populer ulusal, yerel ve tematik 35 farkli Turkce haber sitesinden derlenen 100 adet guncel haber sayfasindan olusmaktadir.

Veri kumesinde yer alan haber siteleri:

* sondakika.com
* cnnturk.com
* haberturk.com
* sde.org.tr
* memurlar.net
* hurriyet.com
* canakkaleolay.com
* mersinhaber.com
* dw.com
* gzt.com
* ahaber.com.tr
* aa.com.tr
* goal.com
* indeksonline.net
* karamangundem.com
* haberler.com
* saglikaktuel.com
* bthaber.com
* yenisafak.com
* medyascope.tv
* milliyet.com.tr
* koroglugazetesi.com
* emsal.com
* birgun.net
* dha.com.tr
* donanimhaber.com
* bbc.com
* evrensel.net
* ntv.com.tr
* gazeteoksijen.com
* t24.com.tr
* paratic.com
* euronews.com
* dogrulukpayi.com
* benguturk.com

### Etiketleme Sureci

Iki asamali bir etiketleme stratejisi izlenmistir.

1. Cekirdek Kume (Uzman Etiketli - 17 veri): 17 adet haber sayfasi, alan uzmanlari tarafindan bu dokumanda yer alan kurallara gore etiketlenmis referans metinleri olusturmustur.
2. Genisletilmis Kume (Yazar Etiketli - 83 Veri): Kalan 83 veri, uzmanlarin cekirdeki kumedeki tutarliligini korumak adina, ayni metodoloji takip edilerek yazarlar tarafindan etiketlenmistir.

### Etiketleme Kurallari

Ground-Truth metinler olusturulurken asagidaki kurallar uygulanmistir.

#### Dahil Edilenler
* Haber basligi
* Haber girisi (ozet)
* Ana metin govdesi
* Sayfa icindeki hiyerarsik alt basliklar
* Liste ogeleri (string)

#### Dahil Edilmeyenler
* Reklamlar
* "Ilgili diger haberler" yonlendirmeleri
* Yorum alanlari
* Sosyal medya paylasim butonlari
* Abonelik formlari
* Scriptler
* CSS ve stil kodları
* Sayfa dışı ve bağlam dışı site ögeleri (Footer, Menü vb.)

### Veri Kumesi Istatistikleri ve Dagilimi

Genisletilmis kumede yer alan 83 verinin konus basliklarina gore dagilimi asagidaki gibidir:

| Konu Kategorisi | Veri Sayısı | Yüzde (%) |
| --- | --- | --- |
| Diğer (Genel, Gündem, Magazin vb.) | 27 | 32.5% |
| Finans | 20 | 24.1% |
| İş Dünyası | 19 | 22.9% |
| Ekonomi | 9 | 10.8% |
| Yerel Haberler | 8 | 9.6% |
| **Toplam** | **83** | **100%** |

### Dataset Formati

YAML dosyası her örnek için:
 
- `url`: haber detay sayfası
- `gt`: gold truth (başlık + metin, tek string)
 
Örnek:
 
```yaml
- url: "https://example.com/news/123"
  gt: |
    Haber Başlığı
    Birinci paragraf...
    İkinci paragraf...
```
 
Notlar:
 
- `gt` içinde başlıkların bulunduğu varsayılır.
- Eval sırasında hem `gt` hem prediction hafif normalize edilir (Markdown heading marker’ları silinir, whitespace normalize edilir).