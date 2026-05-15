---
title: 6.1 NiPype Basics
draft: true
tags:
---

## Nipype

Nipype (Neuroimaging in Python Pipelines and Interfaces) is an open-source framework designed to help you build, share, and run complex neuroimaging workflows in a reproducible, modular way. At its core it provides:

- **Interfaces**
    Lightweight Python wrappers around popular neuroimaging tools (FSL, SPM, AFNI, FreeSurfer, ANTs, etc.).  Each interface exposes the tool’s inputs and outputs as Python attributes, so you can call, say, Bet() or Fast() just like a Python function.
- **Nodes & Workflows**
    You wrap each interface in a **Node**, then connect Nodes together in a **Workflow**, declaring data dependencies through named outputs and inputs.  Workflows can be nested, allowing you to build pipelines of arbitrary complexity.
- **Execution Engines**
    Nipype can execute workflows locally (serial or multi-core), on clusters (SGE, SLURM, PBS), or in the cloud, all with identical code—just change the execution “plugin.”
- **Data Grabbing & Selection**
    Utilities like SelectFiles (to wildcard through BIDS-style folder structures) and DataGrabber let you grab inputs dynamically based on subject/session/run iterables.
- **Provenance & Reporting**
    Every node invocation records command‐line calls, software versions, and runtime metadata.  You can generate execution graphs, audit trails, and HTML reports to help with reproducibility.

A better place to learn Nipype is Miyakel's [Nipype Tutorial](https://miykael.github.io/nipype_tutorial/).
### 1. Interfaces

Interfaces are Python wrappers around command-line neuroimaging tools (FSL, AFNI, ANTs, SPM, FreeSurfer, etc.), exposing each tool’s inputs and outputs as Python attributes so you can script analyses in pure Python.

For example, you want to use bet() from FSL to extract brain:

```python
from nipype.interfaces.fsl import BET

# 1. Instantiate the interface
bet = BET()

# 2. Tell it which file to use, and tweak parameters
bet.inputs.in_file  = 'sub-01_T1w.nii.gz'      # input anatomical image
bet.inputs.frac     = 0.5                     # intensity threshold
bet.inputs.robust   = True                    # more robust center-finding
bet.inputs.out_file = 'sub-01_T1w_brain.nii.gz'  # skull-stripped output / optional

# 3. Run it!
res = bet.run()

# 4. Inspect the outputs
print(res.outputs)
```

### 2. Nodes

A **Node** in Nipype is a simple Python object that wraps an interface (i.e. a command‐line tool) and gives it a unique name. Nodes let you:

- **Encapsulate** one processing step (e.g. skull-strip, motion correct, etc.)
- **Hold** the interface instance and its .inputs in one place
- **Run** either standalone or later inside a Workflow
- **Expose** outputs via .outputs after running

For example, the above BET() interface wrapped in a node:

```python
from nipype import Node
from nipype.interfaces.fsl import BET

# 1. Create a Node: specify the interface and give it a name
bet_node = Node(
    BET(frac=0.5, robust=True),  # interface + default parameters
    name='skullstrip'            # unique node name
)

# 2. Set the inputs just like with an interface
bet_node.inputs.in_file  = 'sub-01_T1w.nii.gz'
bet_node.inputs.out_file = 'sub-01_T1w_brain.nii.gz'

# 3. Run the node
res = bet_node.run()

# 4. Access outputs
print(res.outputs.out_file)      # e.g. 'sub-01_T1w_brain.nii.gz'
print(res.outputs.mask_file)     # e.g. 'sub-01_T1w_brain_mask.nii.gz'
```

### 3. Workflow

A **Workflow** in Nipype is simply a directed graph of **Nodes**: it specifies _how_ the output of one processing step feeds into the input of another.  It also manages execution (locally or on a cluster), logging, and provenance for the whole pipeline.

- **Define** a Workflow(name=?, base_dir=?) to hold your graph
- **Connect** Nodes via wf.connect() so that outputs flow into inputs
- **Set** a base_dir to control where all logs, crash files, and derived outputs go
- **Run** with wf.run() (and later you can swap in different execution plugins)

A minimal example: skull-strip + tissue segmentation

