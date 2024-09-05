# ST-P3 Command Guide

### Initial Setup
**1.** Clone the Repository:

    git clone https://github.com/OpenDriveLab/ST-P3.git

**2.** Create and Activate Conda Environment:
  - Create a Conda environment named `st-p3`:

    ```bash
    conda env create -f environment.yml -n st-p3
    ```
          
  - Activate the environment:

    ```bash
    conda activate st-p3
    ```

**3.** Prepare Data Directory:
  - Create a directory path `ST-P3/data/Nuscenes`.
  
  - Download the following data from nuscenes.org/download into the Nuscenes directory:

    - `Metadata [US, Asia] (0.43 GB)`

    - `File blobs of 85 scenes, parts 1-10 [US, Asia] (totals ~284.17 GB)`

    - `Map and CAN bus expansion packs [US, Asia]`
   
    This process may take up to 2 days due to the size of the files.

### Configuration Adjustments

Modify Configuration Files:

  - Navigate to `ST-P3/stp3/configs/nuscenes/Perception.yml`

  - Adjust the GPUs configuration from `GPUS: [0,1,2,3,4]` to `GPUS: [0]` due to using a single GPU (RTX 4090).

  - Optionally, adjust `EPOCHS` to 20 (default setting).

### Training Modules

  - (Recommended) Perception Module Pretraining:

    ```bash
    bash scripts/train_perceive.sh ${configs} ${dataroot}
    ```

  - (Optional) Prediction Module Training (for training purposes, no need for end-to-end):

    ```bash
    bash scripts/train_prediction.sh ${configs} ${dataroot} ${pretrained}
    ```

  - Entire Model End-to-End Training:

    ```bash
    bash scripts/train_plan.sh ${configs} ${dataroot} ${pretrained}
    ```

### Execute Perception Training:

    bash scripts/train_perceive.sh stp3/configs/nuscenes/Perception.yml data/Nuscenes

Each epoch takes around 6 hours, and the total training may last 6-7 days. Checkpoints are saved in `ST-P3/tensorboard_logs`.

### Set Up on a Second PC for Efficiency and Faster Processing

**1.** Repeat the setup process on another PC with an RTX 3090 GPU for parallel processing.

**2.** Transfer the perception-trained epochs to the new PC to utilize the additional GPU for further processing.

### Planning and Evaluation

**1.** Execute Planning on the First PC with a Better GPU (RTX 4090):

- Due to the superior processing power of the RTX 4090 compared to the RTX 3090, the first PC is used for planning to leverage faster computation and manage larger data loads.
          
- Adjust `GPUS` to `[0]` and `BATCHSIZE` to 1 in `ST-P3/stp3/configs/nuscenes/Planning.yml` to resolve CUDA out-of-memory errors.

    ```bash
    bash scripts/train_plan.sh stp3/configs/nuscenes/Planning.yml data/Nuscenes ST-P3/tensorboard_logs/30August2024at13_22_38KST_SimulationPC_Perception/default/version_0/checkpoints/last.ckpt
    ```

Planning takes approximately one week, and the results are critical for subsequent prediction and evaluation tasks.

**2.** Run Evaluation:

- Recommended:

    ```bash
    bash scripts/eval_plan.sh ${checkpoint} ${dataroot}
    ```

- Executed command:

    ```bash
    bash scripts/eval_plan.sh ST-P3/tensorboard_logs/17September2024at18_15_56KST_SimulationPC_Prediction data/Nuscenes
    ```

Evaluation typically takes up to 1 hour, displaying results on the terminal and saving HD maps to `ST-P3/imgs`.

### Running Prediction Module

**1.** Execute Prediction Training:

- Adjust the GPUs configuration from `GPUS: [0,1,2,3,4]` to `GPUS: [0]` in the configuration file.

    ```bash
    bash scripts/train_prediction.sh stp3/configs/nuscenes/Prediction.yml data/Nuscenes ST-P3/tensorboard_logs/30August2024at13_22_38KST_SimulationPC_Perception/default/version_0/checkpoints/last.ckpt
    ```

This process also takes about a week.

**2.** Planning with Prediction Results:

- Recommended:

    ```bash
    bash scripts/eval_plan.sh ${checkpoint} ${dataroot}
    ```

- Executed command:

    ```bash
    bash scripts/eval_plan.sh ST-P3/tensorboard_logs/17September2024at18_15_56KST_SimulationPC_Prediction data/Nuscenes
    ```

Results and HD maps will be available in directories like `ST-P3/imgs`.
