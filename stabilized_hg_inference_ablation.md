# Stabilized History Guidance 推理消融与项目研究说明

日期：2026-05-19

本文用于总结当前 NowcastDiT/DF-like 雷达预测小项目中已经完成的测试、训练与推理评估，并把模型机制、数学背景、代码实现和文献依据整理成一份可以用于组会或面试阐述的研究说明。

需要先强调一个边界：当前所有评估主要是小规模抽样评估，日志显示每次评估为 192 个样本、20 帧预测，并保存前 10 个 case 的可视化 GIF。因此这些结果可以支持方向判断和机制解释，但还不能视为最终统计结论。

## 1. 当前项目目标

原始 NowcastDiT 更接近 full-sequence generation：给定历史雷达序列后，将未来序列整体作为生成目标。当前改造方向是将其推向 Diffusion Forcing 风格：

- 每个时间 token/frame 可以处于不同噪声水平；
- 历史帧可以保持干净、完全加噪或部分作为 context；
- 推理时可以通过一个二维的“采样调度矩阵”控制不同未来帧的去噪进度；
- 在 history-free fine-tuning 后，可以用 History Guidance 提升条件历史对未来生成的约束力。

这一路线的核心不是单纯换一个 attention mask，而是让模型学习“在任意历史/未来噪声状态组合下，如何从上下文中恢复未来动态”。

## 2. 文献脉络

### 2.1 SEVIR 数据与雷达 nowcasting

SEVIR 是 NeurIPS 2020 的气象事件图像数据集，包含超过 10,000 个天气事件，每个事件覆盖 384 km x 384 km、4 小时时间跨度，并对 GOES 卫星、NEXRAD VIL 雷达、GLM 闪电等多源数据做时空对齐。论文也明确把 precipitation nowcasting 与 synthetic weather radar generation 作为示例任务，并包含评估指标设计。参考：[SEVIR: A Storm Event Imagery Dataset for Deep Learning Applications in Radar and Satellite Meteorology](https://papers.nips.cc/paper_files/paper/2020/hash/fa78a16157fed00d7a80515818432169-Abstract.html)。

本项目使用 SEVIR VIL 序列，当前配置为 `cond_len=5`，`pred_len=20`，即 5 帧历史预测 20 帧未来。

### 2.2 DDPM 与 DDIM

DDPM 的标准前向加噪过程为：

```text
q(x_t | x_0) = N(sqrt(alpha_bar_t) x_0, (1 - alpha_bar_t) I)
x_t = sqrt(alpha_bar_t) x_0 + sqrt(1 - alpha_bar_t) epsilon
```

模型通常学习噪声预测：

```text
L = E_{x_0, epsilon, t} || epsilon - epsilon_theta(x_t, t, c) ||_2^2
```

本项目的代码实现对应 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:134) 的 `q_sample`，以及 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:158) 的 `forward` loss。

DDIM 的价值在于它保留类似训练目标，但推理可使用更少采样步数，并通过 `eta` 控制确定性/随机性。论文摘要指出 DDIM 可比 DDPM 快 10x 到 50x，并允许用同一训练目标构造非马尔可夫采样过程。参考：[DDPM](https://arxiv.org/abs/2006.11239)，[DDIM](https://arxiv.org/abs/2010.02502)。

在代码中，DDIM 更新位于 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:479)，关键公式是：

```text
x_{t_next}
= sqrt(alpha_next) x_0
  + sqrt(1 - alpha_next - sigma^2) epsilon_theta
  + sigma z
```

其中 `sigma` 由 `eta` 控制。实验上，causal checkpoint 在 `eta=1.0` 下明显优于 `eta=0.0`，说明这个任务里保留一定采样随机性有助于恢复未来动态，尤其在强回波移动和不确定区域。

### 2.3 DiT / MM-DiT 与 latent patch transformer

