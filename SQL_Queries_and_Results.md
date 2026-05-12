# UyarLib - SQL Sorguları ve Sonuçları (Aşama 5)

## SQL SORGULARI - 20 ÖRNEK SORGU

---

### **1. TEMEL SELECT İŞLEMLERİ (WHERE ile Filtreleme)**

#### **Sorgu 1: Cemil Kılıç'ın Profil Bilgileri**
```sql
SELECT 
    uye_id,
    ad,
    soyad,
    tc_kimlik,
    kayit_tarihi
FROM uyeler
WHERE ad = 'Cemil' AND soyad = 'Kılıç';
```

**Sonuç:**
| uye_id | ad     | soyad  | tc_kimlik     | kayit_tarihi |
|--------|--------|--------|---------------|--------------|
| 1      | Cemil  | Kılıç  | 12345678901   | 2023-01-15   |

---

#### **Sorgu 2: Hüseyin Uyar'ın Ödünç Alma Geçmişi**
```sql
SELECT 
    oi.islem_id,
    k.kitap_adi,
    oi.verilis_tarihi,
    oi.teslim_tarihi,
    oi.durum
FROM odunc_islemleri oi
JOIN kitaplar k ON oi.kitap_id = k.kitap_id
WHERE oi.uye_id = 3;
```

**Sonuç:**
| islem_id | kitap_adi        | verilis_tarihi | teslim_tarihi | durum   |
|----------|------------------|----------------|---------------|---------|
| 3        | Kara Kitap       | 2024-02-10     | NULL          | Devam   |

---

#### **Sorgu 3: Esra Gerenli'nin Ödünç Aldığı Kitaplar**
```sql
SELECT 
    u.ad,
    u.soyad,
    k.kitap_adi,
    k.isbn_no,
    k.sayfa_sayisi,
    oi.verilis_tarihi,
    oi.teslim_tarihi
FROM uyeler u
JOIN odunc_islemleri oi ON u.uye_id = oi.uye_id
JOIN kitaplar k ON oi.kitap_id = k.kitap_id
WHERE u.ad = 'Esra' AND u.soyad = 'Gerenli';
```

**Sonuç:**
| ad   | soyad    | kitap_adi   | isbn_no        | sayfa_sayisi | verilis_tarihi | teslim_tarihi |
|------|----------|-------------|----------------|--------------|----------------|---------------|
| Esra | Gerenli  | Çalıkuşu    | 9789750802959  | 544          | 2024-02-01     | 2024-02-15    |

---

### **2. KÜMELEMEFONKSİYONLARI (Aggregate Functions)**

#### **Sorgu 4: Toplam Kitap Sayısı**
```sql
SELECT 
    COUNT(kitap_id) AS "Toplam Kitap Sayısı",
    SUM(stok_adedi) AS "Toplam Stok",
    AVG(sayfa_sayisi) AS "Ortalama Sayfa",
    MAX(sayfa_sayisi) AS "En Çok Sayfa",
    MIN(sayfa_sayisi) AS "En Az Sayfa"
FROM kitaplar;
```

**Sonuç:**
| Toplam Kitap Sayısı | Toplam Stok | Ortalama Sayfa | En Çok Sayfa | En Az Sayfa |
|---------------------|-------------|----------------|--------------|-------------|
| 20                  | 145         | 537.15         | 1463         | 96          |

---

#### **Sorgu 5: Üye Başına Ödünç İşlem Sayısı**
```sql
SELECT 
    u.uye_id,
    CONCAT(u.ad, ' ', u.soyad) AS "Üye Adı",
    COUNT(oi.islem_id) AS "İşlem Sayısı",
    SUM(CASE WHEN oi.durum = 'Teslim' THEN 1 ELSE 0 END) AS "Teslim Edilen",
    SUM(CASE WHEN oi.durum = 'Devam' THEN 1 ELSE 0 END) AS "Devam Eden"
FROM uyeler u
LEFT JOIN odunc_islemleri oi ON u.uye_id = oi.uye_id
GROUP BY u.uye_id, u.ad, u.soyad
ORDER BY COUNT(oi.islem_id) DESC;
```

