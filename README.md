# 🩸 B Kalite Kontrol Sistemi v2

Gedik Piliç pilav üreticiliği için **otomatik kan kalite kontrol sistemi**. YOLO v8 deep learning modeliyle pilav paketlerindeki kan (hemolitik) lekeleri tespit edip sınıflandırır. Sistem kullanıcı onayıyla otomatik olarak öğrenip (auto-learning) güncelleme yapar.

---

## 🎯 Proje Amacı

**Ne yapılmak istedi:**
- Pilav paketlerine verilen kalite kontrol süreci otomatikleştirmek
- Kan kontaminasyonu olan paketleri otomatik tespit etmek
- Sistem verileriyle YOLO modelini sürekli eğiterek accuracy artırmak

**Mevcut durumu:**
- Manuel kontrol → YOLO otomatik tespit → Kullanıcı onayı → Otomatik model güncelleme döngüsü

---

## 🛠️ Kullanılan Teknolojiler

| Teknoloji | Amaç |
|-----------|------|
| **Python 3.10+** | Backend ve ML işlemleri |
| **Flask** | Web API sunucusu |
| **YOLOv8 (Ultralytics)** | Kan tespit modeli |
| **PIL/Pillow** | Görüntü işleme |
| **HTML5 + CSS3** | Frontend arayüz |
| **Vanilla JavaScript** | Frontend interaksiyonları |
| **JSON** | Veri depolama (SQLite yok) |
| **NSSM** | Windows servis yönetimi |

---

## 🌐 Erişim

**Web Arayüzü:**
```
http://yapayzeka:8503
```

**Bileşenler:**
- `/` → Ana kontrol paneli (inference + onay)
- `/review` → Arşiv tarama ve veri istatistikleri

---

## 📁 Klasör Yapısı

```
B_Kalite_Sistem/
│
├── ham/                          # 📥 Gelen ham görüntüler (izlenen input)
│   └── [yeni resim ekleme noktası]
│
├── best/                         # 🤖 YOLO model dosyaları
│   ├── best.pt                   # Aktif modelin ağırlıkları
│   ├── best_v1.pt, best_v2.pt... # Model versiyonları (backup)
│   └── *.yaml                    # Model konfigürasyon dosyaları
│
├── save1/                        # ✅ KAN VARSA (eğitime girer)
│   ├── resimler/                 # [000001.png, 000002.png, ...]
│   └── txt/                      # YOLO format etiketler [000001.txt, ...]
│
├── save2/                        # ❌ KAN YOKSA (sadece arşiv)
│   └── resimler/                 # [istatistik ve geçmiş]
│
├── trained/                      # 📚 Eğitim veri seti (snapshot)
│   ├── resimler/                 # [eğitilmiş tüm resimler]
│   └── txt/                      # Karşılık gelen etiketler
│
├── data/                         # 💾 Sistem veritabanı ve loglar
│   ├── db.json                   # İstatistikler ve meta
│   ├── queue.json                # İşlenecek / onay bekleyen resimler
│   ├── image_hashes.json         # Duplicate kontrol indexi
│   ├── training_logs.json        # Eğitim geçmişi ve model metrikleri
│   ├── log_blood.txt             # Debug logları
│   └── annotation_debug/         # Görsel kontrol raporu
│
├── bkalite_config.py             # ⚙️ Merkezi ayar dosyası
├── app.py                        # 🔧 Flask backend
├── trainer.py                    # 🧠 Auto-learning trainer
├── service_blood.py              # 🚀 NSSM servis başlatıcı
├── log_blood.py                  # 📝 Loglama sistemi
├── annotation_debug.py           # 🔍 Etiket kalite kontrol
│
├── index.html                    # 🖥️ Ana dashboard HTML
├── app.js                        # ⚙️ Ana dashboard JavaScript
├── review.html                   # 📊 Arşiv tarama sayfası
├── review.js                     # 📊 Arşiv tarama JavaScript
│
└── README.md                     # Bu dosya
```

---

## 📋 Dosya Açıklamaları

### Backend (Python)

