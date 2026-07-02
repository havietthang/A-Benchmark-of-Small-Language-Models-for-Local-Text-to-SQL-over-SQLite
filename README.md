# A Benchmark of Small Language Models for Local Text-to-SQL over SQLite

This repository contains the code, notebook, generated results, and figure/table artifacts for the paper:

**A Benchmark of Small Language Models for Local Text-to-SQL over SQLite**  
Ha Viet Thang, FPT University, Can Tho, Vietnam

The paper evaluates compact instruction-tuned language models for local natural-language-to-SQL generation over embedded SQLite databases. The benchmark uses the Spider 1.0 development set, direct SQLite execution, 4-bit NF4 quantization, lightweight SQL verification, and four schema-context/repair conditions.

> **Important:** This repository does **not** redistribute the Spider 1.0 dataset, Spider question text, gold SQL, or original SQLite database files. Users should download Spider 1.0 from its official source and place it locally following the layout below.

---

## Repository contents

```text
local-text-to-sql-sqlite-slm-benchmark/
├── README.md
├── requirements.txt
├── notebooks/
│   └── APWEB_WAIM_2026_Text_to_SQL_SLMs.ipynb
├── data/
│   └── README.md
├── results/
│   ├── all_models_main_metrics.csv
│   ├── all_models_error_breakdown.csv
│   ├── all_models_analytic_subset_metrics.csv
│   ├── resource_aware_rank.csv
│   ├── rank_by_execution_accuracy.csv
│   ├── rank_by_executable_query_rate.csv
│   ├── rank_by_median_generation_latency.csv
│   ├── rank_by_model_load_memory.csv
│   └── model_file_status.csv
└── figures/
    ├── fig1_pipeline.png
    ├── fig3_execution_accuracy_by_condition.png
    ├── fig4_top5_accuracy_movement.png
    ├── fig5_accuracy_latency_tradeoff.png
    ├── fig6_accuracy_memory_tradeoff.png
    └── fig7_repair_gain.png
```

If your local filenames differ, update the figure paths in this README accordingly.

---

## Data setup

This benchmark uses the **Spider 1.0 development set** and the original Spider SQLite database files.

The expected local layout is:

```text
data/spider/
├── dev.json
├── tables.json
└── database/
```

The repository does not include these Spider files. To reproduce the experiments, download Spider 1.0 from the official release and place the files in the layout above.

Generated benchmark outputs are stored in `results/`. These are generated artifacts from the benchmark run, not a redistribution of the original Spider dataset.

---

## Code and notebook

The main benchmark notebook is:

```text
notebooks/APWEB_WAIM_2026_Text_to_SQL_SLMs.ipynb
```

The notebook covers:

- Spider development-set loading
- schema-context construction
- full, pruned, and augmented schema conditions
- prompt construction
- local model inference
- SQL extraction
- lightweight SQL verification
- SQLite execution
- execution-accuracy computation
- executable-query-rate computation
- latency and GPU-memory logging
- repair-pass evaluation
- analytical-subset evaluation
- result-table generation

---

## Models evaluated

The benchmark evaluates ten compact instruction-tuned models across six model families.

| Model | Family / source | Notes |
|---|---|---|
| `Qwen3-0.6B` | Qwen | Compact baseline |
| `Qwen3-1.7B` | Qwen | Small Qwen model |
| `Qwen3-4B-Instruct-2507` | Qwen | Best overall resource-aware tradeoff in this benchmark |
| `Gemma-4-E2B-it` | Gemma | Strong accuracy with moderate resource cost |
| `Gemma-4-E4B-it` | Gemma | Highest raw execution accuracy in this benchmark |
| `Llama-3.2-1B-Instruct` | Llama | Small Llama baseline |
| `Llama-3.2-3B-Instruct` | Llama | Stronger Llama baseline |
| `Granite-3.3-2B-Instruct` | Granite | Compact IBM Granite model |
| `SmolLM3-3B` | SmolLM | Compact open model |
| `Phi-4-mini-instruct` | Phi | Microsoft Phi mini model |

Model weights are **not** redistributed in this repository. The notebook lists the model identifiers and loads them from their original hosting locations.

---

## Experimental conditions

| Condition | Schema context | Repair |
|---|---|---|
| A | Full schema | Disabled |
| B | Conservative pruned schema | Disabled |
| C | Conservative pruned + augmented schema | Disabled |
| D | Conservative pruned + augmented schema | Enabled |

Condition D differs from Condition C only by allowing one verification-triggered repair attempt. Executable but semantically incorrect SQL does not trigger repair.

---

## Figures

### Figure 1. Local NL-to-SQL execution pipeline

![Local NL-to-SQL execution pipeline](figures/fig1_pipeline.png)

The benchmark builds a condition-specific schema context, constructs a fixed prompt, runs a compact language model locally, extracts SQL, verifies the SQL, executes it in SQLite, and logs correctness/resource metrics. Repair is enabled only in Condition D and only after verification failure.

### Figure 3. Execution accuracy by model and condition

![Execution accuracy by model and condition](figures/fig3_execution_accuracy_by_condition.png)

