# Generative Art Using VQGAN and CLIP

### Setup

``` python
from google.colab import drive
from pathlib import Path

drive.mount('/content/drive')
full_path = Path('/content/drive/MyDrive', 'image-synthesis')
```

``` python
!nvidia-smi
```

``` python
# download vqgan and clip repos
!git clone https://github.com/CompVis/taming-transformers &> /dev/null
!git clone https://github.com/openai/CLIP &> /dev/null

# ai relevant libraries
!pip install ftfy regex tqdm omegaconf pytorch-lightning kornia einops &> /dev/null

# transformers and vqgan
!pip install transformers &> /dev/null  
!pip install taming.models &> /dev/null
 
# libraries for managing meta data
!pip install stegano &> /dev/null
!apt install exempi &> /dev/null
!pip install python-xmp-toolkit imgtag pillow==7.1.2 &> /dev/null

# reload all modules
%reload_ext autoreload
%autoreload &> /dev/null

# libraries for brand image creation
!pip install fire icon_font_to_png &> /dev/null
!git clone https://github.com/minimaxir/icon-image &> /dev/null
 
# libraries for video creation
!pip install imageio-ffmpeg &> /dev/null

# libraries for super resolution upscaling
!git clone https://github.com/Mirwaisse/SRCNN.git

!mkdir steps
```

### Model Selection

``` python
# VQGAN Model

imagenet_1024 = False
imagenet_16384 = True
coco = False
wikiart_1024 = False
wikiart_16384 = False
faceshq = False
sflckr = False

if imagenet_1024:
  model_alias = 'vqgan_imagenet_f16_1024'
  !curl -L -o vqgan_imagenet_f16_1024.ckpt -C - 'https://heibox.uni-heidelberg.de/f/140747ba53464f49b476/?dl=1' 
  !curl -L -o vqgan_imagenet_f16_1024.yaml -C - 'https://heibox.uni-heidelberg.de/f/6ecf2af6c658432c8298/?dl=1' 
if imagenet_16384:
  model_alias = 'vqgan_imagenet_f16_16384'
  !curl -L -o vqgan_imagenet_f16_16384.ckpt -C - 'https://heibox.uni-heidelberg.de/f/867b05fc8c4841768640/?dl=1'
  !curl -L -o vqgan_imagenet_f16_16384.yaml -C - 'https://heibox.uni-heidelberg.de/f/274fb24ed38341bfa753/?dl=1'
if coco:
  model_alias = 'coco'
  !curl -L -o coco.yaml -C - 'https://dl.nmkd.de/ai/clip/coco/coco.yaml'
  !curl -L -o coco.ckpt -C - 'https://dl.nmkd.de/ai/clip/coco/coco.ckpt'
if faceshq:
  model_alias = 'faceshq'
  !curl -L -o faceshq.yaml -C - 'https://drive.google.com/uc?export=download&id=1fHwGx_hnBtC8nsq7hesJvs-Klv-P0gzT'
  !curl -L -o faceshq.ckpt -C - 'https://app.koofr.net/content/links/a04deec9-0c59-4673-8b37-3d696fe63a5d/files/get/last.ckpt?path=%2F2020-11-13T21-41-45_faceshq_transformer%2Fcheckpoints%2Flast.ckpt'
if wikiart_1024:
  model_alias = 'wikiart_1024'
  !curl -L -o wikiart_1024.yaml -C - 'http://mirror.io.community/blob/vqgan/wikiart.yaml'
  !curl -L -o wikiart_1024.ckpt -C - 'http://mirror.io.community/blob/vqgan/wikiart.ckpt'
if wikiart_16384:
  model_alias = 'wikiart_16384'
  !curl -L -o wikiart_16384.ckpt -C - 'http://eaidata.bmk.sh/data/Wikiart_16384/wikiart_f16_16384_8145600.ckpt'
  !curl -L -o wikiart_16384.yaml -C - 'http://eaidata.bmk.sh/data/Wikiart_16384/wikiart_f16_16384_8145600.yaml'
if sflckr:
  model_alias = 'sflckr'
  !curl -L -o sflckr.yaml -C - 'https://heibox.uni-heidelberg.de/d/73487ab6e5314cb5adba/files/?p=%2Fconfigs%2F2020-11-09T13-31-51-project.yaml&dl=1'
  !curl -L -o sflckr.ckpt -C - 'https://heibox.uni-heidelberg.de/d/73487ab6e5314cb5adba/files/?p=%2Fcheckpoints%2Flast.ckpt&dl=1'


# Super Resolution Model

SR_FACTOR = 3

if SR_FACTOR == 2:
    !curl https://raw.githubusercontent.com/justinjohn0306/SRCNN/master/models/model_2x.pth -o model_2x.pth
elif SR_FACTOR == 3:
    !curl https://raw.githubusercontent.com/justinjohn0306/SRCNN/master/models/model_3x.pth -o model_3x.pth
elif SR_FACTOR == 4:
    !curl https://raw.githubusercontent.com/justinjohn0306/SRCNN/master/models/model_4x.pth -o model_4x.pth
```

