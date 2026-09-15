# Intro to HPC Bootcamp 2025: Fusion Energy Workloads

My notebooks from the **Introduction to High Performance Computing Bootcamp 2025**, held at Argonne National Laboratory in August 2025 with NERSC. The modules cover:
- how computer hardware consumes power
- what the TOP500 list shows about supercomputer performance and energy efficiency
- a fusion-energy project on workload performance and energy trade-offs at an HPC center

For the Module 5 fusion project, I also built notebooks that compute energy per job, science per kWh and cost from the project's fusion and non-fusion workload tables, and a greedy job scheduler that fits fusion jobs under a **4.8 MW** power cap (40% of the facility's 12 MW).

The module lesson text, exercise prompts, starter code and synthetic datasets are **bootcamp-provided material**. This repo holds my executed copies and my project notebooks. The sections below separate what each module covers from what was run, and quote results only when they appear in the saved outputs.

## Contents

| Notebook | Topic | Techniques | Key libraries |
|---|---|---|---|
| [module_1_power_consumption](module_1_power_consumption.ipynb) | Power consumption in computers and HPC systems | Component power bar charts, donut chart, CO₂ estimate from kWh | matplotlib |
| [module_2_top500_analysis](module_2_top500_analysis.ipynb) | TOP500 (June 2025) analysis | Robust column detection, numeric coercion, performance per kW, accelerator fraction, country and vendor aggregation | pandas, matplotlib, seaborn |
| [module_5_fusion_workload_energy_analysis](module_5_fusion_workload_energy_analysis.ipynb) | Fusion-energy workloads: performance and energy trade-offs | Synthetic job table, pivot tables, annotated and row-normalized heatmaps, planning worksheets | pandas, seaborn, matplotlib, numpy |
| [**fusion_project_metrics**](fusion_project_metrics.ipynb) | Energy, science-per-kWh and cost metrics for fusion and non-fusion project tables | `to_numeric`, derived energy/cost columns, key normalization, de-duplication, outer merge | pandas |
| [fusion_project_metrics_draft](fusion_project_metrics_draft.ipynb) | Earlier version of the metrics notebook | same as above | pandas |
| [**fusion_job_scheduler**](fusion_job_scheduler.ipynb) | Greedy fusion-job scheduler under a 4.8 MW cap | Priority score, cycle-based packing, time-zone-aware timeline, Gantt and step charts, CSV export | pandas, numpy, matplotlib |

---

## Module 1 — Understanding Power Consumption in Computers

### What the module covers
- How CPUs, GPUs, RAM, storage, cooling, power supplies, networking and power distribution each draw power, in personal computers and in HPC systems.
- Dynamic, static and short-circuit power in processors, and the dynamic-power relation **P = C · V² · f**.
- Energy-efficiency strategies for HPC: algorithm optimization, efficient hardware, cooling, and monitoring.
- Several optional written exercises, such as comparing a real HPC GPU like the NVIDIA A100 or AMD MI250X, or looking at how Perlmutter or Frontier handle cooling and power distribution.

### What was run in this copy
All code cells were executed. They are the interactive exercises, left at the example values in the "modify these values" placeholders:

1. **PC component power:** a bar chart of CPU 95 W, GPU 120 W, RAM 40 W, storage 10 W, motherboard 50 W, cooling 20 W and other 15 W, drawn twice (reference chart and "custom PC" exercise).
2. **Configuration comparison:** stacked bars of idle vs. full-load power for Gaming, Workstation and Energy-Efficient configurations.
3. **HPC components:** a bar chart of processors, memory, interconnects, cooling, storage and other.
4. **Typical HPC node:** a donut chart of CPU, GPU, RAM, storage, cooling and PSU shares. The section heading mentions Plotly, but the chart is drawn with `matplotlib`.
5. **Carbon footprint:** one node at 0.5 kW for 24 h, using 0.4 kg CO₂/kWh (U.S. grid average). Saved output: **"Estimated CO₂ emissions per day: 4.80 kg"**.
6. **Efficiency measures:** a bar chart of the percentage reduction from efficient cooling, power scaling, optimized algorithms and other measures.

The optional written exercises are not answered in this copy.

---

## Module 2 — Top 500 Supercomputers Analysis

### What the module covers
- Background on the TOP500 and Green500 lists.
- A reusable loader for the June 2025 TOP500 CSV with three helper functions:
  - `normalize` standardizes column-name text.
  - `find_col` finds a column whose name contains given keywords, so the code keeps working when header names change between releases.
  - `coerce_numeric` turns strings like `"1,234.5"` into floats.
- Exercises on performance per watt, accelerator adoption, geographic distribution and vendor comparisons.
- Discussion prompts, and an extension activity that repeats the analysis on the Green500 list.

