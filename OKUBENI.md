# ToolA landing — yeniden tasarım (Ekim 2026)

Dağıtım: klasörün tamamını olduğu gibi yükleyin (Cloudflare Pages / Netlify). `_headers`, `_redirects`, `robots.txt`, `sitemap.xml`, OG görselleri ve video dosyaları değişmedi.

Değişenler
- `index.html` — sıfırdan yazıldı (yapı, tipografi, animasyonlar). Metinler eski siteyle birebir aynı; sadece bölüm sırası ve düzeni değişti.
- `legal.js` — eski sitedeki 5 yasal metin (Gizlilik, KVKK, Pilot Aydınlatma, Çerezler, Kullanım Koşulları) birebir korunarak ayrı dosyaya alındı.
- `fonts/` — Archivo + IBM Plex Mono yerine Geist + Geist Mono (SIL OFL 1.1, lisans dosyası klasörde). Variable font olduğu için 4 dosya yeter.

Korunanlar
- Formspree formu (`https://formspree.io/f/mbgrpzba`), alan adları ve gizli alanlar.
- TR/EN dil değiştirme (`?lang=en`, `/en` yönlendirmesi, localStorage tercihi).
- Hero videosu, poster, schema.org verisi, meta/OG etiketleri.

Yeni metin veya çeviri eklerken: Türkçe metni HTML'e yazın, İngilizcesini `index.html` sonundaki `translations` sözlüğüne aynı anahtarla ekleyin. Çevrilmemesi gereken öğelere `data-no-i18n` verin.
