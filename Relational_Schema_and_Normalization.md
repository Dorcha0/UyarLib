# UyarLib - İlişkisel Şema (Relational Schema)

## 6. İLİŞKİSEL ŞEMA VE NORMALİZASYON

### 6.1 İlişkisel Şema (Relation Schema)

Aşağıda UyarLib veritabanının tüm tabloları, öznitelikleri, birincil anahtarları (PK), yabancı anahtarları (FK) ve veri tipleri gösterilmiştir.

---

#### **1. yazarlar Tablosu**
```
yazarlar (
    yazar_id: INTEGER [PK],
    ad_soyad: VARCHAR(100) [NOT NULL]
)
```
- **Açıklama:** Kitapların yazarlarını depolayan tablo
- **Birincil Anahtar:** yazar_id
- **Kısıtlamalar:** ad_soyad boş olamaz

---

#### **2. kategoriler Tablosu**
```
kategoriler (
    kategori_id: INTEGER [PK],
    kategori_adi: VARCHAR(50) [NOT NULL, UNIQUE]
)
```
- **Açıklama:** Kitapların kategorilerini depolayan tablo
- **Birincil Anahtar:** kategori_id
- **Kısıtlamalar:** kategori_adi boş olamaz ve benzersiz olmalı

---

#### **3. turler Tablosu**
```
turler (
    tur_id: INTEGER [PK],
    tur_adi: CHAR(50) [NOT NULL, UNIQUE]
)
```
- **Açıklama:** Kitapların türlerini depolayan tablo
- **Birincil Anahtar:** tur_id
- **Kısıtlamalar:** tur_adi boş olamaz ve benzersiz olmalı

---

#### **4. yayin_evleri Tablosu**
```
yayin_evleri (
    yayinevi_id: INTEGER [PK],
    yayinevi_adi: VARCHAR(100) [NOT NULL, UNIQUE],
    iletisim: VARCHAR(20)
)
```
- **Açıklama:** Kitapları yayınlayan yayınevi bilgilerini depolayan tablo
- **Birincil Anahtar:** yayinevi_id
- **Kısıtlamalar:** yayinevi_adi boş olamaz ve benzersiz olmalı
- **İletişim:** Telefon numarası veya e-mail (isteğe bağlı)

---

#### **5. personel Tablosu**
```
personel (
    personel_id: INTEGER [PK],
    personel_adi: VARCHAR(100) [NOT NULL, UNIQUE]
)
```
- **Açıklama:** Kütüphane çalışanlarının bilgilerini depolayan tablo
- **Birincil Anahtar:** personel_id
- **Kısıtlamalar:** personel_adi boş olamaz ve benzersiz olmalı

---

#### **6. kitaplar Tablosu (Ana Tablo)**
```
kitaplar (
    kitap_id: INTEGER [PK],
    isbn_no: VARCHAR(13) [NOT NULL, UNIQUE],
    kitap_adi: VARCHAR(255) [NOT NULL],
    sayfa_sayisi: INTEGER [NOT NULL],
    stok_adedi: INTEGER [NOT NULL, DEFAULT=0],
    yazar_id: INTEGER [FK → yazarlar(yazar_id)],
    kategori_id: INTEGER [FK → kategoriler(kategori_id)],
    tur_id: INTEGER [FK → turler(tur_id)],
    yayinevi_id: INTEGER [FK → yayin_evleri(yayinevi_id)]
)
```
- **Açıklama:** Kütüphanedeki kitapların detaylı bilgilerini depolayan ana tablo
- **Birincil Anahtar:** kitap_id
- **Yabancı Anahtarlar:**
  - yazar_id → yazarlar tablosu
  - kategori_id → kategoriler tablosu
  - tur_id → turler tablosu
  - yayinevi_id → yayin_evleri tablosu
- **Kısıtlamalar:** 
  - isbn_no benzersiz ve boş olamaz
  - kitap_adi boş olamaz
  - sayfa_sayisi > 0
  - stok_adedi ≥ 0

---

