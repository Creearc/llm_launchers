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
```

# Get DCMAKE_CUDA_ARCHITECTURES
```bash
echo -e '#include <stdio.h>\n#include <cuda_runtime.h>\nint main() { int dev; cudaDeviceProp prop; cudaGetDeviceProperties(&prop, 0); printf("SM_%d%d", prop.major, prop.minor); return 0; }' > temp.cu && nvcc temp.cu -o temp 2>/dev/null && ./temp 2>/dev/null && rm -f temp.cu temp
```

# Build template
```bash
cmake -B build -DCMAKE_CXX_STANDARD=17 -DCMAKE_CUDA_STANDARD=17 -DLLAMA_CURL=OFF -DLLAMA_BUILD_TESTS=OFF -DCMAKE_CUDA_ARCHITECTURES={YOUR_VALUE_HERE} -DGGML_CUDA=ON -DCMAKE_CUDA_HOST_COMPILER=/usr/bin/gcc-10  -DGGML_RPC=ON
cmake --build build --config Release -j 4
```

# NVIDIA GeForce GTX 2060 6GB example
```bash
cmake -B build -DCMAKE_CXX_STANDARD=17 -DCMAKE_CUDA_STANDARD=17 -DLLAMA_CURL=OFF -DLLAMA_BUILD_TESTS=OFF -DCMAKE_CUDA_ARCHITECTURES=75 -DGGML_CUDA=ON -DCMAKE_CUDA_HOST_COMPILER=/usr/bin/gcc-10
cmake --build build --config Release
```

# Get model from HF
```bash
wget -O models/qwen2.5-coder-0.5b-instruct-q8_0.gguf https://huggingface.co/Qwen/Qwen2.5-Coder-0.5B-Instruct-GGUF/resolve/main/qwen2.5-coder-0.5b-instruct-q8_0.gguf?download=true
```

# Run llama.cpp server
```bash
cd build/bin

./llama-server -m "../../models/qwen2.5-coder-0.5b-instruct-q8_0.gguf" --n-gpu-layers 8 --jinja --host 0.0.0.0 --host 0.0.0.0 --port 8080 -b 512 -ub 256 --ctx-size 32768 -t 6 --cache-ram 0 --top-k 1 --top-p 0.9 --min-p 0.05 --temp 0.01 --reasoning-budget 0 --repeat-penalty 0.9 --api-key "test-key"
```

| Параметр           | Значение       | Описание                | Зачем это нужно                                                                                         |
| ------------------ | -------------- | ----------------------- | ------------------------------------------------------------------------------------------------------- |
| `-m`               | `~/models/...` | **Путь к файлу модели** | Указывает серверу, какой `.gguf` файл загружать.                                                        |
| `--n-gpu-layers`   | `35`           | **Слои на GPU**         | Сколько слоёв нейросети перенести в видеопамять. Чем больше, тем быстрее.                               |
| `-t`               | `6`            | **Потоки CPU**          | Количество потоков процессора для обработки остатка модели (если не влезла в GPU).                      |
| `--ctx-size`       | `8192`         | **Размер контекста**    | Максимальная длина диалога (в токенах). 8192 ≈ 6000–8000 слов.                                          |
| `--cache-ram`      | `0`            | **Кэш в RAM**           | Лимит оперативной памяти для кэша. `0` = не использовать RAM для кэша (экономия RAM, нагрузка на VRAM). |
| `-b`               | `256`          | **Размер батча**        | Сколько токенов обрабатывать за один проход при чтении промпта (влияет на скорость загрузки контекста). |
| `-ub`              | `128`          | **Ю-батч (Ubatch)**     | Физический размер блока памяти. Меньшее значение снижает пиковое потребление VRAM.                      |
| `--jinja`          | *(флаг)*       | **Шаблонизатор**        | Включает движок Jinja2 для правильного форматирования чата (критично для Qwen).                         |
| `--host`           | `0.0.0.0`      | **IP адрес**            | `0.0.0.0` = слушать все сетевые интерфейсы (доступно из локальной сети).                                |
| `--port`           | `8080`         | **Порт**                | Порт для доступа к веб-интерфейсу и API.                                                                |
| `--temp`           | `0.3`          | **Температура**         | Креативность. `0.3` = сбалансированный ответ (не слишком сухой, не случайный).                          |
| `--top-k`          | `40`           | **Top-K**               | Выбирает из 40 наиболее вероятных токенов. Снижает шанс абсурдных слов.                                 |
| `--top-p`          | `0.9`          | **Top-P**               | Отсекает токены, чья совокупная вероятность < 90%.                                                      |
| `--min-p`          | `0.05`         | **Min-P**               | Отсекает токены с вероятностью < 5% от лучшего токена. Улучшает качество.                               |
| `--repeat-penalty` | `1.1`          | **Штраф повторов**      | Наказывает модель за повторение фраз. `1.1` = мягкий штраф (чтобы не ломать текст).                     |
| `--api-key`        | `"test-key"`   | **Ключ API**            | Защищает сервер от посторонних. Требует передачи ключа в заголовке запроса.                             |

## для RTX 4060 Ti

```bash
./llama-server \
  -m ~/models/bartowski_Qwen_Qwen3.5-9B-q4_1.gguf \
  --n-gpu-layers 45 \
  --jinja \
  --host 0.0.0.0 \
  --port 8080 \
  -b 512 \
  -ub 256 \
  --ctx-size 16384 \
  -t 6 \
  --cache-ram 0 \
  --top-k 40 \
  --top-p 0.9 \
  --min-p 0.05 \
  --temp 0.3 \
  --repeat-penalty 1.1 \
  --api-key "test-key" \
  --reasoning-budget 0
