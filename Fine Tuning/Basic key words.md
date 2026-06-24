**What it is**: Post-training customization. You take a pre-trained model and adapt it to your specific task, data, or style.

---

### **Two Main Categories (by parameter usage)**

### 1. **Full Fine-Tuning**

- **What**: Update *every* weight in the model.
- **Pros**: Maximum flexibility; model can fully adapt to new domain.
- **Cons**: Needs massive VRAM (multi-GPU), large datasets, high risk of catastrophic forgetting (model forgets original knowledge).
- **Use when**:
    - You have 10k+ high-quality, task-specific examples.
    - You control a multi-GPU cluster.
    - Task is very different from pre-training (e.g., medical diagnosis on a general LLM).
- **Example**: Fine-tuning BERT on a legal corpus for contract clause extraction.

### 2. **Parameter-Efficient Fine-Tuning (PEFT)**

- **What**: Freeze the base model. Train only small, added components (adapters). 99%+ of weights stay frozen.
- **Pros**: Runs on single GPU (even Colab), faster training, less forgetting, easy to swap adapters for different tasks.
- **Cons**: Slightly lower ceiling than full FT for extremely complex domain shifts.
- **Use when**:
    - You have <1k examples.
    - You're on a single GPU or limited budget.
    - You want to iterate fast or maintain multiple task-specific versions.
- **Subtypes**: LoRA, QLoRA, Prefix Tuning, Adapters. **LoRA/QLoRA are the current standard.**

---

### **Common Fine-Tuning Methods (Task/Goal-Based)**

### 🔹 **SFT (Supervised Fine-Tuning)**

- **How**: Train on input-output pairs (e.g., prompt → ideal response). Model learns to mimic the pattern.
- **Works with**: Full FT, LoRA, QLoRA.
- **Example**:
    - *Input*: "Explain quantum entanglement to a 10-year-old"
    - *Output*: "Imagine two magic dice that always show the same number, no matter how far apart they are…"
- **Use case**: Teaching your NPC AI to respond in a specific character voice or game lore style.

### 🔹 **DPO / ORPO (Preference Optimization)**

- **How**: Show the model two responses to the same prompt—one preferred, one not. Model learns to favor the style/quality you want. No separate reward model needed (unlike older RLHF).
- **DPO**: Directly optimizes from human preference pairs.
- **ORPO**: Combines SFT + preference in one stage; more efficient.
- **Example**:
    - Prompt: "How do I defeat the boss?"
    - Response A (good): "Try dodging left, then strike when his shield drops."
    - Response B (bad): "Just keep attacking."
    - Model learns to generate helpful, contextual hints.
- **Use case**: Aligning NPC dialogue to be helpful, immersive, or on-brand without manual rule-writing.

### 🔹 **Distillation**

- **How**: Use a large, powerful model (teacher) to generate high-quality input-output pairs. Train a smaller model (student) on that data.
- **Goal**: Compress capability into a faster, cheaper model.
- **Example**: Use Llama-3-70B to generate 10k quest-dialogue pairs, then fine-tune a 1B model to run locally in your game engine.
- **Use case**: Deploying smart NPCs on edge devices or low-end hardware.

### 🔹 **Reinforcement Learning Fine-Tuning (RLHF / RLAIF)**

- **How**: Model generates output → gets a reward score (from human or AI) → updates policy to maximize reward.
- **GSPO/GRPO**
- **Example**: NPC gives a hint → player succeeds → positive reward. Player gets frustrated → negative reward. Model learns adaptive hinting.
- **Use case**: NPCs that learn from player behavior over time (your emergent AI goal).

---

### **LoRA & QLoRA: Deep Dive (The PEFT Workhorses)**

### 🔸 **LoRA (Low-Rank Adaptation)**

- **Core idea**: Instead of updating huge weight matrices (e.g., 4096×4096), inject small *trainable* matrices (rank r, e.g., 8 or 64) alongside them.
- **Math simplified**:
    - Original: `W` (frozen)
    - LoRA adds: `W + ΔW`, where `ΔW = A × B` (A and B are small, trainable)
- **Why it works**: Most adaptation lives in a low-dimensional subspace. You don't need to tweak every parameter.
- **VRAM impact**: ~70% less than full FT. A 7B model fits on a 24GB GPU.
- **When to use**: Default choice for most fine-tuning tasks. Fast, stable, reversible.

### 🔸 **QLoRA (Quantized LoRA)**

- **Core idea**: LoRA + 4-bit quantization of the base model.
- **How**:
    1. Quantize base model weights to 4-bit (NF4 format) → shrinks model size ~4x.
    2. Apply LoRA adapters on top.
    3. Use paged optimizers to handle memory spikes.
- **VRAM impact**: A 7B model runs on ~12GB VRAM. 70B model fits on a single 48GB GPU.
- **Trade-off**: Tiny accuracy drop (<1%) for massive efficiency gain.
- **When to use**:
    - You're on Colab, a laptop, or a single consumer GPU.
    - You want to experiment with large models (e.g., Llama-3-70B) without cloud costs.
    - Rapid prototyping for your NPC AI project.

**Practical tip**: Start with QLoRA + SFT

---

### **Quick Decision Guide**

| Your Situation | Recommended Method |
| --- | --- |
| Single GPU, <1k examples | **QLoRA + SFT** |
| Want NPC to learn from player feedback | **QLoRA + DPO/ORPO** |
| Deploying on low-end hardware | **Distillation → QLoRA** |
| Massive domain shift + big compute | **Full FT + SFT** |
| Iterating on multiple NPC personalities | **LoRA adapters (swap per character)** |

---

### **One Thing to Watch**

PEFT methods (LoRA/QLoRA) are *additive*. They don't rewrite the base model. they layer behavior on top. That's a feature: you can keep the model's general knowledge and just inject your game-specific logic. But if your task *fundamentally* conflicts with pre-training (e.g., reversing logic), you may still need full FT.

---