This figure compares execution accuracy across all ten models and four experimental conditions.

### Figure 4. Accuracy movement for the top five Condition-D models

![Accuracy movement for top five models](figures/fig4_top5_accuracy_movement.png)

This figure highlights how the strongest Condition-D models move across the four schema-context and repair conditions.

### Figure 5. Accuracy-latency tradeoff under Condition D

![Accuracy-latency tradeoff](figures/fig5_accuracy_latency_tradeoff.png)

This figure compares execution accuracy against median generation latency. The connected line marks the nondominated Pareto frontier.

### Figure 6. Accuracy-memory tradeoff under Condition D

![Accuracy-memory tradeoff](figures/fig6_accuracy_memory_tradeoff.png)

This figure compares execution accuracy against model-load GPU reserved memory delta. It shows that the most accurate model is not necessarily the strongest local-systems tradeoff.

### Figure 7. Accuracy gain from repair

![Repair accuracy gain](figures/fig7_repair_gain.png)

This figure visualizes the execution-accuracy gain from Condition C to Condition D.

---

## Main result tables

### Table 1. Experimental conditions

| Condition | Schema context | Repair |
|---|---|---|
| A | Full schema | Disabled |
| B | Conservative pruned schema | Disabled |
| C | Conservative pruned + augmented schema | Disabled |
| D | Conservative pruned + augmented schema | Enabled |

### Table 2. Schema-context and prompt-size statistics

| Cond. | Median schema chars | P95 schema chars | Median prompt chars | Mean comp. | Gold table recall | Fallback |
|---|---:|---:|---:|---:|---:|---:|
| A | 455 | 1864 | 725 | 1.00 | 100.0% | 0.0% |
| B | 423 | 1282 | 690 | 0.89 | 97.9% | 22.1% |
| C | 1018 | 3081 | 1310 | 2.19 | 97.9% | 22.1% |
| D | 1018 | 3081 | 1310 | 2.19 | 97.9% | 22.1% |

### Table 3. Main result summary sorted by Condition-D accuracy

Accuracy and executable rates are percentages. Latency is in seconds. GPU load memory is model-load GPU reserved memory delta in MB.

| Model | Acc. A | Acc. B | Acc. C | Acc. D | Exec. D | Lat. D | Mem. D |
|---|---:|---:|---:|---:|---:|---:|---:|
| Gemma-4-E4B-it | 52.52 | 51.45 | 50.78 | 52.81 | 95.55 | 7.53 | 8620 |
| Gemma-4-E2B-it | 41.14 | 40.41 | 44.81 | 47.43 | 95.84 | 7.23 | 6146 |
| Qwen3-4B-Instruct-2507 | 45.06 | 45.06 | 47.19 | 47.38 | 97.97 | 4.85 | 2228 |
| Llama-3.2-3B-Instruct | 30.04 | 30.72 | 33.91 | 35.17 | 87.23 | 3.00 | 1814 |
| Qwen3-1.7B | 31.55 | 31.36 | 31.29 | 31.68 | 87.52 | 2.61 | 2754 |
| Phi-4-mini-instruct | 28.56 | 28.65 | 29.69 | 30.23 | 92.36 | 4.08 | 2558 |
| Granite-3.3-2B-Instruct | 27.30 | 26.72 | 28.85 | 29.53 | 84.91 | 5.16 | 1058 |
| Llama-3.2-1B-Instruct | 16.36 | 16.07 | 15.20 | 16.17 | 58.90 | 1.91 | 664 |
| Qwen3-0.6B | 13.87 | 13.95 | 13.46 | 13.46 | 73.31 | 1.52 | 1044 |
| SmolLM3-3B | 10.64 | 11.80 | 10.08 | 12.40 | 35.01 | 22.31 | 1590 |

### Table 4. Repair effect from Condition C to Condition D

Accuracy and executable rates are percentages.

| Model | Acc. C | Acc. D | Gain | Exec. C | Exec. D | Gain | Repair % | Rep. lat. |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Gemma-4-E4B-it | 50.78 | 52.81 | 2.03 | 92.36 | 95.55 | 3.19 | 7.45 | 0.98 |
| Gemma-4-E2B-it | 44.81 | 47.43 | 2.62 | 89.85 | 95.84 | 6.00 | 9.86 | 0.87 |
| Qwen3-4B-Instruct-2507 | 47.19 | 47.38 | 0.19 | 97.10 | 97.97 | 0.87 | 2.71 | 0.20 |
| Llama-3.2-3B-Instruct | 33.91 | 35.17 | 1.26 | 81.33 | 87.23 | 5.90 | 18.47 | 0.84 |
| Qwen3-1.7B | 31.29 | 31.68 | 0.39 | 84.91 | 87.52 | 2.61 | 14.60 | 0.78 |
| Phi-4-mini-instruct | 29.69 | 30.23 | 0.54 | 90.23 | 92.36 | 2.13 | 9.57 | 1.15 |
| Granite-3.3-2B-Instruct | 28.85 | 29.53 | 0.68 | 80.95 | 84.91 | 3.97 | 18.96 | 1.78 |
| Llama-3.2-1B-Instruct | 15.20 | 16.17 | 0.97 | 52.03 | 58.90 | 6.87 | 47.87 | 1.23 |
| Qwen3-0.6B | 13.46 | 13.46 | 0.00 | 72.15 | 73.31 | 1.16 | 27.76 | 1.88 |
| SmolLM3-3B | 10.08 | 12.40 | 2.33 | 26.60 | 35.01 | 8.41 | 73.21 | 15.62 |