### Load Libraries and Define Helpers

``` python
import argparse
import shutil
import math
from pathlib import Path
import sys

# sys.path is where Python searches for modules
sys.path.append('./taming-transformers')

from IPython import display
from base64 import b64encode
from omegaconf import OmegaConf
from PIL import Image
from taming.models import cond_transformer, vqgan
import torch
from torch import nn, optim
from torch.nn import functional as F
from torchvision import transforms
from torchvision.transforms import functional as TF
from tqdm.notebook import tqdm
 
from CLIP import clip
import kornia.augmentation as K
import numpy as np
import imageio
from PIL import ImageFile, Image
from imgtag import ImgTag
from libxmp import *     
import libxmp            
from stegano import lsb
import json

ImageFile.LOAD_TRUNCATED_IMAGES = True
 
def sinc(x):
    return torch.where(x != 0, torch.sin(math.pi * x) / (math.pi * x), x.new_ones([]))
 
def lanczos(x, a):
    cond = torch.logical_and(-a < x, x < a)
    out = torch.where(cond, sinc(x) * sinc(x/a), x.new_zeros([]))
    return out / out.sum()
 
def ramp(ratio, width):
    n = math.ceil(width / ratio + 1)
    out = torch.empty([n])
    cur = 0
    for i in range(out.shape[0]):
        out[i] = cur
        cur += ratio
    return torch.cat([-out[1:].flip([0]), out])[1:-1]
 
def resample(input, size, align_corners=True):
    n, c, h, w = input.shape
    dh, dw = size
 
    input = input.view([n * c, 1, h, w])
 
    if dh < h:
        kernel_h = lanczos(ramp(dh / h, 2), 2).to(input.device, input.dtype)
        pad_h = (kernel_h.shape[0] - 1) // 2
        input = F.pad(input, (0, 0, pad_h, pad_h), 'reflect')
        input = F.conv2d(input, kernel_h[None, None, :, None])
 
    if dw < w:
        kernel_w = lanczos(ramp(dw / w, 2), 2).to(input.device, input.dtype)
        pad_w = (kernel_w.shape[0] - 1) // 2
        input = F.pad(input, (pad_w, pad_w, 0, 0), 'reflect')
        input = F.conv2d(input, kernel_w[None, None, None, :])
 
    input = input.view([n, c, h, w])
    return F.interpolate(input, size, mode='bicubic', align_corners=align_corners)
 
class ReplaceGrad(torch.autograd.Function):
    @staticmethod
    def forward(ctx, x_forward, x_backward):
        ctx.shape = x_backward.shape
        return x_forward
 
    @staticmethod
    def backward(ctx, grad_in):
        return None, grad_in.sum_to_size(ctx.shape)
 
replace_grad = ReplaceGrad.apply
 
class ClampWithGrad(torch.autograd.Function):
    @staticmethod
    def forward(ctx, input, min, max):
        ctx.min = min
        ctx.max = max
        ctx.save_for_backward(input)
        return input.clamp(min, max)
 
    @staticmethod
    def backward(ctx, grad_in):
        input, = ctx.saved_tensors
        return grad_in * (grad_in * (input - input.clamp(ctx.min, ctx.max)) >= 0), None, None
 
 
clamp_with_grad = ClampWithGrad.apply
 
def vector_quantize(x, codebook):
    d = x.pow(2).sum(dim=-1, keepdim=True) + codebook.pow(2).sum(dim=1) - 2 * x @ codebook.T
    indices = d.argmin(-1)
    x_q = F.one_hot(indices, codebook.shape[0]).to(d.dtype) @ codebook
    return replace_grad(x_q, x)
 
class Prompt(nn.Module):
    ''' Prompt model
      Auxiliary model to perform inferece-by-optimization
      computes the loss between image cutouts and text prompt
    '''
    def __init__(self, embed, weight=1., stop=float('-inf')):
        super().__init__()
        self.register_buffer('embed', embed)
        self.register_buffer('weight', torch.as_tensor(weight))
        self.register_buffer('stop', torch.as_tensor(stop))
 
    def forward(self, input):
        ''' Compute Similarity '''
        # CLIP encoded image cutout batch
        input_normed = F.normalize(input.unsqueeze(1), dim=2) 
        # CLIP encoded text prompt
        embed_normed = F.normalize(self.embed.unsqueeze(0), dim=2)
        # Distance
        dists = input_normed.sub(embed_normed).norm(dim=2).div(2).arcsin().pow(2).mul(2)
        dists = dists * self.weight.sign()
        return self.weight.abs() * replace_grad(dists, torch.maximum(dists, self.stop)).mean()
 
def parse_prompt(prompt):
    ''' Parse text prompt of the form "prompt: weight: stop"
    '''
    vals = prompt.rsplit(':', 2)
    vals = vals + ['', '1', '-inf'][len(vals):]
    return vals[0], float(vals[1]), float(vals[2])
 
class MakeCutouts(nn.Module):
    ''' Image cutout and augmentation module '''
    def __init__(self, cut_size, cutn, cut_pow=1.):
        super().__init__()
        self.cut_size = cut_size
        self.cutn = cutn
        self.cut_pow = cut_pow
        self.noise_fac = 0.1
        self.augs = nn.Sequential(
            K.RandomHorizontalFlip(p=0.5),
            K.RandomSharpness(0.3, p=0.4),
            K.RandomAffine(degrees=30, translate=0.1, p=0.8, padding_mode='border'),
            K.RandomPerspective(0.2, p=0.4),
            K.ColorJitter(hue=0.01, saturation=0.01, p=0.7)
        )

    def forward(self, input):
        sideY, sideX = input.shape[2:4]
        max_size = min(sideX, sideY)
        min_size = min(sideX, sideY, self.cut_size)
        cutouts = []
        for _ in range(self.cutn):
            size = int(torch.rand([])**self.cut_pow * (max_size - min_size) + min_size)
            offsetx = torch.randint(0, sideX - size + 1, ())
            offsety = torch.randint(0, sideY - size + 1, ())
            cutout = input[:, :, offsety:offsety + size, offsetx:offsetx + size]
            cutouts.append(resample(cutout, (self.cut_size, self.cut_size)))
        batch = self.augs(torch.cat(cutouts, dim=0))
        if self.noise_fac:
            facs = batch.new_empty([self.cutn, 1, 1, 1]).uniform_(0, self.noise_fac)
            batch = batch + facs * torch.randn_like(batch)
        return batch

def load_vqgan_model(config_path, checkpoint_path):
    ''' Init VQGAN model in evaluation mode '''
    config = OmegaConf.load(config_path)
    if config.model.target == 'taming.models.vqgan.VQModel':
        model = vqgan.VQModel(**config.model.params)
        model.eval().requires_grad_(False)
        model.init_from_ckpt(checkpoint_path)
    elif config.model.target == 'taming.models.cond_transformer.Net2NetTransformer':
        parent_model = cond_transformer.Net2NetTransformer(**config.model.params)
        parent_model.eval().requires_grad_(False)
        parent_model.init_from_ckpt(checkpoint_path)
        model = parent_model.first_stage_model
    else:
        raise ValueError(f'Unknown model type: {config.model.target}')
    del model.loss
    return model

def resize_image(image, out_size):
    ratio = image.size[0] / image.size[1]
    area = min(image.size[0] * image.size[1], out_size[0] * out_size[1])
    size = round((area * ratio)**0.5), round((area / ratio)**0.5)
    return image.resize(size, Image.LANCZOS)
```

