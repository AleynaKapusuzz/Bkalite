# B Kalite Kontrol Sistemi

Otomatik öğrenen YOLO v8 tabanlı görüntü kontrol sistemi. Kullanıcı onaylarıyla model kendini geliştiriyor.

## 🎯 Ne Yapıyor?

```
Yeni Resim → YOLO Tahmin → Web UI'de Göster → Kullanıcı Onayla/Reddet
                                                        ↓
                                          save1/ (eğitim)  save2/ (arşiv)
                                                        ↓
                                            500+ veri olunca Model Eğit
                                                        ↓
                                              best.pt Güncelle
```

---

## 📁 Dosya & Klasör Yapısı

### **Python Dosyaları**

| Dosya | Görev |
|-------|-------|
| **app.py** | Flask sunucusu. ham/ izler → YOLO çalıştırır → API sağlar |
| **trainer.py** | 30 saniye aralığında save1/ veri sayısını kontrol eder. 500+ olunca YOLO eğitim başlatır |
| **bkalite_config.py** | **Merkezi ayarlar** - sınıf adları, threshold değerleri, eğitim parametreleri |
| **annotation_debug.py** | Hata ayıklama - YOLO kutularını resme çizer, sorunlu açıklamaları raporlar |
| **log_blood.py** | Loglama - debug mesajları dosyaya yaz |
| **service_blood.py** | Windows NSSM servisi başlatıcı (opsiyonel) |

### **Frontend Dosyaları**

| Dosya | Görev |
|-------|-------|
| **index.html** | Ana arayüz - resimleri ve YOLO kutularını göster |
| **app.js** | index.html'in JavaScript - kutulara drag/drop, onay/reddet |
| **review.html** | Thumbnail önizleme sayfası (save1 ve save2 resimleri) |
| **review.js** | review.html'in JavaScript |

### **Çalışma Klasörleri**

```
proje/
├── ham/                    # 📥 INPUT - Yeni resimler buraya konur
│
├── save1/                  # ✅ Onaylanan (EĞİTİME GİRER)
│   ├── resimler/           #    PNG/JPG dosyaları
│   └── txt/                #    YOLO format açıklamalar (0 0.45 0.52 0.15 0.20)
│
├── save2/                  # ❌ Reddedilen (sadece arşiv, eğitime girmez)
│   └── resimler/
│
├── best/                   # 🤖 Model dosyaları
│   └── best.pt             #    Aktif YOLO modeli
│
├── trained/                # 📊 Eğitim geçmişi
│   ├── resimler/           #    Eğitimde kullanılan veriler
│   └── txt/                #    YOLO açıklamaları
│
├── bkalite_config.py       # ⚙️ Ayarlar dosyası
│   # CLASS_NAMES = ["kan"]                    # Sınıf adları
│   # TRAIN_TRIGGER_SAVE1 = 500               # Eğitim tetik
│   # INFER_CONF = 0.60                       # Tahmin kesinlik sınırı
│   # (Yeni sınıf eklerken SADECE burası değiştirilir)
│
├── app.py                  # 🌐 Flask backend
│   # - ham/ klasörünü izle
│   # - YOLO tahminleri yap
│   # - /api/* endpoint'leri sağla
│
├── trainer.py              # 🧠 Auto-trainer
│   # - save1 veri sayısını kontrol et
│   # - 500+ olunca YOLO eğit
│   # - Kalite kontrol (eski model ile karşılaştır)
│   # - İyi olmuşsa best.pt güncelle
│
└── data/                   # 💾 Sistem dosyaları
    ├── db.json             #    Resim metadataları
    ├── queue.json          #    İşlemde olan resimler
    ├── image_hashes.json   #    Duplicate algılama (SHA256)
    ├── training_logs.json  #    Eğitim geçmişi
    └── log_blood.txt       #    Debug loglar
```

---

## 🚀 Başlangıç

```bash
# 1. En az bir best.pt modeli koy: best/ klasörüne

# 2. Başlat
python app.py
```

**3. Tarayıcıda aç:**
```
http://yapayzeka:8503
```
*veya*
```
http://localhost:8503
```

**4. ham/ klasörüne resim koy**
- Sistem otomatik olarak görecek
- YOLO çalıştıracak  
- Web UI'de gösterecek

---

## ⚙️ Konfigürasyon

**bkalite_config.py** dosyasında:

```python
CLASS_NAMES = ["kan"]              # Sınıf adları (yeni sınıf: ["kan", "morluk", "kemik"])

MIN_BOX_AREA_RATIO = 0.00001      # Min kutu boyutu
MAX_BOX_AREA_RATIO = 0.18         # Max kutu boyutu
INFER_CONF = 0.60                 # Tahmin confidence threshold

TRAIN_TRIGGER_SAVE1 = 500         # Kaç yeni veri olunca eğitim başlasın
TRAIN_EPOCHS = 250                # Eğitim tur sayısı
TRAIN_BATCH = 4                   # Batch boyutu
TRAIN_IMGSZ = 1024                # YOLO giriş boyutu
```

---

## 📊 Sistem Akışı

1. **ham/** klasörüne resim konur
2. **app.py** YOLO'yu çalıştırır → kutu tahminleri yapar
3. Web UI'de kutular gösterilir
4. Kullanıcı onaylar ✅ → **save1/** (eğitim verisi)
5. Kullanıcı reddeder ❌ → **save2/** (arşiv)
6. **trainer.py** 30 saniye aralığında kontrol eder:
   - "500+ yeni veri var mı?" → Varsa **YOLO eğit**
7. Yeni model eski modelden iyi midir? 
   - Evet → **best.pt güncelle** ✅
   - Hayır → Eski model kalsın ❌
8. Döngü başa dön (yeni model ile tahmin yapılır)

---

## 🛠️ Teknik

- **AI:** YOLO v8 (Ultralytics)
- **Backend:** Flask + CORS
- **Frontend:** HTML5 + Vanilla JavaScript
- **DB:** JSON (basit, şifre yok)
- **Threading:** Arka plan eğitim (UI bloklanmaz)

---

## 🔍 Hata Ayıklama

YOLO kutularındaki sorunları bulmak için:

```bash
python annotation_debug.py
```

Çıktı: `data/annotation_debug/` klasöründe:
- `image__OK.jpg` - sorun yok
- `image__WARN.jpg` - uyarı var (çok büyük/küçük kutu vb)
- `report.json` - detaylı rapor

---

## 📝 API Endpoints

| Method | Endpoint | Açıklama |
|--------|----------|----------|
| GET | `/api/queue` | Onay bekleyen resimleri listele |
| POST | `/api/save` | Onay/reddet kaydet |
| GET | `/api/classes` | Sınıf listesi |
| GET | `/api/training-status` | Eğitim durumu |

---

## 📌 Önemli Notlar

- **Yeni sınıf eklerken:** `bkalite_config.py`'deki `CLASS_NAMES` değiştir, tüm sistem bunu okuyor
- **YOLO Format (save1/txt/):** `<class_id> <cx> <cy> <width> <height>` (koordinatlar 0-1 arası)
- **Duplicate Kontrol:** Hash tabanlı (aynı resim 2 kez işlenmez)
- **Threshold:** `INFER_CONF` değerini düşürürsen daha fazla kutu göreceksin (yanlış pozitif artar)

---

**v2 - Gedik Piliç - 2025**

## 👨‍💼 Maintainer

- Aleyna Kapusuz
- Hande Bandırmalı

---

**Son Güncelleme:** 