```

## 💡 Ключевые моменты для RTX 4060 Ti

1. **`--n-gpu-layers 35`**:
    - Для модели **9B** это примерно **90% слоёв**.
    - Оставшиеся слои будут на CPU, что нормально для карты с **8 ГБ VRAM**.
    - Если у вас версия **16 ГБ**, можно увеличить до `45` или `99`.
2. **`-b 256` и `-ub 128`**:
    - Эти значения подобраны так, чтобы не переполнить видеопамять при обработке длинных запросов.
    - Если получаете ошибку **OOM (Out Of Memory)** → уменьшите `-ub` до `64`.
3. **`--jinja`**:
    - **Обязательно** для моделей семейства Qwen. Без этого параметра модель может игнорировать систему или плохо понимать структуру диалога.
4. **`--ctx-size 8192`**:
    - Золотая середина. Если увеличить до `32768`, на 8 ГБ карте может закончиться память при длинной переписке.
5. **`--repeat-penalty 1.1`**:
    - Исправленное значение (в предыдущем примере было `0.9`, что ошибочно). Значение **> 1.0** заставляет модель избегать тавтологии.


# RPC
## Host
### Soft
```bash
./llama-server -m "../../models/Qwen3.5-27B-Q4_K_M.gguf" --jinja --host 0.0.0.0 --port 8080 -b 512 -ub 256 --ctx-size 32768 -t 4 --top-k 1 --top-p 0.9 --min-p 0.05 --temp 0.01 --reasoning-budget 0 --repeat-penalty 0.9 --api-key "test-key" --rpc 192.168.0.245:50052,192.168.0.116:50052 --n-gpu-layers 30 --n-gpu-layers-draft 30 --no-kv-offload --flash-attn 1
```

### Max
```bash
./llama-server -m "../../models/Qwen3.5-27B-Q4_K_M.gguf" --jinja --host 0.0.0.0 --port 8080 -b 512 -ub 256 --ctx-size 24576 -t 2 --top-k 1 --top-p 0.9 --min-p 0.05 --temp 0.01 --reasoning-budget 0 --repeat-penalty 0.9 --api-key "test-key" --rpc 192.168.0.245:50052,192.168.0.116:50052 --n-gpu-layers 41 --n-gpu-layers-draft 41 --no-kv-offload --flash-attn 1 --split-mode row
```

## Client
```bash
CUDA_VISIBLE_DEVICES=0 ./rpc-server --host 0.0.0.0 --port 50052 -c
```
