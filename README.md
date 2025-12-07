# StoRM: Wind Noise Filtering

This repository contains the code for wind noise reduction using the StoRM (Stochastic Regeneration Model) framework, based on diffusion-based generative models for speech enhancement.

## Model Download

Before running inference, you need to download the pretrained model checkpoint:

1. Download the model from [Google Drive](https://drive.google.com/file/d/19Naet8MkFCo5P-ZadQk8EntJgRNh8JKe/view?usp=drive_link)
2. Place the downloaded checkpoint file (e.g., `denoiser_ncsnpp_wsj0+wind_epoch=199.ckpt`) in the repository root directory

## Installation

1. Create a virtual environment (Python 3.8 recommended):
   ```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

2. Install dependencies:
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

   **Note:** If you encounter issues with `pytorch-lightning==1.8.3`, you may need to downgrade pip first:
   ```bash
   pip install "pip<24.1"
   pip install -r requirements.txt
   ```

3. For macOS users, install additional system dependencies:
   ```bash
   brew install libsndfile  # If using Homebrew
   ```

## Running Inference

To enhance audio files, use the `enhancement.py` script:

```bash
python enhancement.py \
  --test_dir <input_directory> \
  --enhanced_dir <output_directory> \
  --ckpt <path_to_checkpoint> \
  --mode denoiser-only
```

### Example

```bash
# Create directories for input and output
mkdir -p test_input enhanced_output

# Place your audio files (must be 16kHz) in test_input/
# Then run inference:
python enhancement.py \
  --test_dir test_input \
  --enhanced_dir enhanced_output \
  --ckpt denoiser_ncsnpp_wsj0+wind_epoch=199.ckpt \
  --mode denoiser-only
```

### Important Notes

- **Sample Rate**: Input audio files must be 16kHz. If your files are at a different sample rate, resample them first:
  ```bash
  # Using Python with torchaudio:
  python -c "import torchaudio; y, sr = torchaudio.load('input.wav'); resampled = torchaudio.functional.resample(y, sr, 16000); torchaudio.save('input_16k.wav', resampled, 16000)"
  ```

- **Audio Format**: The script processes all `.wav` files in the input directory

- **Mode Options**: 
  - `denoiser-only`: Uses only the discriminative denoiser model (fastest)
  - `storm`: Uses the full StoRM model with both denoiser and score model
  - `score-only`: Uses only the score-based diffusion model

### Advanced Options

```bash
python enhancement.py \
  --test_dir test_input \
  --enhanced_dir enhanced_output \
  --ckpt denoiser_ncsnpp_wsj0+wind_epoch=199.ckpt \
  --mode denoiser-only \
  --N 50 \
  --corrector ald \
  --corrector-steps 1 \
  --snr 0.5
```

- `--N`: Number of reverse diffusion steps (default: 50)
- `--corrector`: Corrector type - `ald`, `langevin`, or `none` (default: `ald`)
- `--corrector-steps`: Number of corrector steps (default: 1)
- `--snr`: SNR value for Langevin dynamics (default: 0.5)

## Troubleshooting

### macOS Compatibility

This code has been tested on macOS. Some modifications were made for macOS compatibility:
- CUDA extensions are automatically skipped (uses CPU fallback)
- Model will run on CPU if CUDA is not available

### Common Issues

1. **"ModuleNotFoundError"**: Make sure all dependencies are installed in your virtual environment
2. **"Sample rate mismatch"**: Ensure all input audio files are resampled to 16kHz
3. **CUDA errors on macOS**: This is expected - the model will automatically use CPU mode

## Citation

If you use this code, please cite the original StoRM paper:

```bibtex
@article{lemercier2023storm,
  author={Lemercier, Jean-Marie and Richter, Julius and Welker, Simon and Gerkmann, Timo},
  journal={IEEE/ACM Transactions on Audio, Speech, and Language Processing}, 
  title={StoRM: A Diffusion-Based Stochastic Regeneration Model for Speech Enhancement and Dereverberation}, 
  year={2023},
  volume={31},
  number={},
  pages={2724-2737},
  doi={10.1109/TASLP.2023.3294692}
}
```

## License

See LICENSE file for details.

