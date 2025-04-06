<p align="center">
<h1 align="center">MapEx: Indoor Structure Exploration with Probabilistic Information Gain from Global Map Predictions</h1>
<h3 class="is-size-5 has-text-weight-bold" style="color: orange;" align="center">
    IEEE International Conference on Robotics and Automation (ICRA) 2025 
</h3>
  <p align="center">
    <a href="https://cherieho.com/" target="_blank"><strong>Cherie Ho*</strong></a>
    ·
    <a href="https://seungchan-kim.github.io" target="_blank"><strong>Seungchan Kim*</strong></a>
    ·
    <a href="https://bradymoon.com/" target="_blank"><strong>Brady Moon</strong></a>
    ·
    <a href=""><strong>Aditya Parandekar</strong></a>
    ·
    <a href=""><strong>Narek Harutyunyan</strong></a>
    <br>
    <a href="https://sairlab.org/team/chenw/" target="_blank"><strong>Chen Wang</strong></a>
    ·
    <a href="https://www.cs.cmu.edu/~./katia/" target="_blank"><strong>Katia Sycara</strong></a>
    ·
    <a href="https://profiles.uts.edu.au/Graeme.Best" target="_blank"><strong>Graeme Best</strong></a>
    ·
    <a href="https://theairlab.org/team/sebastian/" target="_blank"><strong>Sebastian Scherer</strong></a>
    <br>
  </p>
</p>
  <h3 align="center"><a href="https://arxiv.org/abs/2409.15590">Paper</a> | <a href="https://mapex-explorer.github.io">Project Page</a> | <a href="https://youtu.be/x6vEn8d6lN0">Video</a></h3>
  <div align="center"></div>

## Preliminary Setup
Clone the repository and make sure that you are on the main branch.

    git clone --recurse-submodules git@github.com:castacks/MapEx.git
    cd ~/MapEx
    git checkout main
    git submodule update --init --recursive

### Set up Mamba environment (Recommended)
Mamba is a package manager used for managing python environments and dependencies, known for having better speed and efficiency than conda. For more information, please refer to this <a href="https://mamba.readthedocs.io/en/latest/user_guide/mamba.html">link</a>. 

    wget https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-x86_64.sh
    bash Mambaforge-Linux-x86_64.sh

Go to `lama` submodule folder, and create `lama` environment. 

    cd ~/MapEx/lama
    mamba env create -f conda_env.yml
    mamba activate lama

<!-- Install `torch` and relevant packages

    mamba install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch --y
    mamba install wandb --yes
    pip install pytorch-lightning==1.2.9 -->

### Download pretrained prediction models (KTH dataset)
You can download pretrained models from this <a href="https://drive.google.com/drive/u/0/folders/1u9WZ9ftwaMbP-RVySuNSVEdUDV_x4Dw6">link</a>. Place the zip file under `pretrained_models` directory and unzip the file. 

    mv ~/Downloads/weights.zip ~/MapEx/pretrained_models/
    cd ~/MapEx/pretrained_models/
    unzip weights.zip

The `pretrained_model` directory and its subdirectories should be organized as below: 

    MapEx
    ├── pretrained_models
        ├── weights
            ├── big_lama
                ├── models
                    ├── best.ckpt
            ├── lama_ensemble
                ├── train_1
                    ├── models
                        ├── best.ckpt
                ├── train_2
                    ├── models
                        ├── best.ckpt
                ├── train_3
                    ├── models
                        ├── best.ckpt    

### Install lib_rangec
Go to `range_libc` directory and install by following:

    cd ~/MapEx/lib_rangec/pywrapper
    python3 setup.py install 
    pip3 install Cython

### Install KTH toolbox dependencies for raycasting and observation model

    python3 -m pip install pyastar2d
    mamba install numba --yes

## Experiments

### Run MapEx planner with baselines
In order to test MapEx, run the `explore.py` script. 

    cd scripts/
    python3 explore.py

This will automatically call `base.yaml`, which contains default parameters and specifies filepaths, environment, and starting conditions. If you want to customize parameters, generate your own yaml file and save it in the `configs` directory. 

Moreover, if you want to specify environment and starting positions as arguments to the script. For example, 

    python3 explore.py --collect_world_list 50010535_PLAN1 --start_pose 768 551

The list of environments is in the `kth_test_maps` directory. The list of start_pose for each environment can be seen in the `clusters/run_explore.job`. 

