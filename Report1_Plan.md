# Project Report 1 — Mutation Analysis (BRCA, Group 6)

**Due: Nov 26** · 15% of grade · max 10 pages, Times New Roman 11 pt (code appendix and references don't count toward the limit) Submit **one PDF** on Canvas: the report, then the knitted R Markdown as an appendix.

------------------------------------------------------------------------

## 1. What this milestone is actually asking

The whole project asks one question: **can we find breast-cancer subgroups in TCGA data and link them to clinical outcome and biology?** It has two milestones:

| Milestone          | Data used                                  | Due    |
|--------------------|--------------------------------------------|--------|
| **Report 1 (now)** | **mutations + clinical**                   | Nov 26 |
| Report 2 + video   | expression (RNA-seq), building on Report 1 | Dec 14 |

So for Report 1 you **do not analyze RNA-seq yet**. You only use the expression file to work out which patients have all three data types, so the cohort stays the same in Report 2.

There's no single correct answer. You're graded on: - a clear, logical **workflow** with a flowchart (the instructions call this "the most important part") - **at least 5 investigations**, where quality matters more than quantity - every claim **backed by R code, output and a figure** - biological interpretation and support from the literature

### The three data files (in `dataset/`)

| File | What it is | Key columns |
|----|----|----|
| `data_mutations.txt` | one row per mutation (\~130k rows, 1,009 patients) | `Hugo_Symbol` (gene), `Variant_Classification` (mutation type), `Tumor_Sample_Barcode` (first 12 characters = patient ID) |
| `data_clinical_patient.txt` | one row per patient (1,084) | `PATIENT_ID`, `SUBTYPE` (PAM50: LumA/LumB/Her2/Basal/Normal), `AGE`, `SEX`, `AJCC_PATHOLOGIC_TUMOR_STAGE`, `RACE`, `OS_STATUS`/`OS_MONTHS`, `PFS_*`, `DSS_*`, `DFS_*`, `ICD_O_3_HISTOLOGY` |
| `RNAseq_BRCA.csv` | gene × sample counts | column names are sample barcodes; only used here to intersect patients |

**Patients with all three data types: 1,006.** That's your analysis cohort.

### Mutation types: what "non-synonymous" means here

Mutations in the file break down like this: - **Keep (protein-changing):** Missense_Mutation, Nonsense_Mutation, Frame_Shift_Del, Frame_Shift_Ins, In_Frame_Del, In_Frame_Ins, Splice_Site, Nonstop_Mutation, Translation_Start_Site - **Drop (no protein change):** Silent, Intron, 3'UTR, 5'UTR, 3'Flank, 5'Flank, RNA, Splice_Region

State this filter in the report, since the slides ask about it directly. After filtering, the most frequently mutated genes are **TP53 (347 patients), PIK3CA (347), TTN, CDH1, GATA3, MUC16, KMT2C, MAP3K1**. TTN and MUC16 are very long genes that pick up mutations by chance; mention this as a caveat and focus on known drivers.

------------------------------------------------------------------------

## 2. Suggested report structure and page budget

| Section | Pages | Notes |
|----|----|----|
| Title, group number, names | — | top of page 1 |
| **Key findings** | 0.5 | at most 5 bullets; write these last |
| **Overall analysis workflow + flowchart** | 1.5–2 | most important section; see `Example_Project_1_Report_Flow_Chart.png` |
| **Results**: one subsection per investigation | 5–6 | each one: question → method → figure → finding → 1–2 lines of biology with a citation |
| **Challenges / limitations** | 0.5–0.75 |  |
| **Duties and contributions** | 0.5 | work done so far *and* tasks going forward, including writing |
| References (IEEE) | not counted |  |
| Appendix: knitted Rmd | not counted |  |

------------------------------------------------------------------------

## 3. The investigations (aim for 6 so you have a spare)

Each one follows the slide roadmap: look at the data → mutation summary → correlate with clinical data → clustering.

### Inv. 0 — Data preparation (goes in the workflow section, not counted as an investigation)

- Load the clinical file with `comment.char = "#"` (or `skip = 4`). The first 4 lines are metadata.
- Trim mutation barcodes to 12 characters → `PATIENT_ID`.
- Intersect patients across the 3 files → 1,006 patients.
- Filter to non-synonymous mutations (list above).
- Build a **binary mutation matrix**: patients × genes (1 = mutated). The slides hint at this ("MUT.matrix").
- Clean clinical variables: `OS_STATUS` "1:DECEASED" → 1/0; collapse stage to I/II/III/IV and Early (I–II) vs Late (III–IV); drop the roughly 12 male patients or note them.

### Inv. 1 — Cohort demographics *(covers Lab 9, Action 2)*

- **Boxplot of age** (required by Lab 9) plus a histogram.
- Bar charts: PAM50 subtype, stage, race, sex.
- Overall Kaplan–Meier survival curve for the cohort.
- *What to say:* median age, subtype mix (LumA is the largest group), how few death events there are.

### Inv. 2 — Mutation landscape *(covers Lab 9, Action 3)*

- Bar chart of mutation types (Variant_Classification).
- **Bar plot of non-synonymous mutations per patient**, sorted (required by Lab 9). Expect a long tail of hypermutated tumours.
- Top 20 mutated genes (% of patients). Optional: an oncoplot with `maftools`, or a `pheatmap` of the top genes.

### Inv. 3 — Top driver genes vs PAM50 subtype

- Mutation frequency of TP53, PIK3CA, CDH1, GATA3 and MAP3K1 in each subtype (grouped bar chart).
- Chi-square or Fisher test per gene.
- *Expected (and supported by the TCGA paper):* TP53 is enriched in Basal and Her2, PIK3CA/MAP3K1/GATA3 in LumA. If you see this, it's a strong, well-supported result.

### Inv. 4 — Mutations vs other clinical variables

Choose 2–3 of these: - Mutation burden (number of mutations per patient) vs age (Spearman correlation) and vs stage (Kruskal–Wallis) - TP53 status vs age at diagnosis (Wilcoxon test). TP53-mutant patients tend to be younger. - **CDH1 vs histology**: CDH1 mutations are the hallmark of invasive *lobular* carcinoma (`ICD_O_3_HISTOLOGY` 8520/3) versus ductal (8500/3). Fisher test. Usually a very clean result. - Mutation frequency by race (chi-square), as in Inv. 4 of the example flowchart.

### Inv. 5 — Survival by mutation status

- Kaplan–Meier curves plus log-rank tests: TP53 mutant vs wild-type, PIK3CA mutant vs wild-type, and four groups (TP53 only / PIK3CA only / both / neither).
- Cox regression adjusted for age and stage (`coxph`), reporting hazard ratios.
- BRCA has few OS events, so also try **PFS or DSS**. A non-significant result is fine; report it and explain why.

### Inv. 6 — Clustering of the mutation matrix (the slides ask for this explicitly)

- Use the top \~30–50 most frequently mutated genes as features.
- **Approach A:** hierarchical clustering with a binary/Jaccard distance (`dist(method = "binary")`, Ward.D2), cut into k clusters, shown as a `pheatmap` with subtype annotation.
- **Approach B** (the slides suggest trying two): k-means or PAM on the same matrix, or PCA/MCA followed by k-means.
- Relate the clusters to clinical data: cluster × PAM50 subtype (contingency table plus chi-square), cluster × stage, and Kaplan–Meier by cluster.
- Pathway angle: group genes into pathways (PI3K/AKT: PIK3CA, PTEN, AKT1; p53: TP53; luminal TFs: GATA3, FOXA1; MAPK: MAP3K1, MAP2K4) and compare.

### Optional extras

- Co-occurrence and mutual exclusivity: TP53 vs PIK3CA (2×2 Fisher test). They tend to be mutually exclusive, which reflects the subtypes.
- Logistic regression predicting Late vs Early stage from mutation status plus age, with an ROC curve (as in Inv. 5 of the example).

------------------------------------------------------------------------

## 4. Checklist

### Setup (today)

- [ ] Agree who does which investigation; one member takes the flowchart and writing (see split below)
- [ ] Read the TCGA BRCA paper (Nature 2012, nature11412) and note 5–8 key mutation and clinical findings *(Lab 9 Action 1)*
- [ ] Install packages once in the console, not in the Rmd: `survival`, `survminer`, `pheatmap`, `dplyr`, `tidyr`, `ggplot2` (optional `maftools`)
- [ ] Fix `mutationAnalysis.Rmd`: remove `install.packages(...)` from the chunk, read the clinical file with `comment.char = "#"`, and decide which expression file to use (see the note at the end)

### Analysis (Nov 24–25)

- [ ] Inv. 0 data prep → cohort of 1,006 patients, binary mutation matrix saved as `.rds`
- [ ] Inv. 1 demographics, including the **age boxplot**
- [ ] Inv. 2 mutation landscape, including the **mutations-per-patient bar plot**
- [ ] Inv. 3 drivers vs PAM50 subtype
- [ ] Inv. 4 mutations vs age, histology, stage or race
- [ ] Inv. 5 survival analysis (KM, log-rank, Cox)
- [ ] Inv. 6 clustering with 2 approaches, linked to clinical data
- [ ] Every figure has a title, labelled axes, a legend, and a p-value where a test was run
- [ ] Knit the Rmd to PDF early, since knitting problems take time to fix

### Writing (Nov 25–26)

- [ ] Flowchart (draw.io, Lucidchart or Canva): Load data → filter/merge → one branch per investigation → output (plot or test)
- [ ] Workflow text: for each step, the method, why you chose it, and which variables you used
- [ ] Results subsections, each with a figure, a finding and a citation
- [ ] Challenges section (ideas below)
- [ ] Duties and contributions for each member, both done and upcoming
- [ ] Key findings (≤5 bullets), written last
- [ ] IEEE references
- [ ] Check formatting: Times New Roman 11, ≤10 pages, names and group number on top
- [ ] Merge report PDF and knitted Rmd PDF into one file → upload to Canvas

### Possible points for Challenges

- Few OS death events in BRCA → low statistical power for survival analysis
- Long genes (TTN, MUC16) look "frequently mutated" without being drivers
- A binary mutation matrix is very sparse, so clustering is dominated by TP53 and PIK3CA
- Patient overlap differs between files; some clinical fields are missing (stage, race)
- Multiple testing across many genes → need FDR correction (`p.adjust(method = "BH")`)
- Hypermutated outliers can skew mutation-burden analyses

------------------------------------------------------------------------

## 5. Suggested split for 4 people

| Member | Analysis | Writing |
|----|----|----|
| A | Inv. 0 data prep + Inv. 1 demographics | Flowchart + workflow section |
| B | Inv. 2 landscape + Inv. 3 subtype | Results 2–3, literature/references |
| C | Inv. 4 clinical associations + Inv. 5 survival | Results 4–5 |
| D | Inv. 6 clustering | Results 6, challenges |
| All | — | Key findings, contributions, final proofread and merge |

Member A should share the cleaned objects (`cohort`, `mut_matrix`) as `.rds` files as soon as possible so the others aren't blocked.

------------------------------------------------------------------------

## 6. Results so far (from `mutationAnalysis.pdf`, knitted Nov 24)

Use these as a starting point for Results and Key findings. Each comes from the investigation with the same number in the Rmd.

| \# | Finding | Evidence |
|----|----|----|
| Cohort | 1,006 patients (995 F, 11 M); median age 58; LumA 499, LumB 197, Basal 171, Her2 78, Normal 36; 136 deaths, median follow-up 27 months | data prep, Inv. 1 |
| 1 | Age differs by subtype; OS differs by subtype | Kruskal–Wallis p = 0.007; log-rank p = 0.024 |
| 2 | 85,360 non-synonymous mutations; median 41 per patient (max 5,340, so hypermutators exist). Basal and Her2 carry more mutations than LumA | Kruskal–Wallis p \< 2.2e-16 |
| 3 | PIK3CA (34.4%) and TP53 (34.3%) are the top drivers, followed by CDH1, GATA3, KMT2C, MAP3K1. TP53, PIK3CA, CDH1, GATA3 and MAP3K1 all differ by subtype | Fisher p_adj ≈ 1e-5 for each |
| 3 | TP53 and PIK3CA mutations tend to be **mutually exclusive** | OR = 0.45, p = 4.5e-8 |
| 4 | **CDH1 is mutated in 61% of lobular vs 3% of ductal tumours** | OR = 55, p \< 2.2e-16 |
| 4 | TP53-mutant patients are younger at diagnosis. Burden rises slightly with age and does not differ by stage | Wilcoxon p = 0.007; Spearman rho = 0.10, p = 0.002; KW p = 0.15 |
| 5 | TP53 and PIK3CA are not significant for OS or PFS alone (log-rank p ≥ 0.4), **but TP53 becomes significant after adjusting for age and stage** (likely confounded by age, since TP53-mutant patients are younger) | Cox HR 1.62 (1.12–2.33), p = 0.010; late stage HR 3.08; age HR 1.04 per year |
| 6 | Hierarchical clustering (k = 4) finds a GATA3 cluster (C1, LumA/B), a PIK3CA/CDH1/MAP3K1 cluster (C3, 87% LumA), a TP53-only cluster (C4, 63% Basal) and a large mixed cluster (C2). Clusters relate strongly to subtype, but not to stage or survival | χ² p = 1e-5; stage p = 0.54; log-rank p = 0.9 |

**Challenges already visible in the output:** - Silhouette widths are low (\< 0.2), so mutation-based clusters are weak because the binary matrix is very sparse. - The Cox model breaks the proportional-hazards assumption for stage (cox.zph p = 0.006). - Only 136 deaths, so survival tests have little power. - 86 patients have no mutation in the top 30 genes, so they couldn't be clustered. - Long genes (TTN, MUC16) show up among the top mutated genes.

------------------------------------------------------------------------

## Note on files in this folder

- `dataset/` holds the **course-provided BRCA data**. Use these.
- `RNAseq.csv`, `TCGA-BRCA/` (4.9 GB), `MANIFEST.txt`, `df.rds` and `results.rds` came from re-downloading BRCA RNA-seq yourself with `Instruction_TCGA_RNAseq.Rmd`. They're BRCA, so they were kept, but you don't need them for Report 1, and `dataset/RNAseq_BRCA.csv` already covers expression.
- `TCGA-LIHC/` (liver cancer, 1.7 GB) was moved to the macOS Trash.
