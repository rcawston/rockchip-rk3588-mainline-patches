# Out-of-Tree RK3588 patches for Rockchip RK3588 (WIP)

This repository contains an out-of-tree patches for RK3588 to a near-mainline kernel (tested on v6.19-rc8)

Current Patches:

- H.265 Hardware Encoding Support: 0001-rockchip-rk3588-vepu580-encoder-support-v2.patch
- Set EDID Fix: 0002-rockchip-rk3588-hdmirx-edid-fix-v1.patch

Testing is done on Rock 5B[+] and Orange Pi 5 Ultra.

## RK3588 VEPU580 H.265 Encoder Driver for Mainline Linux (WIP)

This repository contains an out-of-tree patch series adding stateful H.265 (HEVC) encoding support for the Rockchip RK3588's VEPU580 encoder block to a near-mainline kernel (tested on v6.19-rc8).

**Status**: Functional dual-core encoding. Produces valid H.265 bitstreams at 4K@60. Multi-core load balancing works.

**Goal**: Provide a practical, high-performance encoder driver for RK3588 boards (Orange Pi 5 Ultra, Radxa Rock 5B[+], etc.) while staying as close to mainline as possible. Not intended for upstream merge — although feedback welcome on design and upstreamability.

**Motivation**: Enable low-latency hardware-accelerated H.265 encoding for applications like IPKVM (e.g., SentinelKVM project) on mainline kernels without full vendor BSP dependency.

### Pre-Requisite: Build & Install MPP

Build directly on the RK3588 device (or cross-compile for faster results).

1. Install Dependencies:

```bash
sudo apt update
sudo apt install cmake build-essential libdrm-dev libva-dev
```

2. Clone & Build:

```bash
git clone https://github.com/tsukumijima/mpp-rockchip.git
cd mpp-rockchip
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make -j$(nproc)
sudo make install
sudo ldconfig  # Update library cache
```

3. Verify

```bash
mpp_info  # Should show MPP version and supported codecs (including HEVC encode)
```

## Set EDID Fix

This patch addresses an issue where the EDID is not being written correctly to the HDMIRX block.
The patch ensures that the EDID is correctly written and that the source device sees the newly set EDID.

Tested with various EDID including 4k@60hz support.

## How to Apply

```bash
# Clone mainline kernel
git clone --depth 1 -b v6.19-rc8 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux-mainline
cd linux-mainline

# Apply patches
git am /path/to/patches/*.patch
```

## Use with armbian-build

1. Clone Armbian repo

```bash
git clone --depth 1 https://github.com/armbian/build
cd build
```

2. Create Patch Directory

- Armbian applies user patches from userpatches/.
- Create the directory for rockchip64 edge kernel (or matching kernel family dir - replace "rockchip64-6.19" with your target kernel family):

```bash
mkdir -p ./userpatches/kernel/archive/rockchip64-6.19/
```

3. Copy Patches

- Place patch files (e.g., 0001-*.patch, 0002-*.patch) into ./userpatches/kernel/archive/rockchip64-6.19/
- Armbian automatically applies patches in lexical order during build.

4. Kernel Config Fragment

- Set CONFIG_VIDEO_ROCKCHIP_RKVENC=m during kernel config

5. Build the Image

```bash
./compile.sh
```
