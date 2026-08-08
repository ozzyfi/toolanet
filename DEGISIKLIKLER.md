# Değişiklik kaydı

Bu klasör `toola-landing` klasörünün doğrudan yerine geçer. Tasarım dili, renk
sistemi, tipografi ve metinler korunmuştur. Aşağıdakiler dışında hiçbir şeye
dokunulmamıştır.

Doğrulama: Chromium'da 320 / 360 / 375 / 390 / 414 / 480 / 620 / 768 / 900 /
1024 / 1280 / 1440 / 1920 px genişliklerde, TR ve EN olmak üzere 26 kombinasyonda
ölçüldü. Hepsinde yatay taşma yok, JS hatası yok.

---

## Kritik — mobilde sayfayı bozan hatalar

### 1. Satır içi ızgara stilleri medya sorgularını eziyordu
`.stats` (sektör verileri) ve `.ctr-metrics` (merkez kontrol paneli) ızgaraları
`style="grid-template-columns:repeat(3,1fr)"` biçiminde satır içi
tanımlanmıştı. Satır içi stil her zaman stil sayfasını yener, dolayısıyla
`@media(max-width:620px)` içindeki tek sütuna düşürme kuralları **hiç
çalışmıyordu**.

Ölçüm (düzeltme öncesi): 320px ekranda `document.scrollWidth = 617`.
Yani her telefonda sayfa yatay kayıyor, tam genişlikli arka planlar
(`.strip` gibi) ekranın ortasında kesiliyordu.

Yapılan: her iki ızgara da `.stats` / `.ctr-metrics` sınıflarına taşındı.
Sütunlar `minmax(0,1fr)` ile tanımlandı — bu, ileride benzer bir hata olsa bile
sütunların min-content genişliğine kilitlenip taşmasını engeller.
`.stats` artık 760px altında tek sütuna düşüyor.

### 2. H1 ekran dışına taşıyordu
`.display span{white-space:nowrap}` koşulsuzdu. Ölçüm: 375px ekranda başlık
satırı 593px yer kaplıyordu.

Yapılan: `nowrap` yalnızca `min-width:700px` üzerinde uygulanıyor. Altında
başlık doğal olarak sarıyor. Tasarlanan satır sonları masaüstünde aynen korundu.

### 3. Masaüstünde 9px'lik güvenlik payı
1280px+ ekranda İngilizce başlığın ilk satırı 1063px, kapsayıcı 1072px idi.
Font `font-display:swap` ile yüklendiğinden, yedek font gösterildiği sürede
metin daha geniş olup anlık yatay kaymaya yol açabiliyordu.

Yapılan: `clamp(2.25rem,5vw,4rem)` → `clamp(2.05rem,4.8vw,3.75rem)`.
Pay 9px'ten ~84px'e çıktı. Görsel fark ihmal edilebilir.

### 4. Ölü CSS kuralı (özgüllük)
`.hero h1.display{max-width:1500px}` (0,2,1), `.hero .wrap > :not(.band-card)`
(0,3,0) kuralına yeniliyordu — hiçbir zaman uygulanmamıştı. Başlık 820px'lik
kutusundan taşarak "kazara" çalışıyordu.

Yapılan: kural `.hero .wrap>h1.display{max-width:none}` olarak yeniden yazıldı.

---

## Yüksek öncelikli

