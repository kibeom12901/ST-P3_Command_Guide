#!/bin/bash

# ST-P3 Command Guide

# Initial Setup

# Clone the Repository
git clone https://github.com/OpenDriveLab/ST-P3.git

# Create and Activate Conda Environment
# Create a Conda environment named st-p3
conda env create -f environment.yml -n st-p3

# Activate the environment
conda activate st-p3

# Prepare Data Directory
# Create a directory path ST-P3/data/Nuscenes
mkdir -p ST-P3/data/Nuscenes

# Inform the user to download the following data from nuscenes.org/download into the Nuscenes directory:
echo "Download the following data from nuscenes.org/download into the Nuscenes directory:"
echo "- Metadata [US, Asia] (0.43 GB)"
echo "- File blobs of 85 scenes, parts 1-10 [US, Asia] (totals ~284.17 GB)"
echo "- Map and CAN bus expansion packs [US, Asia]"
echo "This process may take up to 2 days due to the size of the files."

# Configuration Adjustments
# Modify Configuration Files
# Inform the user to navigate to ST-P3/stp3/configs/nuscenes/Perception.yml
echo "Navigate to ST-P3/stp3/configs/nuscenes/Perception.yml and make the following adjustments:"
echo "Adjust the GPUs configuration from GPUS: [0,1,2,3,4] to GPUS: [0] for a single GPU (RTX 4090)."
echo "Optionally, adjust EPOCHS to 20 (default setting)."

# Training Modules

# (Recommended) Perception Module Pretraining
bash scripts/train_perceive.sh ${configs} ${dataroot}

# (Optional) Prediction Module Training
bash scripts/train_prediction.sh ${configs} ${dataroot} ${pretrained}

# Entire Model End-to-End Training
bash scripts/train_plan.sh ${configs} ${dataroot} ${pretrained}

# Execute Perception Training
bash scripts/train_perceive.sh stp3/configs/nuscenes/Perception.yml data/Nuscenes

# Set Up on a Second PC for Efficiency and Faster Processing
# Inform the user about setting up on another PC
echo "Repeat the setup process on another PC with an RTX 3090 GPU for parallel processing."
echo "Transfer the perception-trained epochs to the new PC to utilize the additional GPU for further processing."

# Planning and Evaluation

# Execute Planning on the First PC with a Better GPU (RTX 4090)
echo "Adjust GPUS to [0] and BATCHSIZE to 1 in ST-P3/stp3/configs/nuscenes/Planning.yml to resolve CUDA out-of-memory errors."
bash scripts/train_plan.sh stp3/configs/nuscenes/Planning.yml data/Nuscenes ST-P3/tensorboard_logs/30August2024at13_22_38KST_SimulationPC_Perception/default/version_0/checkpoints/last.ckpt

# Run Evaluation
bash scripts/eval_plan.sh ${checkpoint} ${dataroot}
bash scripts/eval_plan.sh ST-P3/tensorboard_logs/17September2024at18_15_56KST_SimulationPC_Prediction data/Nuscenes

# Running Prediction Module

# Execute Prediction Training
echo "Adjust the GPUs configuration from GPUS: [0,1,2,3,4] to GPUS: [0] in the configuration file."
bash scripts/train_prediction.sh stp3/configs/nuscenes/Prediction.yml data/Nuscenes ST-P3/tensorboard_logs/30August2024at13_22_38KST_SimulationPC_Perception/default/version_0/checkpoints/last.ckpt

# Planning with Prediction Results
bash scripts/eval_plan.sh ${checkpoint} ${dataroot}
bash scripts/eval_plan.sh ST-P3/tensorboard_logs/17September2024at18_15_56KST_SimulationPC_Prediction data/Nuscenes

# Final message to user
echo "Results and HD maps will be available in directories like ST-P3/imgs."
