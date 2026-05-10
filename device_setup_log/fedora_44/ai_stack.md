## Initial Setup

https://youtu.be/dgyqBUD71lg?t=698

```

sudo dnf install rocm

rocm-smi

rocminfo

sudo dnf install cargo rust

cargo --version

cargo install amdgpu_top

sudo dnf install libdrm-devel

CARGO_TARGET_DIR=/tmp/cargo-installsSdsbW cargo install amdgpu_top

echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

amdgpu_top
```

## Tuning

https://github.com/kyuz0/amd-r9700-vllm-toolboxes/blob/main/TUNING.md
## Toolboxes

https://youtu.be/dgyqBUD71lg?t=1376

