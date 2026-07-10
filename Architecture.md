Write a modular scientific analysis pipeline in Python with the following architecture.

The pipeline should be organized into clear functional layers:

1. Imports, global defaults, plotting style.
2. argparse-based configuration.
3. output-directory creation and reproducibility logging.
4. data loading and sample/replicate aggregation.
5. construction of an analysis-ready matrix.
6. normalization and row-level feature calculation.
7. all-data pre-grouping visualizations.
8. unsupervised clustering/grouping.
9. clustering diagnostics and centroid-comparison analysis.
10. grouped heatmaps and profile plots.
11. low-dimensional embeddings and embedding panels.
12. final table writing and summary messages.

The program should separate project-specific assumptions from general pipeline logic. Project-specific details such as input paths, timepoint labels, replicate naming, grouping thresholds, clustering method, and plot columns should be controlled by command-line arguments rather than hard-coded.

Use this general structure:

- `parse_args()`
- `safe_token()`
- `build_run_dir()`
- `save_parameters()`
- `write_readme()`
- `savefig_both()`
- `load_input_matrix()`
- `average_replicates_or_samples()`
- `normalize_profiles()`
- `add_row_metrics()`
- `plot_whole_heatmap()`
- `plot_all_profile()`
- `cluster_kmeans()`
- `cluster_hierarchical()`
- `choose_k_by_diagnostics()`
- `assign_groups()`
- `compute_centroids()`
- `compute_centroid_correlation()`
- `plot_centroid_correlation_heatmap()`
- `plot_group_heatmap()`
- `plot_group_profiles()`
- `plot_group_profiles_with_traces()`
- `compute_embedding()`
- `plot_embedding()`
- `plot_embedding_panels()`
- `main()`

The main function should only orchestrate these functions and should remain readable.

The pipeline must always:
- save all parameters to JSON
- save important intermediate tables
- save both PDF and PNG figures
- create a README explaining the output folders
- use a short output folder name with timestamp
- avoid silent overwriting unless `--overwrite` is supplied
- print a concise final run summary

The program should include both k-means and hierarchical clustering as interchangeable methods.
The program should include diagnostic clustering runs across multiple k values.
The program should compute cluster centroids and pairwise centroid correlation matrices.
The program should produce grouped heatmaps, profile panels, individual-trace profile panels, embedding plots, and embedding panel grids.

Design the program so that biological/project-specific grouping rules can be swapped. For example, one project may use global clustering, while another may first split rows by a known phase/timepoint/group label and then cluster within each stratum.

Do not hard-code one biological interpretation into plotting functions. Plotting functions should accept group columns, value columns, order columns, and labels as arguments.

Use pandas, numpy, scipy, sklearn, and matplotlib. Do not require seaborn.
Use matplotlib Agg backend.
Write robust error messages.
Package the script, README, and runner shell script into a ZIP.
