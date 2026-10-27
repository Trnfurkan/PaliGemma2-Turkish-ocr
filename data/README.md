# Dataset

The `veriseti/` folder (scene images + `manifest.csv`) is not included in this repository. The images may contain recognizable real places, products, or signs, and they are also kept separate because of their size.

To run the notebook, `veriseti.zip` should have the following structure:

```
veriseti/
├── manifest.csv
├── ana/
│   └── orijinal/   (plus optional 224/ and 448/ letterbox folders)
└── ...             (other groups defined in GROUPS)
```

**Paths used by the notebook (Colab):**

- The archive is read from `/content/veriseti/veriseti.zip` and extracted into `veriseti/`.
- The manifest is read from `veriseti/manifest.csv`.
- Images are read from `veriseti/veriseti/<grup>/orijinal/<dosya>`.

If your folder layout differs, update the `image_path` function and the extraction cell in the notebook accordingly.

Expected columns in `manifest.csv`:

| Column | Description |
|---|---|
| `dosya` | Image file name |
| `grup` | Image group (e.g. `ana`) |
| `ana_metin` | Primary text segments (ground truth), separated by `\|` |
| `ikincil_metin` | Secondary (small/minor) text segments, separated by `\|` |
| `dogrulandi` | (optional) used to keep only verified rows |

Only the 30 images in the `ana` group were used in this experiment (`GROUPS = ["ana"]` in the notebook). `manifest.csv` also lists `taninmis_marka` and `kontrol` rows, which were not evaluated.

## `results/`: outputs included in this repository

The four CSV files in this folder were produced by the notebook on the 30-image dataset and can be inspected or analyzed without the dataset:

- **`sonuclar.csv`**: raw model outputs. Columns: `cozunurluk` (resolution, 224/448), `prompt` (`ocr`/`tr_soru`/`en_soru`), `dosya` (file), `grup` (group), `cikti` (model output), `sure_sn` (generation time, seconds).
- **`skorlar_parca.csv`**: segment-level raw scores. Columns: `cozunurluk`, `prompt`, `dosya`, `tur` (`ana` = primary / `ikincil` = secondary), `parca` (ground-truth text segment), `tam_bulundu` (exact match), `aksansiz_bulundu` (accent-insensitive match), `hata_orani` (error rate).
- **`ozet.csv`**: summary statistics by `tur` × `cozunurluk` × `prompt` (`parca_sayisi` = number of segments, and the means of `tam_bulundu`, `aksansiz_bulundu`, `ort_hata_orani`).
- **`hallucination.csv`**: a separate qualitative labeling of hallucinations. Columns: `dosya`, `cozunurluk`, `aday_kelime` (candidate word/number not found in the reference), `cikti_satiri` (output line containing the word), `etiket` (label), `etiket_taslak` (draft label, e.g. `U`: fabricated, `G`: genuine text present in the image but missing from the reference, `?`: uncertain).

For metric definitions, see the main [`README.md`](../README.md) and the `score` function in the notebook.
