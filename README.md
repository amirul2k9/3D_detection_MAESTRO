[comment]: # (<img src="docs/open_mmlab.png" align="right" width="30%">)

# 3D Human Detection for MAESTRO Project

This repo is forked from [`OpenPCDet`](https://github.com/open-mmlab/OpenPCDet). The origional code was written to work in Autonomous Driving applications. This code is customised to detect humans in 3D Lidar pointclouds inside the Operating room, where humans pointclouds are more dense and larger than those in the AV application. This repo explains the customisation procedure and how to train and test the model on a customised dataset. 

## Overview
- [Installation](#installation)
- [Data](#Data)
- [Quick Demo](docs/DEMO.md)
- [Getting Started](docs/GETTING_STARTED.md)
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



## Citation 
If you find this project useful in your research, please consider cite:


```
@misc{openpcdet2020,
    title={OpenPCDet: An Open-source Toolbox for 3D Object Detection from Point Clouds},
    author={OpenPCDet Development Team},
    howpublished = {\url{https://github.com/open-mmlab/OpenPCDet}},
    year={2020}
}
```

## Contribution
Welcome to be a member of the OpenPCDet development team by contributing to this repo, and feel free to contact us for any potential contributions. 


