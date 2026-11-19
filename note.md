# Full RFM Segmentation and Customer Profiling

## 1. Məqsəd
Müştəriləri alış davranışına görə (Recency, Frequency, Monetary) qaydaya əsaslanan (rule-based) seqmentlərə bölmək və hər seqment üçün konkret marketinq tədbirləri təklif etmək.

## 2. Dataset
- **Fayl:** `sales_data_sample.csv` (Kaggle: kyanyoga/sample-sales-data). Başqa dataset istifadə olunmayıb.
- **Alət:** R (RStudio, R Markdown). Paketlər: `tidyverse`, `lubridate`, `plotly`.
- **Reference date:** `2005-06-01` (datasetdəki ən son sifariş tarixi + 1 gün).
- **Müştəri sayı:** 92 unikal müştəri (`CUSTOMERNAME`).
- Nəticələr (cədvəllər və qrafiklər) `rfm_segmentation.Rmd` faylının knit edilmiş HTML versiyasında görünür.

## 3. Metodologiya
Hər müştəri üçün `group_by(CUSTOMERNAME)` ilə üç göstərici hesablanıb:

| Göstərici | Hesablanma | Yaxşı nəticə |
|---|---|---|
| **Recency (R)** | reference date ilə müştərinin son sifariş tarixi arasındakı gün sayı | az gün |
| **Frequency (F)** | `n_distinct(ORDERNUMBER)`, yəni unikal sifariş sayı | çox sifariş |
| **Monetary (M)** | `sum(SALES)`, yəni ümumi satış məbləği | çox məbləğ |

Frequency üçün `n()` deyil, `n_distinct(ORDERNUMBER)` istifadə olunub, çünki datasetdə hər sətir bir məhsul xəttidir və bir sifariş bir neçə sətirdən ibarət ola bilər.

### Ümumi statistika
| Göstərici | Min | Orta | Median | Max |
|---|---|---|---|---|
| Recency (gün) | 1 | 182.8 | 186 | 509 |
| Frequency (sifariş) | 1 | 3.3 | 3 | 26 |
| Monetary | 9,129 | 109,050 | 86,523 | 912,294 |

## 4. Ballandırma məntiqi
Hər göstərici `ntile(x, 4)` ilə **1-4 arası** ballandırılıb (kvartillər, hər qrupda 23 müştəri):

- **F və M:** 4 = ən yüksək dəyər.
- **R (recency inversion):** `ntile(desc(Recency), 4)` istifadə olunub. Beləliklə ən az gün keçmiş (ən yaxın tarixli) müştərilər 4, ən köhnələr 1 alır.
- **RFM_Segment:** üç bal string kimi birləşdirilib, məsələn `"444"` (R=4, F=4, M=4) və ya `"311"`.

R-in `ntile()` funksiyası Python-dakı `pd.qcut(...)` ilə eyni işi görür. Bərabər dəyərlər olduqda (məsələn, Frequency-də çoxlu müştəri 3 sifariş verib) `ntile()` onları sıra ilə müxtəlif qruplara bölür.

## 5. Seqment qaydaları
Qaydalar **yuxarıdan aşağı** yoxlanılır, müştəri ilk uyğun gələn seqmentə düşür:

| Seqment | Qayda | Mənası |
|---|---|---|
| **Champions** | R=4, F=4, M=4 | yaxınlarda alıb, tez-tez alır, ən çox xərcləyir |
| **At Risk** | R=1, F>=3, M>=3 | əvvəllər güclü müştəri idi, indi uzaqlaşıb |
| **Lost** | R=1, F=1, M=1 | çoxdan yoxdur, az alıb, az xərcləyib |
| **Loyal** | R>=2, F>=3 | aktivdir və tez-tez alır |
| **Needs Attention** | qalan hamı | orta səviyyəli, uzaqlaşa bilər |

## 6. Nəticələr

| Seqment | Müştəri sayı | Orta Recency (gün) | Orta Frequency | Orta Monetary | Ümumi satışda pay |
|---|---|---|---|---|---|
| Champions | 9 | 19.4 | 8.1 | 290,098 | 26.0% |
| Loyal | 36 | 129.3 | 3.4 | 109,685 | 39.4% |
| Needs Attention | 35 | 216.2 | 2.5 | 78,691 | 27.5% |
| At Risk | 1 | 456.0 | 3.0 | 142,874 | 1.4% |
| Lost | 11 | 360.5 | 1.9 | 52,367 | 5.7% |
| **Cəmi** | **92** | | | | **100%** |

