# 9/D Beden Eğitimi İstatistik Sitesi

## Proje Özeti
9-D sınıfı beden eğitimi derslerindeki futbol maçlarını, oyuncu istatistiklerini ve kadro dengeleme sistemini yöneten tek sayfalık (SPA) bir web uygulaması.

**Canlı Site:** https://mehmetcinart.github.io/9-D-mac_istatistikleri/9dbedenistatistikler.html
**GitHub:** https://github.com/mehmetcinart/9-D-mac_istatistikleri
**Renk Şeması:** Beyaz (#FFFFFF) + Mavi (#1E90FF)

---

## Dosya Yapısı

```
9-D Beden İstatistikler/
├── 9dbedenistatistikler.html    # Ana site dosyası (tek dosya: HTML + CSS + JS)
├── saha.png                      # Futbol sahası görseli (maç detay modalında kullanılır)
├── todo.md                       # Yapılacaklar listesi
├── CLAUDE.md                     # Bu dosya
├── Veriler/
│   ├── Maçlar ve Maç İstatikleri/
│   │   ├── Maç Tahtası.png       # Saha görseli (orijinal)
│   │   └── Maç Sonuçları.xlsx    # Excel şablonu
│   ├── Oyuncular ve İstatistikleri/
│   │   └── Oyuncular ve İstatistikleri.xlsx  # Oyuncu bilgileri
│   └── Gol ve Asist Krallığı Listesi/       # (boş dizin)
└── .claude/
    └── settings.json
```

---

## Teknolojiler

- **Frontend:** Vanilla HTML/CSS/JS (tek dosya, build sistemi yok)
- **Veritabanı:** Firebase Realtime Database (anlık senkronizasyon)
- **Hosting:** GitHub Pages (ücretsiz)
- **Firebase SDK:** CDN üzerinden yükleniyor (v10.12.0 compat)

---

## Firebase Yapılandırması

- **Proje:** `d-beden-istatistik`
- **Database URL:** `https://d-beden-istatistik-default-rtdb.europe-west1.firebasedatabase.app`
- **Veritabanı sahibi:** Bir arkadaşa ait; Firebase konsoluna erişim yok. Her işlem siteden, admin hesabıyla yapılır.

### Veri yapısı (sezon sistemi)

```
seasons_meta/
  current: "2026-27"                       ← en son sezon; maçlar buraya eklenir
  list/{sezonId}: { name, grade, label, order, startDate, createdAt }
                                           ← ör. "2025-26": { grade: 9, label: "9/D", order: 1 }
  migration: { done, at, by, counts }      ← ilk sezon ekleme (taşıma) bir kez çalıştı mı

seasons/{sezonId}/
  players         { [id]: {id, name, positions, primaryPos, foot} }   ← sezonun kadrosu
  player_edits    { [id]: {positions, primaryPos, foot} }             ← mevki/ayak düzenlemeleri
  pending_edits   { pushId: {...} }        ← kullanıcıların mevki değişikliği talepleri
  settings        { playerRatings: {id: puan}, fixedPositions: {id: "SNT"} }  ← gizli, kadro kurucu
  matches         [ ... ]                  ← maç dizisi
  notifications   { pushId: {...} }        ← duyurular
  notifs_cleared  zaman damgası
  announcements   { ... }                  ← eski veri, sitede gösterilmiyor
  polls           { ... }                  ← anketler

Sezondan bağımsız (ortak):
  9d_users, 9d_user_links, 9d_feedbacks, 9d_maintenance
```

- **Kullanıcı-oyuncu bağlantısı** (`9d_user_links/{uid}: oyuncuId`) sezondan bağımsızdır. **1 oyuncu = 1 hesap:** oyuncu yeni bir hesaba bağlanınca eski hesabın bağlantısı aynı `update()` ile silinir (`linkUser`); eski hesap bütün sezonlarda oyuncusuz kalır.
- **Oyuncu ID'leri bütün sezonlarda aynıdır** (kullanıcı bağlantıları ve kariyer toplamı buna dayanır). Yeni oyuncuya bütün sezonlarda kullanılmamış en küçük sonraki ID verilir. **200 ve üstü misafir oyunculara ayrılmıştır.**
- **Eski anahtarlar:** `9d_matches`, `9d_notifications`, `9d_announcements`, `9d_notifs_cleared`, `9d_pending_edits`, `9d_player_edits`, `9d_polls`, `9d_ratings`, `9d_match_ratings` taşıma sonrası **kullanılmıyor** ama silinmedi (birkaç hafta sorunsuz çalıştıktan sonra elle silinecek). `9d_ratings` ve `9d_match_ratings` yeni yapıya kopyalanmadı.
- **Geçiş modu:** `seasons_meta` yoksa (taşıma yapılmadıysa) site eski `9d_*` anahtarlarını tek sezon gibi okur ve yazar.
- **Dinleme:** `seasons_meta` ve `seasons` tek dinleyiciyle bellekte tutulur (bütün sezonlar). Geçiş modunda `9d_matches`, `9d_player_edits`, `9d_notifications`, `9d_pending_edits` dinlenir.
- **Gerekli kurallar:** `seasons` ve `seasons_meta` için `".read": "auth != null", ".write": "auth != null"` (normal kullanıcılar da oy, mevki talebi, duyuru yazıyor).

---

## Giriş Sistemi (İki Mod)

### Gözlemci Paneli
- Şifre gerektirmez
- Maçları, oyuncu istatistiklerini, krallık listesini görüntüleyebilir
- Maç detaylarına tıklayabilir, saha dizilişini görebilir
- **Yapamaz:** Maç ekleme/silme/sıralama, kadro kurucu, Excel'e aktarma

### Kontrol Paneli
- **Şifre:** `kontrolpaneligiris1970`
- Tüm özellikler açık
- CSS sınıfı: `body.control-mode` eklenince `.control-only` elemanlar görünür
- JS düzeyinde de koruma var: `isControlMode()` fonksiyonu kontrol eder

---

## Sezon Sistemi

- Her öğretim yılı bir sezon: **9. Sınıf (2025-26)**, **10. Sınıf (2026-27)**, ileride 11 ve 12.
- **Header'daki sezon seçici** (`#season-select`): site her açılışta en son sezonla başlar. Eski bir sezon seçilince bütün sekmeler (Ana Sayfa, Oyuncular, Maçlar, Krallık, Duyurular, Anketler, Performansım, profiller) o sezonun verisini gösterir. "9/D", "10/D" yazıları (`.class-label`) ve sayfa başlığı seçilen sezona göre değişir. Yükleme/giriş ekranında tarayıcıda kayıtlı son etiket (`localStorage 9d_class_label`) gösterilir.
- **Maç, duyuru ve anket ekleme her zaman güncel sezona yapılır.** Eski sezon görüntülenirken bu formlar gizlenir (`.season-current-only`) ve turuncu uyarı (`.season-past-banner`) çıkar.
- **Geçmiş sezon koruması** (`guardSeasonWrite()`): normal kullanıcılar eski sezonda hiçbir şeyi değiştiremez (anket oyu, mevki talebi dahil). Adminler maç düzenleme/silme, mevki düzenleme, oyuncu ekleme, duyuru/anket silme yapabilir; oturum başına sezon başına bir kez "Geçmiş sezonu düzenliyorsunuz" onayı çıkar.
- **Sezona özel:** maçlar, kadro, mevkiler, güç puanları, sabit mevkiler, oyuncu düzenlemeleri, duyurular, anketler ve bunlardan hesaplanan bütün istatistikler. Bir sezondaki mevki değişikliği veya yeni oyuncu diğer sezonları etkilemez.
- **Duyuru rozeti** (`#notif-badge`) sadece güncel sezonun duyurularını sayar.
- **Hafta numaraları** her sezonda "Hafta 1"den başlar.
- **Kadro Kurucu** her zaman güncel sezonun kadrosunu ve ayarlarını kullanır.
- **Oyuncu profili ve Performansım:** sezon butonları ("9. Sınıf", "10. Sınıf") + "Tüm Sezonlar" (kariyer toplamı; her sezon kendi kadrosuyla hesaplanıp toplanır).
- **Genel mantık (Maçkolik / Süper Lig sitesi gibi):** sezona göre değişen tek şey içerik (maçlar, istatistikler, krallık, duyurular, anketler). Hesaplar, giriş, profiller (isim, fotoğraf, tema), roller, kullanıcı-oyuncu eşleştirmeleri, geri bildirimler ve sitenin çalışma şekli bütün sezonlarda aynı. Tek istisna oyuncu listesi: yeni oyuncu veya mevki değişikliği sadece o sezonda geçerli; yeni sezon açılırken güncel kadro kopyalanır.
- **Sezon adları otomatik:** sınıftan üretilir ("9. Sınıf" + küçük harfle yıl). Yıl = 2025 + (sınıf − 9): 9 → 2025-26, 10 → 2026-27, 11 → 2027-28, 12 → 2028-29. Sezon ID'si yıldır (`seasonYearForGrade()`). Ad veya yıl elle girilmez.
- **Güncel sezon her zaman en son sezondur** (ekleyince yeni sezon, silince bir önceki sezon güncel olur).

### Admin menüsü (sağ üst kullanıcı menüsü, sadece `role === 'admin'`)
- **Veritabanı Yedeğini İndir** — bütün veriyi okur (önce kök, izin yoksa anahtar anahtar), `yedek-YYYY-MM-DD-SSDD.json` indirir, sezon bazında özet gösterir. Hiçbir şey yazmaz.
- **Yedekten Geri Yükle** — .json seçilir, anahtarlar onay kutularıyla listelenir (`10d_*` her zaman atlanır), `GERİ YÜKLE` yazılarak onaylanır. Önce mevcut verinin otomatik yedeği (`geri-yukleme-oncesi-yedek-...json`) iner. Seçili anahtarlar dosyadaki haliyle tamamen değiştirilir, diğerlerine dokunulmaz.
- **Sezonlar** — her zaman görünür (admin). Bütün sezon işleri buradan yapılır:
  - **Geçiş modunda** (`renderLegacySeasonsPanel`): mevcut veri tek sezon olarak "9. Sınıf (2025-26) · Güncel" diye listelenir. "+ 10. Sınıf Sezonunu Ekle" butonu tek seferlik taşımayı başlatır (aşağıda).
  - **Liste:** ad, yıl, maç/duyuru/anket/oyuncu sayısı, "Güncel" etiketi.
  - **Sezon Ekle** (`addNextSeason`): tek butonla bir sonraki sınıf; kadro (düzenlemeler uygulanmış haliyle), güç puanları ve sabit mevkiler en son sezondan kopyalanır, duyurular ve anketler boş başlar, yeni sezon güncel olur. 12. Sınıf'tan sonra eklenemez.
  - **Sil** (`deleteLastSeason`): sadece en son sezon; `seasons/{id}` (maçlar, duyurular, anketler, kadro, mevki kayıtları) ve `seasons_meta/list/{id}` tek `update()` ile silinir, bir önceki sezon güncel olur.
  - **Sıfırla** (`resetLastSeason`): sadece en son sezon; maçlar, duyurular (notifications, announcements, notifs_cleared) ve anketler silinir, sezon ve kadro kalır.
  - **Kurallar** (`canModifySeason`, arayüzde ve fonksiyon içinde): 9. Sınıf hiçbir zaman silinemez/sıfırlanamaz (buton görünmez), sadece en son sezon silinir/sıfırlanır. Önce otomatik yedek iner, sonra "10. Sınıf: 3 maç, 4 duyuru… silinecek" özetli onay çıkar; onaydan sonra kurallar tekrar kontrol edilir.
  - Sadece `seasons/{o sezon}` ve `seasons_meta` yazılır; kullanıcılar, profiller, geri bildirimler ve diğer sezonlar etkilenmez.
- **İlk sezon ekleme = sezon sistemine geçiş** (`startMigration` / `confirmMigration`, Sezonlar ekranında geçiş modunda "+ 10. Sınıf Sezonunu Ekle"), tek seferlik. Sırası: otomatik yedek → `seasons/_yazma_testi` ve `seasons_meta/_yazma_testi` ile yazma testi (izin yoksa durur ve kural satırlarını gösterir) → eski verileri okur → özet gösterir → onayla tek bir `update()` ile kopyalar. Ayırma: tarihi 2026-09-01'den önce olanlar 2025-26, sonrası 2026-27; `9d_player_edits`, kadro ve ayarlar iki sezona da. Hedefte aynı ID/anahtar varsa üzerine yazmaz; `seasons_meta/migration.done` varsa hiç çalışmaz.
- **Oyuncu Ekle** — Oyuncular sekmesinde buton (admin + sezon modu). Ad, mevkiler, asıl mevki, ayak, gizli güç puanı. Sadece görüntülenen sezona eklenir; kadroya kalıcı katılanlar içindir (tek maçlık misafir için maç formundaki "Misafir Ekle").

---

## Kaldırılan Seçenekler

- **Halısaha yeri:** maç ekleme ve düzenleme formundaki "Yer" seçimi kaldırıldı. Okul/Halısaha/Genel filtreleri (Oyuncular, Krallık, profil, Performansım), `hali*` alanları, "Okul"/"HS" rozetleri silindi. Tek istatistik görünümü var. Yeni maçlara `location` alanı yazılmıyor; eski maçlardaki `location: "Okul"` veride duruyor ama kullanılmıyor.
- **Lig maçı türü:** formlarda sadece "Dostluk Maçı" ve "Turnuva Maçı" var. Lig rozet rengi silindi. Turnuva mantığı (2. maç otomatik oluşturma vb.) değişmedi.

---

## Sekmeler ve Özellikler

### 1. Ana Sayfa (`tab-anasayfa`)
- Özet istatistik kartları: Toplam Oyuncu, Toplam Maç, Toplam Gol, Toplam Asist (görüntülenen sezon)
- Son 2 maç kartı
- Oyuncu kadro tablosu (tıklanabilir → oyuncu profili modalı)

### 2. Oyuncular (`tab-oyuncular`)
- Tüm oyuncular tablosu (G+A sıralamalı, görüntülenen sezon)
- "+ Oyuncu Ekle" butonu (admin, sezon modu)
- Her oyuncuya tıklayınca **profil modalı** açılır:
  - Mevkiler, kullanılan ayak (seçilen sezonun kaydı)
  - Sezon butonları + "Tüm Sezonlar" (kariyer)
  - Gol, asist, maç, G+A, galibiyet, beraberlik, mağlubiyet, kazanma oranı
  - Son 5 maç (G/B/M ikonu, skor bold, maç başı gol/asist)
- Excel'e Aktar butonu (görüntülenen sezon; dosya adında sezon yazar)

### 3. Maçlar (`tab-maclar`)
- **Maç ekleme formu** (kontrol paneli, sadece güncel sezon görüntülenirken):
  - Tür (Dostluk / Turnuva), tarih, skor girişi
  - Takım 1 oyuncuları seçildiğinde geri kalanlar otomatik Takım 2'ye atanır
  - **Misafir Oyuncu paneli her zaman açık:** toplam oyuncu sayısından bağımsız misafir eklenebilir (ad, mevki, takım). Misafir sadece o maçın kaydında (`extraPlayers`) tutulur, ID'leri 200'den başlar.
  - Gol/asist olayları: takım skoruna göre limit kontrolü (anlık engelleme)
  - Aynı oyuncunun birden fazla olay satırı birleştirilerek kaydedilir
- **Maç kartları** (hafta sırasıyla):
  - Skor, kazanan bilgisi, gol/asist etiketleri
  - Tıklanınca **maç detay modalı** açılır
  - Yukarı/aşağı sıralama butonları + sil butonu (kontrol paneli)
- **Maç detay modalı:**
  - Skor ve sonuç
  - Saha dizilişi (saha.png üzerinde oyuncular mevkilerine göre konumlandırılır)
  - Takım 1: sol yarı saha, Takım 2: sağ yarı saha
  - Oyuncu isimleri sadece ilk isim gösterilir
  - Kadro listesi ve maç olayları (gol+asist yan yana birleştirilmiş)
- Excel'e Aktar butonu (kontrol paneli)

### 4. Gol & Asist Krallığı (`tab-krallik`)
- Gol krallığı sıralaması (altın/gümüş/bronz renkleri)
- Asist krallığı sıralaması

### 5. Kadro Kurucu (`tab-kadro`, sadece kontrol paneli)
- Takım 1 ve Takım 2 kişi sayısı girişi (min 4, varsayılan 6)
- Gelmeyen oyuncuları seçme sistemi (kadro gerekli sayıdan fazlaysa)
- Dengeli kadro oluşturma algoritması (her zaman güncel sezonun kadrosu)

### 6. Duyurular, Anketler, Performansım
- Görüntülenen sezonun duyuruları/anketleri listelenir. Geçmiş sezon anketleri salt okunur (sonuçlar gösterilir, oy verilemez).
- MVP anketi her zaman güncel sezonun maçları için açılır.
- Performansım: bağlı oyuncunun sezon/kariyer istatistikleri; "Bilgilerimi Düzenle" sadece güncel sezonda görünür (talepler güncel sezona yazılır).

---

## Kadro Kurucu Algoritması

### Oyuncu Güç Puanları (gizli, sitede gösterilmez)
- Koddaki `playerRatings` objesi ve `getPlayerRating()` başlangıç değerleridir (değiştirilmedi).
- Sezona özel değerler `seasons/{id}/settings/playerRatings` içinde; `getSeasonRating()` önce bunu, yoksa `getPlayerRating()`'i kullanır. "Oyuncu Ekle" ile girilen puan buraya yazılır. Ayar ekranı yok.

### Sabit Mevki Atamaları (`fixedPositions`, sezonda `settings/fixedPositions`)
- Mehmet Çınar Turşak (id:1) → her zaman **SNT**
- Fatih Doğanay (id:2) → her zaman **SNT**
- Kağan Durak (id:3) → normalde **MOO**, ama Fatih ve Mehmet Çınar aynı takımdaysa **SNT**
- Egemen Işık (id:6) → her zaman **MOO**

### Dağılım Kuralları
Kurallar oyuncu ID'lerine bağlı ve kodda kalır; ilgili oyuncu o sezonun kadrosunda yoksa kural sessizce atlanır.
1. **Yusuf (id:7) ve Ömer (id:8):** kodda Ömer %50 ihtimalle Yusuf'la aynı takıma, %50 karşı takıma konuyor (eski not "her zaman farklı" diyordu; kod böyle çalışıyor)
2. **Mehmet Çınar (id:1)** %50 ihtimalle Yusuf'la aynı, %50 karşı takımda
3. **Stoperler 3-2 bölüşülür:** Yusuf'un takımında 3 STP (dahil), Ömer'in takımında 2 STP (dahil)
4. **Kaleciler ve forvetler** rastgele dağılır (bazen aynı takımda olabilir)
5. **Her takımda her mevki kategorisinden** (KL, STP, MO, SNT) en az 1 oyuncu
6. Kalan oyuncular kişi başı ortalama güce göre dengeli dağıtılır

### Mevki Kategorileri
- `KL` → Kaleci
- `STP` → Savunma (STP, MDO)
- `MO` → Orta Saha (MO, MOO)
- `SNT` → Forvet (SNT, SLK, SĞK)

### Kadro Listesi Sıralaması
Kaleci → Defans → Orta Saha → Forvet (hat başlıkları ile ayrılmış)

---

## Başlangıç Kadrosu (`defaultPlayers`, 12 kişi)

Bu dizi artık sadece taşıma ve geçiş modu için başlangıç verisidir. Güncel kadro her sezonun `players` + `player_edits` kaydından gelir; yeni oyuncu siteden "Oyuncu Ekle" ile eklenir (kod değişmez).

| # | Ad Soyad | ID | Mevkiler | Ayak |
|---|----------|-----|----------|------|
| 1 | Mehmet Çınar Turşak | 1 | SNT, SĞK, SLK, MOO | Sağ |
| 2 | Fatih Doğanay | 2 | SNT, SĞK, SLK, MOO | Sağ |
| 3 | Kağan Durak | 3 | SNT, SĞK, SLK, MOO | Sağ |
| 4 | Mehmet Ali Çınar | 4 | MOO, MO | Sağ |
| 5 | Mehmet Efe Güvenmez | 5 | MOO, MO | Sağ |
| 6 | Egemen Işık | 6 | SNT, MO | Sağ |
| 7 | Yusuf Karadağ | 7 | STP, MO, MDO | Sağ |
| 8 | Ömer Ege Durmuşoğlu | 8 | STP, MO, MDO | Sağ |
| 9 | Ertuğrul Gazi Kuzucu | 9 | STP | Sağ |
| 10 | Eren Kaya | 10 | STP | Sağ |
| 11 | Boran Mir Şenoğlu | 11 | KL, STP | Sağ |
| 12 | Ege Demir Şen | 12 | KL | Sağ |

---

## Mevki Kısaltmaları

| Kısaltma | Mevki |
|----------|-------|
| KL | Kaleci |
| STP | Stoper |
| MDO | Merkez Defansif Orta Saha |
| MO | Merkez Orta Saha |
| MOO | Merkez Ofansif Orta Saha |
| SLK | Sol Kanat |
| SĞK | Sağ Kanat |
| SNT | Santrfor |

---

## Saha Dizilişi Sistemi

Maç detay modalında `saha.png` üzerinde oyuncular konumlandırılır:
- **Takım 1:** Sol yarı saha (%2-%48 arası left değerleri)
- **Takım 2:** Sağ yarı saha (%52-%98 arası left değerleri)
- Her hat (KL, STP+MDO, MO+MOO, SNT+SLK+SĞK) aynı yatay çizgide
- Aynı hattaki oyuncular dikeyde (%15-%85 arası) eşit aralıklarla dağıtılır
- Oyuncu isimleri sadece **ilk isim** olarak gösterilir
- Takım 1: mavi (#1E90FF), Takım 2: kırmızı (#e74c3c) noktalar

---

## Veri Akışı

1. **Yollar:** `sPath(key)` görüntülenen sezonun yolu, `cPath(key)` güncel sezonun yolu (`seasons/{id}/{key}`); geçiş modunda ikisi de `9d_{key}` döner.
2. **Maç kaydedildiğinde:** `saveData(sPath('matches'), matches)` → Firebase'e yazılır
3. **Firebase listener:** `seasons_meta` → `enterSeasonMode()` / `enterLegacyMode()`; `seasons` → `allSeasons` → `onSeasonDataChanged()` → (görüntülenen sezonun maç/kadro verisi değiştiyse) `applyViewSeason()` + `renderAll()`. Sadece duyuru/anket değişikliği maç formunu sıfırlamaz.
4. **Global değişkenler:** `players` ve `matches` her zaman görüntülenen sezonun verisidir. Başka sezon için `seasonRoster(id)` / `seasonMatches(id)` kullanılır.
5. **İstatistik hesaplama:** `recalcPlayerStats()` görüntülenen sezonda; `calcStats(maçlar, id, kadro)` sezon/kariyer için.
6. **Anlık senkronizasyon:** Bir cihazda yapılan değişiklik tüm cihazlarda anında görünür

---

## CSS Yapısı

Tüm stiller `<style>` etiketi içinde, harici dosya yok. Ana bölümler:
- Header (sticky, gradient mavi)
- Nav tabs
- Stat cards (grid)
- Table wrapper
- Modal overlay + modal
- Match cards (border-left mavi)
- Pitch container (absolute positioning)
- Team builder
- King cards (gol: yeşil gradient, asist: turuncu gradient)
- Login screen (overlay)
- Responsive breakpoint: 768px

---

## Geliştirme Notları

- Tüm proje tek HTML dosyasında (inline CSS + JS)
- Build sistemi, framework veya paket yöneticisi yok
- Oyuncu eklemek için kod değiştirilmez: siteden "Oyuncu Ekle" kullanılır (`defaultPlayers` sadece başlangıç verisi)
- Yeni sezon için kod değiştirilmez: admin menüsü → "Sezonlar" → "Sezon Ekle"
- Excel dosyalarını okumak için `openpyxl` Python paketi kullanılabilir
- GitHub push için `git push origin main` yeterli
- Firebase API key'i client-side'da açık, bu normal (güvenlik Rules ile sağlanır)

---

## Otomatik Commit & Push Kuralı

**Her değişiklikten sonra otomatik olarak commit atılıp push edilmeli.**

Değişiklik yapıldığında şu adımlar izlenir:
1. `git add` ile değişen dosyalar eklenir
2. Türkçe ve açıklayıcı bir commit mesajı yazılır
3. `git push origin main` ile GitHub'a gönderilir

Remote URL: `https://github.com/mehmetcinart/9-D-mac_istatistikleri.git`
Branch: `main`
