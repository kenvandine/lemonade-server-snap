# Lemonade Server Snap

A snap package for [Lemonade Server](https://lemonade-server.ai/) - a lightweight, high-performance local AI inference server with an OpenAI-compatible API.

## Installation

```bash
sudo snap install lemonade-server
```

## Features

- **OpenAI-compatible API** - Drop-in replacement for OpenAI API endpoints
- **Multiple backends** - Vulkan, ROCm (AMD GPUs), and CPU support
- **Automatic model management** - Downloads and caches models from Hugging Face
- **Background service** - Runs automatically on system startup
- **Hardware acceleration** - Optimized for AMD GPUs with ROCm support

## Supported Hardware

| Backend | Hardware | Architecture |
|---------|----------|--------------|
| Vulkan | Any Vulkan-capable GPU | - |
| ROCm | AMD RX 7000 series (RDNA3) | gfx110X |
| ROCm | AMD RX 9000 series (RDNA4) | gfx120X |
| ROCm | AMD Strix Point APUs | gfx1150 |
| ROCm | AMD Strix Halo APUs | gfx1151 |
| CPU | Any x86_64 processor | - |

## Quick Start

After installation, the server starts automatically and listens on port 8000.

The snap automatically detects your GPU and selects the best backend:
- **AMD GPUs** - Uses ROCm for optimal performance
- **Other GPUs** - Uses Vulkan

### Enable ROCm Support (AMD GPUs)

For ROCm GPU acceleration on AMD hardware, connect the `process-control` interface:

```bash
sudo snap connect lemonade-server:process-control
sudo snap restart lemonade-server.service
```

This is required because ROCm needs permission to set CPU thread affinity for optimal performance.

### Check service status

```bash
sudo snap services lemonade-server
```

### View logs

```bash
sudo journalctl -u snap.lemonade-server.service -f
```

### Test the API

```bash
curl http://localhost:8000/api/v1/models
```

## Usage

### Load a model

```bash
curl -X POST http://localhost:8000/api/v1/load \
  -H "Content-Type: application/json" \
  -d '{"model_name": "Llama-3.2-3B-Instruct-GGUF"}'
```

### Chat completion

```bash
curl -X POST http://localhost:8000/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Llama-3.2-3B-Instruct-GGUF",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### List available models

```bash
curl http://localhost:8000/api/v1/models
```

## Configuration

### Model cache location

Models are cached in `/var/snap/lemonade-server/common/.cache/huggingface/`.

### Using external models

To use models from your home directory or removable media, the snap has access to:
- `$HOME` (via the `home` plug)
- `/media` and `/mnt` (via the `removable-media` plug)

### Manual server control

```bash
# Stop the service
sudo snap stop lemonade-server.service

# Start the service
sudo snap start lemonade-server.service

# Restart the service
sudo snap restart lemonade-server.service

# Disable autostart
sudo snap stop --disable lemonade-server.service
```

### Command-line usage

You can also run the server manually:

```bash
lemonade-server --help
lemonade-server serve --port 8080
```

## Connecting to Applications

Lemonade Server is compatible with any application that supports the OpenAI API:

- **Open WebUI** - Set API base URL to `http://localhost:8000/api/v1`
- **AnythingLLM** - Configure as OpenAI-compatible endpoint
- **Continue (VS Code)** - Use OpenAI provider with custom base URL
- **LangChain** - Use `ChatOpenAI` with `base_url="http://localhost:8000/api/v1"`

## Troubleshooting

### Service won't start

Check the logs for errors:
```bash
sudo journalctl -u snap.lemonade-server.service --no-pager -n 50
```

### GPU not detected

Ensure you have the proper GPU drivers installed:
```bash
# For AMD GPUs
sudo apt install mesa-vulkan-drivers

# Check Vulkan support
vulkaninfo | head -20
```

### ROCm not working (AMD GPUs)

If you have an AMD GPU but ROCm isn't working:

1. Ensure `process-control` interface is connected:
   ```bash
   sudo snap connect lemonade-server:process-control
   ```

2. Restart the service:
   ```bash
   sudo snap restart lemonade-server.service
   ```

3. Check the logs for backend detection:
   ```bash
   sudo journalctl -u snap.lemonade-server.service.service -n 20
   ```

   You should see: `[Lemonade Wrapper] AMD GPU detected, using ROCm backend`

### Permission issues

The snap runs in strict confinement. If you need to access files outside the allowed locations, you may need to copy them to your home directory first.

### Model download failures

Check your network connection and ensure you have enough disk space:
```bash
df -h /var/snap/lemonade-server/common/
```

## Building from Source

```bash
git clone https://github.com/kenvandine/lemonade-server.git
cd lemonade-server
snapcraft
sudo snap install --dangerous lemonade-server_*.snap
```

## Links

- [Lemonade Server Documentation](https://lemonade-server.ai/)
- [Lemonade SDK GitHub](https://github.com/lemonade-sdk/lemonade)
- [Report Issues](https://github.com/lemonade-sdk/lemonade/issues)

## License

Snap packaging is GPL-3 License - See [LICENSE](LICENSE) for details.