DiT 论文提出用 Transformer 替代 U-Net 作为扩散模型主干，在 latent patch 上做扩散建模，并观察到更高 Gflops 通常对应更低 FID。参考：[Scalable Diffusion Models with Transformers](https://arxiv.org/abs/2212.09748)。

MM-DiT/Rectified Flow 进一步强调 Transformer 对多模态 token 的双向信息流、噪声尺度采样和高分辨率生成扩展性。参考：[Scaling Rectified Flow Transformers for High-Resolution Image Synthesis](https://arxiv.org/abs/2403.03206)。

本项目虽然不是文本到图像，但借用了同类设计思想：把雷达 latent 序列切成时空 patch token，加入时间/位置/条件嵌入，用 Transformer 预测每帧 latent 的噪声。核心实现位于：

- DiT patch embedding 与 timestep embedding：[model/df_dit/video_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/video_dit.py:234)
- AdaLN DiT block：[model/df_dit/video_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/video_dit.py:96)
- per-frame timestep 注入：[model/df_dit/video_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/video_dit.py:318)

### 2.4 Diffusion Forcing

Diffusion Forcing 的核心观点是：模型不再只接收全序列同一噪声水平，而是训练模型去 denoise 一组 token，每个 token 可以有独立噪声水平。论文摘要明确说它把 next-token prediction 的可变长度/因果生成能力与 full-sequence diffusion 的轨迹引导能力结合起来，并证明其优化所有子序列似然的变分下界。参考：[Diffusion Forcing: Next-token Prediction Meets Full-Sequence Diffusion](https://arxiv.org/abs/2407.01392)。

本项目中的对应实现是：

```text
每个 batch 样本有 t_idx: shape = (B, T)
x_t[b, i] = q(x_0[b, i], t_idx[b, i])
```

代码位置：

- 每帧独立采样 timestep：[model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:151)
- 保持历史帧干净：[model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:163)
- 只在未来帧上算 loss：[model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:186)

### 2.5 DFoT 与 History Guidance

DFoT/History-Guided Video Diffusion 发现视频历史条件 CFG 有两个问题：固定条件长度架构不适合可变历史；直接 history dropout 的 CFG 风格训练可能表现不好。DFoT 提出支持可变历史帧的 Diffusion Forcing Transformer，并引入 History Guidance。其最简单形式 vanilla HG 已能改善视频质量与时间一致性，更高级的 time/frequency HG 能改善运动动态和长视频 rollout 稳定性。参考：[History-Guided Video Diffusion](https://arxiv.org/abs/2502.06764)，官方代码：[kwsong0113/diffusion-forcing-transformer](https://github.com/kwsong0113/diffusion-forcing-transformer)。

官方 README 的示例命令中也出现了 `history_guidance.name=stabilized_vanilla`、`guidance_scale=4.0` 和 `stabilization_level=0.02`，这与本项目采用 `history_guidance_mode=stabilized`、`history_guidance_stabilization_level=0.02` 的启发来源一致。

本项目的 HG 公式是 CFG 形式：

```text
epsilon_guided = epsilon_uncond + s * (epsilon_cond - epsilon_uncond)
```

代码位于 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:303)。其中：

- `epsilon_cond`：保留被选中的历史/已生成 context；
- `epsilon_uncond`：把选中 context 替换为全噪声或无条件 token；
- `s = history_guidance_scale`。

当前 stabilized HG 是推理端近似：对已生成 future context 先加一个很低的噪声水平，再进入 conditional 分支，降低“完全干净的模型自身预测帧”作为历史时导致的分布偏移。代码位于 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:267)。

## 3. 模型结构与关键实现

### 3.1 数据流

当前 SEVIR VIL 图像先经已有 VAE 编码成 latent，扩散模型在 latent 空间预测未来序列：

```text
VIL frames -> VAE encoder -> latent sequence z_0
z_0 + per-frame noise -> z_t
DiT(z_t, per-frame timestep, history condition) -> predicted noise
DDIM/DDPM sampler -> generated latent sequence
VAE decoder -> predicted VIL frames
```

训练/评估入口在 [train_df_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/train_df_dit.py:26) 和 [train_df_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/train_df_dit.py:112)。评估时会加载模型 checkpoint、VAE checkpoint，并切换到第二组 EMA 参数：[train_df_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/train_df_dit.py:142)。

### 3.2 Per-frame timestep

原始 full-sequence diffusion 通常整段序列共享同一扩散时间；本项目改为每帧一个 timestep：

```python
t_idx = torch.randint(0, self.num_timesteps, (b, t), device=x.device)
```

对应 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:151)。DiT 端通过 `_prepare_timestep_conditioning` 把 `(B,T)` timestep 展开到所有 patch token，并保留一个全局平均 timestep 作为 AdaLN conditioning：[model/df_dit/video_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/video_dit.py:318)。

这点很关键：模型不只是知道“现在是第几步扩散”，而是知道每一帧各自的噪声水平。这就是 DF-like 的基础。

### 3.3 Causal pyramid schedule

推理时使用二维调度矩阵：

```text
schedule: shape = (sample_steps + 1, B, T)
schedule[m, b, i] = 第 m 个采样步时，第 i 帧处于哪个 diffusion timestep
```

`-1` 表示该帧已经完成去噪，可以作为生成历史。实现位于 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:412) 和 [model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:453)。

