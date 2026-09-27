## Ollama Setup

**step-by-step procedure for installing Ollama on your Colab Linux VM without sudo**, while keeping NVIDIA/T4 GPU support.

The official Ollama Linux documentation normally installs into system directories with `sudo`, but your Colab `colab` user doesn't have sudo, so we use a user-local installation instead. ([GitHub][1])

## 1. Check GPU

```bash
nvidia-smi
```

You should see:

```text
Tesla T4
15360 MiB
```

---

## 2. Download Ollama

```bash
curl -L https://ollama.com/download/ollama-linux-amd64.tar.zst -o ~/ollama.tar.zst
```

Check:

```bash
ls -lh ~/ollama.tar.zst
```

---

## 3. Install Zstandard support

Your Colab environment may not have the `zstd` command.

Check:

```bash
which zstd
```

If it doesn't exist, Python's `zstandard` package can be used. In your case it was already available:

```bash
python3 -c "import zstandard; print(zstandard.__version__)"
```

---

## 4. Decompress the Ollama archive

```bash
python3 - <<'PY'
import zstandard as zstd

src = "/home/colab/ollama.tar.zst"
dst = "/home/colab/ollama.tar"

print("Decompressing...")

dctx = zstd.ZstdDecompressor()

with open(src, "rb") as inp, open(dst, "wb") as out:
    dctx.copy_stream(inp, out)

print("Done:", dst)
PY
```

---

## 5. Extract Ollama

```bash
mkdir -p ~/.local
tar -xf ~/ollama.tar -C ~/.local
```

Check:

```bash
find ~/.local -type f -name ollama -ls
```

In your setup, this produced:

```text
/home/colab/.local/bin/ollama
```

So **don't copy it anywhere else**. The Ollama binary is already in the correct location.

---

## 6. Add Ollama to PATH

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

Then:

```bash
source ~/.bashrc
```

Verify:

```bash
which ollama
```

Expected:

```text
/home/colab/.local/bin/ollama
```

Then:

```bash
ollama --version
```

---

## 7. Configure NVIDIA library

Your Colab NVIDIA library is:

```text
/usr/lib64-nvidia/libnvidia-ml.so
```

So:

```bash
export LD_LIBRARY_PATH=/usr/lib64-nvidia:$LD_LIBRARY_PATH
```

Make it permanent:

```bash
echo 'export LD_LIBRARY_PATH=/usr/lib64-nvidia:$LD_LIBRARY_PATH' >> ~/.bashrc
```

Reload:

```bash
source ~/.bashrc
```

Verify:

```bash
nvidia-smi
```

---

## 8. Start Ollama

Run:

```bash
ollama serve
```

You'll want to keep this terminal running.

Ollama's Linux documentation uses `ollama serve` when starting the server manually. ([GitHub][1])

---

## 9. Open another VS Code terminal

Run:

```bash
export PATH="$HOME/.local/bin:$PATH"
export LD_LIBRARY_PATH=/usr/lib64-nvidia:$LD_LIBRARY_PATH
```

Check:

```bash
ollama list
```

---

## 10. Pull a model

For Qwen3 4B:

```bash
ollama pull qwen3:4b
```

Then:

```bash
ollama run qwen3:4b
```

---

## 11. Verify GPU usage

While Qwen is running, open another VS Code terminal:

```bash
watch -n 1 nvidia-smi
```

You should see Ollama consuming the T4 VRAM.

You can also use:

```bash
nvidia-smi --query-gpu=name,utilization.gpu,memory.used,memory.total,temperature.gpu,power.draw --format=csv
```

### Final setup

```text
Windows PC
    │
    │ VS Code
    │ SSH
    ▼
Google Colab Linux
    │
    ├── 2 CPU
    ├── 12 GB RAM
    ├── 65 GB free disk
    │
    ├── NVIDIA Tesla T4
    │      └── 15 GB VRAM
    │
    └── Ollama
          │
          └── Qwen3 4B
```

### Important

Because this is a **Colab runtime**, the Ollama binary and downloaded models are stored on the temporary Colab filesystem. If the runtime is reset/disconnected, you'll need to reinstall/re-download unless you arrange persistent storage.

Official Ollama Linux documentation: [Ollama Linux Installation](https://docs.ollama.com/linux?utm_source=chatgpt.com)

[1]: https://github.com/ollama/ollama/blob/main/docs/linux.mdx?utm_source=chatgpt.com "ollama/docs/linux.mdx at main · ollama/ollama · GitHub"
