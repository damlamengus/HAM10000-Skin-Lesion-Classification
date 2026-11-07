# Dermatoskopik Görüntülerde Derin Öğrenme ile Deri Lezyonu Sınıflandırması

**Ders:** Biyomedikal Verilerin Analizi ve Yapay Zekâ  
**Veri Seti:** HAM10000  
**Proje Türü:** Çok sınıflı görüntü sınıflandırma, açıklanabilir yapay zekâ ve model dağıtımı  
**Durum:** 1. Aşama – Veri Keşfi ve Bölme Planı

## 1. Projenin Amacı ve Klinik Araştırma Sorusu

Bu projenin amacı, dermatoskopik görüntüler ve klinik metadata kullanılarak yedi farklı deri lezyonu sınıfının derin öğrenme yöntemleriyle sınıflandırılmasıdır. Özellikle melanomun tespit edilmesi, sınıf dengesizliğinin model performansına etkisi ve tahminlerin klinik güvenilirliği araştırılacaktır.

Proje kapsamında şu sorulara cevap aranacaktır:

1. Transfer öğrenme tabanlı modeller yedi deri lezyonu sınıfını ne kadar güvenilir ayırabilmektedir?
2. Aynı lezyona ait görüntülerin eğitim ve test kümelerine karışması performans sonuçlarını nasıl etkilemektedir?
3. Yaş, cinsiyet ve anatomik lokalizasyon bilgilerinin görüntü tabanlı sınıflandırmaya katkısı var mıdır?
4. Model tahminleri kalibrasyon ve Grad-CAM yöntemleriyle nasıl değerlendirilebilir?
5. Geliştirilen model Docker ve FastAPI kullanılarak bir tahmin servisine dönüştürülebilir mi?

## 2. Veri Seti

Çalışmada HAM10000 (Human Against Machine with 10000 Training Images) veri seti kullanılmaktadır.

**Kaynak:** https://doi.org/10.7910/DVN/DBW86T

Veri seti 10.015 dermatoskopik görüntü, 7.470 benzersiz lezyon ve yedi tanı sınıfından oluşmaktadır.

| Sınıf | Açıklama | Görüntü Sayısı |
|---|---|---:|
| nv | Melanositik nevüs | 6.705 |
| mel | Melanom | 1.113 |
| bkl | Benign keratoz benzeri lezyon | 1.099 |
| bcc | Bazal hücreli karsinom | 514 |
| akiec | Aktinik keratoz ve intraepitelyal karsinom | 327 |
| vasc | Vasküler lezyon | 142 |
| df | Dermatofibrom | 115 |

Veri setinde `lesion_id`, `image_id`, `dx`, `dx_type`, `age`, `sex` ve `localization` alanları bulunmaktadır.

Ham görüntüler GitHub deposunda tutulmamaktadır. Veri seti, kaynak sağlayıcının lisans koşulları dikkate alınarak indirilmelidir.

## 3. Keşifçi Veri Analizi

İlk aşamada sınıf dağılımı, eksik metadata, yaş, cinsiyet, anatomik lokalizasyon ve tekrar eden lezyonlar incelenmiştir.

### 3.1. Sınıf Dengesizliği

Görüntülerin yaklaşık %66,95'i melanositik nevüs sınıfına aittir. Melanom sınıfının oranı %11,11'dir.

Bu nedenle doğruluk (Accuracy) tek başına yeterli bir performans ölçütü olarak değerlendirilmeyecektir. Macro-F1, sınıf bazlı duyarlılık, dengeli doğruluk ve melanom için ROC-AUC gibi ölçütler kullanılacaktır.

### 3.2. Metadata Kalitesi

- Yaşı eksik görüntü kaydı: 57
- Cinsiyeti `unknown` olan kayıt: 57
- Lokalizasyonu `unknown` olan kayıt: 234
- Ortalama yaş: 51,86
- Ortanca yaş: 50

İki benzersiz lezyonda anatomik lokalizasyon bilgilerinin farklı ayrıntı düzeylerinde kodlandığı tespit edilmiştir.

Orijinal lokalizasyon bilgileri korunarak göğüs, sırt, karın ve gövde kategorileri yeni bir `localization_grouped` değişkeninde birleştirilmiştir. Bu işlem sonucunda aynı lezyon içindeki gruplanmış lokalizasyon tutarsızlıkları giderilmiştir.

