# Mac Studio Server Configuration for Ollama

This repository contains configuration and scripts for running Ollama LLM server on Apple Silicon Macs in headless mode (tested on Mac Studio with M1 Ultra).

## Overview

This configuration is optimized for running Mac Studio as a dedicated Ollama server, with:
- Headless operation (SSH access recommended)
- Minimal resource usage (GUI and unnecessary services disabled)
- Automatic startup and recovery
- Performance optimizations for Apple Silicon

## Latest Updates

- **[v1.1.0]** Added GPU Memory Optimization - configure Metal to use more RAM for models
- **[v1.0.0]** Initial release with system optimizations and Ollama configuration

See the [CHANGELOG](CHANGELOG.md) for detailed version history.

## Features

- Automatic startup on boot
- Optimized for Apple Silicon
- System resource optimization through service disabling
- External network access
- Proper logging setup
- SSH-based remote management

## Requirements

- Mac with Apple Silicon
- macOS Sonoma or later
- [Ollama](https://ollama.com/) installed
- Administrative privileges
- SSH enabled (System Settings → Sharing → Remote Login)

## Remote Access

For optimal performance, we recommend:
1. Primary access method: SSH
```bash
ssh username@your-mac-studio-ip
```

2. (Optional) Screen Sharing is kept available for emergency/maintenance access but not recommended for regular use to save resources.

## Installation

1. Clone this repository:
```bash
git clone https://github.com/anurmatov/mac-studio-server.git
cd mac-studio-server
```

2. (Optional) Configure installation:
```bash
# Default values shown
export OLLAMA_USER=$(whoami)  # User to run Ollama as
export OLLAMA_BASE_DIR="/Users/$OLLAMA_USER/mac-studio-server"
export OLLAMA_GPU_PERCENT="80"  # Set to enable GPU memory optimization (percentage of RAM to allocate)
```

3. Run the installation script:
```bash
chmod +x scripts/install.sh
./scripts/install.sh
```

## Configuration

The Ollama service is configured with the following optimizations:
- External access enabled (0.0.0.0:11434)
- 8 parallel requests (adjustable)
- 30-minute model keep-alive
- Flash attention enabled
- Support for 4 simultaneously loaded models
- Model pruning disabled

### Customizing Configuration

To modify the Ollama service configuration:

1. Edit the configuration file:
```bash
vim config/com.ollama.service.plist
```

2. Apply the changes:
```bash
# Stop the current service
sudo launchctl unload /Library/LaunchDaemons/com.ollama.service.plist

# Copy the updated configuration
sudo cp config/com.ollama.service.plist /Library/LaunchDaemons/

# Set proper permissions
sudo chown root:wheel /Library/LaunchDaemons/com.ollama.service.plist
sudo chmod 644 /Library/LaunchDaemons/com.ollama.service.plist

# Load the updated service
sudo launchctl load -w /Library/LaunchDaemons/com.ollama.service.plist
```

3. Check the logs for any issues:
```bash
tail -f logs/ollama.err logs/ollama.log
```

## System Optimizations

The installation process:
- Disables unnecessary system services
- Configures power management for server use
- Optimizes for background operation
- Maintains Screen Sharing capability for remote management

## Logs

Log files are stored in the `logs` directory:
- `ollama.log` - Ollama service logs
- `ollama.err` - Ollama error logs
- `install.log` - Installation logs
- `optimization.log` - System optimization logs

## Performance Considerations

This configuration significantly reduces system resource usage:
- Memory usage reduction from 11GB to 3GB (tested on Mac Studio M1 Ultra)
- Disables GUI-related services
- Minimizes background processes
- Prevents sleep/hibernation
- Optimizes for headless operation

The dramatic reduction in memory usage (around 8GB) is achieved by:
1. Disabling Spotlight indexing
2. Turning off unnecessary system services
3. Minimizing GUI-related processes
4. Optimizing for headless operation

### GPU Memory Optimization

By default, Metal runtime allocates only about 75% of system RAM for GPU operations. This configuration includes optional GPU memory optimization that:
- Runs at system startup (when enabled)
- Allocates a configurable percentage of your total RAM to GPU operations
- Logs the changes for monitoring

The GPU memory setting is critical for LLM performance on Apple Silicon, as it determines how much of your unified memory can be used for model operations.

This allows:
- More efficient model loading
- Better performance for large models
- Increased number of concurrent model instances
- Fuller utilization of Apple Silicon's unified memory architecture

To enable and configure GPU memory optimization, set the environment variable before installation:
```bash
export OLLAMA_GPU_PERCENT="80"  # Allocate 80% of RAM to GPU
./scripts/install.sh
```

Or to adjust after installation:
```bash
# Run with a custom percentage
OLLAMA_GPU_PERCENT=85 sudo ./scripts/set-gpu-memory.sh
```

If you don't set OLLAMA_GPU_PERCENT, GPU memory optimization will be skipped.

For best performance:
1. Use SSH for remote management
2. Keep display disconnected when possible
3. Avoid running GUI applications
4. Consider disabling Screen Sharing if not needed for emergency access
5. Adjust GPU memory percentage based on your available memory and workload

These optimizations leave more resources available for Ollama model operations, allowing for better performance when running large language models.

## Versioning

This project follows [Semantic Versioning](https://semver.org/):
- MAJOR version for incompatible changes
- MINOR version for new features
- PATCH version for bug fixes

The current version is 1.1.0.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License