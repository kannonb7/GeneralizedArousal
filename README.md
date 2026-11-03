# GeneralizedArousal
GitHub repository for the Generalized Arousal paper.

## Ready-to-use arousal models (`0_CPMs`)

`0_CPMs/` contains the four arousal connectome-based predictive models (CPMs) from the paper, each trained on all participants of its dataset, in both atlases:

| Model | Shen 268 (+ / −) | Schaefer-Brainnetome 208 (+ / −) |
|---|---|---|
| `Pupil_Rest` | 3906 / 1055 | 3402 / 852 |
| `EEG_Rest` | 2843 / 1209 | 2277 / 985 |
| `Pupil_Story` | 2366 / 1204 | 2317 / 863 |
| `Movies` (Sherlock + FilmFestival pooled) | 5033 / 1640 | 3768 / 1655 |

**Format.** Each file is `{Model}_binarized_weights_{Shen|Schaefer}_0.05.npy`, a 1-D NumPy array with one value per edge:
- Length: 35,778 edges for Shen (268 × 267 / 2), 21,528 for Schaefer-Brainnetome (208 × 207 / 2).
- Values:
  - `+1`: the edge's FC rises with arousal;
  - `−1`: the edge's FC falls with arousal;
  - `0`: the edge is not in the model.
- An edge is in the model if a two-tailed one-sample t-test of the per-participant edge–arousal correlations gave p < .05. Its sign is the sign of the mean correlation.
- Edges are ordered by the upper triangle, row by row, with 1-based ROI numbers: (1,2), (1,3), …, (1,N), (2,3), …, (N−1,N). This is the same as `itertools.combinations(range(1, N + 1), 2)` or `np.triu_indices(N, 1)`.
- ROI numbers follow `3_atlases/Shen/shen_2mm_268_parcellation.nii.gz` or `3_atlases/Schaefer/Schaefer200_17N_plus_8ROIs_208Parcels.nii.gz` (see the ROI key below).

```python
import numpy as np
w = np.load("0_CPMs/Pupil_Rest_binarized_weights_Shen_0.05.npy")
n = 268                                   # 208 for the Schaefer files
M = np.zeros((n, n))
M[np.triu_indices(n, 1)] = w
M = M + M.T                               # M[i-1, j-1] = weight of the edge between ROI i and ROI j
pos_network, neg_network = (M > 0).astype(int), (M < 0).astype(int)

# Predicted arousal for one FC pattern (an edge vector in the same order, e.g. one sliding window):
sig = np.flatnonzero(w)
predicted_arousal = (fc_vector[sig] * w[sig]).sum() / len(sig)
```

The models were trained on dynamic FC computed as in `a_Arousal_FC`: confound-regressed BOLD, Pearson correlation in a 30-second Gaussian sliding window, with no Fisher z-transform. Predictions are most comparable when new FC is computed the same way.

## Additional items you will need to rerun the pipeline