`causal_pyramid` 的直观含义是：越靠后的未来帧，开始去噪越晚。这样可让模型先生成近未来，再逐渐把近未来作为 context 影响远未来，接近 autoregressive rollout，但仍保留 diffusion 的多步修正能力。

### 3.4 Attention mask

当前主要比较了两类 attention：

- `noncausal`：所有时间帧互相可见；
- `block_causal` + `block_size=5`：第 k 个 5 帧 block 可看自己和过去 block，不看未来 block。

Attention mask 的抽象在 [model/df_dit/attention.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/attention.py:46)，其中 `block_causal` 的时间约束是：

```python
allowed = allowed & ((kv_t // block_size) <= (q_t // block_size))
```

见 [model/df_dit/attention.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/attention.py:99)。

DiT 中实际构造 mask spec 的位置是 [model/df_dit/video_dit.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/video_dit.py:294)。需要注意：`block_size=5` 不是严格逐帧自回归，而是 5 帧一组的块因果。它能降低未来泄漏，但保留块内双向建模，适合雷达这种局部连续运动任务。

### 3.5 History-free fine-tuning 与 HG 的关系

History Guidance 需要模型同时理解“有历史”和“无历史/历史被破坏”的两种条件分布。训练中通过 `history_free_prob=0.15` 做 history-free dropout：

- 对一部分样本，把历史 latent 加到最大噪声；
- 同时把条件输入置零或替换为无条件 token；
- 让模型学会在缺历史时的预测分布。

代码位置：[model/df_dit/denoiser.py](/dataHW/workspace/zhangyuchen/code/NowcastDiT/model/df_dit/denoiser.py:537)。

当前 noncausal 和 causal 的 HG 微调都是从 200 epoch base checkpoint 继续到 epoch 230：

| run | base ckpt | lr | history_free_prob | attention |
|---|---|---:|---:|---|
| noncausal HG FT | `20260516-195905.../checkpoint_epoch_200.pth` | 2e-5 | 0.15 | noncausal |
| causal HG FT | `20260516-195644.../checkpoint_epoch_200.pth` | 2e-5 | 0.15 | block_causal, block=5 |

## 4. 已补跑/已盘点测试

### 4.1 CPU smoke tests

已补跑：

```bash
python smoke_tests/df_dit_mask_timestep_smoke.py --device cpu --batch-size 1 --frames 6 --channels 4 --size 8 --hidden-size 32 --depth 2 --heads 4 --patch-size 2
```

结果：

```text
block_size=0/1/3 forward + backward 均通过
mask_shape=(1, 1, 96, 96), visible=6912
[OK] DF DiT mask/timestep smoke passed
```

已补跑：

```bash
python smoke_tests/df_dit_denoiser_smoke.py --device cpu --batch-size 1 --cond-len 2 --pred-len 3 --channels 2 --img-size 8 --sample-steps 3
```

结果：

```text
loss=1.041530, final_grad_norm=0.761174
generated_shape=(1, 5, 2, 8, 8), history_fixed=True
[OK] DF-like denoiser smoke passed
```

已补跑：

```bash
PYTHONPATH=. pytest -q tests/test_df_attention_backends.py
```

结果：

```text
7 passed, 1 warning in 3.67s
```

说明 attention 后端、dense/debug mask、block sparse 近似路径和 DiT forward 的基本数值稳定性是通过的。

### 4.2 未成功补跑的 full path smoke

尝试运行：

```bash
python smoke_tests/df_dit_full_path_smoke.py --config scripts/df/sevir/df_dit_smoke.yaml --device cpu
```

结果：数据集可加载，VAE 路径可识别，但 sandbox 环境下 DataLoader 多进程共享文件描述符触发 `PermissionError: Operation not permitted`。这更像当前受限运行环境问题，不是模型 forward 的形状或数值错误。该测试不计入失败模型结论。

## 5. 已有训练与评估结果

### 5.1 训练 checkpoint 盘点

主要有效 checkpoint：

