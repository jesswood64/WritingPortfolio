
# READ ME

# Description of Apex Functional Connectivity project - JW 2023
### This document goes through folder set-up, Kong Individual Parcellation code, and FC preprocessing & analyses 

 
#### Folder explanation: /storage/ApexConn/: 

DCM_workshop = Nee/Pitt scripts for running DCM tutorial - not applied to ApexConn dataset 

GrattonScripts = All files and folders given from Ally Dworetsky in Gratton Lab 2022-2023 
	- When running code in Matlab:this folder almost always needs to be on path including all subfolders for scripts to run 

KONG_Example = Houses all scripts and example data from Kong et al., 2019 (CBIG folder cloned from git)

ProtocolTesting = All PREprocessing, GLM, and functional connectivity data
	- preprocessing scripts = `/storage/ApexConn/ProtocolTesting/code` <br /> 
	- preprocessed data = `/storage/ApexConn/ProtocolTesting/derivatives/preproc_fmriprep-22.0.2/`

SFN = analyses & visualizations done for Wood, Meyer, & Nee Poster to SFN23 on CON/FPN cTBS effects

figures = folder for all figures made from project analyses

scripts = POST-preprocessing analysis scripts

workbench = connectome workbench downloaded locally 

====================================================
### When in doubt with matlab errors - make sure to run `addpath(genpath('/storage/ApexConn/GrattonScripts/')`
## KONG Individual Parcellation / Surface Preprocessing
To go through example dataset from github: <br /> 
https://github.com/ThomasYeoLab/CBIG/tree/master/stable_projects/brain_parcellation/Kong2019_MSHBM/examples <br /> 
Entire CBIG/Yeo data and code cloned from git here: `/storage/ApexConn/KONG_Example/CBIG` <br /> 
### MUST FOLLOW YEO'S SETUP README.md TO GET A CONFIGURATION FILE SET IN YOUR BASHRC AND SOFTWARE VERSIONS TO COMPLY
Found here: https://github.com/ThomasYeoLab/CBIG/blob/master/setup/README.md <br /> 
Create a setup folder in your local home directory and copy the `/storage/ApexConn/KONG_Example/CBIG_sample_config2.sh` file. <br /> 
Then put this line in .bashrc: `source ~/setup/CBIG_sample_config2.sh` <br /> 
Must also have paths to FSL and ANTS in .bash_profile:<br /> 
"#FSL
FSLDIR=/usr/local/fsl
. ${FSLDIR}/etc/fslconf/fsl.sh
PATH=${FSLDIR}/bin:${PATH}
export FSLDIR PATH

export FSLOUTPUTTYPE=NIFTI"
<br /> 
"#ANTS
export ANTSPATH=/home/local/antsbin/bin
export PATH=${ANTSPATH}:$PATH

export ANTSPATH=/home/local/ANTS
export PATH=${ANTSPATH}:$PATH"

JW ran first 2 subs (103 & 701), running a 'good sub' based on low movement/later time in data collection with KG: 728<br /> 
### To run on Apex participants:  <br /> 
PRIOR to running Kong, all participants need to: <br /> 

1. Have surface preprocessing completed in fMRIprep/freesurfer <br /> 
`/storage/ApexConn/ProtocolTesting/code/Surface_Preproc_forKONG/fmriprep_ApexConnSurface.sh` <br /> 
**Before running: change the -u and -user fields in fmriprep_ApexConnSurface.sh to YOUR user id using 'id -u' in terminal so that you have permissions**
	- To RUN: in terminal in above folder, type: time bash fmriprep_ApexConnSurface.sh SUBJECT - (i.e. time bash fmriprep_ApexConnSurface.sh 705)<br /> 
	- When re-running participants with surface processing,  to avoid re-writing they need to be in a new folder or have the old sub folders renamed <br /> 
	- Takes about 15 hours to run per subject <br /> 
	- Output of surface preprocessed data is in: `/storage/ApexConn/ProtocolTesting/derivatives/preproc_fmriprep-22.0.2/sourcedata/freesurfer/` <br /> 

