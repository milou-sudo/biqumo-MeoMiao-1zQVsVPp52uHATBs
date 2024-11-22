
![](https://img2024.cnblogs.com/blog/335778/202411/335778-20241121203141134-326554335.jpg)


最近由 magic\-quill 团队开源的 MagicQuill 项目十分引人瞩目，这个项目可以通过定制的 gradio 客户端针对不同的图像元素通过提示词进行修改，从而生成新的图像。值得一提的是，这个项目相当亲民，只需要20步迭代模型预测，甜品卡10秒钟就可以获取图片的修改效果，但是代价是至少需要40个G左右的磁盘空间。


本次分享一下如何在本地(Windows11\)来部署MagicQuill项目。


首先需要下载依赖的权重模型，压缩包地址:



```
https://hkustconnect-my.sharepoint.com/:u:/g/personal/zliucz_connect_ust_hk/EWlGF0WfawJIrJ1Hn85_-3gB0MtwImAnYeWXuleVQcukMg?e=Gcjugg&download=1

```

推荐使用某雷进行下载，速度会比较快一点。


注意，权重解压后大约需要30G的硬盘空间，请预留好相应的磁盘空间。


随后，克隆官方的最新代码:



```
git clone --recursive https://github.com/magic-quill/MagicQuill.git  
cd MagicQuill

```

把解压后的models目录，放入的项目的根目录。


复制权重后的目录结构：



```
E:\work\MagicQuill-main>treee -L 1  
MagicQuill-main  
├── LICENSE  
├── MagicQuill  
├── README.md  
├── check_env.py  
├── docs  
├── gradio_magicquill-0.0.1-py3-none-any.whl  
├── gradio_run.py  
├── hf_download  
├── models  
├── py311_cu118  
├── pyproject.toml  
├── requirements.txt  
├── tf_download  
├── 检测运行环境.bat  
└── 运行.bat

```

随后确保本地已经安装好 python3\.11，安装包可以去Python.org官网下载。


当然也可以使用conda：



```
conda create -n MagicQuill python=3.11 -y  
conda activate MagicQuill

```

官方推荐使用python3\.10，但经过验证，python3\.11也可以运行，且性能更好。


随后安装官方定制版本的gradio客户端：



```
pip install gradio_magicquill-0.0.1-py3-none-any.whl

```

由于项目依赖LLaVA，官方推荐使用pip安装:



```
(For Windows)  
copy /Y pyproject.toml MagicQuill\LLaVA\  
pip install -e MagicQuill\LLaVA\

```

但实际上，我们都知道，pip install \-e 命令用于安装一个处于开发模式下的 Python 包。它不会复制包文件到你的 site\-packages 目录，而是创建一个指向包源代码目录的符号链接（symbolic link）。这意味着你的项目代码的任何更改都会立即反映在你的 Python 环境中，而无需重新安装。这对于开发和调试非常有用。但是如果你的项目经常需要迁移，比如复制到别的磁盘或者发送给同事使用，这个包就会报错。


所以直接去LLaVA的官方项目，直接克隆项目:



```
git clone https://github.com/haotian-liu/LLaVA.git

```

随后把项目内的llava目录拷贝到当前Python环境的Lib\\site\-packages目录下即可，比如笔者的Python3\.11环境安装在：E:\\work\\MagicQuill\-main\\py311\_cu118\\目录，那么就把llava目录拷贝到：E:\\work\\MagicQuill\-main\\py311\_cu118\\Lib\\site\-packages 即可。


这样即使项目迁移，该模块也不会失效。


接着回到 MagicQuill 项目，安装基础依赖:



```
pip3 install -r requirements.txt

```

随后安装 torch 三件套:



```
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

```

接着修改 gradio\_run.py 文件，把这段代码:



```
ckpt_name = gr.Dropdown(  
                    label="Base Model Name",  
                    choices=folder_paths.get_filename_list("checkpoints"),  
                    value='SD1.5/realisticVisionV60B1_v51VAE.safetensors',  
                    interactive=True  
                )

```

修改为:



```
default_model = os.path.join('SD1.5', 'realisticVisionV60B1_v51VAE.safetensors')  
                ckpt_name = gr.Dropdown(  
                label="Base Model Name",  
                choices=folder_paths.get_filename_list("checkpoints"),  
                value=default_model,  
                interactive=True  
                )

```

意思是给基础模型增加一个默认值。


设置一下环境变量：



```
set HF_ENDPOINT=https://hf-mirror.com  
set CUDA_VISIBLE_DEVICES=0

```

最后运行命令启动服务:



```
python3 gradio_run.py

```

程序返回:



```
model_type EPS  
Using pytorch attention in VAE  
Using pytorch attention in VAE  
self.brushnet_loader.inpaint_files:  {'brushnet\\random_mask_brushnet_ckpt\\diffusion_pytorch_model.safetensors': 'E:\\work\\MagicQuill-main\\MagicQuill\\../models\\inpaint', 'brushnet\\segmentation_mask_brushnet_ckpt\\diffusion_pytorch_model.safetensors': 'E:\\work\\MagicQuill-main\\MagicQuill\\../models\\inpaint'}  
BrushNet model file: E:\work\MagicQuill-main\MagicQuill\../models\inpaint\brushnet\random_mask_brushnet_ckpt\diffusion_pytorch_model.safetensors  
BrushNet model type: SD1.5  
BrushNet model file: E:\work\MagicQuill-main\MagicQuill\../models\inpaint\brushnet\random_mask_brushnet_ckpt\diffusion_pytorch_model.safetensors  
Some parameters are on the meta device device because they were offloaded to the cpu.  
BrushNet SD1.5 model is loaded  
INFO:     Started server process [41504]  
INFO:     Waiting for application startup.  
INFO:     Application startup complete.  
INFO:     Uvicorn running on http://127.0.0.1:7860 (Press CTRL+C to quit)  
HTTP Request: GET https://api.gradio.app/pkg-version "HTTP/1.1 200 OK"

```

说明部署成功。


访问：[http://127\.0\.0\.1:7860](https://github.com)


![](https://v3u.cn/v3u/Public/js/editor/attached/20241121191105_28882.png)


如此，就可以在本地愉快地玩耍了。


 本博客参考[FlowerCloud机场](https://yunbeijia.com)。转载请注明出处！
