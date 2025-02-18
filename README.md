# DEMIURGE
![CI](https://github.com/buganart/descriptor-transformer/workflows/CI/badge.svg?branch=main)
[![codecov](https://codecov.io/gh/buganart/descriptor-transformer/branch/main/graph/badge.svg)](https://codecov.io/gh/buganart/descriptor-transformer)

## Development

### Installation

    pip install -e .


### Install package and development tools

Install `python >= 3.6` and `pytorch` with GPU support if desired.

    pip install -r requirements.txt


<!-- Run the tests

    pytest -->

<!-- 
### Option 2: Using nix and direnv

1. Install the [nix](https://nixos.org/download.html) package manager
and [direnv](https://direnv.net/).
2. [Hook](https://direnv.net/docs/hook.html) `direnv` into your shell.
3. Type `direnv allow` from within the checkout of this repository. -->

## INTRODUCTION
*Demiurge* is a tripartite neural network architecture devised to generate and sequence audio waveforms (Donahue et al. 2019). The architecture combines a synthesis engine based on a **una-GAN** + **mel-GAN / hifi-GAN** model with a custom **transformer-based sequencer**. The diagram below explains the relation between elements.

<p align="center">
    <img src="https://user-images.githubusercontent.com/68105693/121159742-be8dd280-c84b-11eb-996f-bcd68971b201.png" width="70%" height="30%" align="center">
</p>

Audio generation and sequencing neural-network-based processes work as follows:

1. Modified versions of **[mel-GAN](https://github.com/buganart/melgan-neurips)** (a vocoder that is a convolutional non-autoregressive feed-forward adversarial network ) / **[hifi-GAN](https://github.com/jik876/hifi-gan)** (introducing a similar approach to mel-GAN but with one multi-scale and one multi-period discriminator) and **[una-GAN](https://github.com/buganart/unagan)** (an auto-regressive unconditional sound generating boundary-equilibrium GAN) will first process audio files `.wav` from an original database `RECORDED AUDIO DB` to produce GAN-generated `.wav` sound files, which are compiled into a new database `RAW GENERATED AUDIO DB`. 

2. The **descriptor** in the **[sequencer model](https://github.com/buganart/descriptor-transformer)** extracts a series of Los Mel Frequency Cepstral Coeﬃcients `MFCC` `.json` strings from the audio files in the `PREDICTOR DB` while the **predictor**, a time-series prediction model, generates projected descriptor sequences based on that data. 

3. As the predicted descriptors are just statistical values, a **query engine** matches them with those extracted from the `RAW GENERATED AUDIO DB` before it replaces matched with predicted descriptors using the audio reference from the `RAW GENERATED AUDIO DB`, merging and combining the resultant sound sequences into an output `.wav` audio file.

Please bear in mind that our model uses **[WandB](https://wandb.ai/)** to track and monitor training.

## SYNTHESIS ENGINE (mel-GAN / hifi-GAN + una-GAN)

The chart below explains the GAN-based sound synthesis process. Please bear in mind that for ideal results the **mel-GAN / hifi-GAN** and **una-GAN** audio databases should be the same. Cross-feeding between different databases generates unpredictable (although sometimes musically interesting) results. Please record the `wandb_run_ids` for the final sound generation process. 


<p align="center">
    <img src="https://user-images.githubusercontent.com/68105693/121171334-a96a7100-c856-11eb-9f4e-19cdafba2921.png" width="50%" height="20%" align="center">
</p>

### 1. mel-GAN

**[mel-GAN](https://github.com/buganart/melgan-neurips)**  (Kumar et al. 2019) is a fully convolutional non-autoregressive feed-forward adversarial network that uses mel-spectrograms as a lower-resolution audio representation model that can be both efficiently computed from and inverted back to raw audio format. An average melGAN run on [Google Colab](https://colab.research.google.com/) using a single V100 GPU may need a week to produce satisfactory results. The results obtained using a multi-GPU approach with parallel data vary. To train the model please use the following [notebook](https://colab.research.google.com/drive/1xUrh2pNUBTMO4s4YPsxAbUdTdlHjTeVU?usp=sharing).

<p align="center">
    <img src="https://user-images.githubusercontent.com/68105693/115818429-53b94100-a42f-11eb-9cb5-1c6c20ba5243.png" width="70%" height="30%" align="center">
</p>
<p align="center"> (Kumar et al. 2019)

### 2. hifi-GAN

**[hifi-GAN](https://github.com/jik876/hifi-gan)** (Bae et al. 2020) is a convolutional non-autoregressive adversarial network that, as the mel-GAN, uses mel-spectrograms as a symbolic audio representation. The model combines a multi-receptive field fusion (MRF) generator with two different discriminators, introducing multi-period (MPD) and multi-scale (MSD) structures. 

<p align="center">
     <img width="600" alt="Screenshot 2021-06-08 at 12 51 30" src="https://user-images.githubusercontent.com/68105693/121173292-efc0cf80-c858-11eb-91eb-eb4a6ca29603.png">
</p>
<p align="center"> (Bae et al. 2020)
    
### 3. una-GAN

**[una-GAN](https://github.com/buganart/unagan)** (Liu et al. 2019) is an auto-regressive unconditional sound generating boundary-equilibrium GAN (Berthelot et al. 2017) that takes variable-length sequences of noise vectors to produce variable-length mel-spectrograms. A first UNAGAN model was eventually revised by Liu et al. at [Academia Sinica](https://musicai.citi.sinica.edu.tw) to improve the resultant audio quality by introducing in the generator a hierarchical architecture  model and circle regularization to avoid mode collapse. The model produces satisfactory results after 2 days of training on a single V100 GPU. The results obtained using a multi-GPU approach with parallel data vary. To train the model please use the following [notebook](https://colab.research.google.com/drive/1JEXcGs-zVoAi84e79OO-7_G9qD3ptNwF?usp=sharing).

### 4. Generator

After training **mel-GAN/hifi-GAN** and **una-GAN**, you will have to use **[una-GAN generate](https://github.com/buganart/descriptor-transformer/blob/main/predict_notebook/Unagan_generate.ipynb)** to output `.wav` audio files. Please set the `melgan_run_id` or `hifi_run_id` and `unagan_run_id` created in the previous training steps. The output `.wav` files will be saved to the `output_dir` specified in the notebook. To train the model please use the following [notebook](https://colab.research.google.com/github/buganart/unagan/blob/master/Unagan_generate.ipynb). The table below provides a selection of our best `wandb_run_ids` that may be used to run **una-GAN generate**.
    
<p align="center">
    <img width="600" alt="Screenshot 2021-06-08 at 13 26 43" src="https://user-images.githubusercontent.com/68105693/121177081-44664980-c85d-11eb-9478-81102595be89.png">
</p>

## SEQUENCER MODEL

The **sequencer** combines an `MFCC` descriptor extraction model with a descriptor predictor generator and query and playback engines that generate `.wav` audio files out of those `MFCC` `.json` files. The diagram below explains the relation between the different elements of the prediction-transformer-query-playback workflow.

<p align="center">
    <img src="https://user-images.githubusercontent.com/68105693/115947129-2e443a00-a4f8-11eb-9abb-6503a389a41f.png" width="60%" height="35%" align="center">
</p>

### 1. Descriptor Prediction Model

As outlined above, the **descriptor model** plays a crucial role in the the prediction workflow. You may use pretrained descriptor data by selecting a `wandb_run_id` from the **[descriptor model](https://github.com/robertoalonsotrillo/descriptor-transformer/blob/main/predict_notebook/descriptor_model_predict.ipynb)** or train your own model using this [notebook](https://colab.research.google.com/github/buganart/descriptor-transformer/blob/main/predict_notebook/descriptor_model_predict.ipynb), following the instructions found there, to generate `MFCC` `.json` files.

Four different time-series predictors were implemented as training options. Both the "LSTM" and "transformer encoder-only model" are one step prediction models, while "LSTM encoder-decoder model" and "transformer model" can predict descriptor sequences with specified sequence length. 

- **LSTM** (Hochreiter et al. 1997)
- **LSTM encoder-decoder model** (Cho et al. 2014)
- **Transformer encoder-only model**
-  **Transformer model** (Vaswani et al. 2017)

Once you train the model, record the `wandb_run_id` and paste it in the **[prediction notebook](https://github.com/buganart/descriptor-transformer/blob/main/predict_notebook/descriptor_model_predict.ipynb)**. Then, provide paths to the `RAW generated audio DB` and `Prediction DB` databases and and run the notebook to generate new descriptors. The descriptors genereted from `Prediction DB` will be used as the input of the neural sequencer to predict subsequent descriptors, which will be converted into `.wav` audio files using the **query and playback engines** (see below). To train the model please use the following [notebook](https://colab.research.google.com/drive/1xUrh2pNUBTMO4s4YPsxAbUdTdlHjTeVU?usp=sharing).

#### Training (script alternative)

You may alternatively train the descriptor model using a database containing files in `.wav` format by running

    python desc/train_function.py --selected_model <1 of 4 models above> --audio_db_dir <path to database> --window_size <input sequence length> --forecast_size <output sequence length> 


### 2. Query and playback engines

This is the workflow of the **query and playback engines**, which will translate `MFCC` `.json` files into `.wav` audio files. This workflow partially overlaps with the instructions provided above on the **descriptor predictor model**.  

1. The **descriptor model** processes the `PREDICTION DB` databse (see diagram above) to generate *descriptor input sequences* and saves them in `DESCRIPTOR DB II`. It then predicts subsequent descriptor strings based on that data.

2. The model processes the audio database into `DESCRIPTOR DB I` and links each descriptor to an `ID reference` connected to the specific audio segment.
 
3. The **query function** replaces the new predicted descriptors generated by the **descriptor model** with the closest match, based on a distance function, found in the `DESCRIPTOR DB I` 

4. The model combines and merges these segments referenced by the replaced descriptors from the query function into a new `.wav` audio file.

To train the model please use the following [notebook](https://colab.research.google.com/drive/1xUrh2pNUBTMO4s4YPsxAbUdTdlHjTeVU?usp=sharing).



