
# Bit-Flips Attack:


![BFA](assets/BFA.jpg?raw=true "Bit Flip Attack")

This repository constains a Pytorch implementation of the paper, titled "[Bit-Flip Attack: Crushing Neural Network with Progressive Bit Search](http://openaccess.thecvf.com/content_ICCV_2019/papers/Rakin_Bit-Flip_Attack_Crushing_Neural_Network_With_Progressive_Bit_Search_ICCV_2019_paper.pdf)", which is published in [ICCV-2019](http://iccv2019.thecvf.com/).

If you find this project useful to you, please cite [our work](http://openaccess.thecvf.com/content_ICCV_2019/papers/Rakin_Bit-Flip_Attack_Crushing_Neural_Network_With_Progressive_Bit_Search_ICCV_2019_paper.pdf):

```bibtex
@inproceedings{he2019bfa,
 title={Bit-Flip Attack: Crushing Neural Network with Progressive Bit Search},
 author={Adnan Siraj Rakin and He, Zhezhi and Fan, Deliang},
 booktitle={Proceedings of the IEEE International Conference on Computer Vision (ICCV)},
 pages={1211-1220},
 year={2019}
}
```

## Table of Contents

- [Introduction](#Introduction)
- [Dependencies](#Dependencies)
- [Usage](#Usage)
- [Model Quantization](#Model-Quantization)
- [Bit Flipping](#Bit-Flipping)

## Introduction

This repository includes a Bit-Flip Attack (BFA) algorithm which search and identify the vulernable bits within a quantized deep neural network.

## Dependencies:
  
* Python 3.6 (Anaconda)
* [Pytorch](https://pytorch.org/) >=1.01
* [TensorboardX](https://github.com/lanpa/tensorboardX) 

For more specific dependency, please refer [environment.yml](./environment.yml) and [environment_setup.md](./docs/environment_setup.md)

## Usage
<!-- Our code performs Following steps for Bit-Flip Attack (BFA):
1. Get a quantized model.
2. Conduct BFA bit-by-bit. -->

Please modify `PYTHON=`, `TENSORBOARD=` and `data_path=` in the example bash code (`BFA_imagenet.sh`) before running the code.

```bash
HOST=$(hostname)
echo "Current host is: $HOST"

# Automatic check the host and configure
case $HOST in
"alpha")
    # PYTHON="/home/elliot/anaconda3/envs/pytorch_041/bin/python" # python environment
    PYTHON="/home/elliot/anaconda3/envs/bindsnet/bin/python"
    TENSORBOARD='/home/elliot/anaconda3/envs/bindsnet/bin/tensorboard'
    data_path='/home/elliot/data/imagenet'
    ;;
esac
```

Then just run the following command in the terminal.
```bash
bash BFA_imagenet.sh
# CUDA_VISIBLE_DEVICES=2 bash BFA_imagenet.sh  # to specify GPU id to ex. 2
```

The example log file of BFA on ResNet34:
```txt
  **Test** Prec@1 73.126 Prec@5 91.380 Error@1 26.874
k_top is set to 10
Attack sample size is 128
**********************************
Iteration: [001/020]   Attack Time 3.241 (3.241)  [2019-08-28 07:59:27]
loss before attack: 0.4138
loss after attack: 0.5209
bit flips: 1
hamming_dist: 1
  **Test** Prec@1 72.512 Prec@5 91.072 Error@1 27.488
iteration Time 65.493 (65.493)
**********************************
Iteration: [002/020]   Attack Time 2.667 (2.954)  [2019-08-28 08:00:35]
loss before attack: 0.5209
loss after attack: 0.7529
bit flips: 2
hamming_dist: 2
  **Test** Prec@1 70.492 Prec@5 89.866 Error@1 29.508
iteration Time 65.671 (65.582)
**********************************
```
It shows to identify one bit througout the entire model only takes ~3 Second (i.e., Attack Time) using 128 sample images for BFA. 


### Model quantization

We direct adopt the post-training quantization on the DNN pretrained model provided by the [model-zoo](https://pytorch.org/docs/stable/torchvision/models.html) of pytorch. 

> __Note__: for save the model in INT-8, additional data conversion is expected.

<!-- For the goal that directly quantize the deep neural network without retraining it, we add the function ```--optimize_step``` to optimize the step-size of quantizer to minimize the loss (e.g., mean-square-error loss) between quantized weight and its full precision base. It is intriguing to find out that:


- directly apply the uniform quantizer can achieve higher accuracy (close to the full precision baseline) without optimize the quantizer, for high-bit quantization (e.g., 8-bit). 

- On the contrary, for the low-bit quantization (e.g., 4-bit), directly quantize the weight causes significant accuracy loss. With the ```--optimize_step``` enabled, accuracy can partially recover without retraining. 

Since for the ImageNet simulation, we want to use directly perform the weight quantization on the pretrained weight. -->

### Bit Flipping

Considering the quantized weight $w$ is a integer ranging from $-(2^{N-1})$ to $(2^{N-1}-1)$, if using $N$ bits quantization. For example, the value range is -128 to 127 with 8-bit representation. In this work, we use the two's complement as its binary format ($b_{N-1},b_{N-2},...,b_0$), where the back and forth conversion can be described as:

$W_b = -127 + 2^7\cdot B_7 + 2^6 \cdot B_6 + \cdots\cdots\cdots 2^1\cdot B_1 + 2^0\cdot B_0$


> __Warning__: The correctness of the code is also depends on the ```dtype``` setup for the quantized weight, when convert it back and forth between signed integer and two's complement (unsigned integer). By default, we use ```.short()``` for 16-bit signed integers to prevent overflowing.


## License
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

The software is for educaitonal and academic research purpose only.