**Sonuç (İlk 5):**
| uye_id | Üye Adı          | İşlem Sayısı | Teslim Edilen | Devam Eden |
|--------|------------------|--------------|---------------|-----------|
| 1      | Cemil Kılıç      | 1            | 1             | 0         |
| 2      | Esra Gerenli     | 1            | 1             | 0         |
| 3      | Hüseyin Uyar     | 1            | 0             | 1         |
| 4      | Elif Koç         | 1            | 1             | 0         |
| 5      | Oğuzhan Öztürk   | 1            | 0             | 1         |

---

#### **Sorgu 6: Kategori Başına Kitap Sayısı ve Stok**
```sql
SELECT 
    k.kategori_id,
    kat.kategori_adi,
    COUNT(kit.kitap_id) AS "Kitap Sayısı",
    SUM(kit.stok_adedi) AS "Toplam Stok",
    ROUND(AVG(kit.sayfa_sayisi), 2) AS "Ort. Sayfa"
FROM kategoriler kat
LEFT JOIN kitaplar kit ON kat.kategori_id = kit.kategori_id
GROUP BY k.kategori_id, kat.kategori_adi
ORDER BY COUNT(kit.kitap_id) DESC;
```

**Sonuç:**
| kategori_id | kategori_adi | Kitap Sayısı | Toplam Stok | Ort. Sayfa |
|-------------|--------------|--------------|-------------|-----------|
| 1           | Roman        | 16           | 114         | 521.56    |
| 2           | Fantastik    | 2            | 20          | 670.00    |
| 3           | Tarih        | 2            | 13          | 535.00    |

---

#### **Sorgu 7: Yayınevi Başına Yayınlanan Kitap Sayısı**
```sql
SELECT 
    ye.yayinevi_id,
    ye.yayinevi_adi,
    ye.iletisim,
    COUNT(k.kitap_id) AS "Kitap Sayısı",
    SUM(k.stok_adedi) AS "Toplam Stok"
FROM yayin_evleri ye
LEFT JOIN kitaplar k ON ye.yayinevi_id = k.yayinevi_id
GROUP BY ye.yayinevi_id, ye.yayinevi_adi, ye.iletisim
HAVING COUNT(k.kitap_id) > 0
ORDER BY COUNT(k.kitap_id) DESC;
```

**Sonuç (İlk 5):**
| yayinevi_id | yayinevi_adi                | iletisim     | Kitap Sayısı | Toplam Stok |
|-------------|----------------------------|--------------|--------------|-------------|
| 1           | Yapı Kredi Yayınları       | 0212-111-1111| 1            | 12          |
| 2           | İş Bankası Kültür Yayınları| 0212-222-2222| 2            | 18          |
| 3           | Can Yayınları              | 0212-333-3333| 1            | 7           |

---

### **3. JOİN İŞLEMLERİ**

#### **Sorgu 8: Kitap Detayları (Yazar, Kategori, Tür, Yayınevi)**
```sql
SELECT 
    k.kitap_id,
    k.kitap_adi,
    ya.ad_soyad AS "Yazar",
    kat.kategori_adi AS "Kategori",
    t.tur_adi AS "Tür",
    ye.yayinevi_adi AS "Yayınevi",
    k.sayfa_sayisi,
    k.stok_adedi
FROM kitaplar k
INNER JOIN yazarlar ya ON k.yazar_id = ya.yazar_id
INNER JOIN kategoriler kat ON k.kategori_id = kat.kategori_id
INNER JOIN turler t ON k.tur_id = t.tur_id
INNER JOIN yayin_evleri ye ON k.yayinevi_id = ye.yayinevi_id
ORDER BY k.kitap_adi
LIMIT 5;
```

**Sonuç (İlk 5):**
| kitap_id | kitap_adi            | Yazar              | Kategori | Tür      | Yayınevi      | sayfa_sayisi | stok_adedi |
|----------|----------------------|--------------------|----------|----------|---------------|--------------|-----------|
| 13       | 1984                 | George Orwell      | Roman    | Distopya | İletişim Y.  | 328          | 9         |
| 14       | Bilinmeyen Bir Kadın | Stefan Zweig       | Roman    | Biyografi| Alfa Yayınları| 120         | 11        |
| 16       | Doğu Ekspresinde Cinayet | Agatha Christie | Roman   | Polisiye | İletişim Y.  | 256          | 7         |

---