### Table 5. Analytical subset comparison under Condition D

Accuracy and executable rates are percentages.

| Model | Main acc. | Analytic acc. | Drop | Main exec. | Analytic exec. |
|---|---:|---:|---:|---:|---:|
| Gemma-4-E4B-it | 52.81 | 47.43 | 5.38 | 95.55 | 94.75 |
| Gemma-4-E2B-it | 47.43 | 43.02 | 4.41 | 95.84 | 95.44 |
| Qwen3-4B-Instruct-2507 | 47.38 | 40.57 | 6.81 | 97.97 | 97.61 |
| Llama-3.2-3B-Instruct | 35.17 | 28.00 | 7.17 | 87.23 | 85.97 |
| Qwen3-1.7B | 31.68 | 24.71 | 6.97 | 87.52 | 86.20 |
| Phi-4-mini-instruct | 30.23 | 21.37 | 8.86 | 92.36 | 91.45 |
| Granite-3.3-2B-Instruct | 29.53 | 23.29 | 6.24 | 84.91 | 83.58 |
| Llama-3.2-1B-Instruct | 16.17 | 12.56 | 3.61 | 58.90 | 56.78 |
| Qwen3-0.6B | 13.46 | 7.08 | 6.38 | 73.31 | 70.13 |
| SmolLM3-3B | 12.40 | 6.74 | 5.66 | 35.01 | 32.50 |

---

## Generated result files

The following generated result files are expected in `results/`.

| File | Description |
|---|---|
| `all_models_main_metrics.csv` | Main model-level metrics across all four conditions |
| `all_models_error_breakdown.csv` | SQL verification and execution failure summaries |
| `all_models_analytic_subset_metrics.csv` | Metrics on the analytical subset of structurally harder SQL examples |
| `resource_aware_rank.csv` | Accuracy-first resource-aware ranking based on accuracy, executability, latency, and memory |
| `rank_by_execution_accuracy.csv` | Ranking by execution accuracy |
| `rank_by_executable_query_rate.csv` | Ranking by executable-query rate |
| `rank_by_median_generation_latency.csv` | Ranking by median generation latency |
| `rank_by_model_load_memory.csv` | Ranking by model-load GPU memory delta |
| `model_file_status.csv` | Completion/status tracking for model output files |

Large generated output files, if included, should contain only benchmark-generated outputs. Do not include raw Spider data, full Spider question text, gold SQL, or original SQLite database contents unless redistribution rights are explicitly confirmed.

---

## Reproducing the benchmark

A typical reproduction workflow is:

1. Clone this repository.
2. Install the Python dependencies listed in `requirements.txt`.
3. Download Spider 1.0 from the official source.
4. Place Spider files under `data/spider/` using the layout shown above.
5. Open the benchmark notebook in Google Colab or a local Jupyter environment.
6. Run the notebook cells to generate model outputs, metrics, tables, and figures.

Example setup:

```bash
git clone https://github.com/YOUR_USERNAME/local-text-to-sql-sqlite-slm-benchmark.git
cd local-text-to-sql-sqlite-slm-benchmark
pip install -r requirements.txt
```

Then place Spider locally:

```text
data/spider/dev.json
data/spider/tables.json
data/spider/database/
```

---

## Security and credential note

Before committing the notebook, make sure it does not contain:

- Hugging Face tokens
- API keys
- Kaggle credentials
- Google Drive paths containing private information
- private Colab secrets

Use environment variables or Colab Secrets instead of hard-coded credentials.

---

## Data and code availability statement

The generated benchmark outputs, including aggregate model-level metrics, repair statistics, analytical-subset results, resource-aware score tables, and figure source data, are available in this repository and/or the linked Kaggle artifact. The raw Spider 1.0 dataset and its original SQLite database files are not redistributed in this artifact; users should obtain them from the official Spider release. The accompanying benchmark notebook provides the code for prompt construction, schema-context generation, SQL verification, SQLite execution, metric computation, and figure/table generation. Model weights are not redistributed; the artifact documents the model identifiers and software versions used in the experiments.

---

## License

The code in this repository is released under the license specified in `LICENSE`.

Generated benchmark result tables and figures may be released under a separate data license, such as CC BY 4.0, if desired.

---

## Citation

If you use this benchmark code or generated results, please cite the paper:

```bibtex
@inproceedings{ha2026localtexttosql,
  title     = {A Benchmark of Small Language Models for Local Text-to-SQL over SQLite},
  author    = {Ha Viet Thang},
  booktitle = {Proceedings of APWeb-WAIM 2026 Workshops},
  year      = {2026},
  note      = {To appear}
}
```

