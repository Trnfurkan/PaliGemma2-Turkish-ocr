# Veri seti

`veriseti/` klasörü (sahne görselleri + `manifest.csv`) bu depoya dahil edilmemiştir — görseller tanınabilir gerçek mekân/ürün/levha içerebileceğinden ve boyut nedeniyle ayrı tutulmuştur.

Notebook'un çalışması için `veriseti.zip` şu yapıda olmalı:

```
veriseti/
├── manifest.csv
├── ana/
│   └── orijinal/   (ve isteğe bağlı 224/, 448/ letterbox klasörleri)
└── ...             (GROUPS içinde tanımlı diğer gruplar)
```

`manifest.csv` beklenen sütunlar:

| Sütun | Açıklama |
|---|---|
| `dosya` | Görsel dosya adı |
| `grup` | Görsel grubu (ör. `ana`) |
| `ana_metin` | Ana metin parçaları, `\|` ile ayrılmış ground-truth |
| `ikincil_metin` | İkincil (küçük/ikincil) metin parçaları, `\|` ile ayrılmış |
| `dogrulandi` | (opsiyonel) yalnızca doğrulanmış satırları filtrelemek için |

## `results/` — bu depoda yer alan çıktılar

Bu klasördeki üç CSV, notebook'un 30 görsellik veri seti üzerinde üretilen çıktılarıdır ve veri seti olmadan da incelenebilir/analiz edilebilir:

- **`sonuclar.csv`** — modelin ham çıktıları. Sütunlar: `cozunurluk` (224/448), `prompt` (`ocr`/`tr_soru`/`en_soru`), `dosya`, `grup`, `cikti` (model çıktısı), `sure_sn` (üretim süresi, saniye).
- **`skorlar_parca.csv`** — parça bazlı ham skorlar. Sütunlar: `cozunurluk`, `prompt`, `dosya`, `tur` (`ana`/`ikincil`), `parca` (ground-truth metin parçası), `tam_bulundu`, `aksansiz_bulundu`, `hata_orani`.
- **`ozet.csv`** — `tur` × `cozunurluk` × `prompt` kırılımında özet istatistikler (`parca_sayisi`, `tam_bulundu`, `aksansiz_bulundu`, `ort_hata_orani` ortalamaları).

Metrik tanımları için ana [`README.md`](../README.md) ve notebook'taki "Ölçüm" bölümüne bakın.
