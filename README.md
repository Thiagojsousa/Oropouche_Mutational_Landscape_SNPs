# Oropouche SNPs Landscape (L / M / S)

[![R](https://img.shields.io/badge/R-%E2%89%A5%204.1-blue)](https://www.r-project.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](#license)

An **R** pipeline to catalogue SNPs (with coding impact) and generate a *publication-ready* **mutational landscape** figure for Oropouche segments **L, M, and S**.

---

## Table of contents
- [Recommended repository layout](#recommended-repository-layout)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [How to run](#how-to-run)
- [Pipeline overview](#pipeline-overview)
- [Plot visual customizations](#plot-visual-customizations)
  - [Font and typography](#font-and-typography)
  - [Colours](#colours)
  - [Legend](#legend)
  - [X-axis: show more tick labels](#x-axis-show-more-tick-labels)
  - [Nonsyn as open circle (red outline)](#nonsyn-as-open-circle-red-outline)
  - [Export (PNG/SVG/PDF)](#export-pngsvgpdf)
- [Troubleshooting](#troubleshooting)
- [License](#license)
- [How to cite](#how-to-cite)

---

## Recommended repository layout

```
.
├── scripts/
│   ├── get_SNPs_effect.R
│   └── plot_snps_landscape_LMS.R
├── data_example/
│   ├── L_aln.fasta
│   ├── M_aln.fasta
│   ├── S_aln.fasta
│   └── (optional) Tabela_codons.csv
├── outputs/
│   ├── LMS_snps_catalogo.csv
│   ├── LMS_snps_catalogo_summary.csv
│   └── figures/
│       ├── Mutational_landscape_L_M_S.png
│       ├── Mutational_landscape_L_M_S.svg
│       └── Mutational_landscape_L_M_S.pdf
└── README.md
```

---

## Inputs

### Required
- Nucleotide **aligned FASTA (MSA)**, one per segment (L/M/S).
  - Must contain **one reference** and **N samples** (e.g., `Seq_01`, `Seq_02`, …).
  - All sequences must have the **same length** (valid alignment).

### Optional
- `Tabela_codons.csv` to override the internal codon dictionary.

---

## Outputs

### Catalogue
- `L_snps_catalogo.csv`, `M_snps_catalogo.csv`, `S_snps_catalogo.csv`
- `LMS_snps_catalogo.csv` (merged L+M+S)
- `LMS_snps_catalogo_summary.csv` (summary by sample/segment/effect)

### Figure
- `Mutational_landscape_L_M_S.png`
- `Mutational_landscape_L_M_S.svg`
- `Mutational_landscape_L_M_S.pdf`

---

## How to run (manual execution in RStudio)

This pipeline is intended for **interactive execution in RStudio**.

### Step 0) Open the scripts
In the RStudio **Files** pane, open:
- `scripts/get_SNPs_effect.R`
- `scripts/plot_snps_landscape_LMS.R`

### Step 1) Edit parameters
In each script, locate and edit the parameter block (e.g., `PARAM` / `PARAM_PLOT`) to set:
- directories (`workdir`, `out_dir_fig`)
- input file names (FASTAs / CSVs)
- reference IDs (`ref_id`), when applicable

### Step 2) Run in RStudio (no `source()`)
Choose one:
- Select the **entire script** and click **Run** (or `Ctrl+A` → `Ctrl+Enter`);
- Run **block-by-block** (recommended): select each block and press `Ctrl+Enter`.

### Execution order
1. Run `get_SNPs_effect.R` (generates `LMS_snps_catalogo.csv` and summary tables)
2. Run `plot_snps_landscape_LMS.R` (exports PNG/SVG/PDF to `outputs/figures/`)

---

## Pipeline overview

1. Load the aligned FASTA and identify the reference sequence.
2. Scan alignment columns and call SNPs vs the reference (ignoring indels/gaps if configured).
3. Convert alignment coordinates → reference coordinates (gap-free reference positions).
4. Translate and classify mutation effects:
   - `synonymous` → **Syn**
   - `nonsynonymous`, `stop_gained`, `stop_lost` → **Nonsyn**
   - otherwise → **Unk**
5. Merge L/M/S and produce the final landscape plot faceted by sample/Node.

---

# Plot visual customizations

The sections below provide snippets to apply in `plot_snps_landscape_LMS.R`.

## Font and typography

### Option A — Portable (no extra dependencies)
```r
theme_bw(base_size = 12, base_family = "sans")
```

### Option B — Consistent font rendering in SVG/PDF (recommended)
```r
if (!require(showtext)) install.packages("showtext")
library(showtext)
font_add_google("Inter", "Inter")
showtext_auto()

p <- p + theme_bw(base_size = 12, base_family = "Inter")
```

---

## Colours

### Mutation type (fixed)
- Syn: `#1f78b4` (blue)
- Nonsyn: `#e31a1c` (red)
- Unk: `grey50`

```r
scale_color_manual(
  name   = "",
  breaks = c("Syn", "Nonsyn", "Unk"),
  values = c(Syn = "#1f78b4", Nonsyn = "#e31a1c", Unk = "grey50"),
  labels = c(
    Syn    = "Synonymous",
    Nonsyn = "Non-synonymous (incl. stop)",
    Unk    = "Unknown/ambiguous"
  )
)
```

---

## Legend

### Order (Type of mutation first; AA class below)
```r
guides(
  color = guide_legend(order = 1, nrow = 1),
  fill  = guide_legend(order = 2, nrow = 1)
) +
theme(
  legend.position = "top",
  legend.box      = "vertical"
)
```

---

## X-axis: show more tick labels

### Fixed step
```r
scale_x_continuous(
  name   = "Amino acid position",
  breaks = seq(0, max(dados_plot$Amino_position, na.rm = TRUE), by = 100)
)
```

### Automatic (pretty breaks)
```r
if (!require(scales)) install.packages("scales")
scale_x_continuous(
  name   = "Amino acid position",
  breaks = scales::pretty_breaks(n = 12)
)
```

---

## Nonsyn as open circle (red outline)

To enforce **Nonsyn = open circle** (`shape = 1`) and keep Syn/Unk filled:

```r
# Syn + Unk filled
geom_point(
  data = subset(dados_plot, Mutation_type != "Nonsyn"),
  aes(color = Mutation_type),
  size = 2
) +
# Nonsyn open circle
geom_point(
  data = subset(dados_plot, Mutation_type == "Nonsyn"),
  aes(color = Mutation_type),
  shape = 1, size = 3, stroke = 1
)
```

---

## Export (PNG/SVG/PDF)

Recommendation:
- **PNG**: 300–600 dpi
- **SVG/PDF**: vector formats (ideal for Inkscape/Illustrator)
- It's likely that some SNPs will overlap, so it's recommended to open the SVG file with Inkscape (https://inkscape.org/) and adjust the positions. 

```r
ggsave("outputs/figures/plot.svg", plot = p, width = 14, height = 8, units = "in")
ggsave("outputs/figures/plot.pdf", plot = p, width = 14, height = 8, units = "in")
ggsave("outputs/figures/plot.png", plot = p, width = 14, height = 8, units = "in", dpi = 600)
```

---

## Troubleshooting

### Reference not found
Ensure `ref_id` **exactly matches** the reference header in the aligned FASTA.

### FASTA is not an alignment
All sequences must have the **same length**.

---

## License

Recommended: **MIT** (permissive and commonly used in research repositories).  
Add a `LICENSE` file at the repository root.

---

## How to cite

Suggestion:

> Thiago Sousa. Oropouche SNP plotting pipeline (L/M/S) (vX.Y.Z) [Computer software]. Orcid: orcid.org/0000-0001-9809-8883. Accessed: YYYY-MM-DD. 
> Thiagojsousa. Oropouche SNP plotting pipeline (L/M/S) [Computer software]. GitHub: Thiagojsousa/Oropouche_Mutational_Landscape_SNPs.git. Accessed: YYYY-MM-DD. Website: thiagojsousa.com.br.
