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