#### Details on methods 
`modes_to_test` in the yaml file specifies the methods that you test. Specifically,  

    modes_to_test: ['visvarprob'] #if you just want to test MapEx
    modes_to_test: ['nearest', 'upen', 'hectoraug', 'visvarprob'] #if you want to compare MapEx and baselines
    modes_to_test: ['obsunk', 'onlyvar', 'visunk', 'visvar', 'visvarprob'] #if you want ablation experiments
    modes_to_test: ['nearest', 'obsunk', 'onlyvar', 'visunk', 'visvar', 'visvarprob', 'upen', 'hector', 'hectoraug'] #if you want to test all methods 

`visvarprob` corresponds to the `MapEx` method, meaning the combination of visibility mask, variance, and probabilistic raycast. `nearest` is nearest frontier-based exploration, `upen` is our implementation of uncertainty-driven planner, proposed by <a href="https://arxiv.org/abs/2202.11907">Georgakis et al. ICRA 2022</a>, and `hectoraug` is our implementation of IG-hector method proposed by <a href="https://ieeexplore.ieee.org/document/8793769">Shrestha et al. ICRA 2019</a>.

<strong>Ablated methods</strong>: `visvar` (visibility mask + variance + deterministic raycast), `visunk` (visibility mask + counting number of pixels in the area), `obsunk` (visibility mask on observed occupancy grid + counting number of pixels in the area), `onlyvar` (using no visibility mask, but only summing variances) correspond with Deterministic, No Variance, Observed Map, and No Visibility methods in the ablation studies section of our original paper. 

## Evaluations, Metric, and Data Processing

### Trajectory Visualization and Data Postprocessing (Generating predictions for metric)

After you run `explore.py`, the script will produce results in the `experiments` folder. The script automatically generates a subdirectory inside it, using the current year, month, date. Under this folder, each result of map & start_pose & method pairs will be saved. For example, 

    MapEx
        ├── experiments
            ├── 20250131_test
                ├── 20250131_172221_50052750_513_880_visvarprob
                    ├── global_obs
                    ├── run_viz
                    ├── odom.npy
                ├── 20250131_172221_50052750_513_880_upen
            ...

`global_obs` contains 2D top-down view observed occupancy grid map in .png file format over all timesteps. `run_viz` folder contains visualization of exploration over timesteps as below.

![RunViz Image](https://github.com/mapex-explorer/mapex-explorer.github.io/blob/main/assets/run_viz_example.png)

Now for the purpose of metric and evaluations, run `simple_lama_pred.py` to generate predictions for these observations. 

    python3 simple_lama_pred.py

Make sure to customize `modelalltrain_path` and `input_experiment_root_folder` as needed. Especially, `input_experiment_root_folder` should be modified if the directory that contains the exploration results change. `simple_lama_pred.py` will generate predictions and save them in the `global_pred` directory under each folder like below:

    MapEx
        ├── experiments
            ├── 20250131_test
                ├── 20250131_172221_50052750_513_880_visvarprob
                    ├── global_obs
                    ├── global_pred
                    ├── run_viz
                    ├── odom.npy
                ├── 20250131_172221_50052750_513_880_upen
            ...

### Coverage and Predicted IoU

After you generated predictions of your observations, run `calc_metrics_subdirectory.py`. Make sure to customize your `root_path` and `exp_parent_dir` in the code. 

    python3 calc_metrics_subdirectory.py

This script will compute Coverage and Predicted IoU, and will visualize the plots like below.

![CovIOU Image](https://github.com/mapex-explorer/mapex-explorer.github.io/blob/main/assets/cov-iou-example.png)

## Code Management

This repository will be maintained and improved by <a href="https://seungchan-kim.github.io" target="_blank"><strong>Seungchan Kim</strong></a>. If you have any questions regarding the codes, please email to seungch2@andrew.cmu.edu. 

## Citation

If you find our paper or code useful, please cite us:

```bib
@article{ho_kim2024mapex,
  title={MapEx: Indoor Structure Exploration with Probabilistic Information Gain from Global Map Predictions},
  author={Ho, Cherie and Kim, Seungchan and Moon, Brady and Parandekar, Aditya and Harutyunyan, Narek and Wang, Chen and Sycara, Katia and Best, Graeme and Scherer, Sebastian},
  journal={arXiv preprint arXiv:2409.15590},
  year={2024}
}
```