```python
from nipype import Node, Workflow
from nipype.interfaces.fsl import BET, FAST

# 1. Create your Nodes
skullstrip = Node(
    BET(frac=0.5, robust=True),
    name='skullstrip'
)
skullstrip.inputs.in_file = 'sub-01_T1w.nii.gz'

segment = Node(
    FAST(img_type=1, segments=True),
    name='segment'
)
# (no need to set segment.inputs.in_files here — we’ll connect it)

# 2. Build the Workflow
wf = Workflow(
    name='t1_preproc',
    base_dir='.'   # all logs & outputs go under ./t1_preproc/
)

# 3. Wire up the graph: pass skullstrip’s brain → FAST’s in_files
wf.connect([
    (skullstrip, segment, [
        ('out_file', 'in_files')      # skullstrip.outputs.out_file → segment.inputs.in_files
    ])
])

# 4. Run it
wf.run()  # by default uses a MultiProc plugin on your local machine
```

The above script will run skull strip to produce a sub-01_T1w_brain.nii.gz, and it then call `segment.run()`, feeding that brain image into FAST. All commands, stdout/stderr, and verion info get logged under `./t1_preproc`. 

### 4. Data Inputs

MRI analyses often involve many data, which you need to select based on their naming patterns. Nipype provides many different modules to grab or select the data, such as DataGrabber and `SelectFiles`. 

A common usage is using `SelectFiles`. You declare a small dict of glob-style template with {subject_id}, {session}, {run}, etc., and Nipype does the rest. 

An example:
```python
from nipype.interfaces.io import SelectFiles
from nipype import Node

templates = {
    'bold'   : 'derivatives/fmriprep/sub-{subject_id}/func/sub-{subject_id}_task-flanker_run-{run}_space-MNI152NLin2009cAsym_desc-preproc_bold.nii.gz',
    'events' :      'sub-{subject_id}/func/sub-{subject_id}_task-flanker_run-{run}_events.tsv',
}

selectfiles = Node(
    SelectFiles(templates, base_directory=proj_dir),
    name='selectfiles'
)
selectfiles.inputs.subject_id = '02'
selectfiles.inputs.run        = '1'
res = selectfiles.run()
print(vars(res.outputs))
```

However, to run the same process on multiple subjects or to do statistical inference on multiple files, you need to loop through all participants etc. Nipype has an execution plugin for Workflow, called __iterables__. 

```python
isosmooth = Node(IsotropicSmooth(), name='iso_smooth')
isosmooth.iterables = ("fwhm", [4,8,16])
```

Another is **IdentityInterface**, a tiny utility interface that doesn’t wrap any external program—it simply defines one or more named fields you can set or iterate over.  It’s most commonly used as an “infosource” to feed parameters (subjects, sessions, runs, etc.) into your workflow without needing real data I/O. 

```python
from nipype import Node
from nipype.interfaces.utility import IdentityInterface

# 1. Define which parameters you need
infosource = Node(
    IdentityInterface(fields=['subject_id', 'run']),
    name='infosource'
)

# 2. Tell it what to loop over
infosource.iterables = [
    ('subject_id', ['01', '02', '03']),
    ('run'       , ['1', '2'])
]

# Now each time the workflow executes, infosource.outputs.subject_id and
# infosource.outputs.run will take on one combination of values,
# which you can connect into SelectFiles, analysis nodes, etc.
```
### 5. Data Output with DataSink

A **DataSink** is Nipype’s simple output router: it takes the outputs of your Nodes or Workflows and writes them into a clean, organized folder hierarchy. You point it at a base directory, tell it what “container” (sub-folder) to use (e.g. your subject or workflow name), and optionally give it filename **substitutions** to tidy up autogenerated names.

An example from the [tutorial](https://miykael.github.io/nipype_tutorial/notebooks/basic_data_output.html):

```python
from nipype.interfaces.io import DataSink

# Create DataSink object
sinker = Node(DataSink(), name='sinker')

# Name of the output folder
sinker.inputs.base_directory = '/output/working_dir/preprocWF_output'

# Connect DataSink with the relevant nodes
wf.connect([(smooth, sinker, [('out_file', 'in_file')]),
            (mcflirt, sinker, [('mean_img', 'mean_img'),
                               ('par_file', 'par_file')]),
            ])
wf.run()
```

You can also output all files in one folder `preproc` by replacing the final `connect` with 

```python
wf.connect([(smooth, sinker, [('out_file', 'preproc.@in_file')]),
            (mcflirt, sinker, [('mean_img', 'preproc.@mean_img'),
                               ('par_file', 'preproc.@par_file')]),
            ])
wf.run()
```

