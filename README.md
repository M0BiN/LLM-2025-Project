# LLMs for Cyberattack Detection on UNSW-NB15

*A Multi-Agent Network Threat Analysis Framework via LLM-Based Log Understanding*
University of Tehran — Large Language Models (Spring 2025)
**Authors:** Mobin Tirafkan · Kiyan Khezri · Mohammad Mehrabian&#x20;

---

## 🔎 Overview

Security logs are cryptic and high-volume; off-the-shelf LLMs struggle on raw tabular flows (semantic gap), are costly to deploy at scale, and lack explainability. We bridge this by converting each flow into a human-readable **story** and training a compact LLM to produce an **explainable verdict** (label + reason).&#x20;

**What we built**

* **Two-agent pipeline:**

  1. **Rule-Based Storyteller** — turns a log row into a contextual narrative using a normal-traffic baseline;
  2. **LLM Reasoner** — a fine-tuned small model that outputs a label *and* a concise, evidence-based reason.&#x20;
* **Coverage-aware sampling:** cluster story embeddings and train on \~**3%** (\~4k) representative samples.&#x20;
* **Teacher→Student distillation:** generate Story→Reason pairs with a strong teacher LLM, then fine-tune **Gemma-3 1B** as the student.&#x20;

---

## ✨ Key Results (UNSW-NB15, binary)

* **Balanced Accuracy:** **93.0%** · **F1:** **93.0%** (test **n=1,000**).
  Errors are balanced (FP=34, FN=36), yielding **Precision 93.2%** and **Recall 92.8%**.&#x20;
* **ICL baselines (Gemma-3 4B IT):** Raw-log prompting ≈ **50% accuracy** (chance); *Story* zero-shot **F1 64.4%**; *Story* few-shot **F1 75.2%**.&#x20;
* **By attack type:** near-perfect on **Backdoor**, **DoS**, **Reconnaissance**; weaker on **Shellcode** (long-tail).&#x20;

---

## 🧠 Method

### 1) Agent-1: Rule-Based Storyteller

* Select 21 salient features from UNSW-NB15’s 49; compute “Usual” baselines from normal traffic.
* For each record, compare to the baseline and produce qualitative phrases (e.g., “\~+124% vs the Usual”, “rare protocol”).
* Concatenate into a compact, readable **story** purpose-built for LLM reasoning.&#x20;

### 2) Efficient Data Curation (\~3%)

* Embed stories (e.g., `all-mpnet-base-v2`), cluster, and **sample proportionally** with a min per cluster to preserve diversity and label balance (\~4,000 stories total).&#x20;

### 3) Agent-2: LLM Reasoner (Student)

* **Base model:** Gemma-3 **1B**.
* **Supervision:** Story → (Reason, Label) pairs authored by the **Teacher LLM**.
* **Training stack:** PyTorch · HF Transformers/TRL · **Unsloth** for memory-efficient finetuning (consumer GPU).
* **Hyperparams (final run):** LR **5e-4**, **7** epochs, batch size **4**.&#x20;

---

## 📦 Dataset

* **UNSW-NB15** (flows; 49 features). We work in **binary** setting (*attack* vs *normal*).
* Storyteller uses normal-only statistics to contextualize deviations; examples of the raw→story→verdict transform are in the report.&#x20;



---

## 📊 Reproduced Metrics (machine-readable)

```json
{
  "dataset": "UNSW-NB15",
  "task": "binary_attack_vs_normal",
  "train_fraction": 0.03,
  "test_n": 1000,
  "model": "Gemma-3 1B (finetuned)",
  "hyperparams": {"lr": 5e-4, "epochs": 7, "batch_size": 4},
  "metrics": {"accuracy_balanced": 93.0, "accuracy": 93.0, "precision": 93.2, "recall": 92.8, "f1": 93.0},
  "errors": {"false_positives": 34, "false_negatives": 36}
}
```

(See **`report.pdf`** for baseline ICL results and per-attack analysis.)&#x20;

---

## 🧪 Notes & Limitations

* ICL with *raw logs* fails (≈50% accuracy), highlighting the semantic gap; the **Story** representation is essential.&#x20;
* Our model underperforms on rare/complex **Shellcode** cases; targeted augmentation is a next step.&#x20;
* A planned third **Action-Proposer** agent was **descoped** due to training/evaluation challenges (lack of reliable teacher and benchmarks).&#x20;

---

## 📚 Citation

If you use this project, please cite the report:

```
@report{tirafkan2025logs2verdict,
  title   = {From Logs to Explainable Verdict: A Multi-Agent Network Threat Analysis Framework via LLM-Based Log Understanding},
  author  = {Mobin Tirafkan and Kiyan Khezri and Mohammad Mehrabian},
  institution = {University of Tehran},
  year    = {2025},
  url     = {reports/report.pdf}
}
```

---

## 🙏 Acknowledgements

Course: **Large Language Models (Spring 2025), University of Tehran**. We thank our instructors and peers for feedback and compute support.&#x20;

---

## License

Code is released under MIT. Datasets and base models follow their respective licenses/terms.