``` python
from IPython.display import Image as Image_

%cd /content/icon-image/
!python icon_image.py --icon_name "fab fa-spotify" --bg_color "white" --icon_color "#7b7568" --bg_width 500 --bg_height 500 --icon_size 400
%cd /content

Image_('icon-image/icon.png')
```

### Image Synthesis

``` python
multi_prompts = [
    ['colorful company logo of modern tech firm'],
    ['colorful company logo in the style of 1970s sci-fi book cover'],
    ['colorful company logo trending on artstation'],
    ['black and white company logo of nature lover'],
    ['colorful company logo of nature lover'],
    ['sketch of company logo of skyscraper'],
]
```

``` python
for prompts in multi_prompts:

    # retain qaudratic proportions, otherwise the output will be repetitive (CLIP only judges square sections)
    width, height =  512, 512
  
    # print frequency of GAN output (every `interval` iterations)
    interval = 50

    # initial image to guide generation (either path as string or None)
    initial_image = None  # str(Path(full_path, 'wooden-banana-temple-in-an-underwater-kingdom-steampunk', '0500.png'))

    # text prompts
    prompts = [p.strip() for p in prompts]
    
    # image prompts (either list of paths as string or None)
    image_prompts = ['/content/icon-image/icon.png'] #[str(Path(full_path, 'vulcano-temple-prompt.jpg'))]
    if not image_prompts:
        image_prompts = []
    else:
        image_prompts = [img.strip() for img in image_prompts]

    if initial_image or image_prompts != []:
        input_images = True

    # put None to get random seed
    seed = 2022

    # number of total iterations (-1: infinite)
    max_iterations = 500  

    args = argparse.Namespace(
        prompts=prompts,
        image_prompts=image_prompts,
        noise_prompt_seeds=[],
        noise_prompt_weights=[],
        size=[width, height],
        init_image=initial_image,
        init_weight=0.,
        clip_model='ViT-B/32',
        vqgan_config=f'{model_alias}.yaml',
        vqgan_checkpoint=f'{model_alias}.ckpt',
        step_size=0.1,
        cutn=64,
        cut_pow=1.,
        print_freq=interval,
        seed=seed,
    )

    device = torch.device('cuda:0' if torch.cuda.is_available() else 'cpu')

    if args.seed is None:
        seed = torch.seed()
    else:
        seed = args.seed
    torch.manual_seed(seed)

    if prompts:
        print(f'Using text prompt: {prompts}')
    if image_prompts:
        print(f'Using image prompts: {image_prompts}')
    print(f'Using seed: {seed}')

    # load vqgan model
    model = load_vqgan_model(args.vqgan_config, args.vqgan_checkpoint).to(device)

    # download CLIP model (and freeze weights)
    perceptor = clip.load(args.clip_model, jit=False)[0].eval().requires_grad_(False).to(device)

    # CLIP image encoder resolution: 224 x 224
    cut_size = perceptor.visual.input_resolution
    # codebook vector dimensionality: 256
    e_dim = model.quantize.e_dim
    # codebook size: 16384
    n_toks = model.quantize.n_e
    # ensure correct image sizing for init image and image prompt
    f = 2**(model.decoder.num_resolutions - 1)
    toksX, toksY = args.size[0] // f, args.size[1] // f
    sideX, sideY = toksX * f, toksY * f
    # min signal in codebook vectors
    z_min = model.quantize.embedding.weight.min(dim=0).values[None, :, None, None]
    # max signal in codebook vectors
    z_max = model.quantize.embedding.weight.max(dim=0).values[None, :, None, None]
    # generate `cutn` times number of augmentations cutouts
    make_cutouts = MakeCutouts(cut_size, args.cutn, cut_pow=args.cut_pow)

    # warm start from user-provided image (embed in VQ-GAN feature space)
    if args.init_image:
        pil_image = Image \
            .open(args.init_image).convert('RGB') \
            .resize((sideX, sideY), Image.LANCZOS)
        z, *_ = model.encode(TF.to_tensor(pil_image).to(device).unsqueeze(0) * 2 - 1)
    # alternatively init random noise vector
    else:
        one_hot = F.one_hot(torch.randint(n_toks, [toksY * toksX], device=device), n_toks).float()
        z = one_hot @ model.quantize.embedding.weight
        z = z.view([-1, toksY, toksX, e_dim]).permute(0, 3, 1, 2)
    z_orig = z.clone()

    # optimize noise vector (*inference-by-optimization*)
    z.requires_grad_(True)
    opt = optim.Adam([z], lr=args.step_size)

    normalize = transforms.Normalize(mean=[0.48145466, 0.4578275, 0.40821073],
                                     std=[0.26862954, 0.26130258, 0.27577711])

    pMs = []

    # parse and encode prompts in CLIP embedding space and init auxiliary Prompt model
    for prompt in args.prompts:
        txt, weight, stop = parse_prompt(prompt)
        embed = perceptor.encode_text(clip.tokenize(txt).to(device)).float()
        pMs.append(Prompt(embed, weight, stop).to(device))

    for prompt in args.image_prompts:
        path, weight, stop = parse_prompt(prompt)
        img = resize_image(Image.open(path).convert('RGB'), (sideX, sideY))
        batch = make_cutouts(TF.to_tensor(img).unsqueeze(0).to(device))
        embed = perceptor.encode_image(normalize(batch)).float()
        pMs.append(Prompt(embed, weight, stop).to(device))

    for seed, weight in zip(args.noise_prompt_seeds, args.noise_prompt_weights):
        gen = torch.Generator().manual_seed(seed)
        embed = torch.empty([1, perceptor.visual.output_dim]).normal_(generator=gen)
        pMs.append(Prompt(embed, weight).to(device))

    def synth(z):
        z_q = vector_quantize(z.movedim(1, 3), model.quantize.embedding.weight).movedim(3, 1)
        return clamp_with_grad(model.decode(z_q).add(1).div(2), 0, 1)

    def add_xmp_data(fname):
        image = ImgTag(filename=fname)
        params = {"prop_array_is_ordered": True, "prop_value_is_array": True}
        image.xmp.append_array_item(libxmp.consts.XMP_NS_DC, 'creator', 'VQGAN+CLIP', params)
        if args.prompts:
            image.xmp.append_array_item(libxmp.consts.XMP_NS_DC, 'title', " | ".join(args.prompts), params)
        else:
            image.xmp.append_array_item(libxmp.consts.XMP_NS_DC, 'title', 'None', params)
        image.xmp.append_array_item(libxmp.consts.XMP_NS_DC, 'i', str(i), params)
        image.xmp.append_array_item(libxmp.consts.XMP_NS_DC, 'model', model_alias, params)
        image.xmp.append_array_item(libxmp.consts.XMP_NS_DC, 'seed',str(seed) , params)
        image.xmp.append_array_item(libxmp.consts.XMP_NS_DC, 'input_images',str(input_images) , params)
        image.close()

    def add_stegano_data(filename):
        ''' Encodes metadata into image via stenography
            Decode via: https://stylesuxx.github.io/steganography/
        '''
        data = {"title": " | ".join(args.prompts) if args.prompts else None,
                "notebook": "VQGAN+CLIP",
                "i": i,
                "model": model_alias,
                "seed": str(seed),
                "input_images": input_images}
        lsb.hide(filename, json.dumps(data)).save(filename)

    @torch.no_grad()
    def checkin(i, losses):
        losses_str = ', '.join(f'{loss.item():g}' for loss in losses)
        tqdm.write(f'i: {i}, loss: {sum(losses).item():g}, losses: {losses_str}')
        out = synth(z)
        TF.to_pil_image(out[0].cpu()).save('progress.png')
        add_stegano_data('progress.png')
        add_xmp_data('progress.png')
        display.display(display.Image('progress.png'))

    def ascend_txt():
        ''' Gradient Ascent '''
        # get counter from global environment 
        global i
        # synthesized (i.e., decode) image based on noise vector
        out = synth(z)
        # batch of image cutouts encoded into CLIP embedding space
        iii = perceptor.encode_image(normalize(make_cutouts(out))).float()

        result = []

        if args.init_weight:
            result.append(F.mse_loss(z, z_orig) * args.init_weight / 2)

        # compute loss by comparing each image cutout with input text prompt(s) (loss per prompt)
        for prompt in pMs:
            result.append(prompt(iii))

        # save image and encode metadata
        img = np.array(out.mul(255).clamp(0, 255)[0].cpu().detach().numpy().astype(np.uint8))[:,:,:]
        img = np.transpose(img, (1, 2, 0))
        fname = f'steps/{i:04}.png'
        imageio.imwrite(fname, np.array(img))
        add_stegano_data(fname)
        add_xmp_data(fname)

        return result

    def train(i):
        opt.zero_grad()
        lossAll = ascend_txt()
        if i % args.print_freq == 0:
            checkin(i, lossAll)
        loss = sum(lossAll)
        loss.backward()
        # update noise vector Z
        opt.step()
        with torch.no_grad():
            z.copy_(z.maximum(z_min).minimum(z_max))

    # actual training loop
    i = 0
    try:
        with tqdm() as pbar:
            while True:
                train(i)
                if i == max_iterations:
                    break
                i += 1
                pbar.update()
    except KeyboardInterrupt:
        pass

    # save images
    if image_prompts:
      img_dir = Path(full_path, '|'.join(prompts).replace(' ', '-') + '_targeted')
    else:
      img_dir = Path(full_path, '|'.join(prompts).replace(' ', '-'))

    if not img_dir.exists():
        img_dir.mkdir()

    for i in tqdm(range(0, max_iterations + 1, args.print_freq)):
        fname = f'{i:04}.png'
        shutil.copy(
            Path('steps', fname),
            Path(img_dir, fname)
        )

    ### create video
    init_frame = 1
    last_frame = max_iterations
    total_frames = last_frame - init_frame

    min_fps = 10
    max_fps = 30

    length = 15 # Desired video runtime in seconds

    frames = []

    for i in range(init_frame, last_frame):
        fname = f'steps/{i:04}.png'
        frames.append(Image.open(fname))

    # adjust based on desired video length and min and max fps, respectively
    fps = np.clip(total_frames/length, min_fps, max_fps)

    video_fname = "video.mp4"

    from subprocess import Popen, PIPE
    cmd = ['ffmpeg', '-y', '-f', 'image2pipe', '-vcodec', 'png', '-r', str(fps),
           '-i', '-', '-vcodec', 'libx264', '-r', str(fps),
           '-pix_fmt', 'yuv420p', '-crf', '17', '-preset', 'veryslow', video_fname]
    p = Popen(cmd, stdin=PIPE)

    for img in tqdm(frames):
        img.save(p.stdin, 'PNG')
    p.stdin.close()
    p.wait()
    print("Video ready")

    shutil.copy(
        Path(video_fname),
        Path(img_dir, video_fname)
    )


    ### increase resolution via SRCNN
    cmd = ['python', '/content/SRCNN/run.py',
          '--zoom_factor', f'{SR_FACTOR}',
          '--model', f'/content/model_{SR_FACTOR}x.pth',
          '--image', f'{max_iterations:04}.png',
          '--cuda']
    process = Popen(cmd, cwd=f'/content/steps')
    stdout, stderr = process.communicate()
    if process.returncode != 0:
        print(stderr)
        raise RuntimeError(stderr)

    shutil.copy(
        Path('content', 'steps', f'zoomed_{max_iterations:04}.png'),
        Path(img_dir, f'zoomed_{max_iterations:04}.png')
    )
```
