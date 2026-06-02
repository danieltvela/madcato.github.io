---
layout:     post
title:      "What I Learned Trying to Convert and Quantize a Model to GGUF"
subtitle:   "How is the process?"
date:       2024-09-26 06:00:00
author:     "Daniel Vela"
locale:     en
lang-ref:   gguf-to-ollama
---

Recently, I attempted to convert an NVIDIA-based model to a format compatible with Ollama. This post outlines the steps I took, the issues I encountered, and the necessary next steps to complete the conversion and import process.

I’m sharing this information in case it might be helpful to others facing a similar challenge.

# The Original Model

The model in question is **Nemotron-51B**, developed by NVIDIA and based on LLaMA 3.1. It’s available on the Hugging Face model repository in the safetensors format.

# How to Import a Model into Ollama

To import a model into Ollama, the first step is to convert the safetensors format into GGUF. For this, I tried using the llama.cpp tool. However, I ran into a problem: this tool is only set up to convert specific models. The NVIDIA model, DeciLM, hasn’t yet been adapted for use with llama.cpp as of September 2024.

To check which models are supported by llama.cpp, you need to inspect the _convert_hf_to_gguf.py_ file in the project repository [here](https://github.com/ggerganov/llama.cpp/blob/master/convert_hf_to_gguf.py).

Below, I’ve shared the scripts I used, as well as the readme.md detailing my attempt to convert and import the model into Ollama.

## Scripts

### README.md

```markdown
# Prepare model Nvidia Nemotron-51B for ollama

IT DOESN’T WORK BECAUSE I CAN’T FIND A SCRIPT THAT SETS THE GGUF PARAMETERS TO CONVERT A **DeciLMForCausalLM** MODEL.

TO CHECK WHICH MODELS CAN BE CONVERTED BY LLAMA.CPP, LOOK AT THE convert_hf_to_gguf.py SCRIPT IN THE LLAMA.CPP FOLDER.

- [ollama: importing a model](https://github.com/ollama/ollama/blob/main/docs/import.md)
- [llama.cpp](https://github.com/ggerganov/llama.cpp)
- [Install ollama](https://ollama.com/download)
- [nvidia/Llama-3_1-Nemotron-51B-Instruct](https://huggingface.co/nvidia/Llama-3_1-Nemotron-51B-Instruct)

## Run

### 0. Install dependencies

- transformers python module
- llama.cpp
- ollama
- torch

    ```sh
    pip3 install transformers
    ```

    ```sh
    git clone https://github.com/ggerganov/llama.cpp
    cd llama.cpp
    make
    pip3 install -r requirements.txt
    ```

    ```sh
    curl -fsSL https://ollama.com/install.sh | sh
    ```

To install torch, follow the instructions on this [page](https://pytorch.org/get-started/locally/).

### 1. Download model

    ```sh
    mkdir model
    ```

    ```python
    from transformers import AutoModelForCausalLM, AutoTokenizer
    import torch

    model_name = "nvidia/Llama-3_1-Nemotron-51B-Instruct"
    model_kwargs = {"torch_dtype": torch.bfloat16, "trust_remote_code": True, "device_map": "auto"}

    tokenizer = AutoTokenizer.from_pretrained(model_name)
    tokenizer.pad_token_id = tokenizer.eos_token_id
    model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype="auto", trust_remote_code=True)

    # Guardar el modelo en una carpeta local
    tokenizer.save_pretrained("./model")
    model.save_pretrained("./model")
    ```

### 2. Convert model to GGUF with llama.cpp

Run this command to convert a **Safetensors** model to **GGUF**. This requires all model files from Hugging Face.

    ```bash
    python3 ../llama.cpp/convert_hf_to_gguf.py model 
    ```

### 3. Create ollama quantizing model to q4_K_M

    ```sh
    ollama create --quantize q4_K_M nvidia-nemotron-51b
    ```

### 4. Test the model:

    ```sh
    ollama run nvidia-nemotron-51b
    ```

### 5. Upload model ollama

First create an ollama account by signing up at [ollama.com](https://ollama.com/signup).

Add your ollama public key in [ollama account settings](https://ollama.com/settings/keys). Folllow the instructions on this page.

    ```sh
    ollama cp mymodel danieltvela/nvidia-nemotron-51b
    ollama push danieltvela/nvidia-nemotron-51b
    ```

### 6. Test it

    ```sh
    ollama run danieltvela/nvidia-nemotron-51b
    ```
```

### download.py

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

model_name = "nvidia/Llama-3_1-Nemotron-51B-Instruct"
model_kwargs = {"torch_dtype": torch.bfloat16, "trust_remote_code": True, "device_map": "auto"}

tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token_id = tokenizer.eos_token_id
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype="auto", trust_remote_code=True)

# Guardar el modelo en una carpeta local
tokenizer.save_pretrained("./model")
model.save_pretrained("./model")
```

### Modelfile

```
FROM model
```