| Konu | Sorun | Çözüm |
|---|---|---|
| i18n | İngilizce sürümde `%92`, `%57`, `%68` yazıyordu | `92%`, `57%`, `68%` — TR/EN ayrı `span` |
| iOS güvenli alan | `body{padding-bottom:76px}` sabitti; alt CTA çubuğu centikli iPhone'larda footer'ı örtüyordu | `calc(78px + env(safe-area-inset-bottom))` |
| Erişilebilirlik | `<main>` işareti yoktu, "İçeriğe atla" bağlantısı bir `<section>`'a gidiyordu | İçerik `<main id="main">` içine alındı; `body>main>section` yerleşim savunma seçicisine eklendi |
| Erişilebilirlik | Gizlilik modalı `aria-labelledby="pTitle"` ile hep Türkçe başlıkla etiketleniyordu | İki dilli `aria-label` |
| Erişilebilirlik | `list-style:none` olan listeler VoiceOver'da liste olarak duyurulmuyordu | `role="list"` eklendi (`.cmp-l`, `.mlist`, `.flow2 ol`, `.dev-steps`) |
| Kırık bağlantı | `href="#ornek"` — sayfada böyle bir öğe yok. JS kapalıysa ölü bağlantı | `mailto:` yedeği |
| Dokunma hedefleri | Dil düğmeleri 34×40, sosyal ikonlar 34×34, footer bağlantıları 31px, logo 37px | Mobilde 44px eşiğine çıkarıldı |
| Modal | Başlık dar ekranda kapatma (×) düğmesinin altına giriyordu | `.modal-t,.legal h3{padding-right:34px}` |
| Tablo | "Yana kaydırın" ipucu ilk tabloda vardı, merkez panel tablosunda yoktu | Eklendi; sarmalayıcı `.tw` sınıfına geçirildi |
| Hareket duyarlılığı | `prefers-reduced-motion` `scroll-behavior:smooth`'u kapatmıyordu | `html{scroll-behavior:auto}` eklendi |

---

## Orta öncelikli

- **Rozet stili:** Durum eki olmayan `.pill` öğeleri (İç ekip / Yüklenici ekip)
  hiçbir zemin almıyordu — rozet gibi değil düz metin gibi görünüyorlardı.
  Nötr bir zemin ve hairline eklendi.
- **CTA tutarsızlığı:** Kapanıştaki "Örnek kaydı e-posta ile alın" düğmesi
  `intro` modalını ("kısa bir tanıtım gönderelim") açıyordu. Artık hero ile
  aynı `ornek` akışını açıyor ve etiketi de onunla eşleşiyor.
- **Kapanış başlığı:** Tek bir cümle iki ayrı `<h2>`'ye bölünmüştü; başlık
  ağacında gürültü yaratıyordu. Tek `<h2>` + `<br>` yapıldı, görünüm aynı.
- **Analitik:** Sektör verileri ve "Teknik prosedürden doğrulanmış işe"
  bölümlerinde `data-sec` yoktu — görüntülenmeleri hiç ölçülmüyordu. Eklendi.
- **Form:** Telefon alanına `inputmode="tel"`, e-posta alanına
  `inputmode="email" autocapitalize="off" spellcheck="false"` eklendi.
- **İkonlar:** Yalnızca SVG data-URI favicon vardı; iOS ana ekran ve Android
  yükleme ikonu yoktu. Markadan `apple-touch-icon.png` (180), `icon-192.png`,
  `icon-512.png`, `favicon-32.png` ve `site.webmanifest` üretildi.
- **`_build_v2.py` silindi.** Repo genelinde tarandı: hiçbir build, deploy,
  package veya script süreci bu dosyaya referans vermiyor. Tek geçtiği yer
  `.gitignore` (yalnızca yoksayma kaydı, aktif süreç değil). Dosya kaldırıldı.

---

## Düzeltilmedi — karar sizin

1. **İçerik tekrarı.** 02 ve 05 bölümlerinin ikisi de problemi kuruyor.
   15 bölümlük bir sayfada ritmi yavaşlatıyor. (02'deki "Kayıt vardır —
   savunulabilir teknik kanıt yoktur" callout'u kaldırıldı; 09. bölümdeki
   "Kayıt var. Teknik kanıt yok." karşılaştırması aynen korundu.)
2. **TR/EN kullanım alanı kartlarının sırası farklı** (TR'de dağıtım önce, EN'de
   veri merkezleri önce). Kasıtlıysa sorun yok.