| Dosya | Görev |
|-------|-------|
| **bkalite_config.py** | 🔑 **Merkezi ayarlar**. Class adları, model parametreleri, eğitim tetikleri. Başka dosyalar bunu import eder. |
| **app.py** | 🔧 **Flask API sunucusu** (1377 satır). Ham klasörü izler, yeni görüntü gelince YOLO çalıştırır, JSON API sunar. |
| **trainer.py** | 🧠 **Auto-Learning motor** (718 satır). 500 yeni resim → eğitim trigger, model versiyonlama, acceptance gate. |
| **log_blood.py** | 📝 **Loglama**. Tüm olaylar data/log_blood.txt'ye yazılır (max 50k satır). |
| **service_blood.py** | 🚀 **NSSM servis başlatıcı**. Windows Task Scheduler'dan başlatılır. |
| **annotation_debug.py** | 🔍 **Kalite kontrol aracı**. save1/txt'deki YOLO kutularını resimlere çizer, sorunlu kutuları raporlar. |

### Frontend (HTML/JS)

| Dosya | Görev |
|-------|-------|
| **index.html** | 🖥️ **Ana arayüz** (1203 satır). Giriş ekranı, kontrol paneli, resim/kutu gösterimi, onay duyguları. |
| **app.js** | ⚙️ **Frontend mantığı** (52 KB). Backend API çağrıları, kutu çizme, timer, live feed, localStorage. |
| **review.html** | 📊 **Arşiv/İstatistik sayfası** (144 KB). save1 ve save2 geçmişini, model eğitim loglarını gösterir. |
| **review.js** | 📊 **Arşiv mantığı** (20 KB). Filtreleme, pagination, veri görselleştirme. |

---

## 🔄 İş Akışı (Sistem Mantığı)

### 1️⃣ Giriş (Input)
```
ham/ klasörüne yeni resim atıl
        ↓
app.py tarafından otomatik algılanır
```

### 2️⃣ Analiz
```
YOLO best.pt modelini çalıştır
        ↓
Kan (hemolitik leke) tespitini yap
        ↓
Kutuları (bbox) JSON'a serileştir
```

### 3️⃣ Arayüz Gösterimi
```
index.html'de resim + kutuları göster
        ↓
Kullanıcı kutuları kontrol et:
  • ✅ Doğru     → save1 (eğitime girer)
  • ❌ Yanlış    → sil / değiştir
  • 📝 Resimleme → can, morluk, kemik açıkla
        ↓
Onayı gönder (POST /api/approve)
```

### 4️⃣ Depolama
```
save1/resimler/ + save1/txt/       (kan VAR → eğitim)
     veya
save2/resimler/                    (kan YOK → arşiv)
```

### 5️⃣ Auto-Learning (Otomatik Eğitim)
```
save1/resimler/ klasöründe 500+ resim
        ↓
trainer.py tarafından tespit edilir
        ↓
YOLO yolov8l.pt'den YOLO full retrain (250 epoch)
        ↓
Eski model vs yeni model karşılaştır
        ↓
Precision/Recall %5'ten fazla düşmediyse → best.pt güncelle
        ↓
Eğitim logu data/training_logs.json'a yazıl
        ↓
Arayüzde model durumu gösteril
```

---

## ⚙️ Ayar Dosyası (bkalite_config.py)

```python
CLASS_NAMES = ["kan"]                    # Sınıflar (gelecek: morluk, kemik)
MIN_BOX_AREA_RATIO = 0.00001             # Minimum kutu boyutu
MAX_BOX_AREA_RATIO = 0.18                # Maksimum kutu boyutu
MAX_BOX_ASPECT_RATIO = 8.0               # Max genişlik/yükseklik oranı

TRAIN_TRIGGER_SAVE1 = 500                # Eğitim trigger: 500 yeni resim
TRAIN_EPOCHS = 250                       # Eğitim dönem sayısı
TRAIN_BATCH = 4                          # Batch boyutu
TRAIN_IMGSZ = 1024                       # Model giriş boyutu

INFER_CONF = 0.60                        # Tahmin güven eşiği
INFER_IOU = 0.45                         # NMS IoU eşiği
```

**⚠️ ÖNEMLİ:** Yeni sınıf eklemek için **SADECE** `bkalite_config.py` düzenle!

---

## 🚀 Kurulum & Çalıştırma

### Ön Koşullar
```bash
Python 3.10+
CUDA 11.8+ (GPU varsa)
```

### Bağımlılıklar
```bash
pip install flask flask-cors ultralytics pillow pyyaml
```

### Başlatma

**Geliştirme (Debug):**
```bash
python app.py
# http://localhost:8503
```

---

## 📊 API Endpoints (Backend)

