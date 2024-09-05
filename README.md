# ST-P3 Workflow Guide

```bash
# 1. Clone the Repository
git clone https://github.com/OpenDriveLab/ST-P3.git

# 2. Create the Environment
# Download the environment with an arbitrary name. In my case, I created an environment called `st-p3` using the command:
conda env create -f environment.yml -n st-p3

# 3. Setup the Data Directory
# Create a directory called `data` and another directory called `Nuscenes` inside it.
mkdir -p ST-P3/data/Nuscenes

# Download the following data from NuScenes (https://www.nuscenes.org/download) into the `Nuscenes` directory:
# Metadata [US, Asia] 0.43 GB
# File blobs of 85 scenes, part 1 [US, Asia] 29.41 GB
# File blobs of 85 scenes, part 2 [US, Asia] 28.06 GB
# File blobs of 85 scenes, part 3 [US, Asia] 27.81 GB
# File blobs of 85 scenes, part 4 [US, Asia] 29.87 GB
# File blobs of 85 scenes, part 5 [US, Asia] 26.25 GB
# File blobs of 85 scenes, part 6 [US, Asia] 25.61 GB
# File blobs of 85 scenes, part 7 [US, Asia] 27.50 GB
# File blobs of 85 scenes, part 8 [US, Asia] 28.19 GB
# File blobs of 85 scenes, part 9 [US, Asia] 31.21 GB
# File blobs of 85 scenes, part 10 [US, Asia] 38.87 GB
# Map expansion pack (v1.3) [US, Asia]
# CAN bus expansion pack [US, Asia]

# 4. Train the Perception Module
# Change the current directory to `ST-P3`.
cd ST-P3

# Edit the configuration file `ST-P3/stp3/configs/nuscenes/Perception.yml` and set the GPUs:
# GPUS: [0]  # From [0,1,2,3,4]

# Optionally, modify the number of epochs. Default is `EPOCHS: 20`.

# Start training the perception module:
bash scripts/train_perceive.sh stp3/configs/nuscenes/Perception.yml data/Nuscenes

# Training takes around 6 hours per epoch, totaling 6-7 days.
# The checkpoints will be saved in `tensorboard_logs` for future use in prediction and planning.
# Example checkpoint path:
# ST-P3/tensorboard_logs/30August2024at13_22_38KST_SimulationPC_Perception/default/version_0/checkpoints/last.ckpt

# 5. Run Planning on the First PC (RTX 4090)
# Edit the configuration file `ST-P3/stp3/configs/nuscenes/Planning.yml` and set:
# GPUS: [0]  # From [0,1,2,3,4]
# BATCHSIZE: 1  # From 2 to avoid CUDA out-of-memory error

# Run the planning module:
bash scripts/train_plan.sh stp3/configs/nuscenes/Planning.yml data/Nuscenes tensorboard_logs/30August2024at13_22_38KST_SimulationPC_Perception/default/version_0/checkpoints/last.ckpt

# This process takes about a week.
# The checkpoint will be saved in tensorboard_logs:
# ST-P3/tensorboard_logs/10September2024at16_58_48KST_SimulationPC_Planning

# 6. Run Evaluation
# Once planning is finished, run the evaluation:
bash scripts/eval_plan.sh ST-P3/tensorboard_logs/10September2024at16_58_48KST_SimulationPC_Planning data/Nuscenes

# The results, including HD maps, will be saved in paths like:
# ST-P3/imgs/09_10_10_46_05

# 7. Setup and Run Prediction on the Second PC (RTX 3090)
# Repeat the setup process (cloning the repo, setting up the environment, downloading the data, and transferring the perception model).

# Modify the GPUs in the `Prediction.yml` config:
# GPUS: [0]  # From [0,1,2,3,4]

# Start training the prediction module:
bash scripts/train_prediction.sh stp3/configs/nuscenes/Prediction.yml data/Nuscenes tensorboard_logs/30August2024at13_22_38KST_SimulationPC_Perception/default/version_0/checkpoints/last.ckpt

# Training takes about a week. The checkpoint will be saved in:
# ST-P3/tensorboard_logs/17September2024at18_15_56KST_SimulationPC_Prediction

# 8. Run Planning with Prediction
# After prediction is complete, run the planning evaluation:
bash scripts/eval_plan.sh ST-P3/tensorboard_logs/17September2024at18_15_56KST_SimulationPC_Prediction data/Nuscenes

# The results, including HD maps, will be saved in paths like:
# ST-P3/imgs/09_12_11_40_30
