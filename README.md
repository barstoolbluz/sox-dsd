# SoX-DSD: Sound eXchange with Direct Stream Digital Support

This is a fork of [Måns Rullgård's SoX](http://sox.sourceforge.net/) that adds comprehensive Direct Stream Digital (DSD) support. SoX is a cross-platform command-line utility that can convert various formats of computer audio files, apply effects to them, and perform other audio processing tasks.

## Key Features

This fork extends the original SoX with:

- **DSD File Format Support**: Native support for DSF (DSD Stream File) and DSDIFF formats
- **DSD over PCM (DoP)**: Support for transmitting DSD data over PCM interfaces
- **Advanced Noise Shaping**: Multiple sigma-delta modulation algorithms including CLANS and SDM
- **Trellis Optimization**: High-quality encoding with configurable lookahead depth
- **Multiple DSD Rates**: Support for DSD64, DSD128, DSD256, and DSD512

## Building with Flox

The easiest way to build SoX-DSD is using [Flox](https://flox.dev/get), which provides a reproducible build environment with all necessary dependencies.

### Prerequisites

1. Install Flox:
   ```bash
   curl -L https://flox.dev/install | sh
   ```

2. Clone this repository:
   ```bash
   git clone <repository-url>
   cd sox-dsd
   ```

### Build Instructions

The repository includes a pre-configured Flox environment. Simply run:

```bash
# Activate the Flox environment
flox activate

# Build SoX-DSD
flox build sox

# The built binary will be available at:
./result-sox/bin/sox
```

The Flox environment automatically handles:
- Platform-specific dependencies (Linux/macOS)
- All required audio codec libraries (FLAC, LAME, Vorbis, etc.)
- Build tools (autoconf, automake, libtool)
- Audio I/O libraries (ALSA, PulseAudio on Linux; CoreAudio on macOS)

### Installing SoX-DSD

#### Option 1: Local Installation (Recommended)

For a portable, self-contained installation to your home directory:

```bash
./local-install
```

This will:
- Build SoX-DSD using Flox
- Create a complete, self-contained package at `$HOME/.local/sox-dsd`
- Work on any Linux or macOS system without root access
- Include all dependencies - no system libraries required

To install to a custom location:
```bash
./local-install /path/to/installation
```

After installation, add to your PATH:
```bash
export PATH="$HOME/.local/sox-dsd/bin:$PATH"
```

#### Option 2: Publish to the Flox Catalog

You can install SoX-DSD and its complete runtime dependencies by publishing it to your private Flox Catalog.

**Why publish to Flox?**
- **Easy distribution**: You can install with `flox install <your-handle>/sox-dsd`
- **Version management**: Track releases and pin specific versions
- **Cross-platform support**: Build once per platform, install anywhere
- **Dependency handling**: Flox manages the entire dependency closure automatically

**Prerequisites:**
1. A [FloxHub](https://hub.flox.dev) account (free)
2. Clean Git repository with all changes committed and pushed
3. Successfully built package (`flox build sox`)

**Publishing steps:**

```bash
# Ensure git is clean and pushed
git status  # Should show "working tree clean"
git push

# Publish to your catalog
flox publish -o <your-floxhub-handle> sox

# Example:
flox publish -o myusername sox
```

After publishing, you can imperatively install your build:
```bash
# Install from catalog
flox install myusername/sox-dsd

# Or add declaratively to your manifest
[install]
"myusername/sox-dsd".pkg-path = "myusername/sox-dsd"
```

**Multi-platform considerations:**
- Build and publish on each platform you want to support (Linux x86_64, macOS ARM64, etc.)
- You'll see platform availability when running `flox show myusername/sox-dsd`
- The package automatically installs the correct version for each platform

**Benefits over local installation:**
- No manual copying of files
- Automatic updates when you publish new versions
- Integration with other Flox environments
- Cached builds across machines

## What's Different about This SoX Build? DSD Encoding!

### Basic Concepts

Direct Stream Digital (DSD) uses 1-bit sigma-delta modulation at very high sampling rates:
- **DSD64**: 2.8224 MHz (64× CD sample rate)
- **DSD128**: 5.6448 MHz
- **DSD256**: 11.2896 MHz
- **DSD512**: 22.5792 MHz

### Noise Shaping and Modulators

SoX-DSD offers multiple modulator types:

1. **CLANS (Closed Loop Analysis of Noise Shapers)**: Optimized for stability and lower distortion
2. **SDM (Standard Delta-Sigma Modulation)**: Conventional approach with potentially lower noise
3. **CRFB (Cascade of Resonators with distributed FeedBack)**: Alternative modulator design

Available filter options include: `clans-4`, `clans-8`, `sdm-4`, `sdm-8`, `crfb-4`, `crfb-8`, etc.

#### Recommended Settings by DSD Rate

| DSD Rate | Recommended Modulator | Alternative |
|----------|----------------------|-------------|
| DSD64    | CLANS-8             | SDM-8 (less stable) |
| DSD128   | CLANS-7             | SDM-7 |
| DSD256   | CLANS-6             | SDM-6 |
| DSD512   | CLANS-5             | SDM-5 |

Higher DSD rates allow lower-order modulators due to inherent noise reduction at elevated sampling frequencies.

### Basic Encoding Examples

Convert PCM to DSD64:
```bash
sox input.wav output.dsf rate -v 2822400 sdm -f clans-8
```

Convert PCM to DSD128:
```bash
sox input.wav output.dsf rate -v 5644800 sdm -f clans-7
```

### Advanced Encoding with Trellis Optimization

Trellis optimization improves encoding quality by exploring alternative bitstream paths:

```bash
# High-quality encoding with trellis optimization
sox input.wav output.dsf rate -v 2822400 sdm -f clans-8 -t 32 -n 32
```

#### Trellis Parameters

- **`-t N`**: Lookahead depth (1-64). Higher values improve quality but increase encoding time
- **`-n N`**: Number of trellis nodes (1-32+). More nodes enhance quality at computational cost
- **`-l N`**: Trellis latency in samples (optional). Usually set automatically, but can be manually increased for slightly better results at the cost of additional delay

**Practical Settings:**
- Moderate quality: `-t 8 -n 8`
- High quality: `-t 16 -n 16`
- Mastering quality: `-t 32 -n 32` (very slow)
- Very high quality with explicit latency: `-t 32 -n 32 -l 64`

### Signal Level Standards

DSD uses different level references than PCM:
- **0dBDSD**: 50% modulation, equivalent to -6dBFS PCM
- **+3.1dBDSD**: SACD peak limit (71.43% modulation, -2.9dBFS PCM)

When converting:
- Target 0dBDSD (-6dBFS) for base level
- Allow peaks up to +3.1dBDSD
- Apply +2.5dB gain when converting DSD→PCM to avoid clipping

### Format Conversion Examples

DSD to PCM:
```bash
sox input.dsf output.wav rate -v 96000 gain +2.5
```

DSF to DSDIFF:
```bash
sox input.dsf output.dff
```

DSD over PCM (DoP):
```bash
sox input.dsf output.dop
```

## Supported Formats

In addition to standard SoX formats, this fork supports:
- **DSF** (DSD Stream File) - Read/Write
- **DSDIFF** (DSD Interchange File Format) - Read/Write
- **DoP** (DSD over PCM) - Read/Write

## Performance Considerations

- **Encoding Speed**: Trellis optimization significantly increases encoding time
- **Memory Usage**: Higher-order modulators and trellis settings require more RAM
- **CPU Usage**: DSD encoding is computationally intensive, especially at higher rates

## Troubleshooting

### Modulator Stability
- High-order modulators (especially SDM-8) may become unstable with high-amplitude signals
- DSD128 can be particularly challenging to encode
- If experiencing distortion or overload, try:
  - Using CLANS instead of SDM
  - Reducing the modulator order
  - Lowering the input signal level

### Platform-Specific Notes
- **Linux**: Requires ALSA or PulseAudio for real-time playback
- **macOS**: Uses CoreAudio; some features may require additional configuration
- **Windows**: See README.win32 for Windows-specific instructions

## License

SoX is distributed under the terms of the GNU Lesser General Public License (LGPL). See COPYING and LICENSE.LGPL for details.

## Contributors

Original SoX authors and contributors - see AUTHORS file
DSD support by Måns Rullgård
Fork maintained by the SoX-DSD community

## Further Reading

- [SOX-DSD-BASICS.md](SOX-DSD-BASICS.md) - Detailed DSD encoding primer
- [SoX Documentation](http://sox.sourceforge.net/sox.html) - Original SoX documentation
- Platform-specific guides: README.osx, README.win32