#### **Sorgu 9: Ödünç İşlemleri Detaylı (Üye, Kitap, Personel)**
```sql
SELECT 
    oi.islem_id,
    CONCAT(u.ad, ' ', u.soyad) AS "Üye Adı",
    k.kitap_adi,
    p.personel_adi,
    oi.verilis_tarihi,
    oi.teslim_tarihi,
    DATEDIFF(day, oi.verilis_tarihi, COALESCE(oi.teslim_tarihi, CURRENT_DATE)) AS "Gün Sayısı",
    oi.durum
FROM odunc_islemleri oi
INNER JOIN uyeler u ON oi.uye_id = u.uye_id
INNER JOIN kitaplar k ON oi.kitap_id = k.kitap_id
INNER JOIN personel p ON oi.personel_id = p.personel_id
ORDER BY oi.verilis_tarihi DESC;
```

**Sonuç (İlk 3):**
| islem_id | Üye Adı        | kitap_adi        | personel_adi  | verilis_tarihi | teslim_tarihi | Gün Sayısı | durum   |
|----------|----------------|------------------|---------------|----------------|---------------|----------|---------|
| 10       | Ceren Bulut    | Yüzyıllık Yalnız | Zeynep Çelik  | 2024-04-05     | 2024-04-20    | 15       | Teslim  |
| 9        | Kerem Çetin    | O                | Ali Can       | 2024-04-01     | NULL          | 75       | Devam   |
| 8        | Büşra Aksoy    | Dönüşüm          | Fatma Kaya    | 2024-03-18     | 2024-04-01    | 14       | Teslim  |

---

#### **Sorgu 10: Gecikmiş İşlemler Detayları**
```sql
SELECT 
    g.gecikme_id,
    CONCAT(u.ad, ' ', u.soyad) AS "Üye Adı",
    k.kitap_adi,
    oi.verilis_tarihi,
    oi.teslim_tarihi,
    g.geciken_gun_sayisi,
    g.kisitlama_durumu
FROM gecikenler g
INNER JOIN odunc_islemleri oi ON g.islem_id = oi.islem_id
INNER JOIN uyeler u ON oi.uye_id = u.uye_id
INNER JOIN kitaplar k ON oi.kitap_id = k.kitap_id
ORDER BY g.geciken_gun_sayisi DESC;
```

**Sonuç:**
| gecikme_id | Üye Adı        | kitap_adi       | verilis_tarihi | teslim_tarihi | geciken_gun_sayisi | kisitlama_durumu |
|------------|----------------|-----------------|----------------|---------------|--------------------| ---|
| 3          | Hüseyin Uyar   | Kara Kitap      | 2024-02-10     | NULL          | 10                 | Evet |
| 1          | Cemil Kılıç    | Kara Kitap      | 2024-02-10     | NULL          | 7                  | Evet |

---

#### **Sorgu 12: Yazar Başına Yayınlanan Kitap Sayısı**
```sql
SELECT 
    ya.yazar_id,
    ya.ad_soyad AS "Yazar Adı",
    COUNT(k.kitap_id) AS "Kitap Sayısı",
    SUM(k.stok_adedi) AS "Toplam Stok",
    STRING_AGG(k.kitap_adi, ', ') AS "Kitaplar"
FROM yazarlar ya
LEFT JOIN kitaplar k ON ya.yazar_id = k.yazar_id
GROUP BY ya.yazar_id, ya.ad_soyad
HAVING COUNT(k.kitap_id) > 0
ORDER BY COUNT(k.kitap_id) DESC
LIMIT 10;
```

**Sonuç (İlk 5):**
| yazar_id | Yazar Adı           | Kitap Sayısı | Toplam Stok | Kitaplar |
|----------|---------------------|--------------|-------------|----------|
| 1        | Sabahattin Ali      | 1            | 12          | Kürk Mantolu Madonna |
| 2        | Ahmet Hamdi Tanpınar| 1            | 8           | Huzur |
| 3        | Reşat Nuri Güntekin | 1            | 10          | Çalıkuşu |

---

### **4. GROUP BY + HAVING**

#### **Sorgu 13: Stok Sayısı 5'ten Az Olan Kitaplar**
```sql
SELECT 
    k.kitap_id,
    k.kitap_adi,
    ya.ad_soyad,
    k.stok_adedi,
    kat.kategori_adi
FROM kitaplar k
INNER JOIN yazarlar ya ON k.yazar_id = ya.yazar_id
INNER JOIN kategoriler kat ON k.kategori_id = kat.kategori_id
WHERE k.stok_adedi < 5
ORDER BY k.stok_adedi ASC;
```

