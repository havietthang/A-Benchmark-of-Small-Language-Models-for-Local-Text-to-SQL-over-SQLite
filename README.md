# A Benchmark of Small Language Models for Local Text-to-SQL over SQLite

This repository contains the benchmark artifact for the paper:

**A Benchmark of Small Language Models for Local Text-to-SQL over SQLite**  
Ha Viet Thang, FPT University, Can Tho, Vietnam

The benchmark evaluates compact instruction-tuned language models for local
natural-language-to-SQL generation over embedded SQLite databases. The study
uses the Spider 1.0 development set, direct SQLite execution, 4-bit NF4
quantization, lightweight SQL verification, and four schema-context/repair
conditions.

## Repository Status

This repository is intended to support reproducibility for the camera-ready
paper. It contains benchmark code, generated figures, generated result tables,
and instructions for preparing the external Spider dataset locally.

Raw Spider data, original SQLite database files, and model weights are **not**
redistributed in this repository.

## Contents

```text
.
├── README.md
├── requirements.txt
├── notebooks/
│   └── APWEB_WAIM_2026_Text_to_SQL_SLMs.ipynb
├── data/
│   └── README.md
├── results/
│   ├── README.md
│   └── generated benchmark result files
├── figures/
│   └── generated paper figures
└── LICENSE
