I want you to recreate my frozen RPKM-only circadian temporal-shape assessment pipeline exactly.

Pipeline goal:
Build an RPKM/count-matrix-only pipeline for circadian ChIP-seq temporal binding analysis. Ignore BED/UpSet files completely. The pipeline should read the RPKM/count matrix, average ZT replicate columns, row-wise z-score each peak across timepoints, plot all peaks before grouping, then perform TopZT-constrained temporal-shape clustering and generate heatmaps, profiles, UMAP/PCA embeddings, and clustering diagnostics.

Input:
- One RPKM/count matrix with a PeakID-like column.
- Timepoints are ordered:
  ZT1 ZT4 ZT7 ZT11 ZT13 ZT16 ZT19 ZT22
- The matrix may contain replicate columns like ZT4_1, ZT4_2, ZT4_3, or HOMER-style columns like “ZT4 FPKM”.
- Average replicates per ZT before analysis.

Core biological rule:
Peaks with different peak TopZT must be treated as different biological groups. Therefore, the default grouping mode must be:
PeakTopZT first, then shape clustering within each PeakTopZT.

Pipeline files to create:
1. rpkm_temporal_shape_pipeline.py
2. run_rpkm_temporal_shape_pipeline.sh
3. README.md
4. Package everything into rpkm_temporal_shape_assessment.zip

Short output directory naming:
Use a short output directory name:
<run-label>_<timestamp>
For example:
whole_project_shape_assess_20260709_181500

Do not use long parameter-stamped folder names. Full parameters must still be saved to:
00_parameters/parameters.json

Required output folders:
00_parameters/
01_all_peaks/
02_shape_groups/
03_embeddings/
04_clustering_diagnostics/
05_tables/

All-peak pre-grouping outputs:
Before clustering, plot the whole dataset:
- whole_zscore_heatmap_all_peaks.pdf/png
- whole_raw_log1p_heatmap_all_peaks.pdf/png
- whole_zscore_profile_all_peaks.pdf/png
- whole_raw_profile_all_peaks.pdf/png

Whole heatmap ordering:
Rows should be sorted by peak-level PeakTopZT using the supplied biological timepoint order, then PeakID.

Peak metrics:
For each peak compute:
- RawTopZT
- ZTopZT
- PeakTopZT = ZTopZT by default
- RawAmplitude
- ZAmplitude
- RawAmplitudeCategory
- ZAmplitudeCategory

Clustering modes:
Support:
--grouping-mode topzt_then_shape
--grouping-mode global_shape

Support clustering methods:
--clustering-method kmeans
--clustering-method hierarchical

Default final grouping should be topzt_then_shape.

Within each PeakTopZT stratum:
- Test k values from --shape-k-range, usually 1 to 4.
- If --shape-k auto, select k per ZT.
- If fixed --shape-k is supplied, use that k where possible.
- Respect --min-peaks-per-shape-cluster so small ZT strata are not over-split.
- Use k=1 when a stratum is too small or does not pass splitting criteria.

Auto-k decision:
Support:
--auto-k-rule conservative
Use conservative criteria:
1. minimum cluster size satisfied
2. silhouette >= --min-silhouette
3. within-ZT centroid correlations are not too high
4. choose the smallest acceptable k unless a larger k is clearly better

Support options:
--shape-k auto
--shape-k-range 1 4
--diagnostic-k-values 1 2 3 4
--min-peaks-per-shape-cluster 50
--min-silhouette 0.15
--max-withinzt-centroid-corr 0.90
--max-silhouette-sample 10000
--shape-random-state 42
--shape-n-init 50

K-means:
Use sklearn KMeans on row-wise z-score profiles.
Use n_init from --shape-n-init.

Hierarchical:
Use correlation distance, distance = 1 - Pearson correlation.
Use average linkage.
Allow cutting into k clusters.

Shape group labels:
For TopZT-constrained mode, final labels should be:
ZT4_Shape01
ZT4_Shape02
ZT7_Shape01
...
ShapeSubtype should be:
Shape01, Shape02, etc.
TopZTGroup should be:
ZT1, ZT4, ZT7, etc.
ShapeGroup should combine TopZT and subtype:
ZT4_Shape01.

Group ordering:
Shape groups should be ordered by:
1. PeakTopZT biological order: ZT1, ZT4, ZT7, ZT11, ZT13, ZT16, ZT19, ZT22
2. Shape subtype order within that ZT

Grouped heatmaps:
Create grouped heatmaps for:
- z-score
- raw log1p
For each heatmap, group blocks are ordered by group mean peak ZT.
Rows within each group are sorted by peak-level PeakTopZT, then PeakID.
Draw horizontal boundaries between groups.
Label groups with group name, group peak ZT, and peak count.

Heatmap interpolation:
Support:
--heatmap-interpolations default none
If multiple values are supplied, write one heatmap per interpolation.
Use filenames such as:
shape_group_zscore_peak_heatmap__interp-default.pdf/png
shape_group_zscore_peak_heatmap__interp-none.pdf/png

Profiles:
Create:
- shape_group_zscore_profiles.pdf/png
- shape_group_raw_profiles.pdf/png
- shape_group_zscore_profiles_with_individual_traces.pdf/png
- shape_group_raw_profiles_with_individual_traces.pdf/png

Profile panels:
Groups ordered by biological peak ZT.
Each panel shows mean profile and SEM or SD band.
The individual-trace version should show faint sampled individual peak traces plus thick group mean.

Clustering diagnostics:
For each diagnostic method requested by:
--diagnostic-methods kmeans
--diagnostic-methods hierarchical
--diagnostic-methods both

and for each k in:
--diagnostic-k-values 1 2 3 4

Within each PeakTopZT, cluster into k where possible and compute cluster centroids.