| Method | Endpoint | Açıklama |
|--------|----------|----------|
| **GET** | `/` | Ana arayüz (index.html) |
| **GET** | `/review` | Arşiv arayüzü (review.html) |
| **GET** | `/api/queue` | Onay bekleyen resimler |
| **POST** | `/api/approve` | Kutuyu onayla → save1 veya save2 |
| **POST** | `/api/delete` | Resmi sil |
| **POST** | `/api/reprocess` | Resmi yeniden işle |
| **GET** | `/api/stats` | İstatistikler |
| **GET** | `/api/model_info` | Model durum, eğitim tarihi |
| **GET** | `/api/training_logs` | Eğitim geçmişi |

---

## 🔍 Kalite Kontrol

YOLO etiketlerinin hata kontrolü:

```bash
python annotation_debug.py
```

**Çıktı:**
- `data/annotation_debug/` → Kutular çizilmiş resimler
- `data/annotation_debug/report.json` → Hata raporu
  - `bad_format` → Yanlış YOLO satırı
  - `out_of_range` → [0,1] dışına çıkan koord
  - `too_large` / `too_small` → Kutu boyut hataları
  - `bad_aspect` → Genişlik/yükseklik oranı aşırı

---

## 📈 Veri İstatistikleri

**data/db.json** içinde tutulur:
```json
{
  "total_processed": 1240,
  "with_blood": 847,
  "without_blood": 393,
  "training_count": 3,
  "latest_model_date": "2025-01-15T10:30:00",
  "model_metrics": { "precision": 0.92, "recall": 0.89, "map50": 0.95 }
}
```

**review.html** → Data sekmesi ile görselleştirme

---

## 🧠 Model Versiyonlama

Her başarılı eğitim yeni model oluşturur:
```
best/
├── best.pt           # Aktif model
├── best_v1.pt        # Eski versiyon (backup)
├── best_v2.pt
└── best_v3.pt
```

**Acceptance Gate:**
```
Yeni model
    ↓
Eski model metrikleriyle karşılaştır
    ↓
Precision/Recall/mAP50 > %5 düşüş?
    ↓
HAYIR → best.pt güncelle
EVET  → Reddedildi, eski best.pt devam et
```

---

## 📝 Log Dosyaları

**data/log_blood.txt** (çalışma zamanı logları):
```
2025-01-15 10:20:30 - [SERVICE] B Kalite servisi baslatiliyor
2025-01-15 10:20:31 - [WATCHER] 4 yeni görüntü algılandı
2025-01-15 10:21:15 - [INFERENCE] img_001.png → 2 kutu tespit
2025-01-15 10:21:20 - [APPROVE] Can detected → save1 kaydedildi
...
```

**data/training_logs.json** (eğitim geçmişi):
```json
{
  "trainings": [
    {
      "id": 1,
      "started": "2025-01-10T14:00:00",
      "status": "success",
      "trained_images": 500,
      "epochs": 250,
      "metrics": {
        "precision": 0.91,
        "recall": 0.88,
        "map50": 0.94
      }
    }
  ]
}
```

---

## 🐛 Troubleshooting

| Sorun | Çözüm |
|-------|-------|
| Ham klasörü izlenmiyor | `service_blood.py` çalışıyor mu? Log kontrol et. |
| YOLO hata | `best/best.pt` var mı? Model path config kontrol et. |
| Model eğitilmiyor | `save1/resimler/` 500+ resim var mı? `TRAIN_TRIGGER_SAVE1` config kontrol. |
| Duplicate resimler | `data/image_hashes.json` sıfırla ve reprocess. |
| API timeout | `INFER_IMGSZ=1024` GPU memory için azalt (512/768). |

---

## 📦 Deployment

### Windows Üretim Sunucusu

1. **Python yükle** (3.10+)
2. **Klasör yapısını oluştur** (`ham/`, `best/`, `save1/`, vb)
3. **Bağımlılık yükle:**
   ```bash
   pip install -r requirements.txt
   ```
4. **NSSM servis kurulumu:**
5. **Başlat:**
   ```
   http://yapayzeka:8503
   ```

---

## 🔐 Güvenlik Notları

- Flask `app.secret_key` güvenli string olarak değiştirilmeli (üretim)
- Sadece lokal ağda (`yapayzeka`) çalışması istenir
- Ham klasörü yazma yetkisine sahip bir kullanıcı run et
- Dosya izinlerini sınırla (644 resim, 755 klasörler)

---

## 👨‍💼 Maintainer

- Aleyna Kapusuz
- Hande Bandırmalı

---

**Son Güncelleme:** 
