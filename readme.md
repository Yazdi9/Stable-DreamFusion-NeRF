# Stable-Dreamfusion

A pytorch implementation of the text-to-3D model **Dreamfusion**, powered by the [Stable Diffusion](https://github.com/CompVis/stable-diffusion) text-to-2D model.

# Results 

<div align=center>
  
<img  src="https://user-images.githubusercontent.com/33378412/229326006-de9d43ab-9e84-4976-9a7c-f05ebfab13e3.gif" width="400" height="420">
<img  src="https://user-images.githubusercontent.com/33378412/229326120-50a2eaf6-722f-4b7a-889b-bdd330388721.gif" width="400" height="420"> 
<img  src="https://user-images.githubusercontent.com/33378412/229326360-bf207b36-d8ff-449a-b07b-3e853ecfa659.gif" width="400" height="420">
<img  src="https://user-images.githubusercontent.com/33378412/229326538-a532fea1-1e25-47c7-a794-9841b43a73c8.gif">

 </div>

# Colab notebooks:
* [![Instant-NGP Backbone](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1MXT3yfOFvO0ooKEfiUUvTKwUkrrlCHpF?usp=sharing)

* [![Vanilla Backbone](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1mvfxG-S_n_gZafWoattku7rLJ2kPoImL?usp=sharing)

# Install

```bash
git clone https://github.com/ashawkey/stable-dreamfusion.git
cd stable-dreamfusion
```

### Install with pip
```bash
pip install -r requirements.txt

# (optional) install nvdiffrast for exporting textured mesh (if use --save_mesh)
pip install git+https://github.com/NVlabs/nvdiffrast/

# (optional) install CLIP guidance for the dreamfield setting
pip install git+https://github.com/openai/CLIP.git

```

### Build extension 

```bash
# install all extension modules
bash scripts/install_ext.sh

# if you want to install manually, here is an example:
pip install ./raymarching # install to python path (you still need the raymarching/ folder, since this only installs the built extension.)
```

### Taichi backend 
Use [Taichi](https://github.com/taichi-dev/taichi) backend for Instant-NGP. It achieves comparable performance to CUDA implementation while **No CUDA** build is required. Install Taichi with pip:
```bash
pip install -i https://pypi.taichi.graphics/simple/ taichi-nightly
```

```bash
#### stable-dreamfusion setting

## train with text prompt (with the default settings)

python main.py --text "a hamburger" --workspace trial -O

# reduce stable-diffusion memory usage with `--vram_O` 
# enable various vram savings (https://huggingface.co/docs/diffusers/optimization/fp16).
python main.py --text "a hamburger" --workspace trial -O --vram_O
# this makes it possible to train with larger rendering resolution, which leads to better quality (see https://github.com/ashawkey/stable-dreamfusion/pull/174)
python main.py --text "a hamburger" --workspace trial -O --vram_O --w 300 --h 300 # Tested to run fine on 8GB VRAM (Nvidia 3070 Ti).

# use CUDA-free Taichi backend with `--backbone grid_taichi`
python3 main.py --text "a hamburger" --workspace trial -O --backbone grid_taichi

# choose stable-diffusion version (support 1.5, 2.0 and 2.1, default is 2.1 now)
python main.py --text "a hamburger" --workspace trial -O --sd_version 1.5

# we also support negative text prompt now:
python main.py --text "a rose" --negative "red" --workspace trial -O

## if the above command fails to generate meaningful things (learns an empty scene), maybe try:
# 1. disable random lambertian/textureless shading, simply use albedo as color:
python main.py --text "a hamburger" --workspace trial -O --albedo
# 2. use a smaller density regularization weight:
python main.py --text "a hamburger" --workspace trial -O --lambda_entropy 1e-5

# you can also train in a GUI to visualize the training progress:
python main.py --text "a hamburger" --workspace trial -O --gui

# A Gradio GUI is also possible (with less options):
python gradio_app.py # open in web browser

## after the training is finished:
# test (exporting 360 degree video)
python main.py --workspace trial -O --test
# also save a mesh (with obj, mtl, and png texture)
python main.py --workspace trial -O --test --save_mesh
# test with a GUI (free view control!)
python main.py --workspace trial -O --test --gui


## train
# `-O2` equals `--dir_text --backbone vanilla`
python main.py --text "a hotdog" --workspace trial2 -O2

## if CUDA OOM, maybe try:
# 1. only use albedo rendering, less GPU memory (~16G), train faster, but results may be worse
python main.py --text "a hotdog" --workspace trial2 -O2 --albedo
# 2. reduce NeRF sampling steps (--num_steps and --upsample_steps)
python main.py --text "a hotdog" --workspace trial2 -O2 --num_steps 64 --upsample_steps 0

## test
python main.py --workspace trial2 -O2 --test
python main.py --workspace trial2 -O2 --test --save_mesh
python main.py --workspace trial2 -O2 --test --gui # not recommended, FPS will be low.
```


