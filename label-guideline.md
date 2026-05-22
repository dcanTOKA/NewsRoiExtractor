# Turkish News RoI Dataset Guidelines

### 1. Veri Kumesi 

Veri kumesi, farkli DOM yapilarini ve sablon cesitliligini temsil etmek amaciyla populer ulusal, yerel ve tematik 45 farkli Turkce haber sitesinden derlenen 100 adet guncel haber sayfasindan olusmaktadir.

Veri kumesinde yer alan haber siteleri:

* aa.com.tr
* ahaber.com.tr
* bbc.com
* benguturk.com
* birgun.net
* bthaber.com
* canakkaleolay.com
* cnnturk.com
* cumhuriyet.com.tr
* dha.com.tr
* dogrulukpayi.com
* donanimhaber.com
* dw.com
* emsal.com
* euronews.com
* evrensel.net
* gazeteoksijen.com
* goal.com
* gzt.com
* haber.sol.org.tr
* haberler.com
* haberturk.com
* hurriyet.com
* indeksonline.net
* karamangundem.com
* kenttv.net
* kocaelidenge.com
* koroglugazetesi.com
* medyascope.tv
* memurlar.net
* mersinhaber.com
* milliyet.com.tr
* ntv.com.tr
* paraajansi.com.tr
* paratic.com
* patronlardunyasi.com
* sacitaslan.com
* saglikaktuel.com
* sde.org.tr
* sondakika.com
* sozcu.com.tr
* star.com.tr
* t24.com.tr
* usakgundem.com
* yenisafak.com

### Etiketleme Sureci

Iki asamali bir etiketleme stratejisi izlenmistir.

1. Cekirdek Kume (Uzman Etiketli - 17 veri): 17 adet haber sayfasi, alan uzmanlari tarafindan bu dokumanda yer alan kurallara gore etiketlenmis referans metinleri olusturmustur.
2. Genisletilmis Kume (Yazar Etiketli - 83 Veri): Kalan 83 veri, uzmanlarin cekirdeki kumedeki tutarliligini korumak adina, ayni metodoloji takip edilerek yazarlar tarafindan etiketlenmistir.

### Etiketleme Kurallari

Ground-Truth metinler olusturulurken asagidaki kurallar uygulanmistir.

#### Dahil Edilenler
* Haber basligi (`<h1>`, `<h2>`)
* Haber girisi (ozet)
* Ana metin govdesi (`<p>`))
* Sayfa icindeki hiyerarsik alt basliklar (`<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`)
* Liste ogeleri (duz metin olarak alinmistir) (`<ul>`, `<ol>`, `<li>`)

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

Tum veri kumesinde (Cekirdek + Genisletilmis Kume) yer alan 100 verinin konus basliklarina gore dagilimi asagidaki gibidir:

| Konu Kategorisi | Veri Sayısı | Yüzde (%) |
| --- | --- | --- |
| Diğer (Genel, Gündem, Magazin vb.) | 30 | 30.0% |
| Finans | 23 | 23.0% |
| İş Dünyası | 23 | 23.0% |
| Yerel Haberler | 15 | 15.0% |
| Ekonomi | 9 | 9.0% |
| **Toplam** | **100** | **100%** |

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