#### **7. uyeler Tablosu**
```
uyeler (
    uye_id: INTEGER [PK],
    tc_kimlik: VARCHAR(11) [NOT NULL, UNIQUE],
    ad: VARCHAR(50) [NOT NULL],
    soyad: VARCHAR(50) [NOT NULL],
    kayit_tarihi: DATE [NOT NULL, DEFAULT=TODAY]
)
```
- **Açıklama:** Kütüphane üyelerinin kişisel bilgilerini depolayan tablo
- **Birincil Anahtar:** uye_id
- **Kısıtlamalar:**
  - tc_kimlik benzersiz, 11 karakter, boş olamaz
  - ad ve soyad boş olamaz
  - kayit_tarihi varsayılan olarak bugünün tarihi

---

#### **8. odunc_islemleri Tablosu (İşlem Tablosu)**
```
odunc_islemleri (
    islem_id: INTEGER [PK],
    uye_id: INTEGER [FK → uyeler(uye_id)] [NOT NULL],
    kitap_id: INTEGER [FK → kitaplar(kitap_id)] [NOT NULL],
    personel_id: INTEGER [FK → personel(personel_id)] [NOT NULL],
    verilis_tarihi: DATE [NOT NULL],
    teslim_tarihi: DATE [NULL],
    durum: VARCHAR(20) [NOT NULL, CHECK IN ('Devam', 'Teslim')]
)
```
- **Açıklama:** Üyelerin kitap ödünç alma ve iade işlemlerini depolayan tablo
- **Birincil Anahtar:** islem_id
- **Yabancı Anahtarlar:**
  - uye_id → uyeler tablosu
  - kitap_id → kitaplar tablosu
  - personel_id → personel tablosu
- **Kısıtlamalar:**
  - verilis_tarihi boş olamaz
  - teslim_tarihi NULL ise işlem devam ediyor, doluysa tamamlanmış
  - durum sadece 'Devam' veya 'Teslim' değerlerini içerebilir
  - verilis_tarihi ≤ teslim_tarihi (teslim_tarihi NULL değilse)

---

#### **9. gecikenler Tablosu (Yardımcı Tablo)**
```
gecikenler (
    gecikme_id: INTEGER [PK],
    islem_id: INTEGER [FK → odunc_islemleri(islem_id)] [NOT NULL, UNIQUE],
    geciken_gun_sayisi: INTEGER [NOT NULL, CHECK > 0],
    kisitlama_durumu: VARCHAR(50) [CHECK IN ('Evet', 'Hayır')]
)
```
- **Açıklama:** Teslim tarihi aşılmış ödünç işlemlerini ve gecikme detaylarını depolayan tablo
- **Birincil Anahtar:** gecikme_id
- **Yabancı Anahtar:** islem_id → odunc_islemleri tablosu (1:1 ilişki)
- **Kısıtlamalar:**
  - islem_id benzersiz (her ödünç işlemine en fazla 1 gecikme kaydı)
  - geciken_gun_sayisi > 0
  - kisitlama_durumu 'Evet' veya 'Hayır' olmalı

---

### 6.2 Normalizasyon Analizi

#### **1. Normal Form (1NF) - Atomik Değerler**

✅ **Uygunluk:** Tüm tablolar 1NF'e uygundur.

- Tüm öznitelikler **atomik (bölünmez)** değerlerdir
- Tekrar eden gruplar veya çok değerli alanlar yoktur
- Örnek: `ad` ve `soyad` ayrı sütunlarda tutulmuştur

---

#### **2. Normal Form (2NF) - Fonksiyonel Bağımlılık**

✅ **Uygunluk:** Tüm tablolar 2NF'e uygundur.

- Anahtar olmayan her öznitelik, **birincil anahtarın tamamına tam bağımlıdır**
- Birincil anahtarın bir parçasına bağımlı öznitelik yoktur

Örnek:
- **kitaplar** tablosunda `kitap_adi` tüm `kitap_id` bağımlıdır (parçalı bağımlılık yok)
- **uyeler** tablosunda `ad` ve `soyad`, `uye_id` tüm bağımlıdır

---

#### **3. Normal Form (3NF) - Geçişli Bağımlılık Yok**

✅ **Uygunluk:** Tüm tablolar 3NF'e uygundur.

