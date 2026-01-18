# Python 3.12
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.12
sudo apt install python3.12-venv
```

# Nvidia drivers
```bash
nvtop
sudo apt install nvidia-driver-575
sudo reboot
nvidia-smi
```

# Cuda toolkit
```bash
nvcc --version 
sudo apt install nvidia-cuda-toolkit
```

# Get llama.cpp
```bash
git clone https://github.com/ggml-org/llama.cpp.git
```

# Env
```bash
cd llama.cpp
python3.12 -m venv .llamacpp_env
source .llamacpp_env/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install "datasets>=4.1" "transformers>=4.56" pandas openpyxl
```

# Build llama.cpp
```bash
cd llama.cpp

sudo apt install cmake
sudo apt update
sudo apt install build-essential g++
sudo apt install libcurl4-openssl-dev
sudo apt install gcc-10 g++-10
export CC=/usr/bin/gcc-10
export CXX=/usr/bin/g++-10
cmake -B build -DCMAKE_CXX_STANDARD=17 -DCMAKE_CUDA_STANDARD=17 -DLLAMA_CURL=OFF -DLLAMA_BUILD_TESTS=OFF -DCMAKE_CUDA_ARCHITECTURES=75 -DGGML_CUDA=ON -DCMAKE_CUDA_HOST_COMPILER=/usr/bin/gcc-10
cmake --build build --config Release
```

# Get model from HF
```bash
wget -O models/LFM2.5-1.2B-Instruct-Q8_0.gguf https://huggingface.co/LiquidAI/LFM2.5-1.2B-Instruct-GGUF/resolve/main/LFM2.5-1.2B-Instruct-Q8_0.gguf?download=true
```

# Run llama.cpp server
```bash
cd build/bin

./llama-server -ngl 99 -m ../../models/LFM2.5-1.2B-Instruct-Q8_0.gguf -cmoe --jinja --host 0.0.0.0 --port 8080 -b 512 -ub 256 --ctx-size 32768 -t 6 --cache-ram 0 --top-k 1 --top-p 0.9 --min-p 0.05 --temp 0.01 --reasoning-budget 0 -ctk q8_0 -ctv q8_0 --repeat-penalty 0.9 --api-key "test-key"
```