4. **Navigasyon yok.** 15 bölümlük sayfada başlıkta sadece dil ve CTA var.
   Tasarım tercihi olabilir ama uzun sayfada gezinmeyi zorlaştırıyor.
5. **Satır içi stiller.** Kritik olanları temizledim; merkez kontrol paneli
   tablosunda ve sapma yönetimi listesinde hâlâ çok sayıda satır içi stil var.
   İşlevsel bir sorun değil, bakım maliyeti.

---

## Dosyalar

```
index.html            düzeltilmiş sayfa
site.webmanifest      yeni
apple-touch-icon.png  yeni (180×180)
icon-192.png          yeni
icon-512.png          yeni
favicon-32.png        yeni
fonts/                değişmedi
og.png, mark.svg, wordmark.svg, robots.txt, sitemap.xml   değişmedi
_legacy/              karantinaya alınan eski build betiği
```

---

## Sonraki tur (talep üzerine)

1. `_legacy/_build_v2.py.bak` tamamen silindi — aktif referans yok.
2. Sektör verileri güncellendi: %92 → **%85** (Uptime Institute, Annual Outage
   Analysis 2025); %57 ve %68 metinleri yeniden yazıldı, kaynak künyeleri
   tam adlarıyla verildi. İngilizce karşılıkları da güncellendi.
3. 02. bölümdeki "Kayıt vardır — savunulabilir teknik kanıt yoktur." callout'u
   kaldırıldı. 08. bölümdeki "Sapma, iş kapanmadan görünür." callout'u ve
   09. bölümdeki CMMS/EAM karşılaştırması değiştirilmedi.

---

## İçerik yeniden yazımı

Sayfa yeni metne göre yeniden kuruldu. Tasarım dili, renkler, fontlar, token
sistemi ve bileşen kütüphanesi aynen korundu.

**Hero'daki ölçüm kartına (band-card) hiç dokunulmadı** — işaretleme, animasyon
ve erişilebilirlik etiketi bit düzeyinde aynı.

| | Önce | Sonra |
|---|---|---|
| Bölüm | 15 | 8 + program şeridi |
| HTML etiketi | 1236 | 525 |
| index.html | 124 KB | 81 KB |
| Sayfa boyu (1440px) | 12.681 px | 5.296 px |
| TR/EN düğüm | 255 / 255 | 82 / 82 |

Yeni akış: Hero → programlar → asıl risk (%85) → kriter dışı örneği → nasıl
çalışır (01–03) → kanıt zinciri + kanıt merdiveni → kullanım alanları →
entegrasyon → pilot (koyu kapanış) → footer.

Çıkarılan bölümler: ticari değer üçlüsü, "işçilik kanıtı sonradan üretilemez",
teknisyen/ToolA akış karşılaştırması, sahadan örnek kayıt tablosu, sapma yönetimi
adımları, CMMS/EAM karşılaştırması, merkez kontrol paneli, teknik hafıza,
pilot detay listeleri.

### Yeni yapı için eklenen CSS (yalnızca yerleşim)

- `.stats.stats-1` — tek istatistik üç sütunluk ızgarada sayfanın 2/3'ünü boş bırakıyordu
- `.ladder.ladder-min` — kanıt merdiveni yalnızca seviye adlarını gösteriyor
- `.tbl-sm table{min-width:0}` — global 660px alt sınır üç hücrelik tabloda gereksiz yatay kaydırma yaratıyordu
- `.close .num span` — `--ink-3` koyu zeminde okunmuyordu

### Diğer

- Footer sloganı `İşi doğru yap. Kanıtla. Öğren.` TR karşılığıyla iki dilli hale getirildi (önceden yalnızca İngilizceydi)
- Footer adresi `Teknopark İstanbul · QSTP Doha` olarak kısaltıldı, kategori satırı kaldırıldı
- `<title>`, `description`, `og:title`, `og:description` yeni konumlandırmaya göre iki dilde güncellendi

