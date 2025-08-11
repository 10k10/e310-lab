# E310-Lab

A Python toolkit and examples for controlling an Ettus E310-class **ANTSDR** (AD9361) over Ethernet using [pyadi-iio](https://github.com/analogdevicesinc/pyadi-iio).
Includes a device wrapper, DSP utilities, and example scripts for IQ capture, processing, and playback.

## Features

* **Full RX/TX control** for 2×2 MIMO (channels 0 & 1)
* Configure LO, gain, RF bandwidth, and buffer size
* Capture raw IQ to NumPy arrays (single or dual channel)
* Save IQ with JSON metadata sidecar
* Safe TX with cyclic or non-cyclic buffers
* Tone generator for quick RF sanity tests
* Ready-to-use example scripts
* `.env` support for easy IP configuration

## Requirements

* Python 3.8+
* Ettus E310 / ANTSDR with AD9361 chip
* Network connection to SDR (default `192.168.1.10`)
* [libiio](https://wiki.analog.com/resources/tools-software/libiio) installed on the host

## Installation

1. Clone this repo:

   ```bash
   git clone https://github.com/<youruser>/e310-lab.git
   cd e310-lab
   ```

2. Create a virtual environment:

   ```bash
   python -m venv .venv
   source .venv/bin/activate   # Linux/macOS
   .venv\\Scripts\\activate    # Windows
   ```

3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Copy `.env.example` to `.env` and set your SDR IP:

   ```env
   SDR_IP=192.168.1.10
   ```

## Usage

Example: simple capture and configuration.

```bash
python examples/main.py
```

Inside `examples/main.py`:

```python
from sdr_device import E310SDR
from dotenv import load_dotenv
import os

load_dotenv()
URI = f"ip:{os.getenv('SDR_IP', '192.168.1.10')}"

with E310SDR(uri=URI) as sdr:
    sdr.configure_rx(fs=2.5e6, lo=100e6, rf_bw=2.5e6,
                     channels=(0,), gain_mode="manual", gain_db=20,
                     buffer_len=1 << 15)
    iq = sdr.capture(n_buffers=4)
    print(f"Captured {len(iq)} samples")
```

## Folder Structure

```
.
├── src/                 # Source code for SDR wrapper and utilities
│   └── sdr_device.py    # E310/AD9361 wrapper class
├── examples/            # Top-level scripts and usage examples
│   └── main.py          # Example capture script
├── requirements.txt
├── .env.example         # Sample IP config
├── .env                 # Local environment config (ignored in git)
├── .gitignore
└── README.md
```

## Notes

* RX channels 0 and 1 share the same RX LO; TX channels 0 and 1 share the same TX LO.
* For Doppler correction, consider **digital mixing** in baseband instead of retuning the LO.
* Keep large IQ files out of Git; use `.gitignore` to exclude `*.npy`, `*.bin`, etc.