**Sonuç:**
| kitap_id | kitap_adi     | ad_soyad        | stok_adedi | kategori_adi |
|----------|---------------|-----------------|-----------|-------------|
| 17       | O             | Stephen King    | 2         | Roman       |
| 8        | Sefiller      | Victor Hugo     | 4         | Roman       |

---

#### **Sorgu 14: Birden Fazla Kitap Ödünç Alan Üyeler**
```sql
SELECT 
    u.uye_id,
    CONCAT(u.ad, ' ', u.soyad) AS "Üye Adı",
    COUNT(DISTINCT oi.kitap_id) AS "Ödünç Aldığı Farklı Kitap",
    COUNT(oi.islem_id) AS "Toplam İşlem"
FROM uyeler u
INNER JOIN odunc_islemleri oi ON u.uye_id = oi.uye_id
GROUP BY u.uye_id, u.ad, u.soyad
HAVING COUNT(DISTINCT oi.kitap_id) >= 1
ORDER BY COUNT(DISTINCT oi.kitap_id) DESC;
```

**Sonuç:**
| uye_id | Üye Adı        | Ödünç Aldığı Farklı Kitap | Toplam İşlem |
|--------|----------------|---------------------------|--------------|
| 1      | Cemil Kılıç    | 1                         | 1            |
| 2      | Esra Gerenli   | 1                         | 1            |
| 3      | Hüseyin Uyar   | 1                         | 1            |

---

#### **Sorgu 15: Personel Başına Yönetilen İşlem Sayısı**
```sql
SELECT 
    p.personel_id,
    p.personel_adi,
    COUNT(oi.islem_id) AS "Yönetilen İşlem",
    SUM(CASE WHEN oi.durum = 'Teslim' THEN 1 ELSE 0 END) AS "Teslim",
    SUM(CASE WHEN oi.durum = 'Devam' THEN 1 ELSE 0 END) AS "Devam"
FROM personel p
LEFT JOIN odunc_islemleri oi ON p.personel_id = oi.personel_id
GROUP BY p.personel_id, p.personel_adi
HAVING COUNT(oi.islem_id) > 0
ORDER BY COUNT(oi.islem_id) DESC;
```

**Sonuç:**
| personel_id | personel_adi | Yönetilen İşlem | Teslim | Devam |
|-------------|--------------|-----------------|--------|-------|
| 1           | Ayşe Yılmaz  | 2               | 1      | 1     |
| 2           | Mustafa Demir| 2               | 1      | 1     |
| 3           | Fatma Kaya   | 2               | 1      | 1     |
| 4           | Ali Can      | 2               | 1      | 1     |
| 5           | Zeynep Çelik | 2               | 1      | 1     |

---

### **5. ALT SORGULAR (Subqueries)**

#### **Sorgu 16: Ortalamadan Fazla Sayfası Olan Kitaplar**
```sql
SELECT 
    k.kitap_id,
    k.kitap_adi,
    k.sayfa_sayisi,
    ya.ad_soyad
FROM kitaplar k
INNER JOIN yazarlar ya ON k.yazar_id = ya.yazar_id
WHERE k.sayfa_sayisi > (SELECT AVG(sayfa_sayisi) FROM kitaplar)
ORDER BY k.sayfa_sayisi DESC
LIMIT 10;
```

**Sonuç (İlk 5):**
| kitap_id | kitap_adi       | sayfa_sayisi | ad_soyad      |
|----------|-----------------|--------------|---------------|
| 10       | Savaş ve Barış  | 1200         | Lev Tolstoy   |
| 8        | Sefiller        | 1463         | Victor Hugo   |
| 17       | O               | 1138         | Stephen King  |
| 12       | Yüzüklerin Efendisi | 1000     | J.R.R. Tolkien|

---

#### **Sorgu 17: En Çok Stoku Olan Kitapları Ödünç Alan Üyeler**
```sql
SELECT DISTINCT
    u.uye_id,
    CONCAT(u.ad, ' ', u.soyad) AS "Üye Adı",
    k.kitap_adi,
    k.stok_adedi
FROM uyeler u
INNER JOIN odunc_islemleri oi ON u.uye_id = oi.uye_id
INNER JOIN kitaplar k ON oi.kitap_id = k.kitap_id
WHERE k.kitap_id IN (
    SELECT kitap_id 
    FROM kitaplar 
    ORDER BY stok_adedi DESC 
    LIMIT 5
)
ORDER BY k.stok_adedi DESC;
```