### Data
- `data/TOP500_202506.csv`, the June 2025 TOP500 list from [top500.org](https://www.top500.org/). **It is not included.** Download the June 2025 list from top500.org and save it as CSV at that path.
- Columns the loader detected in the saved run: `Name`, `Country`, `Manufacturer`, `Rmax [TFlop/s]`, `Rpeak [TFlop/s]`, `Power (kW)`, `Accelerator/Co-Processor Cores`, `Total Cores`.

### What was run in this copy
1. **Load and clean:** detect key columns, fill missing system names from the site ID, and coerce numeric columns.
2. **Accelerator fraction** (cell 8, "Your Work"):
   - `accel_frac = accelerator cores / total cores`, with a histogram of that fraction.
   - Mean `Rmax / Power(kW)` compared between accelerated (`accel_frac > 0`) and CPU-only systems.
3. **Energy efficiency** (cell 10): `efficiency_perf_per_kw = Rmax / Power`, the top-10 table, and a scatter plot of Rmax vs. power for all systems.
4. **Geography** (cell 12): horizontal bar charts of system count and total Rmax for the top 12 countries.
5. **Top 10 by Rmax** (cell 14): a standalone cell that reloads the CSV, prints the first rows, and draws a `seaborn` bar chart.
6. **Vendors** (cell 18): mean Rmax and mean efficiency per manufacturer, with a bar chart of the top-10 vendors by efficiency.

### Results in the saved outputs
- **Top systems by Rmax (TFlop/s):** El Capitan 1,742,000; Frontier 1,353,000; Aurora 1,012,000; JUPITER Booster 793,400; Eagle 561,200 (no power value).
- **Mean efficiency (Rmax TFlop/s per kW, numerically GFlops/W):** accelerated systems **35.9** vs. non-accelerated **5.32**.
- **Top 10 by Rmax per kW:**

| # | System | Rmax (TFlop/s) | Power (kW) | Rmax per kW |
|---|---|---|---|---|
| 1 | Adastra 2 | 2,529 | 36.60 | 69.10 |
| 2 | JEDI | 4,504 | 67.31 | 66.91 |
| 3 | Henri | 2,882 | 44.07 | 65.40 |
| 4 | Portage | 24,100 | 371.48 | 64.88 |
| 5 | Hunter | 31,680 | 490.00 | 64.65 |
| 6 | Isambard-AI phase 1 | 7,417 | 117.08 | 63.35 |
| 7 | HoreKa-Teal | 3,123 | 49.60 | 62.96 |
| 8 | rzAdams | 24,380 | 388.20 | 62.80 |
| 9 | Frontier TDS | 19,200 | 308.68 | 62.20 |
| 10 | Viper-GPU | 31,096 | 499.98 | 62.19 |

- **Vendors by mean Rmax** (systems with power data), mean Rmax per kW in parentheses: Intel (26.2), Nebius AI (36.7), IBM / NVIDIA / Mellanox (12.7), NRCPC (6.05), HPE (27.7), NUDT (3.32), Fujitsu (17.9), ASUSTeK (43.0), EVIDEN (19.3), Nvidia (28.6).

### Notes
- The notebook cells are slightly out of order. The accelerator-fraction code (cell 8) comes before the Exercise 3 description. The Exercise 4 description was pasted into a **code** cell (cell 15, not executed; it would raise a `SyntaxError` if run). The Exercise 5 description appears twice.
- Cell 14 was run in a separate kernel session (lower execution count) and re-imports everything it needs.
- The discussion prompts and the Green500 extension are not answered in this copy.

---

## Module 5 — Project 1: HPC Workload and Energy Analysis in Fusion Research

### What the module covers
A "thinking project" that applies Modules 1 to 4 to DOE fusion-energy computing. Participants choose one track:

- **Track A — Workflow efficiency:** match three fusion workloads (plasma simulation, diagnostics analysis, ML prediction) to CPU, GPU or hybrid architectures, and reason about scaling and performance per watt. Worksheets A1 and A2 are provided.
- **Track B — Data-center design and scheduling:** design a facility with a **12 MW** power cap that gives **40%** of compute to fusion and prices electricity at **$0.10/kWh**, then plan a weekly schedule. Worksheets B1 and B2 are provided, with sample job classes.

The notebook also gives a hypothetical workload table (runtime, nodes, kW per node, scaling efficiency), formulas for energy per job, science per kWh and cost, background on 2025 leadership systems (El Capitan, Frontier, Aurora, Perlmutter, Jupiter and others), DOE fusion strategy and facilities, AI/ML in fusion, and policy and commercialization, with references.

### What was run in this copy
1. **Synthetic dataset** (cell 12): a provided 20-row `DataFrame` of fusion jobs with `JobID`, `Project` (DIII-D, NSTX-U, NIF, SPARC, ITER Physics Support), `Facility` (Perlmutter, Frontier, El Capitan, Aurora, Jupiter, LUMI, Henri), `Cores`, `RuntimeMinutes`, `JobType` and `Energy_kWh`.
2. **Heatmaps** (cell 13):
   - `pivot_table`s of **mean runtime** and **job count** by project × facility, with columns ordered to match the 2025 systems list.
   - A `seaborn` heatmap annotated as "mean (n)".
   - A second **row-normalized** heatmap that scales each project's runtimes to 0–1 across facilities, to compare relative runtime within a project.

The Track A and B worksheet tables are **not filled in** in this copy. Cells 7 and 18 are empty.

---

## Fusion project notebooks

These notebooks use the Module 5 Track B setting: a **12 MW** facility with **40%** of compute for fusion, priced at **$0.10/kWh**.

### fusion_project_metrics.ipynb
- **Loads** the project tables in [`data/`](data/): `fusion_workload_projects_perf.csv`, `fusion_workload_projects_architecture.csv`, `NERSCProject_nonfusion_hpc_projects.csv` and `NERSCProject_nonfusion_hpc_projects_architecture.csv` (it also reads the two `NERSCProject_hpc_*` summary tables). The tables list synthetic projects by workload, platform, node type, bottleneck, library, framework, precision, nodes, kW per node, runtime and work units.
- **Derives, for each table:**
  - `Power_draw_kW = Nodes × Power_per_Node_kW`, and the same in MW
  - `Energy_per_job_kWh = Runtime_h × Power_draw_kW`
  - `Science_per_kWh = Work_Units / Energy_per_job_kWh`
  - `Cost_per_job_$ = 0.10 × kWh`
  - `Science_per_$`
  - For architecture tables, the node count comes from `Nodes` or `Average Nodes`.
- **Merges** each performance/architecture pair with an outer join on lowercase `Workload` + `Project_Name` keys, after de-duplicating on those keys. Saved shapes: **fusion 476 × 36**, **non-fusion 173 × 36**.
- The last cells are empty.

### fusion_project_metrics_draft.ipynb
An earlier, **near-identical** version of *fusion_project_metrics*. The only difference is that it lacks the final `fusion_merged.head()` display cell.

### fusion_job_scheduler.ipynb
1. Loads `fusion_workload_projects_architecture.csv`, derives the same power, energy, science and cost columns, and classifies each row as CPU, GPU or HYBRID from `Node_Type`/`Platform` text. Saved: **153 of 250 rows kept**, all eligible.
2. **Priority** = 0.6 × (share of total work units) + 0.4 × (`Science_per_kWh` ÷ its maximum).
3. **Greedy cycle scheduler:**
   - In each cycle, walk the unscheduled jobs by priority (then size) and place every job that still fits under a global **4.8 MW** cap.
   - The cycle lasts as long as its longest job.
   - Repeat until all jobs are placed.
4. Gives the cycles calendar times starting 2025-08-15 09:00 America/Chicago. Saved plan: **start 2025-08-15 09:00, end 2025-08-17 05:07 (UTC−5)**.
5. Writes `fusion_arch_with_schedule_4p8MW_simultaneous.csv`, `fusion_schedule_cycles_4p8MW_simultaneous.csv` and `fusion_cycle_summary_4p8MW_simultaneous.csv`. These outputs are generated when you run it and aren't included.
6. Plots a timeline by bucket (hatched horizontal bars) and a step chart of MW in use against the 4.8 MW cap, saved as `my_plot.png`.

---

## Data

[`data/`](data/) holds the bootcamp's synthetic project tables:

| File | Contents |
|---|---|
| `fusion_workload_projects_architecture.csv` | Fusion workloads by platform, project, node type, bottleneck, library, framework, precision, average nodes, memory, kW per node, runtime and work units |
| `fusion_workload_projects_perf.csv` | Performance view of the fusion workloads |
| `NERSCProject_nonfusion_hpc_projects.csv`, `NERSCProject_nonfusion_hpc_projects_architecture.csv` | The same views for non-fusion HPC projects |
| `NERSCProject_hpc_architecture_type_analysis.csv`, `NERSCProject_hpc_programming_language_usage.csv` | Summary tables of architecture types and programming languages |

`fusion_job_scheduler.ipynb` reads `fusion_workload_projects_architecture.csv` from its own folder. Copy that file next to the notebook, or change the path to `data/fusion_workload_projects_architecture.csv`. Module 2 needs `data/TOP500_202506.csv`, which isn't included.

## Requirements

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

The notebooks were saved with a Python 3.11 kernel. They need no GPU or cluster resources, and run on a laptop or a NERSC JupyterHub session.

## How to run

```bash
git clone https://github.com/joserico00/Intro-to-HPC-Bootcamp-2025-Fusion-Energy-Workloads.git
cd Intro-to-HPC-Bootcamp-2025-Fusion-Energy-Workloads
jupyter lab
```

## Related

Other Intro to HPC Bootcamp projects:
- [Intro-to-HPC-Bootcamp-2023-Power-Outages](https://github.com/joserico00/Intro-to-HPC-Bootcamp-2023-Power-Outages)
- [Intro-to-HPC-Modeling-Epidemic-Outbreaks](https://github.com/joserico00/Intro-to-HPC-Modeling-Epidemic-Outbreaks)
- [Intro-to-HPC-Bootcamp-Teaching-Students-to-Leverage-LLMs-for-Regulatory-Genomics-on-HPC](https://github.com/joserico00/Intro-to-HPC-Bootcamp-Teaching-Students-to-Leverage-LLMs-for-Regulatory-Genomics-on-HPC)

## Author

Jose E. Rodriguez Rios