### 3.3. Tekrarlanan Lezyonlar

Veri setinde 1.956 lezyonun birden fazla görüntüsü bulunmaktadır. Bu lezyonlara ait toplam 4.501 görüntü, tüm görüntülerin yaklaşık %44,94'ünü oluşturmaktadır.

Bu nedenle görüntü bazlı rastgele bölme işleminin aynı lezyona ait görüntüleri farklı kümelere dağıtma riski bulunmaktadır.

## 4. Veri Bölme Stratejisi

Veri sızıntısını önlemek amacıyla bölme işlemi `lesion_id` temel alınarak gerçekleştirilmiştir.

Her lezyon tek bir kayıtla temsil edilmiş, sınıf etiketine göre tabakalı bölme uygulanmış ve görüntüler ilgili lezyonun atandığı kümeye yerleştirilmiştir.

**Rastgelelik tohumu:** `SEED = 42`

| Veri Kümesi | Görüntü | Benzersiz Lezyon | Oran |
|---|---:|---:|---:|
| Eğitim | 6.981 | 5.229 | %69,71 |
| Doğrulama | 1.532 | 1.120 | %15,30 |
| Test | 1.502 | 1.121 | %15,00 |
| Toplam | 10.015 | 7.470 | %100 |

Üç küme arasında `lesion_id` çakışması bulunmadığı doğrulanmıştır.

Bölme sonuçları `results/ham10000_splits.csv` dosyasında saklanmaktadır.

HAM10000 veri setinde hasta kimliği bulunmadığından hasta düzeyinde bağımsızlık doğrulanamamaktadır.

## 5. Planlanan Modeller ve Değerlendirme

Projenin sonraki aşamalarında ImageNet üzerinde önceden eğitilmiş ResNet-18 ve EfficientNet-B0 modelleri kullanılacaktır.

Görüntü tabanlı modellere yaş, cinsiyet ve anatomik lokalizasyon bilgilerinin eklenmesiyle metadata katkısı ayrıca araştırılacaktır.

Değerlendirme kapsamında Macro-F1, sınıf bazlı Recall, ROC-AUC, PR-AUC, olasılık kalibrasyonu ve uygun klinik karar analizleri kullanılacaktır.

Grad-CAM ile modelin görüntünün hangi bölgelerine odaklandığı incelenecektir.

## 6. Çalıştırma Talimatları

### Google Colab

1. HAM10000 veri setini resmî kaynağından indirin.
2. `HAM10000_metadata.csv` dosyasını Colab ortamına yükleyin.
3. `notebooks/01_veri_kesfi.ipynb` defterini açın.
4. Kod hücrelerini sırayla çalıştırın.
5. Veri bölme sonuçlarını `results/ham10000_splits.csv` dosyasıyla karşılaştırın.

**Not:** Mevcut notebook metadata dosyasının `/content/HAM10000_metadata.csv` konumunda bulunmasını gerektirmektedir. Otomatik veri indirme adımı henüz eklenmemiştir.

### Docker ve FastAPI

Bu bölüm sonraki teslimlerde tamamlanacaktır.

Planlanan servis uç noktaları:

- `GET /health`
- `POST /predict`

Servis, dermatoskopik görüntüden sınıf olasılıkları ve Grad-CAM görselleştirmesi üretmek üzere tasarlanacaktır.

## 7. Sınırlılıklar

- Veri setinde belirgin sınıf dengesizliği bulunmaktadır.
- Hasta kimlikleri mevcut değildir.
- Bazı metadata alanlarında eksik veya bilinmeyen değerler bulunmaktadır.
- Veri setinin gerçek klinik ortamlara genellenebilirliği ayrıca doğrulanmalıdır.
- Geliştirilecek sistem araştırma amaçlıdır; klinik tanı aracı değildir.

## 8. Kaynakça

Tschandl, P., Rosendahl, C., & Kittler, H. (2018). The HAM10000 dataset, a large collection of multi-source dermatoscopic images of common pigmented skin lesions. *Scientific Data, 5*, 180161. https://doi.org/10.1038/sdata.2018.161

Esteva, A., et al. (2017). Dermatologist-level classification of skin cancer with deep neural networks. *Nature, 542*, 115–118. https://doi.org/10.1038/nature21056

**Veri seti:** https://doi.org/10.7910/DVN/DBW86T
