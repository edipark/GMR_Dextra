# Motion Retargeting for Dextra Robot

This repository provides motion retargeting capabilities for the **Dextra** humanoid robot using the [GMR (General Motion Retargeting)](https://github.com/YanjieZe/GMR) framework.

  <a href="https://arxiv.org/abs/2505.02833">
    <img src="https://img.shields.io/badge/paper-arXiv%3A2505.02833-b31b1b.svg" alt="arXiv Paper"/>
  </a> <a href="https://arxiv.org/abs/2510.02252">
    <img src="https://img.shields.io/badge/paper-arXiv%3A2510.02252-b31b1b.svg" alt="arXiv Paper"/>
  </a> <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"/>
  </a>

![Banner for GMR](./assets/GMR.png)

![GMR Pipeline](./assets/GMR_pipeline.png)

## About Dextra

This repository is configured for the **Dextra** humanoid robot with two variants:

- **`Dextra`**: Full-body humanoid robot with arms and legs
- **`Dextra_lowerbody`**: Lower-body only configuration (legs only)

Both variants are fully integrated with the GMR motion retargeting system, allowing you to retarget human motion data (SMPL-X, BVH, FBX formats) to Dextra robot motions.

## Key Features

- **Real-time high-quality retargeting** for Dextra robot
- Support for multiple human motion data formats:
  - SMPL-X (from [AMASS](https://amass.is.tue.mpg.de/) and [OMOMO](https://github.com/lijiaman/omomo_release))
  - BVH (from [LAFAN1](https://github.com/ubisoft/ubisoft-laforge-animation-dataset) and [Nokov](https://www.nokov.com/))
  - FBX (from [OptiTrack](https://www.optitrack.com/))
- MuJoCo-based visualization and simulation
- Optimized IK solver for natural motion transfer

## Installation

> [!NOTE]
> The code is tested on Ubuntu 22.04/20.04.

First create your conda environment:

```bash
conda create -n gmr python=3.10 -y
conda activate gmr
```

Then, install the package:

```bash
pip install -e .
```

After installing SMPLX, change `ext` in `smplx/body_models.py` from `npz` to `pkl` if you are using SMPL-X pkl files.

And to resolve some possible rendering issues:

```bash
conda install -c conda-forge libstdcxx-ng -y
```

## Data Preparation

### SMPL-X Body Model

Download SMPL-X body models to `assets/body_models` from [SMPL-X](https://smpl-x.is.tue.mpg.de/) and structure as follows:

```bash
- assets/body_models/smplx/
-- SMPLX_NEUTRAL.pkl
-- SMPLX_FEMALE.pkl
-- SMPLX_MALE.pkl
```

### Human Motion Data

- **[AMASS](https://amass.is.tue.mpg.de/) motion data**: Download raw SMPL-X data to any folder you want from [AMASS](https://amass.is.tue.mpg.de/). NOTE: Do not download SMPL+H data.

- **[OMOMO](https://github.com/lijiaman/omomo_release) motion data**: Download raw OMOMO data to any folder you want from [this google drive file](https://drive.google.com/file/d/1tZVqLB7II0whI-Qjz-z-AU3ponSEyAmm/view?usp=sharing). And process the data into the SMPL-X format using `scripts/convert_omomo_to_smplx.py`.

- **[LAFAN1](https://github.com/ubisoft/ubisoft-laforge-animation-dataset) motion data**: Download raw LAFAN1 bvh files from [the official repo](https://github.com/ubisoft/ubisoft-laforge-animation-dataset), i.e., [lafan1.zip](https://github.com/ubisoft/ubisoft-laforge-animation-dataset/blob/master/lafan1/lafan1.zip).

## Usage for Dextra

### Retargeting from SMPL-X (AMASS, OMOMO) to Dextra

> [!NOTE]
> NOTE: after install SMPL-X, change `ext` in `smplx/body_models.py` from `npz` to `pkl` if you are using SMPL-X pkl files.

Retarget a single motion to Dextra:

```bash
# Full-body Dextra
python scripts/smplx_to_robot.py --smplx_file <path_to_smplx_data> --robot Dextra --save_path <path_to_save_robot_data.pkl> --rate_limit

# Lower-body only Dextra
python scripts/smplx_to_robot.py --smplx_file <path_to_smplx_data> --robot Dextra_lowerbody --save_path <path_to_save_robot_data.pkl> --rate_limit
```

By default you should see the visualization of the retargeted robot motion in a mujoco window.
If you want to record video, add `--record_video` and `--video_path <your_video_path.mp4>`.

- `--rate_limit` is used to limit the rate of the retargeted robot motion to keep the same as the human motion. If you want it as fast as possible, remove `--rate_limit`.

Retarget a folder of motions:

```bash
python scripts/smplx_to_robot_dataset.py --src_folder <path_to_dir_of_smplx_data> --tgt_folder <path_to_dir_to_save_robot_data> --robot Dextra
```

By default there is no visualization for batch retargeting.

### Retargeting from BVH (LAFAN1, Nokov) to Dextra

Retarget a single motion:

```bash
# single motion
python scripts/bvh_to_robot.py --bvh_file <path_to_bvh_data> --robot Dextra --save_path <path_to_save_robot_data.pkl> --rate_limit --format <format>
```

By default you should see the visualization of the retargeted robot motion in a mujoco window. 
- `--rate_limit` is used to limit the rate of the retargeted robot motion to keep the same as the human motion. If you want it as fast as possible, remove `--rate_limit`.
- `--format` is used to specify the format of the BVH data. Supported formats are `lafan1` and `nokov`.

Retarget a folder of motions:

```bash
python scripts/bvh_to_robot_dataset.py --src_folder <path_to_dir_of_bvh_data> --tgt_folder <path_to_dir_to_save_robot_data> --robot Dextra
```

By default there is no visualization for batch retargeting.

### Retargeting from FBX (OptiTrack) to Dextra

#### Offline FBX Files

Retarget a single motion:

1. Install `fbx_sdk` by following [these instructions](https://github.com/nv-tlabs/ASE/tree/main/ase/poselib#importing-from-fbx) and [these instructions](https://github.com/nv-tlabs/ASE/issues/61#issuecomment-2670315114). You will probably need a new conda environment for this.

2. Activate the conda environment where you installed `fbx_sdk`.
Use the following command to extract motion data from your `.fbx` file:

```bash
cd third_party
python poselib/fbx_importer.py --input <path_to_fbx_file.fbx> --output <path_to_save_motion_data.pkl> --root-joint <root_joint_name> --fps <fps>
```

3. Then, run the command below to retarget the extracted motion data to Dextra:

```bash
conda activate gmr
# single motion
python scripts/fbx_offline_to_robot.py --motion_file <path_to_saved_motion_data.pkl> --robot Dextra --save_path <path_to_save_robot_data.pkl> --rate_limit
```

By default you should see the visualization of the retargeted robot motion in a mujoco window. 

- `--rate_limit` is used to limit the rate of the retargeted robot motion to keep the same as the human motion. If you want it as fast as possible, remove `--rate_limit`.

#### Online Streaming

We provide the script to use OptiTrack MoCap data for real-time streaming and retargeting.

Usually you will have two computers, one is the server that installed with Motive (Desktop APP for OptiTrack) and the other is the client that installed with GMR.

Find the server ip (the computer that installed with Motive) and client ip (your computer). Set the streaming as follows:

![OptiTrack Streaming](./assets/optitrack.png)

And then run:

```bash
python scripts/optitrack_to_robot.py --server_ip <server_ip> --client_ip <client_ip> --use_multicast False --robot Dextra
```

You should see the visualization of the retargeted robot motion in a mujoco window.

### Visualize saved Dextra motion

Visualize a single motion:

```bash
python scripts/vis_robot_motion.py --robot Dextra --robot_motion_path <path_to_save_robot_data.pkl>
```

If you want to record video, add `--record_video` and `--video_path <your_video_path.mp4>`.

Visualize a folder of motions:

```bash
python scripts/vis_robot_motion_dataset.py --robot Dextra --robot_motion_folder <path_to_save_robot_data_folder>
```

After launching the MuJoCo visualization window and clicking on it, you can use the following keyboard controls:
* `[`: play the previous motion
* `]`: play the next motion
* `space`: toggle play/pause

## Human/Robot Motion Data Formulation

To better use this library, you can first have an understanding of the human motion data we use and the robot motion data we obtain.

Each frame of **human motion data** is formulated as a dict of (human_body_name, 3d global translation + global rotation).

Each frame of **robot motion data** can be understood as a tuple of (robot_base_translation, robot_base_rotation, robot_joint_positions).

## Pre-retargeted Motion Files

This repository includes pre-retargeted motion files for Dextra robot that you can use directly:

### Available Motion Files

- **`retarget_motion/dextra_walking_improved.pkl`** - Improved walking motion for Dextra robot
- **`retarget_motion/humanoid_walking.pkl`** - Humanoid walking motion

### Visualize Pre-retargeted Motions

You can visualize these pre-retargeted motions directly:

```bash
# Visualize Dextra walking motion
python scripts/vis_robot_motion.py --robot Dextra --robot_motion_path retarget_motion/dextra_walking_improved.pkl

# Visualize with video recording
python scripts/vis_robot_motion.py --robot Dextra --robot_motion_path retarget_motion/dextra_walking_improved.pkl --record_video --video_path videos/dextra_walking_improved.mp4
```

### Example Videos

- **`videos/Dextra_07_01_stageii.mp4`** - Full-body Dextra motion demonstration
- **`videos/Dextra_lowerbody_07_01_stageii.mp4`** - Lower-body only Dextra motion demonstration

## Dextra Configuration

The IK configuration files for Dextra are located in:
- `general_motion_retargeting/ik_configs/smplx_to_dextra.json` - Full-body configuration
- `general_motion_retargeting/ik_configs/smplx_to_dextra_lowerbody.json` - Lower-body only configuration

These configuration files define the mapping between human body parts and Dextra robot joints, including scaling factors and IK solver parameters optimized for Dextra's kinematics.

## Speed Benchmark

| CPU | Retargeting Speed |
| --- | --- |
| AMD Ryzen Threadripper 7960X 24-Cores | 60~70 FPS |
| 13th Gen Intel Core i9-13900K 24-Cores | 35~45 FPS |

## Citation

This repository is based on the GMR (General Motion Retargeting) framework. If you use this code, please cite:

```bibtex
@article{joao2025gmr,
  title={Retargeting Matters: General Motion Retargeting for Humanoid Motion Tracking},
  author= {Joao Pedro Araujo and Yanjie Ze and Pei Xu and Jiajun Wu and C. Karen Liu},
  year= {2025},
  journal= {arXiv preprint arXiv:2510.02252}
}
```

```bibtex
@article{ze2025twist,
  title={TWIST: Teleoperated Whole-Body Imitation System},
  author= {Yanjie Ze and Zixuan Chen and João Pedro Araújo and Zi-ang Cao and Xue Bin Peng and Jiajun Wu and C. Karen Liu},
  year= {2025},
  journal= {arXiv preprint arXiv:2505.02833}
}
```

and the original GMR repository:

```bibtex
@software{ze2025gmr,
  title={GMR: General Motion Retargeting},
  author= {Yanjie Ze and João Pedro Araújo and Jiajun Wu and C. Karen Liu},
  year= {2025},
  url= {https://github.com/YanjieZe/GMR},
  note= {GitHub repository}
}
```

## Known Issues

Designing a single config for all different humans is not trivial. We observe some motions might have bad retargeting results. If you observe some bad results, please let us know! We now have a collection of such motions in [TEST_MOTIONS.md](TEST_MOTIONS.md).

## Acknowledgement

This repository is based on [GMR (General Motion Retargeting)](https://github.com/YanjieZe/GMR) by Yanjie Ze and collaborators. The IK solver is built upon [mink](https://github.com/kevinzakka/mink) and [mujoco](https://github.com/google-deepmind/mujoco). The visualization is built upon [mujoco](https://github.com/google-deepmind/mujoco). The human motion data we use includes [AMASS](https://amass.is.tue.mpg.de/), [OMOMO](https://github.com/lijiaman/omomo_release), and [LAFAN1](https://github.com/ubisoft/ubisoft-laforge-animation-dataset).

---

## Original GMR Usage Examples

The following usage examples are from the original [GMR repository](https://github.com/YanjieZe/GMR) and demonstrate the general framework capabilities:

### Retargeting from GVHMR to Robot

First, install GVHMR by following [their official instructions](https://github.com/zju3dv/GVHMR/blob/main/docs/INSTALL.md).

And run their demo that can extract human pose from monocular video:

```bash
cd path/to/GVHMR
python tools/demo/demo.py --video=docs/example_video/tennis.mp4 -s
```

Then you should obtain the saved human pose data in `GVHMR/outputs/demo/tennis/hmr4d_results.pt`.

Then, run the command below to retarget the extracted human pose data to your robot:

```bash
python scripts/gvhmr_to_robot.py --gvhmr_pred_file <path_to_hmr4d_results.pt> --robot unitree_g1 --record_video
```

*Source: [GMR Repository](https://github.com/YanjieZe/GMR)*

---

## Supported Robots in Original GMR

The original GMR framework supports multiple humanoid robots. This repository focuses on Dextra, but the framework can be extended to other robots. For the full list of supported robots, see the [original GMR repository](https://github.com/YanjieZe/GMR).
