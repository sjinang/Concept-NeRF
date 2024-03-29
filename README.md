# Concept-NeRF: Image-enabled 3D editing based on ViCA-NeRF

Authors: Jinang Shah

Email: jinang@stanford.edu

Project: Concept-NeRF

Project Website: [Concept-NeRF](https://sites.google.com/view/3d-editing/)

Here we present only our modification to the existing codebase of ViCA-NeRF at https://github.com/Dongjiahua/VICA-NeRF.

After setting up the repository and installation, please follow following steps. It is highly recommended to use conda environment to setup the repository and code.

1. One is the textual_inversion_notebook.ipynb, file we provide with textual_inversion folder. With set path to reference images, and output directory, you can generate our textual embeddings and updated text encoder and tokenizers. You can also use instead textual_inversion_train.py file.

2. Train your NeRF model with ns-train nerfacto --load-data /path/to/processed/data

3. Once we have the trained NeRF on our data, replace or copy the contents of file "pipeline_stable_diffusion_instruct_pix2pix.py" provided in this folder into the file at: /opt/conda/envs/(your environment name)/lib/python3.8/site-packages/diffusers/pipelines/stable_diffusion/pipeline_stable_diffusion_instruct_pix2pix.py (for LINUX)

We do above in order to import tokenizer and text_encoder learned through textual inverison process in the first step.

Changes we make are in the function **_encode_prompt** of **class StableDiffusionInstructPix2PixPipeline**, and at the start of the function _encode_prompt(), it adds following:
```
# deleting existing tokenizer and text_encoder to free up space
del self.tokenizer
del self.text_encoder

from transformers import CLIPTokenizer, CLIPTextModel

# Path to directory that has learned/trained text_encoder and tokenizer from 1st step
# CHANGE PATH per your configuration
load_dir = "/home/rworxn/vica-nerf/textual_inversion/data/hulk_outputs"

# Load the trained models
self.tokenizer = CLIPTokenizer.from_pretrained(load_dir, subfolder="tokenizer",)
self.text_encoder = CLIPTextModel.from_pretrained(load_dir, subfolder="text_encoder").to(device)
```

4. Now you can train your Concept-NeRF with following command: ns-train vica --data {PROCESSED_DATA_DIR}  --load-dir {outputs/.../nerfstudio_models}  --pipeline.prompt {"Instruction"}

If your training fails due to lack of CUDA memory, you can try vica-small or vica-tiny in place of "vica" in the above command.


## Here is the setup from ViCA-NeRF repository:


### ViCA-NeRF: View-Consistency-Aware 3D Editing of Neural Radiance Fields
<img src="./assets/teaser.png">

This is the official implementation of [VICA-NeRF](https://openreview.net/pdf?id=Pk49a9snPe).

[Project-Page](https://dongjiahua.github.io/VICA-NeRF/)
## Installation
### Install pytorch and NeRFStudio:
**Recommented:** Follow the instructions on the [NeRFStudio](https://docs.nerf.studio/en/latest/quickstart/installation.html#dependencies).

**Directly install:**
```
conda create --name vica -y python=3.8 -y
conda activate vica -y
python -m pip install --upgrade pip

pip install torch==2.0.1+cu118 torchvision==0.15.2+cu118 --extra-index-url https://download.pytorch.org/whl/cu118 
pip install ninja git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch
pip install nerfstudio torchtyping
```

### Install VICA-NeRF
```
git clone https://github.com/Dongjiahua/VICA-NeRF
cd VICA-NeRF

pip install --upgrade pip setuptools
pip install -e .
```

## Usage
To edit the NeRF, it should be firstly trained on a scene. Following the instructions on [NeRFStudio](https://docs.nerf.studio/en/latest/quickstart/installation.html#dependencies), it can be trained by:
```
ns-train nerfacto --data {PROCESSED_DATA_DIR}
```
The model will be saved in `outputs/.../nerfstudio_models`. You can use the following command to edit the NeRF:
```
ns-train vica --data {PROCESSED_DATA_DIR}  --load-dir {outputs/.../nerfstudio_models}  --pipeline.prompt {"Instruction"}
```

#### Options
Our method supports great controll on the final result by 2D edits. The following options can be used:
```
--pipeline.control {bool} # Whether to mannuly set the target content. Default: False
--pipeline.warm_up_iters {int} # The number of iterations to warm up the model. Default: -1 (automatically set the warm up iterations)
--pipeline.post_refine {bool} # Whether to refine the result by post-refinement. Default: False. May hurt the result for large scene.
```

## Other tips for hyper-parameters
One previous work [Instruct-NeRF2NeRF](https://github.com/ayaanzhaque/instruct-nerf2nerf?tab=readme-ov-file#training-notes) give great suggestions on some tips for editing hyper-parameters. Please refer to their suggestions for better results.

## Acknowledgments
Our code is integrated based on the code framework of Instruct-NeRF2NeRF, thanks to their wonderful work

## Citation
```
@inproceedings{dong2023vica,
  title={ViCA-NeRF: View-Consistency-Aware 3D Editing of Neural Radiance Fields},
  author={Dong, Jiahua and Wang, Yu-Xiong},
  booktitle={Thirty-seventh Conference on Neural Information Processing Systems},
  year={2023}
}
```
