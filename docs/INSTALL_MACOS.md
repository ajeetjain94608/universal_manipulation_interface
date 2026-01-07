# macOS Installation Guide (Intel)

This guide provides instructions for installing and running Universal Manipulation Interface (UMI) on macOS with Intel processors.

## ⚠️ Important Limitations

Before proceeding, please be aware of the following limitations when running UMI on macOS:

### Hardware Limitations
1. **No CUDA Support**: macOS does not support NVIDIA CUDA, so GPU acceleration is not available. Training and inference will run on CPU, which is significantly slower.
2. **Camera Capture**: Video4Linux (`v4l2`) is not available on macOS. Real-time camera capture functionality may be limited.
3. **SpaceMouse**: The Linux `libspnav` library is not available on macOS. SpaceMouse support requires alternative macOS drivers.

### Recommended Use Cases
macOS installation is suitable for:
- **Dataset Processing**: Running SLAM pipelines on pre-recorded data
- **Model Training**: Training diffusion policies (slower than Linux/CUDA but functional)
- **Development**: Code development and testing
- **Visualization**: Analyzing and visualizing collected data

### Not Recommended For
- **Real-time Robot Control**: Hardware interfacing is primarily designed for Linux
- **Production Deployment**: Performance-critical applications requiring GPU acceleration
- **High-throughput Training**: Large-scale model training (very slow on CPU)

## 🛠️ Installation Steps

### 1. Install Homebrew
If you don't have Homebrew installed:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. Install System Dependencies

Install required system libraries:
```bash
brew install glfw
brew install mesa
brew install patchelf
```

**Note**: These are macOS equivalents of the Linux packages. Some MuJoCo-related functionality may require additional setup.

### 3. Install Docker Desktop for Mac