| 模型 | 目录 | checkpoint |
|---|---|---|
| causal base | `20260516-195644_df_dit_cached_latents_full_causal` | epoch 200 |
| noncausal base | `20260516-195905_df_dit_cached_latents_noncausal` | epoch 200 |
| noncausal HG FT | `20260517-164239_df_dit_noncausal_history_free_ft_ep200` | epoch 230 |
| causal HG FT | `20260517-214104_df_dit_causal_history_free_ft_ep200` | epoch 230 |

早期 `df_dit_cached_latents_block_sparse` 有多次输出，其中部分评估很差，后续主要使用 `full_causal` 和 `noncausal` 两条 base。

### 5.2 早期关键评估

| 实验 | ckpt | 推理要点 | CSI | FAR | POD | HSS | MSE | MAE | SSIM | LPIPS |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| block_sparse 早期 run | block_sparse ep200 | eta=1.0 | 0.1405 | 0.3423 | 0.1558 | 0.1813 | 1094.92 | 14.40 | 0.5965 | 0.3093 |
| noncausal base ep200 | noncausal ep200 | synchronized/eta=1.0 | 0.1272 | 0.3071 | 0.1352 | 0.1680 | 1092.00 | 14.75 | 0.5521 | 0.3290 |
| noncausal base ep200 | noncausal ep200 | causal_pyramid, HG scale=1.0 | 0.2224 | 0.5190 | 0.2493 | 0.2919 | 704.69 | 11.68 | 0.5927 | 0.2564 |
| noncausal HG FT ep230 | noncausal ep230 | causal_pyramid, HG scale=1.5 | 0.2346 | 0.5730 | 0.2739 | 0.2989 | 612.96 | 11.23 | 0.6212 | 0.2252 |
| causal base ep200 | causal ep200 | no HG, causal_pyramid | 0.1712 | 0.3470 | 0.1896 | 0.2236 | 934.81 | 13.34 | 0.5900 | 0.3007 |

早期结论：

- 单纯 base 模型不够强，尤其 synchronized 或未 HG 的结果 CSI 偏低；
- causal_pyramid 对 noncausal base 有明显帮助；
- history-free fine-tuning + HG scale=1.5 对 noncausal ep230 有提升；
- causal base ep200 本身不如 noncausal HG FT，但因果结构的潜力需要结合 HG FT 再看。

### 5.3 Noncausal stabilized HG quick4

Checkpoint：

```text
/dataHW/workspace/zhangyuchen/outputs/nowcastdit_df_sevir_cached_latents/20260517-164239_df_dit_noncausal_history_free_ft_ep200/ckpts/checkpoint_epoch_230.pth
```

Output：

```text
/dataHW/workspace/zhangyuchen/outputs/nowcastdit_df_sevir_eval_grid_stabilized_hg_quick4
```

| context | eta | scale | CSI | FAR | POD | HSS | MSE | MAE | SSIM | LPIPS |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| full | 0.0 | 1.0 | 0.2184 | 0.6506 | 0.2813 | 0.2747 | 779.23 | 13.67 | 0.5529 | 0.2416 |
| full | 0.0 | 1.5 | 0.2261 | 0.6497 | 0.2947 | 0.2838 | 753.50 | 13.65 | 0.5432 | 0.2406 |
| latest | 0.0 | 1.0 | 0.2089 | 0.6264 | 0.2509 | 0.2664 | 763.27 | 12.70 | 0.5927 | 0.2385 |
| latest | 0.0 | 1.5 | 0.2224 | 0.6129 | 0.2672 | 0.2832 | 716.90 | 12.34 | 0.5997 | 0.2329 |

结论：

- 在 `eta=0.0` 下，stabilized HG 没有超过早期 noncausal HG 参考结果；
- `latest` context 比 `full` 更稳，FAR/MSE/MAE/SSIM/LPIPS 都更好；
- `full` 会稍微提高 POD，但 false alarm 风险更大。

### 5.4 Causal stabilized HG quick8

Checkpoint：

```text
/dataHW/workspace/zhangyuchen/outputs/nowcastdit_df_sevir_cached_latents/20260517-214104_df_dit_causal_history_free_ft_ep200/ckpts/checkpoint_epoch_230.pth
```

Output：

```text
/dataHW/workspace/zhangyuchen/outputs/nowcastdit_df_sevir_eval_grid_stabilized_hg_causal_ep230_quick8
```

