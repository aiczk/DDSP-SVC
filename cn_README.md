Language: [English](./README.md) ** 简体中文 **
# DDSP-SVC
<div align="center">
<img src="https://storage.googleapis.com/ddsp/github_images/ddsp_logo.png" width="200px" alt="logo"></img>
</div>
基于 DDSP 的端到端歌声转换系统

## 1. 安装依赖
我们推荐从 [**PyTorch 官方网站 **](https://pytorch.org/) 下载 PyTorch.

接着运行
```bash
pip install -r requirements.txt 
```
注： 我只在 python 3.8 (windows) + pytorch 1.9.1 + torchaudio 0.6.0 测试过代码，太旧或太新的依赖可能会报错。
## 2. 配置预训练模型
- **(必要操作)** 下载预训练 [**HubertSoft**](https://github.com/bshall/hubert/releases/download/v0.1/hubert-soft-0d54a1f4.pt) 编码器并将其放到 `pretrain/hubert` 文件夹.
-  从 [DiffSinger 社区声码器项目](https://openvpi.github.io/vocoders) 下载基于预训练声码器的增强器，并解压至 `pretrain/` 文件夹。
## 3. 预处理

将所有的训练集数据 (.wav 格式音频切片) 放到 `data/train/audio`.

将所有的验证集数据 (.wav 格式音频切片) 放到 `data/val/audio`.

接着运行
```bash
python preprocess.py -c configs/combsub.yaml
```

训练基于梳齿波减法合成器的模型，或者运行

```bash
python preprocess.py -c configs/sins.yaml
```
训练基于正弦波加法合成器的模型。

您可以在预处理之前修改配置文件 `config/<model_name>.yaml`。

默认配置适用于GTX-1660 显卡训练 44.1khz 高采样率合成器。

注 1: 请保持所有音频切片的采样率与 yaml 配置文件中的采样率一致！如果不一致，程序可以跑，但训练过程中的重新采样将非常缓慢。

注 2：训练数据集的音频切片总数建议为约 1000 个，另外长音频切成小段可以加快训练速度，但所有音频切片的时长不应少于 2 秒。如果音频切片太多，则需要较大的内存，配置文件中将 `cache_all_data` 选项设置为 false 可以解决此问题。

注 3：验证集的音频切片总数建议为 10 个左右，不要放太多，不然验证过程会很慢。

## 4. 训练
```bash
# 以训练 combsub 模型为例 
python train.py -c configs/combsub.yaml
```

训练其他模型方法类似。

您可以随时中止训练，然后运行相同的命令来继续训练。

您也可以在中止训练后，重新预处理新数据集或更改训练参数（batchsize、lr等），然后运行相同的命令，就可以对模型进行微调 (finetune)。
## 5. 可视化
```bash
# 使用tensorboard检查训练状态
tensorboard --logdir=exp
```
## 6. 测试
```bash
# 以下是 ddsp-svc 变声器的原始输出
# 速度快，但音质相对较低
python main.py -i <input.wav> -m <model_file.pt> -o <output.wav> -k <keychange (semitones)>
```
```bash
# 使用预训练声码器增强输出结果
# 正常音域范围内的将有更高的音质，但速度较慢
python main.py -i <input.wav> -m <model_file.pt> -o <output.wav> -k <keychange (semitones)> -e true
```
```bash
# 关于 f0 提取器的其他选项，参见
python main.py -h
```