---

## Geri eklenen üç bölüm

Kısaltma sonrası eksik kalan üç parça, orijinal işaretlemesiyle geri alındı.
Yeni CSS yazılmadı; hepsi mevcut bileşenleri kullanıyor.

1. **Sapma yönetimi** (`03b`, `.devmgmt` + `.dev-steps` + `.card`)
   Kriter dışı örneğinin hemen ardında. 31 Nm sapmasının nasıl atandığı,
   düzeltildiği ve 40,2 Nm yeniden ölçümle kapandığı görünüyor.
   Bölüm başlığı olarak "Sapma yalnızca gösterilmez. Kapanana kadar yönetilir."
   kullanıldı; aynı cümle gövdeden çıkarıldı ki iki kez geçmesin.

2. **Kanıt merdiveni gerekçesi** — merdivenin üstüne iki cümle:
   kaydın yönteme göre etiketlendiği ve doğrulanabilirliğin yukarıdan aşağıya
   azaldığı. Dört seviyenin neden var olduğunu ve "Beyan"ın neden en altta
   durduğunu açıklıyor.

3. **CMMS / EAM karşılaştırması** (`06b`, `.cmp`)
   Kullanım alanları ile entegrasyon bölümü arasında. "Kim yaptı?" sorularının
   karşısına "Doğru işlem uygulandı mı?" sorularını koyuyor ve hemen ardından
   entegrasyon bölümü "yerlerine geçmez" diyerek soruyu kapatıyor.

Bölüm zeminleri açık/beyaz almaşık kalsın diye `Kanıt zinciri` ve
`Kullanım alanları` bölümlerinin `sec-a` durumu takas edildi.

| | Kısaltılmış | Şimdi |
|---|---|---|
| Bölüm | 8 | 10 |
| HTML etiketi | 525 | 622 |
| index.html | 81 KB | 86 KB |
| TR/EN düğüm | 82 / 82 | 106 / 106 |

---

## Akış revizyonu

Sayfa, dış değerlendirme sonrası yeniden sıralandı. Ölçüm kartı (`band-card`)
yine bit düzeyinde dokunulmadan taşındı — md5 orijinalle aynı.

**Yeni sıra**
Hero → Programlar → Problem → Nasıl çalışır → Sapma çıktıktan sonra →
Kaynak ve kanıt → Kullanım alanları → CMMS farkı + entegrasyon → Pilot → Footer

### Yapılanlar

1. **İkinci 31 Nm tablosu kaldırıldı.** Hero kartı zaten aynı üç değeri
   gösteriyordu; iki ekran sonra tablo olarak tekrar ediyordu.
   "Teknisyen kablo bağlantısını sıkar. Test geçer." cümlesi korunup problem
   bölümüne taşındı — tablo veriyi tekrar ediyordu, cümle etmiyor.

2. **Sapma yönetimi "Nasıl çalışır"ın ardına alındı.** Önceden mekanizma
   anlatılmadan lifecycle'a giriliyordu. Üçlü zaten "Sapmayı gösterir" ile
   bitiyor; bir sonraki bölüm "peki sonra?" diye devam ediyor.

3. **Ölçüm geçiş kartı** eklendi: İlk ölçüm 31 Nm · Sapma → Düzeltme sonrası
   40,2 Nm · Doğrulandı. Önceki düz metin kartın yerine geçti.
   31 Nm artık sayfada iki kez var: hero kartında *durum* olarak,
   geçiş kartında *değişim* olarak — farklı iş yapıyorlar.

4. **Kanıt merdiveni gerekçesi tek satıra indi.** "Doğrulanabilirlik yukarıdan
   aşağıya azalır" cümlesi çıkarıldı; bars görseli ve kırmızı alt satır
   bunu zaten anlatıyor.

