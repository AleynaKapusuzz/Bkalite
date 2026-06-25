# Bkalite

# B Kalite Kontrol Sistemi

Otomatik öğrenen YOLO v8 tabanlı görüntü kontrol sistemi. Kan, morluk, kemik vb. problemli bölgeleri tespit eder ve kullanıcı onaylarıyla modeli otomatik olarak iyileştirir.

## 🎯 Sistem Amacı

**Temel İş Akışı:**
1. Yeni görüntüler `ham/` klasörüne girilir
2. AI modeli otomatik olarak problemli bölgeleri tespit eder
3. Kullanıcı web arayüzünden sonuçları onaylar/düzenler/reddeder
4. Onaylanan veriler eğitim setine eklenir
5. Belirli miktarda veri biriktince model otomatik olarak yeniden eğitilir
6. Yeni model eski modelden daha iyi ise sisteme uygulanır

Bu sayede sistem kullanıldıkça kendi kendini geliştiriyor.

---

## 📊 Sistem Mimarisi

```
┌─────────────────────────────────────────────────────────────────┐
│                     B Kalite Kontrol Sistemi                    │
└─────────────────────────────────────────────────────────────────┘

┌──────────────┐
│  ham/ klasör │  ← Yeni görüntüler buraya konur (PNG/JPG)
└────────┬─────┘
         │
         ▼
┌──────────────────────────────────────────────────────────────┐
│ Flask Backend (app.py)                                       │
│  • Dosyaları izler                                           │
│  • YOLO modelini çalıştırır                                  │
│  • Sonuçları web arayüzüne gönderir                          │
└──────────────┬─────────────────────────────────┬─────────────┘
               │                                 │
        ┌──────▼──────┐                   ┌──────▼──────┐
        │  Web UI      │◄─────────────────►│  API        │
        │ (index.html) │     JSON          │ (REST)      │
        └──────┬───────┘                   └─────────────┘
               │
      (Kullanıcı onaylar)
               │
        ┌──────▼──────────────────────────────────┐
        │  Onaylanan mı? ← save1/  (eğitim seti) │
        │  Reddedilen mi? ← save2/  (arşiv)      │
        └──────┬───────────────────────────────────┘
               │
        (Her 30 saniyede kontrol)
               │
        ┌──────▼──────────────────────────────────┐
        │ Trainer (trainer.py)                   │
        │  • save1'deki veri sayısını kontrol    │
        │  • 500+ yeni veri olunca eğitim       │
        │  • YOLO v8 modeli fine-tune            │
        │  • Kalite kontrol (metrikleri kontrol) │
        └──────┬───────────────────────────────────┘
               │
        ┌──────▼──────────────────────────────────┐
        │ best/best.pt (Yeni Model)              │
        │  • Eski modelden daha iyiyse           │
        │    sisteme uygulanır                   │
        │  • Değilse önceki model kalır          │
        └──────────────────────────────────────────┘
```

---

## 📁 Dosya Yapısı & Görevleri

### **Backend (Python)**

| Dosya | Görev |
|-------|-------|
| **app.py** | Flask uygulaması. ham/ klasörünü izler, YOLO çalıştırır, API endpoint'leri sağlar |
| **trainer.py** | Auto-learning. save1/ klasöründe 500+ yeni veri olunca modeli yeniden eğitir |
| **bkalite_config.py** | **Merkezi ayarlar** (sınıf adları, threshold değerleri, eğitim parametreleri) |
| **annotation_debug.py** | Hata ayıklama. YOLO kutularını resme çizerek sorunlu açıklamaları bulur |
| **log_blood.py** | Basit loglama sistemi |
| **service_blood.py** | Windows NSSM servisi başlatıcı (opsiyonel) |

### **Frontend (JavaScript/HTML)**

| Dosya | Görev |
|-------|-------|
| **index.html** | Ana arayüz. Kategorize edilmemiş resimleri gösterir |
| **app.js** | Ana arayüzün JavaScript (YOLO kutularını görselleştirir, drag-drop) |
| **review.html** | Küçük resim önizleme (thumbnail) sayfası |
| **review.js** | Review sayfasının JavaScript |

---

## 📂 Çalışma Klasörleri

