# GeneralizedArousal
GitHub repository for the Generalized Arousal paper.

## Additional items you will need

1) **fMRI data.** All 5 datasets are publicly available on OpenNeuro. We preprocessed each one with fMRIPrep (details in the paper). Put the resulting BOLD time courses and confounds files in the matching `1_data/*_fMRI` folder:
   `1_EEG_Rest_fMRI`, `1_EEG_Sleep_fMRI`, `2_Pupil_Rest_fMRI`, `3_Pupil_Story_fMRI`, `4_FilmFestival_fMRI`, `5_Sherlock_fMRI`.

2) **Arousal data.**
   - **Included in this repository:** arousal time courses for Pupil Story, FilmFestival and Sherlock, plus the event timestamp files for the two movies.
   - **Pupil Rest arousal:** on Lee et al. (2022)'s OpenNeuro page under `derivatives`. Put it in `1_data/2_Pupil_Rest_Arousal/` (one folder per subject, e.g. `sub-pa1372/sub-pa1372_task-rest_run-01_et.tsv`).
   - **EEG Sleep stages:** on Gu et al. (2023)'s OpenNeuro page under `sourcedata`. Put the sleep-stage files in `1_data/1_EEG_Sleep_fMRI/wakestage/`.
   - **EEG Rest arousal:** calculate spectral slopes from Gu et al. (2023)'s EEG data (details in the paper), and save the resulting array as `1_data/1_EEG_Arousal/SpectralSlopes_05Hz_80Hz.npy`.

3) **External predictive models (`5_external_cpms`).** `e_FeatureMaps` compares our arousal networks with two published connectome-based models, both defined in the Shen 268 atlas. Download them from the original authors and place them as follows:
   ```
   5_external_cpms/
   ├── saCPM_Rosenberg2016/
   │   └── saCPM.mat                      # sustained-attention CPM (Rosenberg et al., 2016):
   │                                      #   variables high_attention_mask, low_attention_mask (268 x 268, binary)
   └── SurpriseEFPM_Zhang2025/
       ├── surprise_EFPM_pos_mask.txt     # surprise edge-fluctuation model (Zhang et al., 2025):
       └── surprise_EFPM_neg_mask.txt     #   positive / negative networks (268 x 268, binary)
   ```

## Pipeline

Run the notebooks in this order. Each one reads the previous one's outputs from `4_results/`.

**a_Arousal_FC.ipynb**
Removes the fMRIPrep confounds from the preprocessed BOLD data, extracts ROI time series, and calculates dynamic functional connectivity.
- Pupil Rest, Pupil Story and EEG Rest: a 30-second Gaussian sliding window.
- FilmFestival and Sherlock: one FC pattern per movie event.

It also preprocesses the arousal time courses: HRF convolution, then smoothing over the same 30-second window, and one time course per movie from the event-based ratings.

**b_Classification.ipynb**
- Within- and across-dataset CPM predictions, including leave-one-subject-out models within Movies and within Pupil Story.
- Phase-randomisation permutation tests.
- EEG sleep-stage (wake) prediction from each arousal model.
- Saves each model's edge correlations and weights to `4_results/CPM/` for `e_FeatureMaps`.

**c_Network.ipynb**
- Finds edges selected with the same sign by 2, 3 or all 4 arousal models, and tests each overlap against a permutation null.
- Summarises the 4-way network by canonical brain network.
- Labels every overlapping edge.
- Predicts arousal in every dataset from the 4-way network.

**d_Recall.Rmd**
Bayesian multilevel models (brms) predicting movie recall fidelity from decoded arousal, with LOO model comparison. It uses the recall data in `4_results/4_recall/` (included).

**e_FeatureMaps.ipynb**
Places the four arousal models and the two external models in a common reference space: edge-map similarity, then interval MDS with Procrustes alignment. Needs the outputs of both atlases (see below) and `5_external_cpms/`.

### Atlas
`a_Arousal_FC` and `b_Classification` run with either:
- the **Shen** atlas (268 ROIs, default), or
- **Schaefer 200 + 8 Brainnetome subcortical** ROIs (208 ROIs).

To switch, set `ATLAS_NAME` in `a` and `ATLAS` in `b` and `c` to `"Schaefer"`. Schaefer outputs end in `_Schaefer` (results in `2_classification_Schaefer_{ALPHA}/` and `3_network_Schaefer_{ALPHA}/`), so the two atlases never overwrite each other. The first Schaefer run of `c` needs internet, because nilearn downloads the Schaefer ROI names. `e_FeatureMaps` uses both atlases, so run `a` and `b` once for each atlas before running `e`.

### Feature-selection threshold
`ALPHA` in `b` (default 0.05) sets the CPM feature-selection threshold. Results go to `4_results/2_classification_{ATLAS}_{ALPHA}/`. `c_Network` reads the folder that matches its own `ATLAS` and `ALPHA`.

## Requirements
- **Python:** numpy, scipy, pandas, nibabel, nilearn, scikit-learn, statsmodels, matplotlib, seaborn, openpyxl.
- **R (for `d_Recall.Rmd`):** brms (needs a C++ toolchain, e.g. Rtools on Windows), posterior, bayestestR, ggplot2, cowplot, dplyr, tidyr, readr, and RcppCNPy or reticulate.
- **Optional:** the R package smacof for the interval MDS in `e`. Without it, `e` uses an equivalent Python implementation.
