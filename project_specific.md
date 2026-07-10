Write a temporal-profile clustering pipeline using the following reusable architecture.

The pipeline analyzes a matrix of features/regions across ordered temporal conditions. It should first visualize all rows before grouping, then normalize each row across time, assign temporal metrics such as peak time and amplitude, perform clustering, assess how many temporal-shape clusters are supported, and generate diagnostic plots.

The structure must include:

1. Input/configuration
   - command-line options for input matrix, output root, run label, ordered condition labels, normalization method, clustering method, k range, diagnostic k values, embedding method, and plotting options

2. Preprocessing
   - detect row ID column
   - aggregate replicates if needed
   - construct raw matrix
   - construct row-wise normalized matrix
   - compute row-level metrics such as top condition, amplitude, and amplitude category

3. Pre-grouping plots
   - whole normalized heatmap
   - whole raw/log heatmap
   - whole normalized profile
   - whole raw profile

4. Group assignment
   - support global clustering
   - support constrained clustering where rows are first split by a biological label such as top condition, phase, tissue, treatment, or manually assigned class
   - within each stratum, support k-means or hierarchical clustering
   - final group labels should combine stratum label and subtype label

5. Auto-k diagnostics
   - test k values across a user-supplied range
   - compute silhouette where applicable
   - compute minimum cluster size
   - compute centroid separation
   - select the smallest acceptable k under a conservative rule
   - write a selected-k table and a full diagnostic score table

6. Centroid analysis
   - compute mean temporal profile for each candidate cluster
   - create pairwise Pearson correlation matrices among all centroids
   - write centroid correlation matrices and pairwise centroid tables
   - optionally compute phase-aligned or peak-aligned centroid correlations

7. Group visualization
   - grouped heatmaps ordered by biological order
   - rows within groups sorted by top condition and row ID
   - group profile panels ordered by biological order
   - individual-trace profile panels

8. Embeddings
   - compute UMAP if available, otherwise PCA fallback
   - support PCA and t-SNE options
   - support normalized and raw/log-scaled input
   - make regular color plots
   - make panel/color grid plots from user-supplied metadata columns

9. Outputs
   - parameters JSON
   - all intermediate tables
   - all plots as PDF and PNG
   - README describing outputs
   - short timestamped output directory

Keep the code modular. Do not write one huge procedural script. Each analysis step should be a function.
