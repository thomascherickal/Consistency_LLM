# Consistency Large Language Models: A Family of Efficient Parallel Decoders
<p align="center">
| <a href="http://arxiv.org/abs/2403.00835"><b>Paper</b></a> | <a href="https://hao-ai-lab.github.io/blogs/cllm/"><b>Blog</b></a> |
</p>

<p align="center">
  <a href="https://opensource.org/licenses/Apache-2.0">
    <img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License">
  </a>
  <a href="https://github.com/hao-ai-lab/Consistency_LLM/issues">
    <img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" alt="Maintenance">
  </a>
  <a href="https://github.com/hao-ai-lab/Consistency_LLM/pulls">
    <img src="https://img.shields.io/badge/Contributions-welcome-brightgreen.svg?style=flat" alt="Contributions welcome">
  </a>
</p>

## Contents
- [Introduction](#introduction)
- [Installation](#installation)
- [Model Weights](#model-weights)
- [Usage](#usage)
  - [Inference](#inference)
  - [Training](#training)
  - [Evaluation](#evaluation)
- [References](#references)
- [Acknowledgements](#acknowledgements)

## Introduction
Consistency Large Language Models (CLLMs) is a family of efficient parallel decoders refined from target LLMs.
Show demo here.

Compared with existing acceleration techniques, CLLMs achieve fast parallel decoding without any:
- Draft models
- Additional architectural components

This implies numerous advantages of CLLMs:
- CLLMs eliminate the complexity of training 'good' draft models and managing two different models in a single system.
- CLLMs share the same architecture with target LLMs which simplifies training and eliminates the need to design additional architecture for specific LLMs.
- CLLMs can be integrated seamlessly with other techniques for efficient LLM inference (e.g. Lookahead Decoding) and achieve greater speedup.

Empirical results have shown the effectiveness of CLLMs.
<p align="center">
  <picture>
  <img src="assets/img/cllm_speedup.png" width="45%">
  </picture>
</p>

## Installation
1. Environment setup:
```
conda create -n cllm python=3.9
conda activate cllm
```
2. Clone this repository and build from source:
```
git clone git@github.com:snyhlxde1/Consistency_LLM.git
cd Consistency_LLM
```
3. Install dependency:
```
pip install -r requirements.txt
```
## Model Weights
#### Target Model
| Size | Dataset |  Hugging Face Repo                             |
| ---- | -------- | --------------------------------------------- | 
| 7B   | ShareGPT |  [cllm/vicuna-7b-sharegpt-gpt4-48k](https://huggingface.co/cllm/vicuna-7b-sharegpt-gpt4-48k)   |
| 7B  | GSM8K | [GAIR/Abel-7B-001](https://huggingface.co/GAIR/Abel-7B-001) |
| 7B  | Spider | [cllm/deepseek-7b-instruct-spider](https://huggingface.co/cllm/deepseekcoder-7b-instruct-spider) |
| 7B  | Code-Search-Net Python | [cllm/deepseekcoder_7b_codesearch_net_python](https://huggingface.co/cllm/deepseekcoder_7b_codesearch_net_python) |
#### CLLM
| Size | Dataset |  Hugging Face Repo                             |
| ---- | -------- | --------------------------------------------- | 
| 7B   | ShareGPT |  [cllm/consistency-llm-7b-sharegpt48k](https://huggingface.co/cllm/consistency-llm-7b-sharegpt48k)   |
| 7B  | GSM8K | [cllm/consistency-llm-7b-gsm8k](https://huggingface.co/cllm/consistency-llm-7b-gsm8k) |
| 7B  | Spider | [cllm/consistency-llm-7b-spider](https://huggingface.co/cllm/consistency-llm-7b-spider) |
| 7B  | Code-Search-Net Python | [cllm/consistency-llm-7b-codesearchnet](https://huggingface.co/cllm/consistency-llm-7b-codesearchnet) |
## Usage
### Inference 
```
bash applications/run_chat_cllm.sh {model_path} {cllm_type}
```
`cllm_type` can take the value of `spider`, `python`, `gsm8k`, `sharegpt`.

### Training
1. Collect Jacobi trajectory
- Method 1: Directly download Jacobi trajectory in hugging face to `data/collected_jacobi_trajectory/` from [our Huggingface Hub page](https://huggingface.co/cllm).
- Method 2 (Generate trajectory suitable to your own target model and dataset): Download raw dataset (e.g. [ShareGPT](https://huggingface.co/datasets/cllm/sharegpt_20230521_2k_clean_lang_split_identity_gpt4), [Spider](https://huggingface.co/datasets/cllm/spider)) to `data/raw_data`. Then run `scripts/generate_trajectory.sh` and the training dataset for a CLLM will be saved in  `data/collected_jacobi_trajectory/`.

For example, for the gsm8k dataset, run:
```
# max_new_tokens corresponds to the size of n_token_sequence
CUDA_VISIBLE_DEVICES=0 bash scripts/generate_trajectory.sh {filename} {model_path} {max_new_tokens} {max_new_seq_len}
```

2. Train a CLLM:
```
bash scripts/generate_trajectory.sh {model_path} {trajectory_file} {output_path} {n_token_seq_size}
```

### Evaluation
We follow the same settings in [human-eval](https://github.com/openai/human-eval), [Spider](https://github.com/taoyds/spider), [MT-bench](https://github.com/lm-sys/FastChat/tree/main/fastchat/llm_judge) and [GSM8K](https://github.com/openai/grade-school-math) evaluate CLLMs' generation quality. An example code to evaluate CLLMs' throughput measured in tokens/s, fast-forwarded token count, stationary token count can be found in `eval` folder. Take GSM8K dataset as the example, run:
```
CUDA_VISIBLE_DEVICES=0 bash eval/gsm8k/speedup.sh {model_path} {target_model_path} {max_new_tokens}
```
To test accuracy:
```
cd eval/gsm8k
CUDA_VISIBLE_DEVICES=0 acc.py --model_dir {cllm_model_path} --max_new_tokens_for_consistency 16 --temperature 0.0 --top_p 1.0 \
--output_file_name 'cllm_generated_gsm8k.jsonl' --dev_set "gsm8k" --prompt_type math-single --max_tokens 1024 --use_consistency_decoding
```
## References

## Acknowledgements
