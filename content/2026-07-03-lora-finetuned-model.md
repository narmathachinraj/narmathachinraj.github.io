Title: LoRA Fine-Tuned Models
Date: 2026-07-03
Category: GenAI
Tags: GenAI, LLM, LoRA, fine-tuning, PEFT, training
Slug: lora-finetuned-model

Fine-tuning a large language model from scratch is expensive. LoRA changes that by training only a tiny fraction of the model's parameters — and the results are surprisingly close to full fine-tuning at a fraction of the cost.

## What is LoRA?

**Low-Rank Adaptation (LoRA)** — A parameter-efficient fine-tuning technique that freezes the original pretrained model weights and injects small trainable matrices into specific layers. Instead of updating billions of parameters, you only train these small adapter matrices.

**Why low-rank?** — Any weight update matrix can be approximated as the product of two smaller matrices (A and B), where their inner dimension (rank r) is much smaller than the original. LoRA exploits this — a rank-8 update to a 4096×4096 weight matrix means training 65,536 parameters instead of 16 million.

## How It Works

**Frozen base model** — The pretrained weights stay completely unchanged during fine-tuning. Only the injected LoRA matrices learn from your task-specific data.

**Rank decomposition** — For a weight matrix W, LoRA learns two matrices: A (shape d × r) and B (shape r × k), where r is the rank hyperparameter you choose. The effective weight update is B × A, which is added to the frozen W at inference time.

**Merge at inference** — Once training is done, the LoRA weights can be merged back into the base model weights (W + BA) with zero added latency. You get a standard model that behaves identically to a fully fine-tuned one.

## Key Hyperparameters

**Rank (r)** — Controls the expressiveness of the adapter. Lower rank (4–8) means fewer parameters and faster training. Higher rank (32–64) captures more complex adaptations but costs more memory. Most tasks work well with r=8 or r=16.

**Alpha (α)** — A scaling factor applied to the LoRA update. The effective scaling is α/r. Keeping α = 2r is a common default that stabilizes training without extensive tuning.

**Target modules** — LoRA is typically applied to the query and value projection matrices in attention layers. You can extend it to all linear layers for stronger adaptation, at the cost of more trainable parameters.

## LoRA vs Full Fine-Tuning

**Trainable parameters** — Full fine-tuning updates every weight in the model. LoRA typically trains less than 1% of the total parameters while achieving comparable downstream performance.

**Memory footprint** — Because most weights are frozen, optimizer states (momentum, variance in Adam) are only maintained for the tiny LoRA matrices — dramatically reducing GPU memory requirements.

**Portability** — LoRA adapters are small files (often just a few MB). You can share, swap, or stack multiple adapters on top of the same base model without duplicating the full model weights each time.

## QLoRA — Taking It Further

**Quantized LoRA** — Combines LoRA with 4-bit quantization (double quantization) of the base model weights. The base model is loaded in 4-bit precision, and only the LoRA adapters are trained in full precision. This enables fine-tuning of 70B+ parameter models on a single consumer GPU with minimal quality loss.

> LoRA has become the default approach for custom LLM fine-tuning. It's fast, cheap, and the adapter files are tiny enough to version-control alongside your code.
