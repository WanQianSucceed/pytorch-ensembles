The official PyTorch implementation of:  
**Pitfalls of In-Domain Uncertainty Estimation and Ensembling in Deep Learning**, ICLR'20 [Paper](https://openreview.net/forum?id=BJxI5gHKDr) [Blog](https://senya-ashukha.github.io/pitfalls-uncertainty&ensembling)

Arsenii Ashukha* (1, 2), Alexander Lyzhov* (1, 4, 3), Dmitry Molchanov* (1, 2) and Dmitry Vetrov (2, 1)

The work is done at:
1. SAIC-M: Samsung AI Center Moscow
2. Samsung-HSE: Samsung-HSE Lab, National Research University Higher School of Economics
3. National Research University Higher School of Economics
4. Skolkovo Institute of Science and Technology

## Environment Setup

The following allows to create and to run a python environment with all required dependencies using [miniconda](https://docs.conda.io/en/latest/miniconda.html): 

```(bash)
conda env create -f condaenv.yml
conda activate megabayes
```

### Logs, Plots, Tables

At the [notebooks](./notebooks) folder we provide:
- Saved logs with all computed results 
- Examples of ipython notebooks to reproduce plots, tables, and compute the deep ensemble equivalent (DEE) score

### Evaluation

The evaluation of ensembling methods can be done using the scripts from the [ens](./ens) folder, which contains a separate script for each ensembling method.
The scrips have the following interface:

```
ipython -- ens/ens-<method>.py -h
usage: ens-onenet.py [-h] --dataset DATASET [--data_path PATH]
                     [--models_dir PATH] [--aug_test] [--batch_size N]
                     [--num_workers M] [--fname FNAME]

optional arguments:
  -h, --help         show this help message and exit
  --dataset DATASET  dataset name CIFAR10/CIFAR100/ImageNet (default: None)
  --data_path PATH   path to a data-folder (default: ~/data)
  --models_dir PATH  a dir that stores pre-trained models (default: ~/megares)
  --aug_test         enables test-time augmentation (default: False)
  --batch_size N     input batch size (default: 256)
  --num_workers M    number of workers (default: 10)
  --fname FNAME      comment to a log file name (default: unnamed)
```
* All scripts assume that `pytorch-ensembles` is the current working directory (`cd  pytorch-ensembles`).
* The scripts will write .csv logs in `pytorch-ensembles/logs` in the following format `rowid, dataset, architecture, ensemble_method, n_samples, metric, value, info`. 
* The [notebooks](./notebooks) folder contains ipython notebooks to reproduce the tables and plots using these logs
* The scripts will write final log-probs of every method in '.npy' format to `pytorch-ensembles/.megacache` folder. 
* The interface for K-FAC-Laplace differs and is described below.

Examples:
```
ipython -- ens/ens-onenet.py  --dataset=CIFAR10/CIFAR100/ImageNet
ipython -- ens/ens-deepens.py --dataset=CIFAR10/CIFAR100/ImageNet
ipython -- ens/ens-sse.py     --dataset=CIFAR10/CIFAR100/ImageNet
ipython -- ens/ens-csgld.py   --dataset=CIFAR10/CIFAR100
ipython -- ens/ens-fge.py     --dataset=CIFAR10/CIFAR100/ImageNet
ipython -- ens/ens-dropout.py --dataset=CIFAR10/CIFAR100
ipython -- ens/ens-vi.py      --dataset=CIFAR10/CIFAR100/ImageNet
ipython -- ens/ens-swag.py    --dataset=CIFAR10/CIFAR100
```

## Training models

#### Deep Ensemble members, Variational Inference (CIFAR-10/100)

All the models trained on CIFAR use a single GPU for training.
Examples of training commands:
```(bash)
bash train/train_cifar.sh \
--dataset CIFAR10/CIFAR100 \
--arch VGG16BN/PreResNet110/PreResNet164/WideResNet28x10 \
--method regular/vi
```

#### Fast Geometric Ensembling, SWA-Gaussian, Snapshot Ensembles, and Cyclical SGLD (CIFAR-10/100)

Examples of training commands:
```(bash)
bash train/train_fge.sh CIFAR10 WideResNet28x10 1 ~/weights ~/datasets
bash train/train_swag.sh CIFAR10 WideResNet28x10 1 ~/weights ~/datasets
bash train/train_sse_mcmc.sh CIFAR10 WideResNet28x10 1 ~/weights ~/datasets SSE
bash train/train_sse_mcmc.sh CIFAR10 WideResNet28x10 1 ~/weights ~/datasets cSGLD
```

Script parameters: dataset, architecture name, training run id, root directory for saving snapshots (created automatically), root directory for datasets (downloaded automatically)

#### K-FAC Laplace (CIFAR-10/100, ImageNet)
Given a checkpoint, `ens/ens-kfacl.py` builds the Laplace approximation and produces the results of the approximate posterior averaging.
Use keys `--scale_search` and `--gs_low LOW --gs_high HIGH --gs_num NUM` to find the optimal posterior noise scale on the validation set.
We have used the following noise scales (also listed in Table 3, Appendix D in the paper):

|Architecture | CIFAR-10 | CIFAR-10-aug | CIFAR-100 | CIFAR-100-aug |
|-------------|---------:|-------------:|----------:|---------------:|
|VGG16BN         | 0.042 | 0.042 | 0.100 | 0.100 |
|PreResNet110    | 0.213 | 0.141 | 0.478 | 0.401 |
|PreResNet164    | 0.120 | 0.105 | 0.285 | 0.225 |
|WideResNet28x10 | 0.022 | 0.018 | 0.022 | 0.004 |

For ResNet50 on ImageNet, the optimal scale found was 2.0 with test-time augmentation and 6.8 without test-time augmentation.

Refer to `ens/ens-kfacl.py' for the full list of arguments and default values.
Example use:
```bash
ipython -- ens/ens-kfacl.py --file CHECKPOINT --data_path DATA --dataset CIFAR10 --model PreResNet110 --scale 0.213
```

#### Deep Ensemble members, Variational Inference, Snapshot Ensembles (ImageNet)

Examples of training commands:
```
bash train/train_imagenet.sh --method regular/sse/fge/vi
```

We strongly recommend using multi-gpu training for Snapshot Ensembles. 

## Attribution

Parts of this code are based on the following repositories:
- [Stochastic Weight Averaging (SWA)](https://github.com/timgaripov/swa). Pavel Izmailov, Dmitrii Podoprikhin, Timur Garipov, Dmitry Vetrov, Andrew Gordon Wilson.
- [A Simple Baseline for Bayesian Deep Learning](https://github.com/wjmaddox/swa_gaussian). Wesley Maddox, Timur Garipov, Pavel Izmailov,  Dmitry Vetrov, Andrew Gordon Wilson.
- [Cyclical Stochastic Gradient MCMC for Bayesian Deep Learning](https://github.com/ruqizhang/csgmcmc). Ruqi Zhang, Chunyuan Li, Jianyi Zhang, Changyou Chen and Andrew Gordon Wilson.
- [Loss Surfaces, Mode Connectivity, and Fast Ensembling of DNNs](https://github.com/timgaripov/dnn-mode-connectivity).  Timur Garipov, Pavel Izmailov, Dmitrii Podoprikhin, Dmitry Vetrov and Andrew Gordon Wilson.
- [PyTorch](https://github.com/pytorch/pytorch)
- [PyTorch Examples](https://github.com/pytorch/examples/tree/ee964a2/imagenet)

## Citation

If you found this code useful, please cite our paper
```
@inproceedings{
    ashukha2020pitfalls,
    title={Pitfalls of In-Domain Uncertainty Estimation and Ensembling in Deep Learning},
    author={Arsenii Ashukha and Alexander Lyzhov and Dmitry Molchanov and Dmitry Vetrov},
    booktitle={International Conference on Learning Representations},
    year={2020},
    url={https://openreview.net/forum?id=BJxI5gHKDr}
}
```
