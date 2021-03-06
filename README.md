This project is in progress. I try to re-produce the impressive results of paper [AlignedReID: Surpassing Human-Level Performance in Person Re-Identification](https://arxiv.org/abs/1711.08184) using [pytorch](https://github.com/pytorch/pytorch).

If you adopt AlignedReID in your research, please cite the paper
```
@article{zhang2017alignedreid,
  title={AlignedReID: Surpassing Human-Level Performance in Person Re-Identification},
  author={Zhang, Xuan and Luo, Hao and Fan, Xing and Xiang, Weilai and Sun, Yixiao and Xiao, Qiqi and Jiang, Wei and Zhang, Chi and Sun, Jian},
  journal={arXiv preprint arXiv:1711.08184},
  year={2017}
}
```

# TODO List

- Model
  - [x] ResNet-50
- Loss
  - [x] Triplet Global Loss
  - [x] Triplet Local Loss
  - [x] Identification Loss
  - [ ] Mutual Loss
- Testing
  - [ ] Re-Ranking
- Speed
  - [x] Speed up forward & backward
  - [ ] Speed up local distance at test time. (Replace numpy by pytorch cuda operation.)


# Current Results

On Market1501 with setting
- global margin 0.3
- local margin 0.3
- Adam optimizer
- Base learning rate 2e-4, decaying exponentially after 75 epochs. Train for 150 epochs in total. Refer to paper [In Defense of the Triplet Loss for Person Re-Identification](https://arxiv.org/abs/1703.07737).

We achieve the following results. Note that training data only comes from one dataset, while the paper combines 4 datasets.

|   | Rank-1 (%) | mAP (%) |
| --- | --- | --- |
| GL-NF | 81.53 | 64.87 |
| --- | --- | --- |
| GL-LL-NF-LHSFGD-TWGD | 84.29 | 67.94 |
| GL-LL-NNF-LHSFGD-TWGD | 84.74 | 68.11 |
| --- | --- | --- |
| GL-LL-NNF-LHSFLD-TWGD | 85.18 | 68.31 |
| GL-LL-NNF-LHSFLD-TWLD | 85.60 | 68.72 |
| GL-LL-NNF-LHSFLD-TWGALD | 86.46 | 70.13 |
| --- | --- | --- |
| GL-IDL-NNF | 85.10 | 68.26 |
| GL-LL-IDL-NNF-LHSFLD-TWGD | 84.74 | 68.34 |
| GL-LL-IDL-NNF-LHSFLD-TWLD | 85.51 | 67.77 |
| GL-LL-IDL-NNF-LHSFLD-TWGALD | 85.45 | 69.47 |

**Notations for the table**
- GL: Global Loss
- LL: Local Loss
- IDL: IDentification Loss
- NF: Normalize Feature (both before Equation (1) of the paper and at test time)
- NNF: Not Normalize Feature (both before Equation (1) of the paper and at test time)
- LHSFGD: Local Hard Sample From Global Distance
- LHSFLD: Local Hard Sample From Local Distance
- TWGD: Test With Global Distance
- TWLD: Test With Local Distance
- TWGALD: Test With Global And Local Distance

The above number of iterations may be insufficient for training ID Loss. When starting decaying at epoch 150 and with total epochs 300, the results are better:

|   | Rank-1 (%) | mAP (%) |
| --- | --- | --- |
| GL-LL-IDL-NNF-LHSFLD-TWGD | 86.46 | 70.51 |
| GL-LL-IDL-NNF-LHSFLD-TWLD | 87.29 | 70.31 |
| GL-LL-IDL-NNF-LHSFLD-TWGALD | 87.08 | 71.61 |


# Installation

It's recommended that you create and enter a python virtual environment before installing our package.

```bash
git clone https://github.com/huanghoujing/AlignedReID-Re-Production-Pytorch.git
cd AlignedReID-Re-Production-Pytorch
```

## Requirements

I use Python 2.7 and Pytorch 0.1.12. For installing Pytorch 0.1.12, follow the [official guide](http://pytorch.org/previous-versions/). Other packages are specified in `requirements.txt`.

```bash
pip install -r requirements.txt
```

Then install this project:

```bash
python setup.py install --record installed_files.txt
```

# Dataset Preparation

Inspired by Tong Xiao's [open-reid](https://github.com/Cysu/open-reid) project, dataset directories are refactored to support a unified dataset interface.

Transformed dataset has following features
- All used images, including training and testing images, are inside the same folder named `images`
- The train/val/test partitions are recorded in a file named `partitions.pkl` which is a dict with the following keys
  - `'trainval_im_names'`
  - `'trainval_ids2labels'`
  - `'train_im_names'`
  - `'train_ids2labels'`
  - `'val_im_names'`
  - `'val_marks'`
  - `'test_im_names'`
  - `'test_marks'`
- Validation set consists of 100 persons (configurable during transforming dataset) unseen in training set, and validation follows the same ranking protocol of testing.
- Each val or test image is accompanied by a mark denoting whether it is from
  - query (mark == 0), or
  - gallery (mark == 1), or
  - multi query (mark == 2) set

## Market1501

You can download what I have transformed for the project from [Google Drive](https://drive.google.com/open?id=1CaWH7_csm9aDyTVgjs7_3dlZIWqoBlv4) or [BaiduYun](https://pan.baidu.com/s/1nvOhpot). Otherwise, you can download the original dataset and transform it using my script, described below.

Download the Market1501 dataset from [here](http://www.liangzheng.org/Project/project_reid.html). Run the following script to transform the dataset, replacing the paths with yours.

```bash
python script/dataset/transform_market1501.py \
--zip_file ~/Dataset/market1501/Market-1501-v15.09.15.zip \
--save_dir ~/Dataset/market1501
```

## CUHK03

We follow the new training/testing protocol proposed in paper
```
@article{zhong2017re,
  title={Re-ranking Person Re-identification with k-reciprocal Encoding},
  author={Zhong, Zhun and Zheng, Liang and Cao, Donglin and Li, Shaozi},
  booktitle={CVPR},
  year={2017}
}
```
Details of the new protocol can be found [here](https://github.com/zhunzhong07/person-re-ranking).

You can download what I have transformed for the project from [Google Drive](https://drive.google.com/open?id=1Ssp9r4g8UbGveX-9JvHmjpcesvw90xIF) or [BaiduYun](https://pan.baidu.com/s/1hsB0pIc). Otherwise, you can download the original dataset and transform it using my script, described below.

Download the CUHK03 dataset from [here](http://www.ee.cuhk.edu.hk/~xgwang/CUHK_identification.html). Then download the training/testing partition file from [Google Drive](https://drive.google.com/open?id=14lEiUlQDdsoroo8XJvQ3nLZDIDeEizlP) or [BaiduYun](https://pan.baidu.com/s/1miuxl3q). This partition file specifies which images are in training, query or gallery set. Finally run the following script to transform the dataset, replacing the paths with yours.

```bash
python script/dataset/transform_cuhk03.py \
--zip_file ~/Dataset/cuhk03/cuhk03_release.zip \
--train_test_partition_file ~/Dataset/cuhk03/re_ranking_train_test_split.pkl \
--save_dir ~/Dataset/cuhk03
```


## DukeMTMC-reID

You can download what I have transformed for the project from [Google Drive](https://drive.google.com/open?id=1P9Jr0en0HBu_cZ7txrb2ZA_dI36wzXbS) or [BaiduYun](https://pan.baidu.com/s/1miIdEek). Otherwise, you can download the original dataset and transform it using my script, described below.

Download the DukeMTMC-reID dataset from [here](https://github.com/layumi/DukeMTMC-reID_evaluation). Run the following script to transform the dataset, replacing the paths with yours.

```bash
python script/dataset/transform_duke.py \
--zip_file ~/Dataset/duke/DukeMTMC-reID.zip \
--save_dir ~/Dataset/duke
```

## Configure Dataset Path in Training Script

The training code requires you to configure the dataset paths. In `script/tri_loss/train_cfg.py`, modify the following snippet according to your saving paths used in preparing datasets.

```python
# In file script/tri_loss/train_cfg.py

if self.dataset == 'market1501':
  self.im_dir = osp.expanduser('~/Dataset/market1501/images')
  self.partition_file = osp.expanduser('~/Dataset/market1501/partitions.pkl')
elif self.dataset == 'cuhk03':
  self.im_type = ['detected', 'labeled'][0]
  self.im_dir = osp.expanduser(osp.join('~/Dataset/cuhk03', self.im_type, 'images'))
  self.partition_file = osp.expanduser(osp.join('~/Dataset/cuhk03', self.im_type, 'partitions.pkl'))
elif self.dataset == 'duke':
  self.im_dir = osp.expanduser('~/Dataset/duke/images')
  self.partition_file = osp.expanduser('~/Dataset/duke/partitions.pkl')
```

# Training Examples

**NOTE:** After changing files in directory `aligned_reid`, you have to install the package again by `python setup.py install --record installed_files.txt`. Because scripts that import from this `aligned_reid` package in fact import from site-packages (Decided by where you run the script? Not sure.), you have to install to update it.

To train and test `ResNet-50 + Global Loss` on Market1501:

```bash
python script/tri_loss/train.py \
-d '(0,)' \
--dataset market1501 \
--normalize_feature false \
--local_dist_own_hard_sample true \
-gm 0.3 \
-lm 0.3 \
-glw 1 \
-llw 0 \
-idlw 0 \
-gtw 1 \
-ltw 0
```

To train `ResNet-50 + Global Loss + Local Loss` on Market1501, and test with `Global + Local Distance`:

```bash
python script/tri_loss/train.py \
-d '(0,)' \
--dataset market1501 \
--normalize_feature false \
--local_dist_own_hard_sample true \
-gm 0.3 \
-lm 0.3 \
-glw 1 \
-llw 1 \
-idlw 0 \
-gtw 1 \
-ltw 1
```

You can run the [TensorBoard](https://github.com/lanpa/tensorboard-pytorch) to watch the loss curves etc during training. E.g.

```bash
# Modify the path for `--logdir` accordingly.
tensorboard --logdir exp/tri_loss/market1501/train/not_nf_ohs_gm_0.3_lm_0.3_glw_1_llw_1_idlw_0_gtw_1_ltw_1/run1/tensorboard
```

For more usage of TensorBoard, see the website and the help:

```bash
tensorboard --help
```