1) **fMRI data.** All 5 datasets are publicly available on OpenNeuro. We preprocessed each one with fMRIPrep and then filtered it (details in the paper). Put the preprocessed BOLD time courses and fMRIPrep confounds files in the matching `1_data/*_fMRI` folder. `a_Arousal_FC` regresses the confounds and standardises the data but does not filter. The Sherlock BOLD files contain only the movie TRs; the scripts take the matching rows of each confounds file.

   **Expected file names.** The scripts find runs and confounds by file name, so name the files like this. `*` means anything. Only BOLD files may end in `.nii.gz` in these folders, and subjects are read in sorted order.

   | Folder | BOLD | Confounds |
   |---|---|---|
   | `1_EEG_Rest_fMRI` | `sub-XX_task-rest_run1_*.nii.gz` | `sub-XX_task-rest_run-1_desc-confounds_timeseries.tsv` |
   | `1_EEG_Sleep_fMRI` | `sub-XX_task-sleep_run1_*.nii.gz` | `sub-XX_task-sleep_run-1_desc-confounds_timeseries.tsv` |
   | `2_Pupil_Rest_fMRI` | `sub-paXXXX_task-rest_run1_*.nii.gz` | `sub-paXXXX_task-rest_run-01_desc-confounds_timeseries.tsv` |
   | `3_Pupil_Story_fMRI` | `sub-tbXXXX_task-story1_*.nii.gz` | `sub-tbXXXX_task-story1_desc-confounds_timeseries.tsv` |
   | `4_FilmFestival_fMRI` | `sub-XX_task-movie_run1_*.nii.gz` | `sub-XX_task-movie_run-01_desc-confounds_timeseries.tsv` |
   | `5_Sherlock_fMRI` | `sub-XX_task-sherlock_part1_*.nii.gz` | `sub-XX_task-sherlockPart1_desc-confounds_timeseries.tsv` |

   **Subjects.** Some subjects are excluded by their position in the sorted file list, so each folder must contain exactly these subjects:
   - **EEG Rest (26):** sub-02, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 17, 18, 19, 20, 21, 22, 23, 25, 26, 28, 29, 31, 33
   - **Pupil Rest (27):** sub-pa1372, 1387, 1393, 1416, 1560, 1582, 1604, 1674, 1700, 1766, 1792, 1811, 1834, 1841, 1859, 1913, 1940, 1978, 2079, 2100, 2106, 2196, 2213, 2216, 2220, 2221, 2226
   - **Pupil Story (22):** sub-tb2994, 3132, 3240, 3279, 3512, 3592, 3602, 3626, 3646, 3744, 3757, 3784, 3810, 3846, 3858, 3920, 3929, 3964, 3977, 4450, 4547, 4572
   - **FilmFestival (20):** sub-01 to sub-20
   - **Sherlock (15):** sub-01, 02, 03, 04, 05, 06, 08, 09, 10, 11, 12, 13, 14, 15, 16
   - **EEG Sleep:** the sleep runs from Gu et al. (2023). `b_Classification` excludes the subjects listed in its `EXCLUDE` setting.

2) **Arousal data.**
   - **Included in this repository:** arousal time courses for Pupil Story (`1_data/3_Pupil_Story_Arousal/`), FilmFestival and Sherlock, plus the event timestamp files for the two movies.
   - **Pupil Rest arousal:** on Lee et al. (2022)'s OpenNeuro page under `derivatives`. Put it in `1_data/2_Pupil_Rest_Arousal/` (one folder per subject, e.g. `sub-pa1372/sub-pa1372_task-rest_run-01_et.tsv`).
   - **EEG Sleep stages:** on Gu et al. (2023)'s OpenNeuro page under `sourcedata`. Put them in `1_data/1_EEG_Sleep_fMRI/wakestage/` as `sub-XX-sleep-stage.tsv` (column `30-sec_epoch_sleep_stage`).
   - **EEG Rest arousal:** calculate spectral slopes from Gu et al. (2023)'s EEG data (details in the paper). Save the resulting array, shape (26 subjects, 2 runs, TRs per run) in the same subject order as the EEG Rest BOLD files, as `1_data/1_EEG_Arousal/SpectralSlopes_05Hz_80Hz.npy`.

3) **External predictive models (`5_external_cpms`).** `e_FeatureMaps` compares our arousal networks with two published connectome-based models, both defined in the Shen 268 atlas. Download them from the original authors and place them as follows:
   ```
   5_external_cpms/
   ├── saCPM_Rosenberg2016/
   │   └── saCPM.mat                      # sustained-attention CPM:
   │                                      #   variables high_attention_mask, low_attention_mask (268 x 268, binary)
   └── SurpriseEFPM_Zhang2025/
       ├── surprise_EFPM_pos_mask.txt     # surprise edge-fluctuation model:
       └── surprise_EFPM_neg_mask.txt     #   positive / negative networks (268 x 268, binary)
   ```

The atlases (`3_atlases/`) and the movie recall data (`4_results/4_recall/`) are included.

## Pipeline

Run the notebooks in `2_scripts/` in this order. Each one reads earlier outputs from `4_results/`.

**a_Arousal_FC.ipynb**
Removes the fMRIPrep confounds from the preprocessed BOLD data, extracts ROI time series, and calculates dynamic functional connectivity.
- Pupil Rest, Pupil Story and EEG Rest: a 30-second Gaussian sliding window.
- FilmFestival and Sherlock: one FC pattern per movie event.