2. Finish FC preprocessing in matlab
	- Run the 5-step script `/storage/ApexConn/ProtocolTesting/code/Surface_Preproc_forKONG/SurfacePreprocSteps.m`
		- Step 1: Make sure the subject file is filled with correct subject info `/storage/ApexConn/ProtocolTesting/code/CCT_datalist_FCProcess_AllVisit.txt` <br /> 
		- Step 2: Need to run the `/storage/ApexConn/ProtocolTesting/code/make_fs_masks_CallFunc.m` script to ensure proper folder and file order. (you will need user permissions from fMRIprep output!) <br /> 
		- **NOTE: make_fs_masks_CallFunc.m script creates an overall anat folder in subject's preprocessing folder. if you are overwriting an old sub's folder (i.e., you didn't create a new folder during fMRIprep), DELETE their overall anat folder first as it will not overwrite it (e.g., /storage/ApexConn/ProtocolTesting/derivatives/preproc_fmriprep-22.0.2/sub-744/anat/)**
	<br /> <br /> 
		- Step 3: Run FCProcess function for motion, demean and detrend, nuissance regression, bandpass filtering 		
		- Step 4: Create surfaces in correct space (from freesurfer output) -> fsLR32k space <br /> 
		- RUN the function PostFreeSurferPipeline_fsavg2fslr_long_GrattonLabJW.m: 
		- **NOTE: Whoever runs the `PostFreeSurferPipeline_fsavg2fslr_long_GrattonLabJW.m` script owns that subject's folder, no one else can run without permissions**
		- PostFreeSurferPipeline_fsavg2fslr_long_GrattonLabJW.m Takes about 30 minutes to run <br /> 
		- Check this output in workbench to make sure its mapping on to the surface properly: 
		- open workbench by clicking `/usr/local/workbench/exe_rh_linux64/wb_view`
		- load in spec file, e.g.: `/storage/ApexConn/ProtocolTesting/derivatives/preproc_fmriprep-22.0.2/sourcedata/freesurfer/FREESURFER_fs_LR/sub-701/MNI/fsaverage_LR32k/sub-701.32k_fs_LR.wb.spec` to Load all files & check that the brain looks normal and colors are mapped on to anatomical gyri correctly (will look obvious if something went wrong)<br /> 

		- Step 5: Create surface cifti's per run -> post_fc_processing_batch_GrattonLabJW.m outputs a .dtseries.nii file for use in Kong pipeline<br /> 
		- parameters/file paths should already be set in `post_fc_processing_batch_params_Apex.m` <br /> 
		- Takes ~ 2 hours per subject if just running 1 6-run session<br /> 

### Kong 2019 has 3 steps. These are in one script:
	`/storage/ApexConn/ProtocolTesting/KongParcellations/Kong2019_1sub.m` 
	- Takes about 90 minutes to run step 1, then skip step 2, step 3 takes about 1 minute

#### Once Kong code is completed and cifti creation script is ran (all in above script), open .dtseries.nii file in Connectome Workbench
- to open workbench: `/usr/local/workbench/exe_rh_linux64/` -> wb_view 
- Load in surface template from `/storage/ApexConn/GrattonScripts/SurfacePipeline/Conte69_32k_atlas_surfaces/` (inflated brain looks best, midthickness brain is anatomically correct for MNI coordinates)
- to create spheres on brain (foci) ust matlab function: `/storage/ApexConn/GrattonScripts/focifilemaker_workbench_ben.m`<br /> 
can use this example script to format your coordinates/colors of foci: `/storage/ApexConn/ProtocolTesting/code/IndAndGpFociMaker.m`
- open .foci files with wb_view 
- Then in workbench: Select Data->Project Foci (view with Features toolbox) 
- JW overlay settings in wb were: Palette: PowerSurf, uncheck Interpolate colors 

====================================================
====================================================

# Order of Operations for FC project

## Order of Operations: ***Preproc***
Original Gratton examples of files are in `/storage/ApexConn/GrattonScripts`
### 1. All data needs to be in BIDS format for fMRIprep:
Need to have conda installed <br /> 
	1. https://conda.io/projects/conda/en/latest/user-guide/install/linux.html <br /> 
	2. I used miniconda & downloaded locally
 
**Make dcm2bids environment**
put `environment.yml` in your directory that houses miniconda  (or make your own: https://unfmontreal.github.io/Dcm2Bids/docs/get-started/install/) <br /> 
then from that directory call:
`conda env create --file environment.yml`

then you can type to make sure it worked:
`conda activate dcm2bids`

**Make config file**
	- Identifies each scan based on header information <br /> 
	- my example: (in hidden folder so that it doesn't mess up BIDS format) `/storage/ApexConn/ProtocolTesting/BIDS/Nifti/.bidsignore/config_apex.json`
 
**Make dcm2bids script**
	- my example: (in hidden folder so that it doesn't mess up BIDS format) `/storage/ApexConn/ProtocolTesting/BIDS/Nifti/.bidsignore/Apex_dcm2bids.sh` <br /> 
	- Need to have a folder where output will go and a Raw Dir folder both under the same overarching folder (I called it “BIDS”)  <br /> 
	- Uses pydeface to deface anatomical images – *not 100 necessary* <br /> 
	- Need to install pydeface and add FSL to path <br /> 
	- pydeface  <br /> 
	- `pip install pydeface` <br /> 
	- add FSL to path <br /> 
	- cd to ~ <br /> 
	- `gedit .bash_profile` <br /> 
			“#FSL
			FSLDIR=/usr/local/fsl
			. ${FSLDIR}/etc/fslconf/fsl.sh
			PATH=${FSLDIR}/bin:${PATH}
			export FSLDIR PATH
			export FSLOUTPUTTYPE=NIFTI”
 
**To convert to BIDS**
In terminal:
`conda activate dcm2bids` <br /> 
Your bash should now start with (dcm2bids) <br /> 
To run: `bash dcm2bids_scriptname Sub#`

### 2. fMRIPrep
Templateflow - downloaded and unzipped locally 
`home/jw20ha/.local/lib/python3.8/site-packages/templateflow/conf/templateflow` <br /> 
locate freesurfer license file 
`home/local/freesurfer/license.txt`

Docker needs to be on path, e.g.: <br /> 
	- Put on .bashrc <br /> 
	- Gedit .bashrc <br /> 
	- Restart terminal <br /> 
Find /home/jw20ha/.local/ -name fmriprep-docker <br /> 
Export PATH="/home/jw20ha/.local/bin:$PATH"

Notes for fMRIPrep script: `/storage/ApexConn/ProtocolTesting/code/fMRIprep_STC_NEW.sh` <br /> 
1. disable FreeSurfer surface preprocessing with --fs-no-reconall <br /> 
2. fMRIPrep does slice timing corrections to the middle slice (not the first which we do). This is important for the first level GLM to set the microtime onset: use --slice-time-ref start \ <br /> 
3. Add yourself as user to own output/avoid root user permissions with -u or --user flags <br /> 

`/storage/ApexConn/ProtocolTesting/code/fMRIprep_STC_NEW.sh` = This script produces output preprocessing up through normalization. Output files named as desc_preprocess_bold.nii in each subject's visit folder `/storage/ApexConn/ProtocolTesting/derivatives/preproc_fmriprep-22.0.2/sub-XX/ses-VXX/func/` <br /> 
		- Slice timing correction <br /> 
		- Spatial normalization <br /> 
		- Co-registration <br /> 
		- Motion realignment  <br /> 
	- Doesn't do smoothing, motion corrections 

### 3. AFNI GLM
`/storage/ApexConn/ProtocolTesting/AFNI_GLM/code` <br /> 
Original script: `Apex_GLM_38_Visitnew.tcsh` <br /> 
Script with shifts in slice timing if NOT modified from fMRIprep: `Apex_GLM_STC_storage.tcsh`

### 4. Correct for motion, make masks, make regressor files, finalize FC preprocessing = all done in Matlab
**Detailed in `/storage/ApexConn/scripts/FCMatlabSteps.m`**

====================================================

## Order of Operations: ***Analyses***
Different scripts for residual analyis, individual-target analysis, and task-included analysis  <br /> 
After all data is preprocessed, used Matlab for all analyses:  <br /> 
Convert SPM MNI coordinates to FSL MNI coordinates: <br /> 
`/storage/ApexConn/ProtocolTesting/code/IndTargetConvertFSL.m`
<br /> 
`/storage/ApexConn/scripts/`:
1. Create ROI mask using FSL Coordinates <br /> 
ROI_AFNI_FSLCorrds.m <br /> 
ROI_AFNI_IndCorrds.m <br /> 

2. Pull ROI timeseries for each subject/visit <br /> 
FC_analysis.m <br /> 
FC_analysis_Ind.m <br /> 
FC_analysisTASK.m <br /> 

3. Calculate motion to create Good Subs List <br /> 
checkFD.m <br /> 

4. Run Stats and create graphs on ROIs on only Good Subs <br /> 
Graphs_Stats_motionCont.m <br /> 
Graphs_Stats_motionContTASK.m <br /> 
Graphs_Stats_Ind.m <br /> 

5. Run Stats and create graphs for Whole Brain on only Good Subs <br /> 
corrmatsSeitzman_motionCont.m <br /> 
corrmatsSeitzman_motionContTASK.m <br /> 

6. Create Seed-Whole brain FC Maps <br /> 
SeedToWholeBrainOutput.m<br /> 
SeedToWholeBrainOutput_Ind.m <br /> 
SeedToWholeBrainOutputTASK.m <br /> 

7. Create final figures  <br /> 
DefenseFigures.m <br /> 
DefenseFigures_fslTask.m <br /> 
DefenseFigures_Ind.m <br /> 

====================================================