| context | eta | scale | CSI | FAR | POD | HSS | MSE | MAE | SSIM | LPIPS |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| full | 0.0 | 1.0 | 0.1846 | 0.5954 | 0.2107 | 0.2421 | 879.21 | 13.23 | 0.5812 | 0.2538 |
| full | 0.0 | 1.5 | 0.2358 | 0.6498 | 0.3103 | 0.2958 | 742.97 | 13.31 | 0.5617 | 0.2263 |
| full | 1.0 | 1.0 | 0.2077 | 0.4739 | 0.2320 | 0.2683 | 700.72 | 11.53 | 0.6412 | 0.2442 |
| full | 1.0 | 1.5 | 0.2512 | 0.5478 | 0.2952 | 0.3210 | 558.90 | 10.62 | 0.6469 | 0.2069 |
| latest | 0.0 | 1.0 | 0.1618 | 0.5782 | 0.1825 | 0.2134 | 982.34 | 13.89 | 0.5832 | 0.2865 |
| latest | 0.0 | 1.5 | 0.2465 | 0.6675 | 0.3530 | 0.3078 | 806.29 | 13.90 | 0.5734 | 0.2255 |
| latest | 1.0 | 1.0 | 0.1909 | 0.4596 | 0.2118 | 0.2475 | 775.82 | 12.11 | 0.6296 | 0.2712 |
| latest | 1.0 | 1.5 | **0.2659** | 0.5803 | **0.3373** | **0.3349** | 574.46 | 11.18 | 0.6244 | 0.2267 |

结论：

- `eta=1.0` 是 causal checkpoint 的关键。相比 `eta=0.0`，它大幅降低 MSE/MAE，并提升 SSIM。
- `latest + eta=1.0 + scale=1.5` 得到当前最高 CSI/HSS：`CSI=0.2659`, `HSS=0.3349`。
- `full + eta=1.0 + scale=1.5` 是更均衡的设置：`CSI=0.2512`, `FAR=0.5478`, `MSE=558.90`, `SSIM=0.6469`, `LPIPS=0.2069`。
- 如果面试中强调“检测强回波能力”，可以讲 `latest + eta=1.0 + scale=1.5`；如果强调“整体预测质量和虚警控制”，可以讲 `full + eta=1.0 + scale=1.5`。

## 6. 如何解释当前成果

可以把结果概括为三点：

1. **从 full-sequence generation 到 DF-like inference 的迁移是有效的。**  
   早期 synchronized/base 结果 CSI 约 0.12-0.17；引入 causal_pyramid、history-free FT、HG 后，最佳小样本 CSI 到 0.2659。

2. **causal attention 本身不是银弹，但 causal + HG FT + eta=1.0 的组合有价值。**  
   causal base ep200 的 CSI 只有 0.1712；但 causal ep230 HG FT 在 stabilized HG 下达到 0.2659。这说明关键不是“换成 causal mask 就解决误差累积”，而是训练/推理机制要对齐。

3. **HG 的 scale 是检测率/虚警率的旋钮。**  
   scale=1.5 通常提高 CSI/POD，但也会提高 FAR。雷达 nowcasting 里这很合理：更强的 history guidance 促使模型更大胆延续强回波运动，但容易产生 false alarm。

## 7. 面试中可能被问到的问题

### Q1：你的项目到底创新在哪里？

可以回答：

> 我不是从零提出一个新 diffusion model，而是在已有 NowcastDiT 雷达预测框架上，把 full-sequence generation 改造成 DF-like 的时序扩散推理框架。关键包括 per-frame timestep、causal pyramid schedule、block causal attention、history-free fine-tuning，以及借鉴 DFoT 的 history guidance/stabilized history guidance。我的主要贡献是把这些机制落到雷达 latent forecasting 任务上，并通过小规模 ablation 证明 causal HG FT + eta=1.0 的推理配置明显改善 CSI/HSS。

### Q2：为什么 causal base 反而不如 noncausal，但 causal FT 最后更好？

可以回答：

> 单纯 causal attention 会限制模型利用未来帧之间的双向一致性，base 训练时如果推理 schedule 没对齐，就可能变差。history-free FT 后，模型学会在历史缺失/历史被噪声破坏时仍进行预测；再结合 causal pyramid 和 HG，因果结构的优势才体现出来。也就是说 causal mask 需要与训练目标、推理 schedule、guidance 一起看，而不是单独看。

### Q3：为什么 `eta=1.0` 比 `eta=0.0` 好？

可以回答：

