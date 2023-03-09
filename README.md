# A Training and Inference Strategy Using Noisy and Enhanced Speech as Target for Speech Enhancement without Clean Speech

This is the official GitHub repo for "A Training and Inference Strategy Using Noisy and Enhanced Speech as Target for Speech Enhancement without Clean Speech". 

The code is mainly built on the [facebookresearch/denoiser](https://github.com/facebookresearch/denoiser). For now, the complete code is in [our forked denoiser](https://github.com/Sinica-SLAM/denoiser/tree/nytt). We are still working on moving the structure to an independent place.

## Training baseline model (initial teacher model)
1. Setting up a new dataset (folder contains noisy.json and noise.json, in our case noisy is from valentini, and noise is from CHiME3 background)
  ```bash
  noisy=<path to the dir with the noisy files>
  noise=<path to the dir with the noise files>
  out=<path to store dataset json>
  mkdir -p $out
  python3 -m denoiser.audio $noisy > $out/noisy.json
  python3 -m denoiser.audio $noise > $out/noise.json
  ```
2. Place the new config files under the dset folder. Check [conf/dset/debug.yaml](https://github.com/Sinica-SLAM/denoiser/blob/nytt/conf/dset/debug.yaml) for more details on configuring your dataset.
3. Use [launch_baseline.sh](https://github.com/Sinica-SLAM/denoiser/blob/nytt/launch_baseline.sh) to train your baseline model (need to assign the new config file name in 2. to DSET_NAME in the script)

## Training Ny/EnhTT-4 student model with static teacher
1. Use teacher model to generat the enhanced files
  ```bash
  python3 -m denoiser.enhance_noise --model_path=<path to the model> --noisy_dir=<path to the dir with the noisy files> --out_dir=<path to store enhanced files>
  ```
2. Setting up a new dataset and place the new config files under the dset folder (as in the previous paragraph, but noise is from 1. output files)
3. Use [launch_static_nyenhtt4.sh](https://github.com/Sinica-SLAM/denoiser/blob/nytt/launch_static_nyenhtt4.sh) to train your nytt1 model (need to assign the new config file name in 2. to DSET_NAME in the script)

## Training Ny/EnhTT-4 student model with exponentially moving average teacher
1. Use teacher model to generat the enhanced files (as in the previous paragraph)
2. Setting up a new dataset and place the new config files under the dset folder (as in the previous paragraph)
3. Use [launch_ema_nyenhtt4.sh](https://github.com/Sinica-SLAM/denoiser/blob/nytt/launch_ema_nyenhtt4.sh) to train your nytt1 model.
   There are some values that need to assign in the script:
   - `DSET_NAME=<config file name in 2.>`
   - `NOISY_DIR=<path to the dir with the noisy files>`
   - `ENHANCE_DIR (your can change this part of script to update noise in $out/noise.json)`
   - `TEACHER_CHECKPOINT=<path to the teacher model ckpt>`

## If you want to use a method other than Ny/EnhTT-4
### Ny/EnhTT-1
- origin noisy -> noisy.json
- enhanced -> clean.json
- Change train_nytt.py to train.py in script
### Ny/EnhTT-2
- enhanced -> noisy.json
- output noise -> noise.json
### Ny/EnhTT-3
- enhanced -> noisy.json
- output noise -> noise.json
- Change train_nytt.py to train_nytt_chime3.py in script
- Assign path to CHiME3 background to CHIME3_BACKGROUND_PATH
### Ny/EnhTT-5
- origin noisy -> noisy.json
- (output noise and CHiME3 background) -> noise.json
### Ny/EnhTT-6
- origin noisy -> noisy.json
- output noise -> noise.json
- Change train_nytt.py to train_nytt_chime3.py in script
- Assign path to CHiME3 background to CHIME3_BACKGROUND_PATH


