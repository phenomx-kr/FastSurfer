[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Deep-MI/FastSurfer/blob/master/Tutorial/Tutorial_FastSurferCNN_QuickSeg.ipynb)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Deep-MI/FastSurfer/blob/master/Tutorial/Complete_FastSurfer_Tutorial.ipynb)

# Overview

This directory contains all information needed to run FastSurfer - a fast and accurate deep-learning based neuroimaging pipeline.  This approach provides a full FreeSurfer alternative for volumetric analysis (within 1 minute)  and  surface-based  thickness  analysis  (within  only  around  1h  run  time). The whole pipeline consists of two main parts:

(i) FastSurferCNN - an advanced deep learning architecture capable of whole brain segmentation into 95 classes in under
1 minute, mimicking FreeSurfer’s anatomical segmentation and cortical parcellation (DKTatlas)

(ii) recon-surf - full  FreeSurfer  alternative for cortical surface reconstruction, mapping of cortical labels and traditional point-wise and ROI thickness analysis in approximately 60 minutes.

Image input requirements are identical to FreeSurfer: good quality T1-weighted MRI acquired at 3T with a resolution close to 1mm isotropic (slice thickness should not exceed 1.5mm). Preferred sequence is Siemens MPRAGE or multi-echo MPRAGE. GE SPGR should also work. Sub-mm scans (e.g. .75 or .8mm isotropic) will be downsampled by us automatically to 1mm isotropic, for example, we had success segmenting de-faced HCP data.

Within this repository, we provide the code and Docker files for running FastSurferCNN (segmentation only) and recon-surf (surface pipeline only) independently from each other or as a whole pipeline (run_fastsurfer.sh, segmentation + surface pipeline). For each of these purposes, see the README.md's in the corresponding folders.

![](/images/teaser.png)

## Usage
There are two ways to run FastSurfer - (a) as a native install or (b) using Docker. 

