
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