### Əsas müşahidələr
- **Champions** müştərilərin cəmi ~10%-ni (9 nəfər) təşkil edir, amma satışın **26%-ni** verir. Orta xərcləri Loyal seqmentindən təxminən 2.6 dəfə çoxdur.
- **Loyal** ən böyük gəlir mənbəyidir: satışın **39.4%-i**. Bu seqmentin bir hissəsini Champions-a yüksəltmək ən böyük potensialdır.
- **Needs Attention** seqmenti böyükdür (35 müştəri, satışın 27.5%-i), orta recency 216 gündür. Aktiv işlənməsə, bu müştərilər At Risk və Lost-a keçə bilər.
- **Lost** seqmentində 11 müştəri var, satışın yalnız 5.7%-ni verir. Reaktivasiya kampaniyası lazımdır, amma büdcə aşağı saxlanmalıdır.
- **At Risk** seqmentində yalnız 1 müştəri var (aşağıdakı "Məhdudiyyətlər" bölməsinə bax).

### Vizuallar
1. **Bar chart:** hər seqmentdəki müştəri sayı.
2. **Scatter plot:** Recency vs Monetary, seqmentə görə rənglənib, y oxu log miqyasındadır (Monetary dəyərləri 9K-dan 912K-a qədər dəyişir).
3. **Heatmap:** hər seqmentin orta R, F, M balı.
4. **Bonus:** interaktiv 3D scatter plot (Recency, Frequency, Monetary), `plotly` ilə.
5. **Bonus:** final RFM cədvəli `rfm_report.csv` kimi export edilib (marketinq komandası üçün).

## 7. Seqment üzrə marketinq tədbirləri

| Seqment | Tədbirlər |
|---|---|
| **Champions** | VIP/loyallıq proqramı, yeni məhsullara erkən çıxış, fərdi menecer, tövsiyə (referral) və rəy istəmək |
| **Loyal** | Əlavə satış (upsell/cross-sell) paketləri, Champions-a yüksəlmək üçün pilləli mükafatlar, həcmə görə endirimlər |
| **Needs Attention** | Vaxt məhdudiyyətli təkliflər, fərdi məhsul tövsiyələri, əvvəlki alışlara əsasən xatırlatma e-poçtları |
| **At Risk** | Fərdi əlaqə (zəng/e-poçt), geri qaytarma endirimi, getmə səbəbini öyrənmək üçün qısa sorğu, yeniliklərin təqdimatı |
| **Lost** | Aşağı büdcəli reaktivasiya kampaniyası, son şans təklifi; cavab gəlmirsə, marketinq büdcəsini bu qrupa xərcləməyi dayandırmaq |

## 8. Tapşırıqdakı referans rəqəmlərlə müqayisə
| Göstərici | Referans | Mənim nəticəm |
|---|---|---|
| Müştəri sayı | 92 | 92 |
| Champions | 9 | 9 |
| Lost | 11 | 11 |
| At Risk | 7 | 1 |
| Recency max / orta | 532 / 187 | 509 / 182.8 |
| Frequency max / orta | 43 / 13.5 | 26 / 3.3 |
| Monetary max / orta | 276K / 38.7K | 912K / 109K |

Müştəri sayı, Champions və Lost referansla üst-üstə düşür. Digər göstəricilər fərqlidir. Frequency və Monetary üçün referans rəqəmlər datasetdən birbaşa hesablanmış dəyərlərlə uyğun gəlmir, ona görə onları təxmini istiqamətverici kimi qəbul etdim. At Risk fərqinin səbəbi növbəti bölmədədir.

## 9. Məhdudiyyətlər
- **At Risk seqmenti çox kiçikdir (1 müştəri).** Qayda (R=1, F>=3, M>=3) çox dardır: Frequency-də müştərilərin çoxu 2-3 sifariş verib (median = 3), ona görə recency-si ən zəif olan qrupda F və M eyni anda yüksək olan müştəri azdır. Qaydanı genişləndirmək (məsələn, F>=2) At Risk-in ölçüsünü artırardı.
- **Frequency-də çoxlu bərabər dəyər var** (kvartillər 2, 3, 3). `ntile()` bərabər dəyərlərə malik müştəriləri sıra ilə müxtəlif qruplara böldüyü üçün F balı müəyyən dərəcədə müştərilərin cədvəldəki sırasından asılıdır. Seqment sayları Python-dakı `pd.qcut` nəticəsindən az fərqlənə bilər.
- **Yalnız 92 müştəri var**, hər bal qrupunda cəmi 23 nəfərdir. Seqmentlərin ölçüləri kiçikdir, nəticələr qaydaların seçiminə həssasdır.
- Seqment qaydaları **subyektivdir**, biznesin ehtiyacına görə dəyişdirilə bilər.
- Analiz yalnız keçmiş sifariş məlumatına əsaslanır. Məhsul kateqoriyası, qaytarmalar və ya müştəri xərcləri nəzərə alınmayıb.

## 10. Təhvil verilən fayllar
- `rfm_segmentation.Rmd`: bütün RFM pipeline-ı, şərhlərlə
- `index.html`: knit edilmiş hesabat (cədvəllər və qrafiklər daxildir)
- `rfm_report.csv`: final RFM cədvəli (bonus)
- `note.md`: bu fayl

