# Install Flash Attention 2

"Read Terminal" failed
Completed with input: {
  "shellId": "7",
  "delay": 5
}
Tool execution failed: shellId: 7 not found or was never created. Please supply a valid shell ID.

<no active shell sessions>

Ran terminal command: /home/malneyugnfl/anaconda3/envs/huggingface/bin/python -m pip show flash-attn
Tool execution failed with exit code 1

Yes. In a terminal, run these commands to install FlashAttention from the official source into `huggingface`:

```bash
conda activate huggingface

# Confirm PyTorch can see the Thor GPU
python -c "import torch; print(torch.__version__, torch.version.cuda); print(torch.cuda.get_device_name(0), torch.cuda.get_device_capability(0))"

# Install build prerequisites
python -m pip install ninja packaging

# Build for Thor (SM 11.0); limit parallel jobs to reduce memory use
MAX_JOBS=6 FLASH_ATTN_CUDA_ARCHS=110 \
  python -m pip install --no-build-isolation \
  "git+https://github.com/Dao-AILab/flash-attention.git@main"
```

Then verify the install:

```bash
python -c "import flash_attn, torch; print('FlashAttention:', flash_attn.__version__); print('GPU:', torch.cuda.get_device_name(0))"
```

Your environment has Python 3.11, PyTorch with CUDA 13.2, and an NVIDIA Thor GPU, so this uses a source build rather than relying on a prebuilt wheel. I checked afterward: FlashAttention is **not installed yet**. If the build fails, share the final error output and I can help troubleshoot it.