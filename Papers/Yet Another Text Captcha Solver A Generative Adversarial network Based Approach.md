# Yet Another Text Captcha Solver: A Generative Adversarial network Based Approach
Summary of [paper](https://www.lancaster.ac.uk/staff/wangz3/publications/ccs18.pdf) by Guixin Ye, Zhanyong Tang, Dingyi Fang, Zhanxing Zhu, Yansong Feng, Pengfei Xu, Xiaojiang Chen, Zheng Wang

Focuses on the methods rather than the results

## 1. Introduction
_Presents a generic, yet effective text captcha solver based on the generative adversarial network._
_Implemented using Python. Preprocessing built upon the Pix2Pix framework, implemented using Tensorflow. Solver coded using keras_
### 장점
- Requires fewer training data
- Better performance
- Requires little human involvement and efforts
### 이론
- Generative adversarial network
- Transfer learning


## 2. Background
### Threat model
![image.png](https://github.com/alstjgg/alstjgg.github.io/blob/master/Yet%20Another%20Text%20Captcha%20Solver%20A%20Generative%20Adversarial%20network%20Based%20Approach/1.png)
- 1, 2, 6: **Anti-segmentation**
    1. Occluding Lines 
    2. Character Overlapping
    3. Noisy Background
    7. Character Set
- 3, 4, 5: **Anti-recognition**
    3. Font Style
    4. Waving / Distortion / Rotation
    5. Font color

### GAN(Generative Adversarial Networks)
Consists of two models
1. Generative Network: create synthetic examples
2. Discriminative Network: distinguish the synthesized examples from the real ones
-> Use backpropagation to train networks.

## 3. Overview
![image.png](https://github.com/alstjgg/alstjgg.github.io/blob/master/Yet%20Another%20Text%20Captcha%20Solver%20A%20Generative%20Adversarial%20network%20Based%20Approach/2.png)
### 1. Captcha Synthesis
-> Create a generator that can automatically generate an unbounded number of captchas for which the characters are known.
- GAN Captcha Generator = Captcha Generator + Discriminator
![image.png](https://github.com/alstjgg/alstjgg.github.io/blob/master/Yet%20Another%20Text%20Captcha%20Solver%20A%20Generative%20Adversarial%20network%20Based%20Approach/3.png)
1. Captcha Generator : Generate captchas that look similar to target captchas.
2. Discriminator : Tries to identify the synthetic captchas from the real ones.
3. Terminate when the discriminator fails to identify most synthetic captchas.
    -> if not, adjust synthesizer parameters
4. Trained generator creates a large number of captchas which labels are known.

### 2. Preprocessing
-> Preprocess captcha image to remove security features and standardize the font style
- Pix2Pix(GAN 중 하나) : Trained from synthetic captchas for which clean captchas are given(images without security features)

### 3. Train Base Solver
-> Train base solver with dataset created by generator
- Convolutional Neural Network

### 4. Fine-tune Base Solver
- Transfer Learning : Refine base solver with non-synthetic captcha images(labeled manually)


## 4. Implementation
### 1. Captcha Synthesizer
- input :  ① set of real captchas, ② set of security features
- output : set of optimal parameter values
1. Generator _G_
-> set of security features + captcha word = create captcha image
    -> CNN model modifies image at pixel level (configurable parameter values, fonts, etc..) using grid search model
        -> measure similarity as the ratio of synthetic captchas that cannot be distinguished by the discriminator
            -> modify model to search for optimal parameters
   
2. Discriminator _D_
- CNN => outputs the probability of an input captcha being a synthetic one
- mini-batch : randomly sampled synthetic captchas _x_(label: 1), and real captchas _y_(label: 0).
    -> Loss Function : ![image.png](https://github.com/alstjgg/alstjgg.github.io/blob/master/Yet%20Another%20Text%20Captcha%20Solver%20A%20Generative%20Adversarial%20network%20Based%20Approach/4.png)
- Stochastic Gradient Descent + Adam Solver + L1 norm
    -> Training Objective : ![image.png](https://github.com/alstjgg/alstjgg.github.io/blob/master/Yet%20Another%20Text%20Captcha%20Solver%20A%20Generative%20Adversarial%20network%20Based%20Approach/5.png)
    - _G_ tries to minimize difference of images(synthetic & real), _D_ tries to maximize it.
    - Update _G_: Fix parameters of discriminator / Update _D_: Fix parameters of generator


### 2. Preprocessing Model
- Deep learning : Pix2Pix image-to-image translation framework
    - Remove noise & occluding lines, Standardize font style(fill hollow parts, widen and standardizing gap between charaacters)
- GAN with generator & discriminator
    - Generator: Tries to remove security features
    - Discriminator: Tries to distinguish preprocessed images from clean images(w/o security features)

### 3. Solver
- CNN : LeNet-5
- n characters used to create captch imge -> n neurons in network
    - activation of each neuron : model's confidence that the character is the right one
    - choose neurons with largest activations
1. Train Base Solver - preprocessed synthetic captcha images
    - train a base solver for each target captcha scheme, for each possible number of characters
    - Bayesian based parameter tuner : choose hyperparameters
2. Build Fine-tuned Solver - manually labeled real captcha images
    - Transfer Learning : update later layers

