# GeneralizedArousal
Github Repository for Generalized Arousal Paper. 

File Tree:
├───1_data
│   ├───1_EEG_Arousal
│   ├───1_EEG_Rest_fMRI
│   ├───1_EEG_Sleep_fMRI
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
2) Arousal Data. Arousal time courses for Pupil Story, FilmFestival, and Sherlock are included in this repository. Pupil Rest Arousal is on Lee et al. (2022)'s OpenNeuro page under "derivatives". EEG Sleep Arousal is under Gu et al. (2023)'s OpenNeuro under "sourcedata". Put these in 1_EEG_
