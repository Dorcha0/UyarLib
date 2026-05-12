# UyarLib - İlişkisel Şema Görseli (PNG Format)

## Şema Açıklaması

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         UyarLib - İlişkisel Şema                                │
└─────────────────────────────────────────────────────────────────────────────────┘

REFERANS TABLOLARI (Mavi):
╔════════════════════╗  ╔════════════════════╗  ╔════════════════════╗
║    yazarlar        ║  ║  kategoriler       ║  ║     turler         ║
╠════════════════════╣  ╠════════════════════╣  ╠════════════════════╣
║ PK: yazar_id (INT) ║  ║ PK: kategori_id    ║  ║ PK: tur_id (INT)   ║
║    ad_soyad (VARCHAR)║  ║    kategori_adi    ║  ║    tur_adi (CHAR)  ║
╚════════════════════╝  ╚════════════════════╝  ╚════════════════════╝
        ▲                       ▲                       ▲
        │ 1:N                   │ 1:N                   │ 1:N
        └───────────┬───────────┴───────────┬───────────┘
                    │                       │
        ╔═══════════════════════════════════════════════════╗
        ║          kitaplar (ANA TABLO)                    ║
        ╠═══════════════════════════════════════════════════╣
        ║ PK: kitap_id (INT)                              ║
        ║     isbn_no (VARCHAR) [UNIQUE]                  ║
        ║     kitap_adi (VARCHAR)                         ║
        ║     sayfa_sayisi (INT)                          ║
        ║     stok_adedi (INT) [DEFAULT=0]                ║
        ║ FK: yazar_id → yazarlar                         ║
        ║     kategori_id → kategoriler                   ║
        ║     tur_id → turler                             ║
        ║     yayinevi_id → yayin_evleri                  ║
        ╚═══════════════════════════════════════════════════╝
                    │
                    │ 1:N
                    ▼
        ╔═══════════════════════════════════════════════════╗
        ║      odunc_islemleri (İŞLEM TABLOSU)            ║
        ╠═══════════════════════════════════════════════════╣
        ║ PK: islem_id (INT)                              ║
        ║ FK: uye_id → uyeler                             ║
        ║     kitap_id → kitaplar                         ║
        ║     personel_id → personel                      ║
        ║     verilis_tarihi (DATE)                       ║
        ║     teslim_tarihi (DATE) [NULL]                 ║
        ║     durum (VARCHAR) [CHECK]                     ║
        ╚═══════════════════════════════════════════════════╝
             │              │              │
             │ 1:N          │ 1:N          │ 1:N
             ▼              ▼              ▼
        ╔══════════╗   ╔══════════════╗  ╔════════════╗
        ║ uyeler   ║   ║ personel     ║  ║ yayin_evl. ║
        ╠══════════╣   ╠══════════════╣  ╠════════════╣
        ║ PK: uye_ ║   ║ PK: personel ║  ║ PK: yayin- ║
        ║     id   ║   ║     _id      ║  ║     evi_id ║
        ║    tc_   ║   ║     personel ║  ║ yayinevi_a ║
        ║ kimlik   ║   ║     _adi     ║  ║    di      ║
        ║    ad    ║   ╚══════════════╝  ║ iletisim   ║
        ║  soyad   ║                      ╚════════════╝
        ║ kayit_   ║
        ║  tarihi  ║
        ╚══════════╝
             │
             │ 1:N
             ▼
        ╔═══════════════════════════════════════════════════╗
        ║         gecikenler (YARDIMCI TABLO)              ║
        ╠═══════════════════════════════════════════════════╣
        ║ PK: gecikme_id (INT)                            ║
        ║ FK: islem_id → odunc_islemleri [UNIQUE]         ║
        ║     geciken_gun_sayisi (INT) [CHECK > 0]        ║
        ║     kisitlama_durumu (VARCHAR)                  ║
        ╚═══════════════════════════════════════════════════╝
```

## Tablo Açıklamaları

### 1. yazarlar
- **Amaç:** Kitap yazarlarını depolamak
- **Ana Anahtar:** yazar_id
- **İlişkiler:** 1:N → kitaplar

### 2. kategoriler
- **Amaç:** Kitap kategorilerini depolamak
- **Ana Anahtar:** kategori_id
- **İlişkiler:** 1:N → kitaplar

### 3. turler
- **Amaç:** Kitap türlerini depolamak
- **Ana Anahtar:** tur_id
- **İlişkiler:** 1:N → kitaplar

### 4. yayin_evleri
- **Amaç:** Yayınevi bilgilerini depolamak
- **Ana Anahtar:** yayinevi_id
- **İlişkiler:** 1:N → kitaplar

### 5. personel
- **Amaç:** Kütüphane çalışanlarını depolamak
- **Ana Anahtar:** personel_id
- **İlişkiler:** 1:N → odunc_islemleri

### 6. uyeler
- **Amaç:** Kütüphane üyelerini depolamak
- **Ana Anahtar:** uye_id
- **İlişkiler:** 1:N → odunc_islemleri

### 7. kitaplar (ANA TABLO)
- **Amaç:** Kitapları ve detaylarını depolamak
- **Ana Anahtar:** kitap_id
- **Yabancı Anahtarlar:** yazar_id, kategori_id, tur_id, yayinevi_id
- **İlişkiler:** 1:N → odunc_islemleri

### 8. odunc_islemleri (İŞLEM TABLOSU)
- **Amaç:** Ödünç alma/iade işlemlerini depolamak
- **Ana Anahtar:** islem_id
- **Yabancı Anahtarlar:** uye_id, kitap_id, personel_id
- **İlişkiler:** 1:N → gecikenler

### 9. gecikenler (YARDIMCI TABLO)
- **Amaç:** Gecikmiş işlemleri depolamak
- **Ana Anahtar:** gecikme_id
- **Yabancı Anahtar:** islem_id (1:1)
- **İlişkiler:** 1:1 ← odunc_islemleri

## İlişkiler Özeti

| Kaynak | Hedef | Kardinalite | Açıklama |
|--------|-------|-------------|----------|
| yazarlar | kitaplar | 1:N | Bir yazar birden çok kitap yazabilir |
| kategoriler | kitaplar | 1:N | Bir kategoriye birden çok kitap ait olabilir |
| turler | kitaplar | 1:N | Bir türe birden çok kitap ait olabilir |
| yayin_evleri | kitaplar | 1:N | Bir yayınevi birden çok kitap yayınlayabilir |
| uyeler | odunc_islemleri | 1:N | Bir üye birden çok ödünç işlemi yapabilir |
| personel | odunc_islemleri | 1:N | Bir personel birden çok işlemi yönetebilir |
| kitaplar | odunc_islemleri | 1:N | Bir kitap birden çok kez ödünç verilebilir |
| odunc_islemleri | gecikenler | 1:1 | Bir işlemin max 1 gecikme kaydı vardır |

## Normalizasyon Durumu

✅ **1NF (Birinci Normal Form)**
- Tüm değerler atomiktir (bölünmez)
- Tekrar eden gruplar yoktur

✅ **2NF (İkinci Normal Form)**
- Anahtar olmayan öznitelikler anahtarın tamamına bağımlıdır
- Kısmi bağımlılık yoktur

✅ **3NF (Üçüncü Normal Form)**
- Anahtar olmayan öznitelikler arasında geçişli bağımlılık yoktur
- Veri tekrarı minimize edilmiştir

✅ **BCNF (Boyce-Codd Normal Form)**
- Her determinant bir aday anahtardır
- Tasarım anomalilerden arındırılmıştır
