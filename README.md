# PaliGemma 2 ile Türkçe Sahne Metni OCR Deneyi

PaliGemma 2 (3B, `mix`) modelinin Türkçe sahne metnini (tabela, fiş, ambalaj, uyarı levhası vb.) ne kadar doğru okuduğunu; çözünürlüğün (224px / 448px) ve prompt biçiminin (`ocr`, Türkçe soru, İngilizce soru) sonucu nasıl etkilediğini inceleyen küçük ölçekli bir deney.

Ayrıntılı analiz ve eleştirel değerlendirme için bkz. [`docs/PALIGEMMA_2_rapor.docx`](docs/PALIGEMMA_2_rapor.docx).

## Araştırma soruları

- PaliGemma 2'nin Türkçe sahne metnini tanıma (OCR) performansı nedir?
- 224px ile 448px çözünürlük arasında fark var mı, varsa ne kadar?
- Prompt biçimi (doğrudan `ocr` komutu vs. soru formatı) sonucu değiştiriyor mu?
- Model harfleri doğru okuyup Türkçe aksanları (ş, ğ, ı, ç, ö, ü) kaçırıyor mu?

## Yöntem (özet)

- **Model:** `google/paligemma2-3b-mix-224` ve `google/paligemma2-3b-mix-448` (Hugging Face, gated — erişim onayı gerekir)
- **Donanım:** Google Colab, T4 GPU; `float16` (T4 `bfloat16`'yı desteklemediği için; boş/bozuk çıktı olursa `float32`'ye düşülür)
- **Veri seti:** 30 sahne görseli (tabela, fiş, ambalaj, uyarı levhası vb.), farklı font, ışık/yansıma ve silikleşme koşullarında; her görsel için elle yazılıp kontrol edilmiş ground-truth metin
- **Koşullar:** 2 çözünürlük × 3 prompt = 6 koşul, toplamda 45 ana + 87 ikincil metin parçası üzerinden 792 parça-koşul ölçümü
- **Metrikler** (`notebooks/paligemma2_ocr_deney.ipynb` içinde tanımlı):
  - `tam_bulundu`: parça, model çıktısında birebir (büyük/küçük harf ve noktalama göz ardı edilerek, İ/I ve ı/i farkı korunarak) geçiyor mu
  - `aksansiz_bulundu`: aynı kontrol, Türkçe aksanlar (ç/ğ/ı/ö/ş/ü → c/g/i/o/s/u) sadeleştirilerek
  - `hata_orani`: parçanın çıktı içindeki en iyi eşleşen konumla arasındaki karakter düzeyinde düzenleme mesafesi, parça uzunluğuna bölünmüş

## Önemli bulgular

| | 224px | 448px |
|---|---|---|
| Ana metin, `ocr` promptu, aksansız eşleşme | %53.3 | %82.2 |
| İkincil metin, `ocr` promptu, aksansız eşleşme | %23.0 | %65.5 |

- 448px çözünürlük, özellikle küçük/ikincil metinlerde belirgin bir iyileşme sağlıyor; ancak etki 30 görselin tamamında tutarlı değil (18 görselde iyileşme, 11'inde değişim yok, 1'inde gerileme).
- Doğrudan `ocr` promptu, soru formatındaki promptlardan (`tr_soru`, `en_soru`) her koşulda daha iyi sonuç veriyor.
- Modelin harfleri doğru okuyup Türkçe aksanları atladığı durumlar var; bu fark tüm prompt biçimlerinde gözlemleniyor.
- `ocr` promptunda bazı görsellerde (60 görsel-çözünürlük denemesinden 12'si) model aynı kelime/satırı onlarca kez tekrarlayarak üretim süresine takılıyor — kullanılan eşleşme metriği bu tekrar döngülerini her zaman hata olarak yakalamıyor.

Tüm sayılar ve tartışma için rapora bakın. Bu, 30 görsellik küçük bir deney; genel bir Türkçe OCR başarım iddiası değildir.

## Depo yapısı

```
.
├── notebooks/
│   └── paligemma2_ocr_deney.ipynb   # Colab notebook: model çalıştırma + ölçüm
├── data/
│   ├── veriseti/                    # (depoda yok, bkz. data/README.md)
│   └── results/
│       ├── sonuclar.csv             # modelin ham çıktıları
│       ├── skorlar_parca.csv        # parça bazlı ham skorlar
│       └── ozet.csv                 # tur × çözünürlük × prompt özeti
├── docs/
│   └── PALIGEMMA_2_rapor.docx       # tam analiz ve rapor
├── requirements.txt
└── LICENSE
```

## Çalıştırma

1. Google Colab'da `notebooks/paligemma2_ocr_deney.ipynb` dosyasını açın, çalışma zamanını **T4 GPU** yapın.
2. Hugging Face'te `google/paligemma2-3b-mix-224` ve `google/paligemma2-3b-mix-448` sayfalarındaki lisansı kabul edin.
3. Hugging Face *read* token'ınızı Colab Secrets'a `HF_TOKEN` adıyla ekleyin.
4. `veriseti.zip` dosyasını (görseller + `manifest.csv`, bkz. [`data/README.md`](data/README.md)) Colab'a yükleyin.
5. Hücreleri sırayla çalıştırın. Çıktılar `sonuclar.csv`, `skorlar_parca.csv`, `ozet.csv` olarak kaydedilir.

Yerelde çalıştırmak isterseniz: `pip install -r requirements.txt` yeterli bağımlılıkları kurar; GPU önerilir.

## Veri seti hakkında

Görseller ve `manifest.csv` bu depoya dahil değildir (bkz. [`data/README.md`](data/README.md)). `data/results/` altındaki üç CSV, notebook'un bu veri seti üzerinde üretilen çıktılarıdır ve doğrudan paylaşılabilir.

## Kaynakça

- Beyer, L., Zhai, X., Steiner, A., Wang, X., Tschannen, M., & Houlsby, N. (2024). *PaliGemma 2: A Family of Versatile VLMs for Transfer*. arXiv:2412.03555.
- Gemma Team, Google DeepMind. (2024). *Gemma 2: Improving Open Language Models at Scale*. arXiv:2408.00118.

## Lisans

Kod [MIT Lisansı](LICENSE) ile paylaşılmıştır. Rapor ve deney verileri (`docs/`, `data/results/`) için depo sahibi ayrıca bir lisans belirtmediği sürece tüm hakları saklıdır.