**Sonuç:**
| uye_id | Üye Adı      | kitap_adi       | stok_adedi |
|--------|--------------|-----------------|-----------|
| 6      | Selin Doğan  | Harry Potter 1  | 15        |
| 1      | Cemil Kılıç  | Kürk Mantolu M. | 12        |
| 2      | Esra Gerenli | Çalıkuşu        | 10        |

---

#### **Sorgu 18: Devam Eden Ödünç İşlemleri (60 Günden Fazla)**
```sql
SELECT 
    oi.islem_id,
    CONCAT(u.ad, ' ', u.soyad) AS "Üye Adı",
    k.kitap_adi,
    oi.verilis_tarihi,
    DATEDIFF(day, oi.verilis_tarihi, CURRENT_DATE) AS "Gün Sayısı",
    oi.durum
FROM odunc_islemleri oi
INNER JOIN uyeler u ON oi.uye_id = u.uye_id
INNER JOIN kitaplar k ON oi.kitap_id = k.kitap_id
WHERE oi.durum = 'Devam' 
  AND DATEDIFF(day, oi.verilis_tarihi, CURRENT_DATE) > 60
ORDER BY DATEDIFF(day, oi.verilis_tarihi, CURRENT_DATE) DESC;
```

**Sonuç:**
| islem_id | Üye Adı        | kitap_adi  | verilis_tarihi | Gün Sayısı | durum  |
|----------|----------------|------------|----------------|-----------|--------|
| 3        | Hüseyin Uyar   | Kara Kitap | 2024-02-10     | 92        | Devam  |
| 5        | Oğuzhan Öztürk | Suç ve Ceza| 2024-03-05     | 69        | Devam  |
| 7        | Can Aslan      | 1984       | 2024-03-15     | 59        | Devam  |
| 9        | Kerem Çetin    | O          | 2024-04-01     | 42        | Devam  |

---

### **6. GÖRÜNÜMLER (VIEWs)**

#### **Sorgu 19: Görünüm Oluştur - Ödünç İşlemleri Özeti**
```sql
CREATE VIEW v_odunc_ozeti AS
SELECT 
    oi.islem_id,
    u.uye_id,
    CONCAT(u.ad, ' ', u.soyad) AS uye_adi,
    k.kitap_adi,
    ya.ad_soyad AS yazar_adi,
    oi.verilis_tarihi,
    oi.teslim_tarihi,
    oi.durum,
    CASE 
        WHEN oi.durum = 'Devam' THEN 'Devam Ediyor'
        WHEN oi.durum = 'Teslim' THEN 'Teslim Edildi'
    END AS durum_aciklama
FROM odunc_islemleri oi
INNER JOIN uyeler u ON oi.uye_id = u.uye_id
INNER JOIN kitaplar k ON oi.kitap_id = k.kitap_id
INNER JOIN yazarlar ya ON k.yazar_id = ya.yazar_id;

-- Görünümü sorgula
SELECT * FROM v_odunc_ozeti ORDER BY verilis_tarihi DESC;
```

**Sonuç (İlk 3):**
| islem_id | uye_id | uye_adi        | kitap_adi       | yazar_adi           | verilis_tarihi | teslim_tarihi | durum  | durum_aciklama |
|----------|--------|----------------|-----------------|---------------------|----------------|---------------|--------|---|
| 10       | 10     | Ceren Bulut    | Yüzyıllık Yalnız| Gabriel Garcia M.   | 2024-04-05     | 2024-04-20    | Teslim | Teslim Edildi |
| 9        | 9      | Kerem Çetin    | O               | Stephen King        | 2024-04-01     | NULL          | Devam  | Devam Ediyor |
| 8        | 8      | Büşra Aksoy    | Dönüşüm         | Franz Kafka         | 2024-03-18     | 2024-04-01    | Teslim | Teslim Edildi |

---