5. **"Teknisyene 'doğru yaptın mı?' diye sormak kanıt değildir."** kaldırıldı.
   Aynı bölümdeki "Beyan, kanıt gibi gösterilmez." ile aynı şeyi söylüyordu ve
   merdivenin hemen altında ikincisi daha iyi oturuyor.

6. **CMMS karşılaştırması 11 maddeden 8'e indi** (4'e 4). İki sütunlu yapı
   korundu: okuyucunun kendi kendine cevap vermesi düzyazıdan daha ikna edici.
   Başlık "Kayıt var. Teknik kanıt yok." yerine
   **"İş emrini değil, teknik uygulamayı doğrular."** — farkı iddia etmek
   yerine tanımlıyor.

7. **Ayrı entegrasyon bölümü CMMS bölümüne katıldı.** Tek cümlelik bir bölüm
   olarak zayıf duruyordu; itirazın hemen ardında güçlü.

### Eklenen CSS

`.ev-row` / `.ev-l` / `.ev-v` — ölçüm geçiş kartı. Etiket sütunu 158px
(`DÜZELTME SONRASI` ve `FIRST MEASUREMENT` iki dilde de tek satır).
520px altında etiket üste çıkıyor. Kullanılmayan `.tbl-sm` kuralı kaldırıldı.

| | Önceki tur | Şimdi |
|---|---|---|
| Bölüm | 10 | 9 |
| HTML etiketi | 622 | 586 |
| index.html | 86 KB | 85 KB |
| Sayfa boyu (1440px) | 6.729 px | 5.961 px |
| CMMS maddesi | 11 | 8 |

---

## İki kolonlu yerleşim (3 bölüm)

Yalnızca yerleşim. Metin, kart, ikon, bölüm eklenmedi veya çıkarılmadı —
üç bölümün de düz metin içeriği karakterine kadar aynı doğrulandı.

**Değişen:** Problem · Sapma yönetimi · Kaynak ve kanıt
**Değişmeyen (bayt düzeyinde doğrulandı):** header, hero (ölçüm kartı dahil),
programlar, sektörler, CMMS karşılaştırması, pilot, footer, modallar ve JS

### Yapı

Her üç bölümde başlık tam genişlikte üstte kalıyor; altında `.two` ızgarası:

| Bölüm | Sol kolon | Sağ kolon |
|---|---|---|
| Problem | %85 istatistiği | problem açıklaması |
| Sapma yönetimi | 01–05 adımları | ölçüm kartı + sonuç cümlesi |
| Kaynak ve kanıt | zincir + açıklamalar | kanıt seviyesi kartı + "Beyan, kanıt gibi gösterilmez." |

Sonuç cümleleri kendi kolonlarında kaldı; iki kolonun altına tam genişlikte
düşmüyorlar.

### Eklenen CSS

```
.two{display:grid;grid-template-columns:minmax(0,1fr) minmax(0,1fr);
     gap:52px;align-items:start;margin-top:34px}
.two>div>:first-child{margin-top:0}
@media(max-width:900px){.two{grid-template-columns:1fr;gap:34px}}
```

Ölçü dili `.pil` ile aynı (1fr 1fr / 52px / align-items:start) ve aynı 900px
kırılımını kullanıyor. Mobilde tek kolona düşüyor, DOM sırası korunduğu için
içerik sırası bozulmuyor: önce sol kolon, sonra sağ kolon.

Kolona sığmayan iki sabit genişlik kaldırıldı: `.ladder-min` üzerindeki
520px ve blok içi `max-width` değerleri — artık genişlik kolondan geliyor.

### Yakalanan hata

901px'te (iki kolonun en dar hâli, kolon 393px) İngilizce `DEVIATION`
rozeti ölçüm kartının 8px dışına taşıyordu. `.ev-row` artık her genişlikte
sarabiliyor; dar kolonda rozet alt satıra iniyor, taşma yok.
Türkçede sorun görünmüyordu — çift dilli test olmasa gözden kaçardı.

