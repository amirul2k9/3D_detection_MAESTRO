[comment]: # (<img src="docs/open_mmlab.png" align="right" width="30%">)

# 3D Human Detection for MAESTRO Project

This repo is forked from [`OpenPCDet`](https://github.com/open-mmlab/OpenPCDet). The origional code was written to work in Autonomous Driving applications. This code is used in [Situation Awareness for Automated Surgical Check-listing in AI-Assisted Operating Room](https://arxiv.org/abs/2209.05056) and it is customised to detect humans in 3D Lidar pointclouds inside the Operating room, where humans pointclouds are more dense and larger than those in the AV application. This repo explains the customisation procedure and how to train and test the model on a customised dataset. 

## Overview
- [Installation](#installation)
- [Data](#data)
- [Code Customisation](#customisation)
- [Training](#training)
- [Testing](#testing)
- [Citation](#citation)


## Installation

<details>
<summary>Requirements</summary>
  
All the codes are tested in the following environment:
* Linux (tested on Ubuntu 18.04)
* Python 3.8
* PyTorch 1.9 
* CUDA 10.2
* [`spconv v1.0 (commit 8da6f96)`](https://github.com/traveller59/spconv/tree/8da6f967fb9a054d8870c3515b1b44eca2103634) or [`spconv v1.2`](https://github.com/traveller59/spconv) or [`spconv v2.x`](https://github.com/traveller59/spconv)
    Foldable Content[enter image description here][1]
</details>


<details>
<summary>Install pcdet</summary>

NOTE: Please re-install `pcdet v0.5` by running `python setup.py develop` even if you have already installed previous version.

a. Clone this repository.
```shell
git clone https://github.com/IzzeddinTeeti/3D_detection_MAESTRO.git
```

b. Install the dependent libraries as follows:

[comment]: <> (* Install the dependent python libraries: )

[comment]: <> (```)

[comment]: <> (pip install -r requirements.txt )

[comment]: <> (```)

* Install the SparseConv library, we use the implementation from [`[spconv]`](https://github.com/traveller59/spconv). 
    * If you use PyTorch 1.1, then make sure you install the `spconv v1.0` with ([commit 8da6f96](https://github.com/traveller59/spconv/tree/8da6f967fb9a054d8870c3515b1b44eca2103634)) instead of the latest one.
    * If you use PyTorch 1.3+, then you need to install the `spconv v1.2`. As mentioned by the author of [`spconv`](https://github.com/traveller59/spconv), you need to use their docker if you use PyTorch 1.4+. 
    * You could also install latest `spconv v2.x` with pip, see the official documents of [spconv](https://github.com/traveller59/spconv).
  
c. Install this `pcdet` library and its dependent libraries by running the following command:
```shell
python setup.py develop
```
</details>



## Data
### Data Collecting
The pointclouds were generated using 8 kinect cameras ditribuated inside an opertaing room. The pointclouds coming from the 8 cameras were combined then downsampled. 

### Data Format
The initial format of the data is `.ply` but it was converted to `.pcd` in order to uploaded to [Supervisely](https://supervise.ly/) (the online pointcloud annotation platform). Supervisely outputs the annotation format as `.json` format.

### Data Conversion
Both the data and the annotaions were converted using [this code](https://github.com/IzzeddinTeeti/Convert-supervisely-to-KITTI) to other formats in order to work with the detection code. The data were conveted from `.pcd` to `.bin` and the annotation were converted from `.json` to `.txt`. 



## Customisation 
To customise the code, follow these steps:
1. Move the custom dataset into the right directory, as follows:

```
3D_detection_MAESTRO
├── data
│   ├── custom
│   │   │── ImageSets
│   │   │   │── train.txt
│   │   │   │── val.txt
│   │   │── points
│   │   │   │── 000000.npy
│   │   │   │── 999999.npy
│   │   │── labels
│   │   │   │── 000000.txt
│   │   │   │── 999999.txt
├── pcdet
├── tools
```
Dataset splits need to be pre-defined and placed in `ImageSets`

2. In [`custom_dataset.yaml`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/dataset_configs/custom_dataset.yaml) change the following lines:
- [`POINT_CLOUD_RANGE: [-3.2, -1.6, -3.2, 3.2, 1.6, 3.2]`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/dataset_configs/custom_dataset.yaml#L4), the pointcloud range is in the format [X1, Y1, Z1, X2, Y2, Z2].
- [`MAP_CLASS_TO_KITTI: {'human': 'Pedestrian'}`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/dataset_configs/custom_dataset.yaml#L6-L8)
- [`VOXEL_SIZE: [0.05, 0.1, 0.05]`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/dataset_configs/custom_dataset.yaml#L63), IMPORTANT: range divided by voxelsize must be multiple of 16.

3. In 'custom models' folder, change the following lines in [`pv_rcnn.yaml`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/custom_models/pv_rcnn.yaml) file
- [`CLASS_NAMES: ['human']`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/custom_models/pv_rcnn.yaml#L1)
- [`_BASE_CONFIG_: ../dataset_configs/custom_dataset.yaml`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/custom_models/pv_rcnn.yaml#L4), the directory for `custom_dataset.yaml`
- [`NUM_BEV_FEATURES: 896`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/custom_models/pv_rcnn.yaml#L17), this is one of the trickest changes. It is related to the tensor size and depends on the type of the custom data, if the code throws an error related to the tensor size, copy the number in the error message and paste it here.
- [`'anchor_sizes': [[0.8, 2, 0.8]],`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/tools/cfgs/custom_models/pv_rcnn.yaml#L40) for 'human' class.
  
4. In [`custom_dataset.py`](https://github.com/IzzeddinTeeti/3D_detection_MAESTRO/blob/master/pcdet/datasets/custom/custom_dataset.py) change one line to 
  ```
  create_custom_infos(
            dataset_cfg=dataset_cfg,
            class_names=['human'],
            data_path=ROOT_DIR / 'data' / 'custom',
            save_path=ROOT_DIR / 'data' / 'custom',
        )
  ```

## Training
To train the model (PV-RCNN in our case), follow these steps:
1. Generate the data info by running the following code while being in the main directory
  ```
  python -m pcdet.datasets.custom.custom_dataset create_custom_infos tools/cfgs/dataset_configs/custom_dataset.yaml
  ```
2. Change the directory to 'tools' then run `train.py` using the following scipt
  ```
  CUDA_VISIBLE_DEVICES=1 python train.py --cfg_file cfgs/custom_models/pv_rcnn.yaml --batch_size 2 --workers 1 --epochs 300
  ```
  

## Testing


## Citation 
If you find this project useful in your research, please consider cite:
```
  @misc{onyeogulu2022situation,
    title={Situation Awareness for Automated Surgical Check-listing in AI-Assisted Operating Room},
    author={Tochukwu Onyeogulu and Salman Khan and Izzeddin Teeti and Amirul Islam and Kaizhe Jin and Adrian Rubio-Solis and Ravi Naik and George Mylonas and Fabio Cuzzolin},
    year={2022},
    eprint={2209.05056},
    archivePrefix={arXiv},
    primaryClass={cs.CV}
}
```
To cite the origional OpenPCDet:

```
@misc{openpcdet2020,
    title={OpenPCDet: An Open-source Toolbox for 3D Object Detection from Point Clouds},
    author={OpenPCDet Development Team},
    howpublished = {\url{https://github.com/open-mmlab/OpenPCDet}},
    year={2020}
}
```