It also preprocesses the arousal time courses: HRF convolution, then smoothing over the same 30-second window, and one time course per movie from the event-based ratings.

**b_Classification.ipynb**
- Within- and across-dataset CPM predictions, including leave-one-subject-out models within Movies and within Pupil Story.
- Phase-randomisation permutation tests.
- EEG sleep-stage (wake) prediction from each arousal model.
- Saves each model's edge correlations and binarized weights to `4_results/CPM/`. The `0_CPMs` files are the binarized weights from the p < .05 runs.

**c_Network.ipynb**
- Finds edges selected with the same sign by 2, 3 or all 4 arousal models, and tests each overlap against a permutation null.
- Summarises the 4-way network by canonical brain network.
- Labels every overlapping edge.
- Predicts arousal in every dataset from the 4-way network.

**d_Recall.Rmd**
Bayesian multilevel models (brms) predicting movie recall fidelity from decoded arousal, with LOO model comparison. It uses the recall data in `4_results/4_recall/` and needs `b` and `c` to have run first.

**e_FeatureMaps.ipynb**
Places the four arousal models and the two external models in a common reference space: edge-map similarity, then interval MDS with Procrustes alignment. It writes the similarity matrices, MDS coordinates and stress values as CSVs to `4_results/4_feature_maps/`. Needs the outputs of both atlases (see below) and `5_external_cpms/`.

### Atlas
`a_Arousal_FC`, `b_Classification` and `c_Network` run with either:
- the **Shen** atlas (268 ROIs, default), or
- **Schaefer 200 (17 Network) + 8 Brainnetome subcortical** ROIs (208 ROIs).

To switch, set `ATLAS_NAME` in `a` and `ATLAS` in `b` and `c` to `"Schaefer"`. Schaefer outputs end in `_Schaefer` (results in `2_classification_Schaefer_{ALPHA}/` and `3_network_Schaefer_{ALPHA}/`), so the two atlases never overwrite each other. The first Schaefer run of `c` needs internet, because nilearn downloads the Schaefer ROI names. `e_FeatureMaps` uses both atlases, so run `a` and `b` once for each atlas before running `e`.

**Schaefer + 8 subcortical ROIs.** ROIs 1–200 are the Schaefer 2018 parcels (200 parcels, 17 networks) in label order. ROIs 201–208 are subcortical regions made by merging Brainnetome parcels:

| ROI | Region | Brainnetome parcels merged |
|---|---|---|
| 201 | Left amygdala | 211, 213 |
| 202 | Left basal ganglia | 219, 221, 223, 225, 227, 229 |
| 203 | Left hippocampus | 215, 217 |
| 204 | Left thalamus | 231, 233, 235, 237, 239, 241, 243, 245 |
| 205 | Right amygdala | 212, 214 |
| 206 | Right basal ganglia | 220, 222, 224, 226, 228, 230 |
| 207 | Right hippocampus | 216, 218 |
| 208 | Right thalamus | 232, 234, 236, 238, 240, 242, 244, 246 |

Where a subcortical ROI overlapped a Schaefer parcel, the subcortical label was kept.

### Feature-selection threshold
`ALPHA` in `b` (default 0.05) sets the CPM feature-selection threshold. Results go to `4_results/2_classification_{ATLAS}_{ALPHA}/`. `c_Network` reads the folder that matches its own `ATLAS` and `ALPHA`.

## Requirements
- **Python:** numpy, scipy, pandas, nibabel, nilearn, scikit-learn, statsmodels, matplotlib, seaborn, openpyxl (and joblib if you set `N_JOBS > 1` in `a`).
- **R (for `d_Recall.Rmd`):** brms (needs a C++ toolchain, e.g. Rtools on Windows), posterior, bayestestR, ggplot2, cowplot, dplyr, tidyr, readr, and RcppCNPy or reticulate.
- **Optional:** the R package smacof for the interval MDS in `e`. Without it, `e` uses an equivalent Python implementation.
