# PoolFormer: MetaFormer is Actually What You Need for Vision ([arXiv](https://arxiv.org/abs/2111.11418))

This is a PyTorch implementation of **PoolFormer** proposed by our paper "**MetaFormer is Actually What You Need for Vision**".


![MetaFormer](https://user-images.githubusercontent.com/15921929/142736039-d4237fb4-7d11-46e3-888d-496b52e7244c.png)

Figure 1: **MetaFormer and performance of MetaFormer-based models on ImageNet-1K validation set.** 
We argue that the competence of transformer/MLP-like models primarily stems from the general architecture MetaFormer instead of the equipped specific token mixers.
To demonstrate this, we exploit an embarrassingly simple non-parametric operator, pooling, to conduct extremely basic token mixing. 
Surprisingly, the resulted model PoolFormer consistently outperforms the DeiT and ResMLP as shown in (b), which well supports that MetaFormer is actually what we need to achieve competitive performance.

![PoolFormer](https://user-images.githubusercontent.com/15921929/142746124-1ab7635d-2536-4a0e-ad43-b4fe2c5a525d.png)
Figure 2: (a) **The overall framework of PoolFormer.** (b) **The architecture of PoolFormer block.** Compared with transformer block, it replaces attention with an extremely simple non-parametric operator, pooling, to conduct only basic token mixing.

## Reference
```
@article{yu2021metaformer,
  title={MetaFormer is Actually What You Need for Vision},
  author={Yu, Weihao and Luo, Mi and Zhou, Pan and Si, Chenyang and Zhou, Yichen and Wang, Xinchao and Feng, Jiashi and Yan Shuicheng},
  journal={arXiv preprint arXiv:2111.11418},
  year={2021}
}
```

## 1. Requirements
**For Image Classification** (Configs of detection and segmentation will be available soon)

torch>=1.7.0; torchvision>=0.8.0; pyyaml; [apex-amp](https://github.com/NVIDIA/apex) (if you want to use fp16); [timm](https://github.com/rwightman/pytorch-image-models) (pip install git+https://github.com/rwightman/pytorch-image-models.git@9d6aad44f8fd32e89e5cca503efe3ada5071cc2a)

data prepare: ImageNet with the following folder structure, you can extract ImageNet by this [script](https://gist.github.com/BIGBALLON/8a71d225eff18d88e469e6ea9b39cef4).

```
│imagenet/
├──train/
│  ├── n01440764
│  │   ├── n01440764_10026.JPEG
│  │   ├── n01440764_10027.JPEG
│  │   ├── ......
│  ├── ......
├──val/
│  ├── n01440764
│  │   ├── ILSVRC2012_val_00000293.JPEG
│  │   ├── ILSVRC2012_val_00002138.JPEG
│  │   ├── ......
│  ├── ......
```

Directory structure in this repo:
```
│poolformer/
├──misc/
├──models/
│  ├── __init__.py
│  ├── poolformer.py
├──LICENSE
├──README.md
├──distributed_train.sh
├──train.py
├──validate.py
```

## 2. PoolFormer Models

| Model    |  #params | Image resolution | Top1 Acc| Download | 
| :---     |   :---:    |  :---: |  :---:  |  :---:  |
| poolformer_s12  |    12M     |   224 |  77.2  | [here](https://github.com/sail-sg/poolformer/releases/download/v1.0/poolformer_s12.pth.tar) |
| poolformer_s24 |   21M     |   224 |  80.3  | [here](https://github.com/sail-sg/poolformer/releases/download/v1.0/poolformer_s24.pth.tar) |
| poolformer_s36  |   31M     |   224 |  81.4  | [here](https://github.com/sail-sg/poolformer/releases/download/v1.0/poolformer_s36.pth.tar) |
| poolformer_m36 |   56M     |   224 |  82.1  | [here](https://github.com/sail-sg/poolformer/releases/download/v1.0/poolformer_m36.pth.tar) |
| poolformer_m48  |   73M     |   224 |  82.5  | [here](https://github.com/sail-sg/poolformer/releases/download/v1.0/poolformer_m48.pth.tar) | 


All the pretrained models can also be downloaded by [BaiDu Yun](https://pan.baidu.com/s/1HSaJtxgCkUlawurQLq87wQ) (password: esac).

### Usage
We also provide a [Colab notebook](https://colab.research.google.com/github/sail-sg/poolformer/blob/main/misc/poolformer_demo.ipynb) which run the steps to perform inference with poolformer.


## 3. Validation

To evaluate our PoolFormer models, run:

```bash
MODEL=poolformer_s12 # poolformer_{s12, s24, s36, m36, m48}
python3 validate.py /path/to/imagenet  --model $MODEL \
  --checkpoint /path/to/checkpoint -b 128
```



## 4. Train
We show how to train PoolFormers on 8 GPUs. The relation between learning rate and batch size is lr=bs/1024*1e-3.
For convenience, assuming the batch size is 1024, then the learning rate is set as 1e-3 (for batch size of 1024, setting the learning rate as 2e-3 sometimes sees better performance). 


```bash
MODEL=poolformer_s12 # poolformer_{s12, s24, s36, m36, m48}
DROP_PATH=0.1 # drop path rates [0.1, 0.1, 0.2, 0.3, 0.4] responding to model [s12, s24, s36, m36, m48]
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 ./distributed_train.sh 8 /path/to/imagenet \
  --model $MODEL -b 128 --lr 1e-3 --drop-path $DROP_PATH --apex-amp
```

## LICENSE
This repo is under the Apache-2.0 license. For commercial use, please contact the authors.