```
proje/
├── ham/                    # Input: Yeni görüntüler buraya konur
│   ├── resim1.png
│   ├── resim2.jpg
│   └── ...
│
├── save1/                  # ✅ Onaylanan veriler (EĞİTİME GİRER)
│   ├── resimler/           #    Onaylanan resimler
│   └── txt/                #    YOLO format açıklamalar
│
├── save2/                  # ❌ Reddedilen veriler (sadece arşiv)
│   └── resimler/           #    Sorun olduğu tespit edilen resimler
│
├── best/                   # 🤖 Modeller
│   └── best.pt             #    Aktif YOLO modeli (eğitim için)
│
├── trained/                # 📊 Eğitim geçmişi
│   ├── resimler/           #    Eğitimde kullanılan veriler
│   └── txt/
│
└── data/                   # 💾 Sistem dosyaları
    ├── db.json             #    Görüntü metadataları
    ├── queue.json          #    İşlemde olan/onay bekleyen resimler
    ├── image_hashes.json   #    Duplicate algılama
    └── training_logs.json  #    Eğitim geçmişi ve metrikleri
```

---

## 🚀 Nasıl Çalışır?

### **1. Başlatma**
```bash
python app.py
```
- Flask servisi başlar (localhost:8503)
- `ham/` klasörü izlenmeye başlar
- Auto-training monitörü başlar

### **2. Resim Girişi**
- `ham/` klasörüne PNG/JPG resimler konur
- Sistem otomatik olarak bunları tespit eder
- YOLO modeli çalıştırır

### **3. Arayüz**
- Tarayıcıdan `http://localhost:8503` açılır
- Tespit edilen kutular resme çizilmiş olarak görülür
- Kullanıcı onaylar ✅ veya reddeder ❌

### **4. Auto-Learning**
- Onaylanan resimler: `save1/` → **eğitim verisi**
- Reddedilen resimler: `save2/` → arşiv
- 500+ yeni veri olunca `trainer.py` otomatik eğitim başlatır
- Yeni model eski modelden %5'ten daha kötü değilse uygulanır

---

## ⚙️ Konfigürasyon

Tüm ayarlar **`bkalite_config.py`** dosyasında:

```python
CLASS_NAMES = ["kan"]                    # Sınıf adları (genişlettiğinde bunu değiştir)

MIN_BOX_AREA_RATIO = 0.00001            # Min kutu boyutu
MAX_BOX_AREA_RATIO = 0.18               # Max kutu boyutu
MAX_BOX_ASPECT_RATIO = 8.0              # En/boy oranı limit

TRAIN_TRIGGER_SAVE1 = 500               # Eğitim için gerekli yeni veri sayısı
TRAIN_EPOCHS = 250                      # Eğitim tur sayısı
TRAIN_BATCH = 4                         # Batch boyutu
INFER_CONF = 0.60                       # Tahmin kesinlik threshold
```

---

## 🛠️ Teknik Detaylar

- **AI Framework:** YOLO v8 (Ultralytics)
- **Backend:** Flask + CORS
- **Frontend:** HTML5 + Vanilla JavaScript
- **Veritabanı:** JSON (basit, sync-free)
- **Threading:** Arka plan eğitim (UI'yi bloklamaz)

---

## 📝 API Endpoint'leri

| Method | Endpoint | Açıklama |
|--------|----------|----------|
| GET | `/api/queue` | Onay bekleyen resimleri listele |
| POST | `/api/save` | Resim onayı kaydet (save1 veya save2) |
| GET | `/api/classes` | Sınıf listesini döndür |
| GET | `/api/training-status` | Eğitim durumunu kontrol et |
| POST | `/api/image/<hash>` | Resim detaylarını getir |

---

## 📊 Dosya Isimlendirmesi

**YOLO Format (save1/txt/):**
```
<class_id> <cx> <cy> <width> <height>
0 0.45 0.52 0.15 0.20
```
- Merkez koordinatlar ve boyutlar 0-1 arası normalize edilir
- class_id: 0 = "kan"

---

## 🔍 Hata Ayıklama

YOLO kutularındaki sorunları bulmak için:
```bash
python annotation_debug.py
```
- YOLO kutularını resme çizer
- Sorunlu kutuları raporlar
- Çıktı: `data/annotation_debug/` klasörü

---

## 📋 Not

- **Yeni sınıf eklerken:** `bkalite_config.py`'deki `CLASS_NAMES` listesini değiştir, tüm sistem bunu okur
- **Ham veri yapısı:** Resimler PNG/JPG, açıklamalar YOLO format TXT
- **Duplicate kontrolü:** Hash tabanlı (aynı resim 2 kez işlenmez)

---

## 👨‍💼 Maintainer

- Aleyna Kapusuz
- Hande Bandırmalı

---

**Son Güncelleme:** 
