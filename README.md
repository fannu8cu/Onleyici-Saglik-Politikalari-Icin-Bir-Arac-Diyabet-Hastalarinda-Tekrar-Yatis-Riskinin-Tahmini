# Onleyici Saglik Politikalari Icin Bir Arac: Diyabet Hastalarinda Tekrar Yatis Riskinin Tahmini

Bu proje, 10 yillik hasta verisini kullanarak, bir hastanin taburcu olduktan sonra hastaneye tekrar yatma olasiligini tahmin eden bir makine ogrenmesi modeli gelistirmeyi amaclamaktadir.

## Problem Tanimi

Hastaneye tekrar yatislar (readmissions), saglik sistemleri icin hem finansal bir yuktur hem de bakim kalitesinin bir gostergesidir. Onlenebilir tekrar yatislar, genellikle yetersiz taburculuk takibi veya hastanin risk durumunun dogru degerlendirilememesi anlamina gelir.

## Projenin Amaci

* **Ana Amac:** Hastaneden taburcu olan diyabet hastalarinin tekrar yatis riskini tahmin eden bir siniflandirma modeli gelistirmek.
* **Ikincil Amac:** Riski artiran en onemli faktorleri (feature importance) belirleyerek, kanita dayali politika onerileri sunmak.

## Veri Seti

Projede [Kaggle'da](https://www.kaggle.com/datasets/dubradave/hospital-readmissions) bulunan "Hospital Readmissions" (10 yillik diyabet verisi) seti kullanilmistir. Veri seti, demografik bilgileri, teshisleri (diag_1, diag_2), laboratuvar proseduurlerini ve ilac bilgilerini icermektedir.

Veri seti `kagglehub` kutuphanesi araciligiyla dogrudan Google Colab ortamina cekilmistir.

## Metodoloji ve Is Akisi

Proje akisi asagidaki adimlari kapsamaktadir:

1.  **Problem Cerceveleme:** Hedef degisken (`readmitted`) 1/0 olarak etiketlendi.
2.  **Kesifsel Veri Analizi (EDA):** Veri temizleme kararlarini desteklemek icin gorsellestirmeler yapildi. `age` (yas araligi) ve `medical_specialty` (doktor uzmanligi) gibi kategorik degiskenlerin risk uzerindeki etkisi incelendi.
3.  **Veri On Isleme:**
    * `diag_1`, `diag_2`, `diag_3` sutunlarindaki az sayidaki 'Missing' satir silindi.
    * `medical_specialty` sutunundaki %50'ye varan 'Missing' degeri, "bilgi kaybi" yerine "anlamli bir kategori" olarak korundu.
    * `age` (`[70-80)`) gibi araliklar, orta noktasi (`75`) alinarak sayisallastirildi.
    * Iki-sinifli (`change`) ve sirali (`A1Ctest`) kategorik degiskenler `map` ile, nominal degiskenler (`diag_1`, `medical_specialty`) ise `One-Hot Encoding` ile donusturuldu.
4.  **Ozellik Muhendisligi (Feature Engineering):** Modelin vaka karmasikligini daha iyi anlamasi icin yeni ozellikler uretildi:
    * `total_prior_visits`: Gecmis yilki toplam hastane kullanimi (acil + yatisli + ayakta).
    * `meds_per_day`: Yatis yogunlugu (toplam ilac sayisi / hastanede kalis suresi).
    * `diabetes_complexity_score`: Diyabet yonetim karmasikligini gosteren kompozit bir skor (test sonuclari + ilac kullanimi + ilac degisimi).
5.  **Modelleme:** Iki farkli model karsilastirildi:
    * **Lojistik Regresyon** (Baseline Model) - `StandardScaler` ile olceklendirildi.
    * **Random Forest Classifier** (Sampiyon Model) - Olceklendirilmemis veri kullanildi.

## Temel Bulgular - Model Karsilastirmasi

Bu projede temel basari metrigi "Accuracy" (Genel Dogruluk) degil, yuksek riskli hastalari teshis etme basarisi olan **Recall (Duyarlilik)** olarak belirlenmistir.

| Metrik | Lojistik Regresyon (Baseline) | Random Forest (Sampiyon) | Yorum |
| :--- | :---: | :---: | :--- |
| **Recall ('Riskli' Sinifi)** | **%41.0** | **%52.0** | **Buyuk Iyilesme** |
| Gozden Kacirilan Hastalar | 1371 Hasta | 1133 Hasta | **238 Hasta Kurtarildi** |
| Accuracy (Genel Dogruluk) | %61.4 | %60.9 | (Benzer seviyede) |

**Sonuc:** Lojistik Regresyon, riskli hastalarin yarisindan fazlasini (%59) gozden kacirdi. Random Forest modelimiz, **Recall skorunu %41'den %52'ye cikararak**, Lojistik Regresyon'un kacirdigi **238 yuksek riskli hastayi daha dogru tespit etti.**

## Politika Onerisi - En Onemli Risk Faktorleri

Random Forest modelinin "Feature Importance" analizi, riskin en buyuk belirleyicisinin hastanin **"vaka karmasikligi"** oldugunu ortaya koydu.

**En Onemli 5 Risk Faktoru:**

1.  `n_lab_procedures` (Yapilan Laboratuvar Testi Sayisi)
2.  `meds_per_day` (Gunluk Ilac Yogunlugu) - *(Kendi urettigimiz ozellik!)*
3.  `n_medications` (Toplam Ilac Sayisi)
4.  `time_in_hospital` (Hastanede Kalis Suresi)
5.  `age` (Yas)

**Eylem Onerisi:** Saglik kaynaklari (evde bakim, telefonla takip vb.), sadece 'yasli' veya 'diyabetli' hastalara degil; **yuksek sayida laboratuvar testi, yogun ilac tedavisi ve karmasik bir yatis sureci** geciren hastalara oncelikli olarak odaklanmalidir.

## Kullanilan Teknolojiler

* Python
* Pandas
* NumPy
* Scikit-learn
* Seaborn & Matplotlib
* KaggleHub API
* Google Colab