| | Önce | Sonra |
|---|---|---|
| Sayfa boyu (1440px) | 5.961 px | 5.455 px |
| Problem bölümü | 761 px | 551 px |
| Sapma yönetimi | 700 px | 533 px |
| Kaynak ve kanıt | 947 px | 639 px |

---

## Problem bölümü hizalaması

Yalnızca bu bölümün masaüstü ızgarası. Diğer dokuz blok (header, hero, programlar,
sapma yönetimi, kaynak ve kanıt, sektörler, CMMS, pilot, footer, modallar + JS)
bayt düzeyinde aynı doğrulandı. Bölümün metni de karakterine kadar aynı;
markup farkı yalnızca kabına eklenen `two-rule` sınıfı.

**Sorun:** ayraç `.stats` kutusunun kendi `border-top`'uydu, dolayısıyla yalnızca
sol kolonu kapsıyordu. `.stats>div` üzerindeki 24px iç boşluk `%85`'i aşağı
itiyor, sağ kolon ise doğrudan ızgaranın tepesinden başlıyordu — iki kolon
24px kaymış görünüyordu.

**Çözüm:** ayraç `.stats`'tan alınıp ızgara kabına taşındı; iç boşluk her iki
kolona eşit olarak verildi.

```
@media(min-width:901px){
  .two-rule{grid-template-columns:minmax(0,1fr) minmax(0,1fr);gap:72px;
    border-top:1px solid var(--line);border-bottom:1px solid var(--line)}
  .two-rule>div{padding:24px 0 22px}
  .two-rule .stats{border-top:none;border-bottom:none}
  .two-rule .stats>div{padding:0}
  .two-rule .stats .body{max-width:none}
}
```

Kural `min-width:901px` ile sınırlı — mobil görünüm hiç etkilenmiyor,
hairline'lar orada eskisi gibi `.stats` kutusunun çevresinde kalıyor.

`.stats .body` üzerindeki 38ch sınırı bu bağlamda kaldırıldı; 500px'lik kolonda
metni gereksiz erken sarıyordu. Hairline rengi, kalınlığı ve iç boşluk değerleri
değişmedi — yalnızca hangi öğeye bağlandıkları değişti.

### Ölçüm

| | Önce | Sonra |
|---|---|---|
| Ayraç genişliği | 500 px (yalnız sol kolon) | **1072 px (iki kolon boyunca)** |
| `%85` ile sağ başlık üst farkı | 24 px | **0 px** |
| Kolon arası boşluk | 52 px | **72 px** |
| Bölüm yüksekliği (1440px) | 551 px | **528 px** |

---

## Metin revizyonu (3 cümle)

Üç cümle güncellendi. Her birinde amaç aynıydı: dile getirilen doğruluk /
akıcılık sorununu çözerken sayfanın başka bir yerinde zaten söylenmiş bir şeyi
tekrar etmemek.

