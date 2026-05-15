---
title: 6.2 NiPype for 1st-level fMRI analysis
draft: true
tags:
---
If you use SPM for 1st-level analysis, please follow this official [tutorial](https://miykael.github.io/nipype_tutorial/notebooks/basic_model_specification_fmri.html). Here we walk through the FSL approach with the preprocessed data from `fmriprep`.  We use the open MRI data `ds000102` as an example dataset.

## 1. Select files with iterable

We will use the `fmriprep` output directory as input. The `fmriprep` output directory is the one that contains the preprocessed data. We will use the `fmriprep` output directory as input to the workflow. We will loop over `subject_id` and `run`, and grabe:

- the unsmoothed bold image from `fmriprep` output directory
- the event TSV file
- the confound TSV file

```python
# setup directories, change to yours accordingly
proj_dir = '/dss/open/ds000102'
results_dir=proj_dir + '/derivatives/'
!mkdir -p {results_dir}

#setup caching temp directory
cache_dir = '/dss/tmp/s102'
!mkdir -p {cache_dir}

# using glob to obtain subject_list
sub_dirs = glob.glob(os.path.join(proj_dir, 'sub-*'))
# extract the numeric ID and sort
subject_list = sorted(os.path.basename(d).split('-')[1] for d in sub_dirs)

# if multiple run, add run_list
run_list = ['1','2']

# infosource to iterate over subjects and runs
infosource = Node(IdentityInterface(fields=['subject_id', 'run']),
			name="infosource")
infosource.iterables = [('subject_id', subject_list), ('run', run_list)]
 

# template for SelectFiles (relative to proj_dir)

templates = {
	'func' : 'derivatives/fmriprep/sub-{subject_id}/func/sub-{subject_id}_task-flanker_run-{run}_space-MNI152NLin2009cAsym_desc-preproc_bold.nii.gz',
	'mask' : 'derivatives/fmriprep/sub-{subject_id}/func/sub-{subject_id}_task-flanker_run-{run}_space-MNI152NLin2009cAsym_desc-brain_mask.nii.gz',
	'events' : 'sub-{subject_id}/func/sub-{subject_id}_task-flanker_run-{run}_events.tsv',
	'confounds': 'derivatives/fmriprep/sub-{subject_id}/func/sub-{subject_id}_task-flanker_run-{run}_desc-confounds_timeseries.tsv',
}

# build Node
selectFiles = Node(
	SelectFiles(templates, base_directory=proj_dir),
	name ="selectFiles",
)

```