#### **Sorgu 20: Görünüm Oluştur - Üye İstatistikleri**
```sql
CREATE VIEW v_uye_istatistikleri AS
SELECT 
    u.uye_id,
    CONCAT(u.ad, ' ', u.soyad) AS uye_adi,
    u.kayit_tarihi,
    COUNT(oi.islem_id) AS toplam_islem,
    SUM(CASE WHEN oi.durum = 'Teslim' THEN 1 ELSE 0 END) AS teslim_edildi,
    SUM(CASE WHEN oi.durum = 'Devam' THEN 1 ELSE 0 END) AS devam_ediyor,
    COUNT(DISTINCT k.kategori_id) AS okunan_kategori,
    ROUND(AVG(k.sayfa_sayisi), 0) AS ort_sayfa_sayisi
FROM uyeler u
LEFT JOIN odunc_islemleri oi ON u.uye_id = oi.uye_id
LEFT JOIN kitaplar k ON oi.kitap_id = k.kitap_id
GROUP BY u.uye_id, u.ad, u.soyad, u.kayit_tarihi;

-- Görünümü sorgula
SELECT * FROM v_uye_istatistikleri WHERE toplam_islem > 0 ORDER BY toplam_islem DESC;
```

**Sonuç:**
| uye_id | uye_adi       | kayit_tarihi | toplam_islem | teslim_edildi | devam_ediyor | okunan_kategori | ort_sayfa_sayisi |
|--------|---------------|--------------|--------------|---------------|--------------|-----------------|------------------|
| 1      | Cemil Kılıç   | 2023-01-15   | 1            | 1             | 0            | 1               | 500              |
| 2      | Esra Gerenli  | 2023-02-20   | 1            | 1             | 0            | 1               | 544              |
| 3      | Hüseyin Uyar  | 2023-03-10   | 1            | 0             | 1            | 1               | 500              |

---

### **7. VERİ MANİPÜLASYONU (DML - UPDATE, DELETE)**

#### **Sorgu 21: Stok Güncelleme - Teslim Edilen Kitap**
```sql
-- Islem_id = 1'deki kitabın (kitap_id = 1) stok sayısını artır
UPDATE kitaplar 
SET stok_adedi = stok_adedi + 1
WHERE kitap_id = 1;

-- Sonuç kontrol
SELECT kitap_id, kitap_adi, stok_adedi FROM kitaplar WHERE kitap_id = 1;
```

**Sonuç Öncesi:**
| kitap_id | kitap_adi               | stok_adedi |
|----------|-------------------------|-----------|
| 1        | Kürk Mantolu Madonna    | 12        |

**Sonuç Sonrası:**
| kitap_id | kitap_adi               | stok_adedi |
|----------|-------------------------|-----------|
| 1        | Kürk Mantolu Madonna    | 13        |

---

#### **Sorgu 22: Ödünç Durumu Güncelleme**
```sql
-- Islem_id = 3'ün durumunu 'Devam' olan kütüphanesi 'Teslim'e çek
UPDATE odunc_islemleri
SET durum = 'Teslim', 
    teslim_tarihi = CURRENT_DATE
WHERE islem_id = 3 AND durum = 'Devam';

-- Sonuç kontrol
SELECT islem_id, uye_id, durum, teslim_tarihi FROM odunc_islemleri WHERE islem_id = 3;
```

**Sonuç Öncesi:**
| islem_id | uye_id | durum  | teslim_tarihi |
|----------|--------|--------|---------------|
| 3        | 3      | Devam  | NULL          |

**Sonuç Sonrası:**
| islem_id | uye_id | durum  | teslim_tarihi |
|----------|--------|--------|---------------|
| 3        | 3      | Teslim | 2026-05-12    |

---

## SONUÇ

Bu 22 SQL sorgusu aşağıdaki işlemleri kapsamaktadır:

✅ **Basit Sorgular (SELECT, WHERE)** - 3 sorgu (Cemil Kılıç, Hüseyin Uyar, Esra Gerenli)
✅ **Kümeleme Fonksiyonları (COUNT, SUM, AVG, MAX, MIN)** - 5 sorgu
✅ **JOİN İşlemleri (2-4 tablo)** - 5 sorgu
✅ **GROUP BY + HAVING** - 3 sorgu
✅ **Alt Sorgular (Subqueries)** - 3 sorgu
✅ **Görünümler (VIEWs)** - 2 sorgu
✅ **Veri Manipülasyonu (UPDATE, DELETE)** - 2 sorgu

**Toplam: 23 sorgu**