Download and install Docker Desktop from the [official website](https://www.docker.com/products/docker-desktop/).

After installation:
1. Open Docker Desktop
2. Go to Preferences → Resources to allocate sufficient CPU and memory
3. Ensure Docker is running before using SLAM pipeline features

### 4. Install Conda/Miniforge

We recommend [Miniforge](https://github.com/conda-forge/miniforge) for faster installation:

```bash
# Download Miniforge for macOS Intel
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-x86_64.sh"

# Install
bash Miniforge3-MacOSX-x86_64.sh

# Follow the prompts, then restart your shell
```

### 5. Create Conda Environment

Clone this repository and create the conda environment:
```bash
cd universal_manipulation_interface
mamba env create -f conda_environment_macos.yaml
```

Or if you're using conda:
```bash
conda env create -f conda_environment_macos.yaml
```

**Note**: The first installation may take 15-30 minutes depending on your internet connection.

### 6. Activate Environment

```bash
conda activate umi
```

## 🧪 Testing Your Installation

### Test Python Environment
```bash
python -c "import torch; import numpy; import cv2; print('PyTorch version:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"
```

Expected output:
```
PyTorch version: 2.1.0
CUDA available: False
```

**Note**: `CUDA available: False` is expected on macOS.

### Test MuJoCo (Optional)
```bash
python -c "import mujoco_py; print('MuJoCo installed successfully')"
```

If MuJoCo import fails, you may need to install MuJoCo separately. See [MuJoCo documentation](https://github.com/openai/mujoco-py#install-mujoco) for macOS-specific instructions.

## 📊 Running UMI SLAM Pipeline

The SLAM pipeline should work on macOS for processing pre-recorded data.

### Download Example Data

First, install wget if you don't have it:
```bash
brew install wget
```

Then download the example data:
```bash
wget --recursive --no-parent --no-host-directories --cut-dirs=2 --relative --reject="index.html*" https://real.stanford.edu/umi/data/example_demo_session/
```

**Alternative using curl** (if you prefer not to install wget):
```bash
# Note: For recursive downloads, wget is recommended. Install via: brew install wget
```

### Run SLAM Pipeline
```bash
python run_slam_pipeline.py example_demo_session
```

### Generate Dataset
```bash
python scripts_slam_pipeline/07_generate_replay_buffer.py -o example_demo_session/dataset.zarr.zip example_demo_session
```

## 🎓 Training Diffusion Policy

Training will work on macOS but will be **significantly slower** without GPU acceleration.

### CPU-Only Training
```bash
python train.py --config-name=train_diffusion_unet_timm_umi_workspace task.dataset_path=example_demo_session/dataset.zarr.zip
```

**Performance Note**: Training on CPU is approximately 10-50x slower than on GPU. Consider:
- Using a smaller model
- Training for fewer epochs for testing
- Using a Linux machine with CUDA for production training

### Download Pre-trained Model
Instead of training, you can download and use pre-trained models:
```bash
# Using wget (install first: brew install wget)
wget https://real.stanford.edu/umi/data/pretrained_models/cup_wild_vit_l_1img.ckpt

# Or using curl
curl -O https://real.stanford.edu/umi/data/pretrained_models/cup_wild_vit_l_1img.ckpt
```

## 🦾 Real-World Deployment (Limited Support)

Real-world robot control on macOS has limited support due to hardware interfacing constraints.

### Limitations
1. **Camera Capture**: UVC camera support requires Video4Linux, which is not available on macOS
2. **SpaceMouse**: Requires `libspnav`, which is Linux-specific
3. **USB Devices**: Robot control interfaces may have limited macOS support

### Alternative Approach
For real-world deployment, we recommend:
1. Use macOS for development and data processing
2. Deploy models on a Linux machine (Ubuntu 22.04 recommended) for robot control
3. Use SSH/remote access to control the Linux deployment machine

## 🐛 Troubleshooting

### Issue: OpenCV Video Capture Not Working
**Solution**: On macOS, OpenCV may need additional permissions:
```bash
# Grant camera access to Terminal in System Preferences → Security & Privacy → Camera
```

### Issue: MuJoCo Import Error
**Solution**: Install MuJoCo manually:
1. Download MuJoCo from [official website](https://mujoco.org/)
2. Extract to `~/.mujoco/mujoco210`
3. Set environment variable:
   ```bash
   export MUJOCO_PY_MUJOCO_PATH=~/.mujoco/mujoco210
   ```

### Issue: "No module named 'v4l2py'"
**Solution**: This is expected on macOS. The `v4l2py` module is Linux-specific and is excluded from the macOS environment. Real-time camera capture features that depend on this module will not work.

### Issue: "No module named 'spnav'"
**Solution**: SpaceMouse support via `spnav` is Linux-specific. On macOS, you would need to use alternative SpaceMouse drivers if available. This functionality is not included in the macOS setup.

### Issue: Slow Training Performance
**Solution**: This is expected on CPU-only systems. Options:
1. Reduce batch size in training config
2. Use a smaller model architecture
3. Train on a Linux machine with CUDA support
4. Use cloud GPU instances (AWS, Google Cloud, etc.)

### Issue: Docker SLAM Pipeline Fails
**Solution**: 
1. Ensure Docker Desktop is running
2. Allocate more resources in Docker Desktop preferences
3. Check Docker logs for specific errors

## 📝 Differences from Linux Installation

| Feature | Linux (Ubuntu 22.04) | macOS (Intel) |
|---------|---------------------|---------------|
| GPU Acceleration | ✅ CUDA Support | ❌ CPU Only |
| SLAM Pipeline | ✅ Full Support | ✅ Full Support |
| Training | ✅ Fast (GPU) | ⚠️ Slow (CPU) |
| Real-time Camera | ✅ V4L2 Support | ❌ Limited Support |
| SpaceMouse | ✅ Full Support | ❌ No Support |
| Robot Control | ✅ Full Support | ⚠️ Limited Support |
| Development | ✅ Recommended | ✅ Supported |

## 🔗 Additional Resources

- [Main README](../README.md) - General UMI documentation
- [Hardware Guide](https://docs.google.com/document/d/1TPYwV9sNVPAi0ZlAupDMkXZ4CA1hsZx7YDMSmcEy6EU/edit?usp=sharing) - Hardware setup instructions
- [SLAM Repository](https://github.com/cheng-chi/ORB_SLAM3) - ORB-SLAM3 fork for UMI
- [SLAM Docker](https://hub.docker.com/r/chicheng/orb_slam3) - Docker image for SLAM

## 💡 Tips for macOS Users

1. **Use Linux for Production**: For best results, use macOS for development and Linux for deployment
2. **Cloud Options**: Consider using cloud GPU instances for training
3. **Docker Resources**: Allocate at least 8GB RAM and 4 CPU cores to Docker Desktop
4. **Battery Life**: Training on CPU is power-intensive; keep your MacBook plugged in
5. **Cooling**: CPU-intensive training may cause thermal throttling; ensure good ventilation

## 🤝 Contributing

If you find ways to improve macOS support or work around limitations, please contribute:
1. Test on your macOS system
2. Document your findings
3. Submit a pull request with improvements

## ⚠️ Disclaimer

This macOS installation guide is provided for development and testing purposes. For production deployment and real-time robot control, we strongly recommend using Ubuntu 22.04 Linux as documented in the main [README](../README.md).
