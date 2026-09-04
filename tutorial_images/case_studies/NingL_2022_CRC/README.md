# NingL 2022 CRC WGS case study

This example applies the metagenomic SPECTRA workflow to the independent
PRJNA731589 colorectal cancer cohort. It starts from a published MetaPhlAn3
species relative-abundance matrix and does not process raw sequencing reads.

## Cohort

The source cohort contains 163 samples. This example uses the prespecified
normal-BMI range `18.5 <= BMI < 25`, leaving 116 samples:

- 52 colorectal cancer samples (`CL`)
- 64 healthy controls (`HC`)

`0.metadata.csv` retains the project ID, source group, age, sex, BMI, and final
SPECTRA label.

## Inputs

- `SuppTables_241010.xlsx`: published metadata and the 1,685-species
  MetaPhlAn3 relative-abundance matrix.
- `source_sample_metadata_163.csv`: accession-aligned clinical metadata used
  for BMI selection.
- `41564_2021_1030_MOESM3_ESM.xlsx`: supplementary information from the
  biological cohort publication.

The biological cohort is described by Liu et al., *Nature Microbiology*
(2022), DOI: https://doi.org/10.1038/s41564-021-01030-7. The published
MetaPhlAn3 matrix is provided with Wang et al. (2024), DOI:
https://doi.org/10.3390/microorganisms12102086.

## Run the example

Prepare the normal-BMI cohort and standardize WGS species names:

```bash
python prepare_input_data.py \
  --workbook SuppTables_241010.xlsx \
  --bmi-metadata source_sample_metadata_163.csv \
  --skill-dir ../../skills/spectra-wgs-name-converter \
  --project-id PRJNA731589 \
  --output-dir .
```

Run `SPECTRA_process.ipynb` from this directory. The notebook starts from
`1. Relative abundance.csv`, performs CLR, and writes the CLR, MRI, and final
probability tables.

## Feature preparation and CLR

The source matrix contains 1,685 species. The WGS Skill maps 1,270 species to
the SPECTRA MetaPhlAn3 names and retains 415 unmatched source species. Adding
284 missing reference features produces a 1,969-feature matrix.

We recommend closing each sample to a relative composition, replacing zeros
with multiplicative replacement, and then applying CLR:

```python
from skbio.stats.composition import clr, multi_replace

abun_clr = clr(multi_replace(relative_abundance))
```

This is the CLR procedure used by the xMICARE workflow. Apply name conversion,
collision handling, and reference-feature completion before CLR so that the
log-ratio basis is defined on the final feature table.

## Results

Metrics are displayed in the notebook and are not exported as separate metric
files.

| Metric | Result |
|---|---:|
| SPECTRA CRC AUC | 0.7115 |
| CRC Top1 | 42.31% |
| CRC Top2 | 71.15% |
| CRC Top3 | 80.77% |

## Main files

- `prepare_input_data.py`: extracts the project, selects normal BMI, converts
  species names, and writes metadata and relative abundance.
- `1. Microbial features.csv`: source species-name list.
- `1. Microbial feature 2 MSig by Skill.csv`: source-to-model name mapping.
- `1. Relative abundance.csv`: converted and completed relative abundance.
- `SPECTRA_process.ipynb`: CLR, prediction, and evaluation.
- `2. abun_clr.csv`, `3. MRI score.csv`, and `4. Probability.csv`: numerical
  outputs used in the notebook.