> `eta=0.0` 更接近确定性 DDIM，容易过早锁定轨迹；雷达未来存在多模态不确定性，尤其强回波运动区域，完全确定性采样可能会导致偏移。`eta=1.0` 引入随机项，给后续 step 留出修正空间。实验上 causal ep230 在 `eta=1.0` 下 MSE/SSIM/CSI 都明显更好。

### Q4：History Guidance 和 CFG 是什么关系？

可以回答：

> 数学形式和 CFG 类似，都是用 conditional 与 unconditional 两个预测的差值做引导：
>
> `eps_guided = eps_uncond + s * (eps_cond - eps_uncond)`
>
> 区别是 CFG 通常针对 class/text condition；这里的 condition 是 history frames。为了让 unconditional branch 有意义，训练时要做 history-free dropout，让模型学会缺历史时的分布。

### Q5：你现在的 stabilized HG 和 DFoT 完全一致吗？

应该诚实回答：

> 不完全一致。当前实现是推理端近似：对已生成 future context 以低噪声水平扰动，减少模型把自己生成的干净帧当作 ground-truth history 时的分布偏移。DFoT 论文还有更系统的 flexible history training objective，以及 time/frequency history guidance。本项目尚未完整复现 frequency guidance 和训练端可变 context mask。

### Q6：为什么 FAR 仍然偏高？

可以回答：

> 这是雷达 nowcasting 的核心 trade-off。强 guidance 提高 POD/CSI 的同时会让模型更倾向预测强回波延续，从而增加 false alarm。当前最强 CSI 设置 `latest + eta=1.0 + scale=1.5` FAR 是 0.5803；更稳的 `full + eta=1.0 + scale=1.5` FAR 是 0.5478，但 CSI 降到 0.2512。后续需要在业务目标上决定更重视命中率还是虚警控制。

## 8. 尚未充分探索的问题

这些不是立即优化方向，而是后续做 well-studied 研究时应该准备的“问题清单”。

### 8.1 统计可靠性

当前评估样本只有 192 个，且可视化只保存 10 个 case。需要回答：

- 在完整验证集/测试集上，causal best 是否仍优于 noncausal best？
- 不同 storm intensity、不同 lead time 上收益是否一致？
- CSI/HSS 的提升是否主要来自低阈值，还是高阈值强回波也提升？

### 8.2 Schedule 与训练分布对齐

当前训练是随机 per-frame timestep，推理是 structured causal_pyramid。需要进一步思考：

- 训练时是否应显式采样 causal_pyramid-like timestep matrix？
- 当前 `sample_schedule_shift=0.25` 为什么合理？是否有理论或经验最优区间？
- schedule 中 `-1` 完成态与 generated context 的语义是否和训练时的 clean history 对齐？

### 8.3 Attention 因果性

`block_causal, block_size=5` 是块因果，不是逐帧因果。需要准备解释：

- 块内 noncausal 是否会造成未来泄漏？
- 由于推理目标本身是一次处理 20 帧 latent，这种块内双向是否可以理解为短窗口内联合修正？
- 是否需要比较 `block_size=1/2/5/10`？

### 8.4 Stabilized HG 的严格性

当前 stabilized HG 尚未实现 DFoT 的 frequency guidance。需要进一步研究：

- 雷达回波的频域分解是否有物理意义？
- 对高频强回波边界和低频大尺度平流分别引导，是否能降低 FAR？
- 频域 HG 是否会破坏 VIL 值域或强度校准？

### 8.5 损失与指标错配

训练 loss 是 latent noise MSE，评估主要是 CSI/FAR/POD/HSS + pixel/perceptual metrics。需要准备：

- 为什么 latent MSE 不必然优化 CSI？
- 是否需要 threshold-aware loss 或 high-reflectivity weighting？
- 这会不会偏离“Diffusion Forcing 主线”？如果作为后续工作，应明确它是 task-specific calibration，不是核心 DF 机制。

### 8.6 与已有 nowcasting 模型的关系

需要准备对比口径：

- ConvLSTM/PredRNN/SimVP/Earthformer 等通常是 deterministic 或 latent deterministic sequence forecasting；
- 本项目强调 diffusion latent 生成和 per-frame noise schedule；
- 目前没有做严格 SOTA 对比，不能宣称超过成熟 nowcasting baselines，只能说在本项目内部 ablation 中机制有效。

## 9. 参考文献与代码

