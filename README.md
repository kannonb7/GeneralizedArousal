# GeneralizedArousal
Github Repository for Generalized Arousal Paper. 

File Tree:
├───1_data
│   ├───1_EEG_Arousal
│   ├───1_EEG_Rest_fMRI
│   ├───1_EEG_Sleep_fMRI
│   │   └───wakestage
│   ├───2_Pupil_Rest_Arousal
│   ├───2_Pupil_Rest_fMRI
│   ├───3_Pupil_Story_Arousal
│   ├───3_Pupil_Story_fMRI
│   ├───4_FilmFestival_Arousal
│   ├───4_FilmFestival_fMRI
│   ├───5_Sherlock_Arousal
│   └───5_Sherlock_fMRI
├───2_scripts
├───3_atlases
│   └───Shen
└───4_results
    ├───1_arousal
    │   ├───1_EEG_Rest
    │   ├───1_EEG_Sleep
    │   │   └───combined_sleepwake
    │   ├───2_Pupil_Rest
    │   ├───3_Pupil_Story
    │   ├───4_FilmFestival
    │   └───5_Sherlock
    ├───2_classification
    ├───3_network
    └───4_recall

Additional items you will need:
1) fMRI Data. All 5 datasets are available publicly on OpenNeuro. We used fMRIPrep to process each dataset (details in paper) and outputted the resulting BOLD time courses and confounds files to each respective /1_data/*_fMRI dir.
2) Arousal Data. Arousal time courses for Pupil Story, FilmFestival, and Sherlock are included in this repository. Pupil Rest Arousal is on Lee et al. (2022)'s OpenNeuro page under "derivatives". EEG Sleep Arousal is under Gu et al. (2023)'s OpenNeuro under "sourcedata". Put these in /1_data/2_Pupil_Rest_Arousal and 1_data/1_EEG_Sleep_fMRI/wakestage respectively. For EEG Arousal, you will have to use Gu et al. (2023)'s EEG Data to calculate spectral slopes (details in paper) and put the resulting array in /1_data/1_EEG_Arousal.
3) Atlases. Find the 2013 Shen Atlas online and download shen_268_parcellation_networklabels.csv and shen_2mm_268_parcellation.nii.gz, then put them in /3_atlases/Shen.

The steps in this coding pipeline are as follows:

a_Calculate_FC.ipynb
    This script uses the preprocessed fMRIPrep BOLD time courses and regresses out fMRIPrep-calculated confounds. Then, it uses the Shen atlas to calculate dynamic functional connectivity patterns over a 30 second sliding window. For event-based fMRI datasets (Sherlock and FilmFestival) it calculates event-based functional connectivity.

b_Convolve_Arousal.ipynb
    This script preprocesses the arousal time courses by convolving with a hemodynamic response function and smoothed over the same 30-second sliding window to match the fMRI data. It also preprocesses the event-based movie arousal into one time course.

c_Arousal_Classification.ipynb
    This script performs the within- and across- dataset CPM predictions.

d_Classification_Analysis.ipynb
    This script analyzes the predictions generated in c_Arousal_Classification.ipynb.

e_EEG_Sleep_Analysis.ipynb
    This script preprocesses the EEG Sleep fMRI data and uses the CPMs to predict wakestage.

f_Network_Analysis.ipynb
    This script examines the overlapping features in each arousal network.

g_Recall_Models.Rmd
    This scripts generates the models for predicting recall fidelity from decoded arousal. 