(a) For a __native install__ on a modern linux (e.g. Ubuntu 16.04 or Centos 7, or maybe higher), download this github repository (use git clone or download as zip and unpack) for the necessary source code and python scripts. You also need to have the necessary python 3 libraries installed (see __requirements.txt__) as well as bash-4.0 or higher. This is already enough to generate the whole-brain segmentation using FastSurferCNN (see the README.md in the FastSurferCNN directory for the exact commands). In order to run the whole FastSurfer pipeline locally on your machine, a working version of __FreeSurfer__ (v6.0, https://surfer.nmr.mgh.harvard.edu/fswiki/rel6downloads) is needed (specifically to run recon-surf). See [Example 1](#example-1:-fastSurfer-on-subject1) and [Example 2](#example-2:-fastSurfer-on-multiple-subjects-(parallel-processing)) for an illustration of the commands to run the entire FastSurfer pipeline (FastSurferCNN + recon-surf) natively.

(b) For a __docker version__, simply use the provided Dockerfiles in our Docker directory to build your image (see the README.md in the Docker directory). No other local installations are needed (FreeSurfer and everything else will be included, you only need to provide a FreeSurfer license file). We will also make a Docker image available on Dockerhub in the near future (probably with the official release of version 1.0, the current release is beta). See [Example 3](#example-3:-fastSurfer-inside-docker) for an example how to run FastSurfer inside a Docker container.

The main script called __run_fastsurfer.sh__ can be used to run both FastSurferCNN and recon-surf sequentially on a given subject. There are a number of options which can be selected and set via the command line. 
List them by running the following command:
```bash
./run_fastsurfer.sh --help
```

### Required arguments
* --sd: Output directory \$SUBJECTS_DIR (equivalent to FreeSurfer setup --> $SUBJECTS_DIR/sid/mri; $SUBJECTS_DIR/sid/surf ... will be created).
* --sid: Subject ID for directory inside \$SUBJECTS_DIR to be created ($SUBJECTS_DIR/sid/...)
* --t1: T1 full head input (not bias corrected, global path). The network was trained with conformed images (UCHAR, 256x256x256, 1 mm voxels and standard slice orientation). These specifications are checked in the eval.py script and the image is automatically conformed if it does not comply.

### Requiered for docker
* --fs_license: Path to FreeSurfer license key file. Register (for free) at https://surfer.nmr.mgh.harvard.edu/registration.html to obtain it if you do not have FreeSurfer installed so far. Strictly necessary if you use Docker, optional for local install (your local FreeSurfer license will automatically be used)

### Network specific arguments (optional)
* --seg: Global path with filename of segmentation (where and under which name to store it). Default location: $SUBJECTS_DIR/$sid/mri/aparc.DKTatlas+aseg.deep.mgz
* --weights_sag: Pretrained weights of sagittal network. Default: ../checkpoints/Sagittal_Weights_FastSurferCNN/ckpts/Epoch_30_training_state.pkl
* --weights_ax: Pretrained weights of axial network. Default: ../checkpoints/Axial_Weights_FastSurferCNN/ckpts/Epoch_30_training_state.pkl
* --weights_cor: Pretrained weights of coronal network. Default: ../checkpoints/Coronal_Weights_FastSurferCNN/ckpts/Epoch_30_training_state.pkl
* --seg_log: Name and location for the log-file for the segmentation (FastSurferCNN). Default: $SUBJECTS_DIR/$sid/scripts/deep-seg.log
* --clean_seg: Flag to clean up FastSurferCNN segmentation
* --run_viewagg_on: Define where the view aggregation should be run on. 
By default, the program checks if you have enough memory to run the view aggregation on the gpu. 
                    The total memory is considered for this decision. 
                    If this fails, or you actively overwrote the check with setting "--run_viewagg_on cpu", view agg is run on the cpu. 
                    Equivalently, if you define "--run_viewagg_on gpu", view agg will be run on the gpu (no memory check will be done).
* --no_cuda: Flag to disable CUDA usage in FastSurferCNN (no GPU usage, inference on CPU)
* --batch: Batch size for inference. Default: 16. Lower this to reduce memory requirement
* --order: Order of interpolation for mri_convert T1 before segmentation (0=nearest, 1=linear(default), 2=quadratic, 3=cubic)

### Surface pipeline arguments (optional)
* --fstess: Use mri_tesselate instead of marching cube (default) for surface creation
* --fsqsphere: Use FreeSurfer default instead of novel spectral spherical projection for qsphere
* --fsaparc: Use FS aparc segmentations in addition to DL prediction (slower in this case and usually the mapped ones from the DL prediction are fine)
* --surfreg: Run Surface registration with FreeSurfer (for cross-subject correspondence)
* --parallel: Run both hemispheres in parallel
* --threads: Set openMP and ITK threads to <int>

### Other
* --py: which python version to use. Default: python3.6
* --seg_only: only run FastSurferCNN (generate segmentation, do not run the surface pipeline)
* --surf_only: only run the surface pipeline recon_surf. The segmentation created by FastSurferCNN must already exist in this case.
    

### Example 1: FastSurfer on subject1 (with parallel processing of hemis)

Given you want to analyze data for subject1 which is stored on your computer under /home/user/my_mri_data/subject1/orig.mgz, run the following command from the console (do not forget to source FreeSurfer!):

```bash
# Source FreeSurfer
export FREESURFER_HOME=/path/to/freesurfer/fs60
source $FREESURFER_HOME/SetUpFreeSurfer.sh

# Define data directory
datadir=/home/user/my_mri_data
fastsurferdir=/home/user/my_fastsurfer_analysis

# Run FastSurfer
./run_fastsurfer.sh --t1 $datadir/subject1/orig.mgz \
                    --sid subject1 --sd $fastsurferdir \
                    --parallel --threads 4
```

The output will be stored in the $fastsurferdir (including the aparc.DKTatlas+aseg.deep.mgz segmentation under $fastsurferdir/subject1/mri (default location)). Processing of the hemispheres will be run in parallel (--parallel flag). Omit this flag to run the processing sequentially.

### Example 2: FastSurfer on multiple subjects

In order to run FastSurfer on a number of cases which are stored in the same directory, prepare a subjects_list.txt file listing the names line per line:
subject1\n
subject2\n
subject3\n
...
subject10\n

And invoke the following command (make sure you have enough ressources to run the given number of subjects in parallel!):

```bash
export FREESURFER_HOME=/path/to/freesurfer/fs60
source $FREESURFER_HOME/SetUpFreeSurfer.sh

cd /home/user/FastSurfer
datadir=/home/user/my_mri_data
fastsurferdir=/home/user/my_fastsurfer_analysis
mkdir -p $fastsurferdir/logs # create log dir for storing nohup output log (optional)

while read p ; do
  echo $p
  nohup ./run_fastsurfer.sh --t1 $datadir/$p/orig.mgz 
                            --sid $p --sd $fastsurferdir > $fastsurferdir/logs/out-${p}.log &
  sleep 90s 
done < ./data/subjects_list.txt
```

### Example 3: FastSurfer inside Docker
After building the Docker (see instructions in ./Docker/README.md), you do not need to have a separate installation of FreeSurfer on your computer (included in the Docker). However, you need to register at the FreeSurfer website (https://surfer.nmr.mgh.harvard.edu/registration.html) to acquire a valid license (for free). This license need to be passed to the script via the --fs_license flag.

To run FastSurfer on a given subject using the provided Docker, execute the following command:

```bash
docker run --gpus all -v /home/user/my_mri_data:/data \
                      -v /home/user/my_fastsurfer_analysis:/output \
                      -v /home/user/my_fs_license_dir:/fs60 \
                      --rm --user XXXX fastsurfer:gpu \
                      --fs_license /fs60/.license \
                      --t1 /data/subject2/orig.mgz \
                      --sid subject2 --sd /output \
                      --parallel
```

* The fs_license points to your FreeSurfer license which needs to be available on your computer (e.g. in the /home/user/my_fs_license_dir folder). 
* The --gpus flag is used to access GPU resources. With it you can also specify how many GPUs to use. In the example above, _all_ will use all available GPUS. To use a single one (e.g. GPU 0), set --gpus device=0. To use multiple specific ones (e.g. GPU 0, 1 and 3), set --gpus '"device=0,1,3"'.
* The -v command mounts your data (and output) directory into the docker image. Inside it is visible under the name following the colon (in this case /data or /output).
* The --rm flag takes care of removing the container once the analysis finished. 
* The --user XXXX part should be changed to the appropriate user id (a four digit number; can be checked with the command "id -u" on linux systems). All generated files will then belong to the specified user. Without the flag, the docker container will be run as root.

## References

If you use this for research publications, please cite:

Henschel L, Conjeti S, Estrada S, Diers K, Fischl B, Reuter M, FastSurfer - A fast and accurate deep learning based neuroimaging pipeline, NeuroImage 219 (2020), 117012. https://doi.org/10.1016/j.neuroimage.2020.117012

## Acknowledgements
The recon-surf pipeline is largely based on FreeSurfer including mris_make_surfaces which we bundle in binary form to patch a problem with the one in FreeSurfer 6.0.
https://surfer.nmr.mgh.harvard.edu/fswiki/FreeSurferMethodsCitation