- SEVIR: [SEVIR: A Storm Event Imagery Dataset for Deep Learning Applications in Radar and Satellite Meteorology](https://papers.nips.cc/paper_files/paper/2020/hash/fa78a16157fed00d7a80515818432169-Abstract.html)
- DDPM: [Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239)
- DDIM: [Denoising Diffusion Implicit Models](https://arxiv.org/abs/2010.02502)
- CFG: [Classifier-Free Diffusion Guidance](https://arxiv.org/abs/2207.12598)
- DiT: [Scalable Diffusion Models with Transformers](https://arxiv.org/abs/2212.09748)
- MM-DiT / Rectified Flow Transformer: [Scaling Rectified Flow Transformers for High-Resolution Image Synthesis](https://arxiv.org/abs/2403.03206)
- Diffusion Forcing: [Diffusion Forcing: Next-token Prediction Meets Full-Sequence Diffusion](https://arxiv.org/abs/2407.01392)
- DFoT / History Guidance: [History-Guided Video Diffusion](https://arxiv.org/abs/2502.06764)
- DFoT official code: [kwsong0113/diffusion-forcing-transformer](https://github.com/kwsong0113/diffusion-forcing-transformer)

## 10. 当前最推荐的讲述版本

一句话版本：

> 我把 NowcastDiT 的 full-sequence latent diffusion 推理改造成 DF-like 的 per-frame-noise 生成框架，引入 causal pyramid schedule、block causal attention、history-free fine-tuning 和 stabilized History Guidance；在 SEVIR 小规模评估上，causal HG fine-tuned checkpoint 配合 `eta=1.0` 与 `scale=1.5`，将最佳 CSI 提升到 0.2659，同时 `full context` 设置在 MSE/SSIM/FAR 上更稳。

三句话版本：

> 这个项目的核心是研究雷达预测中的训练-推理对齐：模型训练时学习任意帧噪声水平的 denoising，推理时用 causal pyramid schedule 让近未来先去噪、远未来后去噪，从而缓解 full-sequence generation 与 autoregressive rollout 的冲突。  
> History Guidance 则用 conditional/unconditional 两个历史条件分支做差值引导，history-free fine-tuning 保证无历史分支有意义，stabilized HG 进一步降低 generated context 作为历史时的分布偏移。  
> 当前最强小样本结果来自 causal epoch230 + latest context + eta=1.0 + scale=1.5，CSI/HSS 最好；而 full context + eta=1.0 + scale=1.5 的 MSE、SSIM、LPIPS、FAR 更稳，是更适合展示视觉质量与虚警控制的设置。




---
## 原因 1：attention mask 结构不同

causal attention 限制 future token 的可见范围。  
因此模型更容易依赖：

- clean history；
- 当前 block；
- 已经较可靠的上下文。

noncausal attention 没有这个限制。  
因此它可以让不同 future token 相互通信，即使这些 token 仍然带噪。

这直接导致：

- causal clean key mass 更高；
- noncausal expected key noise 更高。

---

## 原因 2：DF 推理调度制造了 mixed-noise context

causal pyramid schedule 使同一个 sampling step 中，不同 future frames 处于不同噪声等级。

所以 attention 面临的不是普通视频模型里的“同质 token 序列”，而是：

> clean history + low-noise future + mid-noise future + high-noise future

noncausal 模型更充分地利用这个 mixed-noise context。  
causal 模型则更倾向于把 clean history 作为稳定锚点。

---

## 原因 3：雷达预测中的 FAR/POD 张力

noncausal 看 noisy future 更多，可能帮助它生成更丰富的未来结构，因此 POD 更高。  
但 noisy future 本身不稳定，也可能让模型“过度相信”不确定结构，因此 FAR 更高。

causal 看 clean history 更多，更保守，因此 FAR 较低。  
但它对强移动、强变化区域可能响应不足，因此 POD 较低。

这和你前面评估结果一致：

- noncausal：CSI/POD 更好，但 FAR 更高；
- causal：FAR 更低，但召回更弱。

---

# 8. 这一页的核心结论

你可以在 PPT 底部放这三句话：

> Causal attention behaves as history-anchored denoising.  
> Noncausal attention behaves as noisy-future collaborative denoising.  
> This explains the POD/FAR trade-off observed in evaluation.

中文版本：

> causal 更像“历史锚定式去噪”，稳定但保守；  
> noncausal 更像“noisy future 协同去噪”，召回更强但更易虚警；  
> 这为评估中 POD/FAR 的差异提供了机制解释。