**1 · Hero ölçüm kartı notu**
`Gerekçe kaydedilmeden…` → `Gerekçe ve düzeltme kaydedilmeden teknik doğrulama
tamamlanmaz.`
Gerekçenin tek başına yetmediği doğru. Ancak "Sapma kapanmadan teknik doğrulama
tamamlanmaz" 4. bölümün payoff cümlesiyle (`Sapma kapanmadan faaliyet teknik
olarak doğrulanmış sayılmaz.`) neredeyse birebir aynı olurdu; hero'da 15 saniye
önce söylenince orada patlamazdı. İki kelime eklemek doğruluk itirazını
karşılıyor, payoff'u yerinde bırakıyor.

Bu, ölçüm kartına (`band-card`) ilk kez yapılan içerik değişikliğidir; blok o
ana kadar orijinalden bayt düzeyinde aynıydı. Değişen tek satır `band-note`.

**2 · %85 açıklaması**
`…ilişkilendirilen oranı.` → `İnsan hatasının rol oynadığı büyük kesintiler,
prosedürlerin uygulanmaması veya süreç ve prosedür kusurlarıyla
ilişkilendiriliyor.`
Rapor dilinden çıktı. `%85` gövdede tekrarlanmadı; büyük rakamın işi zaten
"yüzde kaç" sorusunu cevaplamak. İngilizcede de `Share of…` kalıbı aynı nedenle
düz cümleye çevrildi.

**3 · Problem paragrafı**
`…Teknisyen kablo bağlantısını sıkar. Test geçer.` → `…Teknisyen bağlantıyı
sıkar, test geçer; ancak tork hâlâ kriterin altındadır.`
Kopuk iki cümle bağlandı. Somut teknisyen görüntüsü korundu (hero kartındaki
*36 kV XLPE kablo eki · bağlantı torku* ile eşleşiyor). Bitiş "kriter dışı"yı
tekrar etmiyor — hemen üstündeki kalın satır zaten öyle diyor — ama
31 Nm / 38–42 Nm'ye doğrudan bağlanıyor.

### Değişim kapsamı

Bayt düzeyinde aynı kaldığı doğrulananlar: header, programlar, Nasıl çalışır,
Sapma yönetimi, Kaynak ve kanıt, sektörler, CMMS, pilot, footer, modallar + JS
ve **CSS bloğunun tamamı**. Hero'da değişen satır sayısı: 2 (yalnızca
`band-note`).

---

## Metin dondurma (son 2 cümle)

**1 · Hero ölçüm kartı notu**
`Gerekçe ve düzeltme kaydedilmeden…` → **`Sapma kapatılmadan teknik doğrulama
tamamlanmaz.`**

Önceki hâli beş adımdan yalnızca ikisini sayıyordu. Kısmi sayım, o ikisinin
yeterli olduğunu ima ettiği için hiç saymamaktan daha yanıltıcıydı; yeni ölçüm
ve teknik onay dışarıda kalıyordu. Yeni cümle 4. bölümdeki beş adımlı süreçle
birebir tutarlı.

Sonuç: 4. bölümün kapanış cümlesi (`Sapma kapanmadan faaliyet teknik olarak
doğrulanmış sayılmaz.`) artık yeni bir sonuç değil, hero'da kurulan kuralın
ispat sonrası tekrarı — bilinçli bir bookend.

**2 · %85 açıklaması**
`…ilişkilendiriliyor.` → **`…ilişkilendirilen kısmı.`**
EN: `Major human-error outages are linked to…` → **`Share of major human-error
outages linked to…`**

Bu bir doğruluk düzeltmesi. Önceki turda rakamı gövdeden çıkarırken "oranı"
bağlayıcısı da düşmüştü; geriye kalan cümle kesintilerin **tamamının**
prosedürlerle ilişkili olduğunu söylüyordu. Kaynağın iddiası bu değil ve
%85 rakamıyla da çelişiyordu. Aynı hata İngilizce metinde de vardı, o da
düzeltildi.

### Değişim kapsamı

Bayt düzeyinde aynı kaldığı doğrulananlar: header, programlar, Nasıl çalışır,
Sapma yönetimi, Kaynak ve kanıt, sektörler, CMMS, pilot, footer, modallar + JS
ve CSS bloğunun tamamı. Hero'da 2 satır, Problem bölümünde 2 satır değişti.

**Copy bu noktada donduruldu.**

---

## Son düzenleme — CMMS listesi

`Bunu gösteren kanıt ne?` → **`Bunu gösteren kanıt var mı?`**
EN: `What evidence shows that?` → **`Is there evidence showing that?`**

Ürünün temel sorusu önce kanıtın varlığı, sonra kaynağı ve kalitesi.
Değişiklik ToolA sütununu da paralel hale getiriyor: dört madde de artık
varlık / uygunluk sorusu.

Tüm dosyada değişen satır: 2. Başka hiçbir yerde fark yok.

**Copy nihai.**
