# Turkish Scene Text OCR Experiment with PaliGemma 2

A small-scale experiment investigating how accurately the PaliGemma 2 (3B, `mix`) model reads Turkish scene text (signboards, receipts, packaging, warning signs, etc.), and how resolution (224px / 448px) as well as prompt style (`ocr`, Turkish question, English question) affect the results.

## Research questions

* What is PaliGemma 2's performance in recognizing Turkish scene text (OCR)?
* Is there a difference between 224px and 448px resolution, and if so, how much?
* Does the prompt style (direct `ocr` command vs. question format) change the outcome?
* Does the model correctly identify letters while missing Turkish accents (ş, ğ, ı, ç, ö, ü)?

## Methodology (summary)

* **Model:** `google/paligemma2-3b-mix-224` and `google/paligemma2-3b-mix-448` (Hugging Face, gated — requires access approval)
* **Hardware:** Google Colab, T4 GPU; `float16` (since T4 does not support `bfloat16`; falls back to `float32` if output is empty/corrupted)
* **Dataset:** 30 scene images (signboards, receipts, packaging, warning signs, etc.) under varying font, light/reflection, and degradation conditions; includes hand-written and verified ground-truth text for each image
* **Conditions:** 2 resolutions × 3 prompts = 6 conditions, with a total of 792 text-segment-condition measurements evaluated across 45 primary and 87 secondary text segments
* **Metrics** (defined in `notebooks/paligemma2_ocr_deney.ipynb`):

  * `tam_bulundu`: whether the segment appears verbatim in the model output (ignoring case and punctuation, while preserving İ/I and ı/i distinctions)
  * `aksansiz_bulundu`: same check, but with Turkish accents (ç/ğ/ı/ö/ş/ü → c/g/i/o/s/u) simplified
  * `hata_orani`: character-level edit distance between the segment and its best-matching position in the output, divided by segment length

## Key findings

|                                                | 224px | 448px |
| ---------------------------------------------- | ----: | ----: |
| Primary text, `ocr` prompt, accentless match   | 53.3% | 82.2% |
| Secondary text, `ocr` prompt, accentless match | 23.0% | 65.5% |

* The 448px resolution provides a significant improvement, especially on small/secondary text; however, the effect is not uniform across all 30 images (improvement in 18 images, no change in 11, regression in 1).
* The direct `ocr` prompt performs better across all conditions compared to question-formatted prompts (`tr_soru`, `en_soru`).
* Instances occur where the model identifies characters correctly but misses Turkish accents; this behavior is observed across all prompt styles.
* Under the `ocr` prompt in certain images (12 out of 60 image-resolution trials), the model enters a repetitive loop by outputting the same word/line dozens of times until hitting generation limits. The evaluation metric used does not always capture these repetition loops as errors.

## Hallucination and erroneous text generation

VLM-based OCR models may generate text that is not present in the image. A separate qualitative analysis was performed to identify such cases in the generated outputs.

For example, in `sahne_05_dadas_doner.png`, the reference phone number is:

`0530 610 9806`

At 224px resolution with the `ocr` prompt, the model generated:

`0540 108 4800`

The generated number does not match the reference text and demonstrates an instance where the model produced a different number instead of correctly reading the text in the image.

Repetition behavior was also observed in the same experiment. In one example, the model repeatedly generated the phrase **"Odun Ateşinde Lezzet"** until reaching the generation limit.

These hallucination and repetition observations are treated as additional qualitative error analysis rather than as part of the primary OCR accuracy metrics.

## Repository structure

```text
.
+-- notebooks/
|   +-- paligemma2_ocr_deney.ipynb   # Colab notebook: model execution + evaluation
+-- data/
|   +-- results/
|       +-- sonuclar.csv             # Raw model outputs
|       +-- skorlar_parca.csv        # Segment-level raw scores
|       +-- ozet.csv                 # Type x resolution x prompt summary
|       +-- hallucination.csv        # hallucination results
+-- requirements.txt
+-- manifest.csv
+-- veriseti.zip
+-- LICENSE
+-- CITATION.cff
+-- README.md
```

## How to run

1. Open `notebooks/paligemma2_ocr_deney.ipynb` in Google Colab and set the runtime to T4 GPU.
2. Accept the terms on the `google/paligemma2-3b-mix-224` and `google/paligemma2-3b-mix-448` model pages on Hugging Face.
3. Add your Hugging Face read token to Colab Secrets under the name `HF_TOKEN`.
4. Upload `veriseti.zip` (images + `manifest.csv`) to Colab.
5. Run the cells sequentially. Outputs will be saved as `sonuclar.csv`, `skorlar_parca.csv`, and `ozet.csv`.

To run locally:

```bash
pip install -r requirements.txt
```

GPU is recommended.

## About the dataset

The original image dataset is not included in this repository. The experiment uses 30 scene images together with manually prepared and verified ground-truth annotations.

The three CSV files under `data/results/` contain the model outputs and evaluation results generated from the dataset and can be used to inspect the reported results.

## Limitations

This is a small-scale experiment conducted on 30 images and should not be interpreted as a comprehensive benchmark of PaliGemma 2's Turkish OCR capabilities.

The reported results are specific to the tested model variants, resolutions, prompts, dataset, and evaluation methodology.

In particular, the evaluation focuses on whether reference text segments can be identified in the generated output. This does not fully capture generation quality, hallucination, or repetition-loop behavior; these issues are therefore analyzed separately.

## References

* Steiner, A., Pinto, A. S., Tschannen, M., Keysers, D., Wang, X., Bitton, Y., Gritsenko, A., Minderer, M., Sherbondy, A., Long, S., Qin, S., Ingle, R., Bugliarello, E., Kazemzadeh, S., Mesnard, T., Alabdulmohsin, I., Beyer, L., & Zhai, X. (2024). *PaliGemma 2: A Family of Versatile VLMs for Transfer*. arXiv:2412.03555.

* Gemma Team, Google DeepMind. (2024). *Gemma 2: Improving Open Language Models at a Practical Size*. arXiv:2408.00118.

## License

Code is distributed under the [MIT License](LICENSE).

The experimental results and dataset annotations are provided for research and evaluation purposes. Please refer to the repository documentation and accompanying files for details about the experiment and its limitations.