- Anahtar olmayan öznitelikler arasında **geçişli bağımlılık yoktur**
- Veri tekrarı minimize edilmiştir

Örnek:
- ❌ **Hatalı Tasarım:**
```
kitaplar_yanlıs (
    kitap_id [PK],
    kitap_adi,
    yazar_id,
    yazar_adi,  ← geçişli bağımlılık (kitap_id → yazar_id → yazar_adi)
    kategori_id,
    kategori_adi ← geçişli bağımlılık (kitap_id → kategori_id → kategori_adi)
)
```

- ✅ **Doğru Tasarım:**
```
kitaplar (
    kitap_id [PK],
    kitap_adi,
    yazar_id [FK],
    kategori_id [FK]
)

yazarlar (
    yazar_id [PK],
    yazar_adi
)

kategoriler (
    kategori_id [PK],
    kategori_adi
)
```

---

#### **4. Boyce-Codd Normal Form (BCNF) - İlave Kontrol**

✅ **Uygunluk:** Tüm tablolar BCNF'e uygundur.

- Her **determinant** (belirleyici) bir **aday anahtar (candidate key)**'dir
- Tasarımda anomalileri engelleyen güçlü yapı vardır

---

### 6.3 Veri Bütünlüğü Kısıtlamaları

#### **Entity Bütünlüğü (Entity Integrity)**
- Her tabloda **benzersiz birincil anahtar** vardır
- PK değerleri **NULL olamaz**

#### **Referential Bütünlüğü (Referential Integrity)**
- FK değerleri, **referans verilen tablonun PK değerleriyle eşleşmelidir**
- FK değerleri NULL olabilir (opsiyonel ilişkiler için)

Örnek:
```
kitaplar.yazar_id → yazarlar.yazar_id (FK constraint)
odunc_islemleri.uye_id → uyeler.uye_id (FK constraint)
gecikenler.islem_id → odunc_islemleri.islem_id (FK constraint)
```

#### **Domain Bütünlüğü (Domain Integrity)**
- Veri tipleri uygun şekilde tanımlanmıştır
- CHECK kısıtlamaları uygulanmıştır:
  - `stok_adedi ≥ 0`
  - `geciken_gun_sayisi > 0`
  - `durum IN ('Devam', 'Teslim')`
  - `kisitlama_durumu IN ('Evet', 'Hayır')`

---

### 6.4 İlişkiler Özeti

| İlişki | Kardinalite | Türü | Açıklama |
|--------|-------------|------|----------|
| yazarlar → kitaplar | 1 : N | 1-to-Many | Bir yazar birden çok kitap yazabilir |
| kategoriler → kitaplar | 1 : N | 1-to-Many | Bir kategoriye birden çok kitap ait olabilir |
| turler → kitaplar | 1 : N | 1-to-Many | Bir türe birden çok kitap ait olabilir |
| yayin_evleri → kitaplar | 1 : N | 1-to-Many | Bir yayınevi birden çok kitap yayınlayabilir |
| uyeler → odunc_islemleri | 1 : N | 1-to-Many | Bir üye birden çok ödünç işlemi yapabilir |
| personel → odunc_islemleri | 1 : N | 1-to-Many | Bir personel birden çok ödünç işlemini yönetebilir |
| kitaplar → odunc_islemleri | 1 : N | 1-to-Many | Bir kitap zaman içinde birden çok kez ödünç verilebilir |
| odunc_islemleri → gecikenler | 1 : 1 | 1-to-1 | Bir ödünç işleminin en fazla 1 gecikme kaydı vardır |

---

### 6.5 Sonuç

UyarLib veritabanı tasarımı:

✅ **3. Normal Form (3NF) ve Boyce-Codd Normal Form (BCNF) standartlarına uygundur**

✅ **Veri tekrarı minimize edilmiştir**

✅ **Referential integrity kısıtlamaları uygulanmıştır**

✅ **Domain constraints tanımlanmıştır**

✅ **Ölçeklenebilir ve bakım yapılabilir bir yapı sunar**

Bu tasarım, kütüphane sisteminin güvenilir ve efisyen şekilde çalışmasını sağlar.