For each k and method, create all-centroid Pearson correlation matrices:
- centroid_correlation_kmeans_k2.tsv/pdf/png
- centroid_correlation_kmeans_k3.tsv/pdf/png
- centroid_correlation_kmeans_k4.tsv/pdf/png
- centroid_correlation_hierarchical_k2.tsv/pdf/png
- centroid_correlation_hierarchical_k3.tsv/pdf/png
- centroid_correlation_hierarchical_k4.tsv/pdf/png

For 8 ZTs and k=4, this gives up to 32 centroids and a 32 x 32 matrix.

Also write centroid pair tables:
- centroid_pairs_kmeans_k4.tsv
- centroid_pairs_hierarchical_k4.tsv

Centroid labels should include:
ZT, method, k, cluster number, and n peaks.

Optional phase-aligned centroid correlation:
Support:
--write-phase-aligned-centroid-corr
When enabled, shift each centroid so its maximum is aligned before computing Pearson correlation. Write phase-aligned centroid correlation matrices alongside the standard same-coordinate centroid correlations.

Embedding:
Support:
--embedding-method umap pca tsne
UMAP should fall back to PCA if umap-learn is unavailable.
Support:
--embedding-input zscore raw_log1p_scaled both

For each embedding input, write embedding tables:
03_embeddings/peak_embedding_<input>_<method>.tsv

UMAP/PCA plots:
I want regular embedding plots and a 3 x 3 panel/color grid based on:
ShapeSubtype
ShapeGroup
PeakTopZT

Support:
--embedding-columns ShapeSubtype ShapeGroup PeakTopZT
--embedding-panel-max-groups 80

For each embedding input, create:
Regular color plots:
- color-ShapeSubtype
- color-ShapeGroup
- color-PeakTopZT

Panel/color combinations:
- panel-ShapeSubtype_color-ShapeSubtype
- panel-ShapeSubtype_color-ShapeGroup
- panel-ShapeSubtype_color-PeakTopZT
- panel-ShapeGroup_color-ShapeSubtype
- panel-ShapeGroup_color-ShapeGroup
- panel-ShapeGroup_color-PeakTopZT
- panel-PeakTopZT_color-ShapeSubtype
- panel-PeakTopZT_color-ShapeGroup
- panel-PeakTopZT_color-PeakTopZT

This creates 9 kinds of panel UMAPs per embedding input.

Important output tables:
05_tables/all_peak_raw_zscore_table.tsv
05_tables/peak_shape_group_assignments.tsv
05_tables/shape_k_silhouette_scores.tsv
05_tables/shape_group_zscore_summary.tsv
05_tables/selected_k_by_topzt.tsv

README:
Write a README explaining:
- this is RPKM-only and ignores BED files
- row-wise z-score profiles define temporal shape
- PeakTopZT separation is enforced before shape clustering
- k-means and hierarchical clustering can be compared
- centroid correlation matrices diagnose whether k=2,3,4 creates distinct or redundant subclasses
- UMAPs are visualizations, not final clustering evidence

Runner script:
Create run_rpkm_temporal_shape_pipeline.sh using this command structure:

#!/usr/bin/env bash
set -euo pipefail

PROJECT_DIR="/media/lazarlab/Ubuntu3/ProcessedData/QIN/4.PROCESS_QIN_Apr10_26_ChipSeq_Reverb_38cohort"
COUNTS_FILE="${PROJECT_DIR}/FinalPeakSets2/tagCounts/whole_project_union_filtered.rpkm.txt"
OUTPUT_ROOT="${PROJECT_DIR}/PEAK_ANALYSIS/RESULTS/rpkm_temporal_shape_runs"
SCRIPT_DIR="${PROJECT_DIR}/PEAK_ANALYSIS/rpkm_temporal_shape_assessment_pkg"

VENV_PATH="/media/lazarlab/Ubuntu3/ProcessedData/QIN/QIN_Chipseq_Reverb_circadian_Oct24_2024_PROCESS/RESULTS/bwa/PeaksHomerMerged/venv"
source "${VENV_PATH}/bin/activate"
python --version

python "${SCRIPT_DIR}/rpkm_temporal_shape_pipeline.py" \
  --counts-file "${COUNTS_FILE}" \
  --output-root "${OUTPUT_ROOT}" \
  --run-label "whole_project_shape_assess" \
  --timepoints ZT1 ZT4 ZT7 ZT11 ZT13 ZT16 ZT19 ZT22 \
  --grouping-mode topzt_then_shape \
  --clustering-method kmeans \
  --diagnostic-methods both \
  --shape-k auto \
  --shape-k-range 1 4 \
  --diagnostic-k-values 1 2 3 4 \
  --min-peaks-per-shape-cluster 50 \
  --min-silhouette 0.15 \
  --max-withinzt-centroid-corr 0.90 \
  --auto-k-rule conservative \
  --max-silhouette-sample 10000 \
  --embedding-method umap \
  --embedding-input both \
  --umap-n-neighbors 15 \
  --umap-min-dist 0.30 \
  --embedding-columns ShapeSubtype ShapeGroup PeakTopZT \
  --embedding-panel-max-groups 80 \
  --amplitude-scheme quartiles \
  --heatmap-sort topzt \
  --heatmap-interpolations default none \
  --write-phase-aligned-centroid-corr

deactivate

Matplotlib:
Use Agg backend.
Save both PDF and PNG for figures.
Use interpolation='none' only when user selects none; otherwise allow matplotlib default.

Coding details:
- Use argparse.
- Use pandas, numpy, scipy, sklearn, matplotlib.
- Avoid seaborn dependency.
- Make all file/folder names safe with a safe_token function.
- Write robust error messages.
- Make the script executable.
- Package into ZIP.
