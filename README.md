# HDMI Sniffer Project

## Problem Statement

Traveling video artists and VJs frequently encounter HDMI connectivity issues at venues that are difficult to diagnose and troubleshoot in real-time. These issues manifest in two critical ways:

1. **Initial Connection Failures**: Unable to establish a stable HDMI connection with venue equipment (projectors, video routers, switchers) due to EDID incompatibilities or negotiation failures
2. **Connection Dropouts During Performance**: HDMI connections that fail mid-performance due to timing sync issues, signal retraining, or other protocol-level problems

Currently, video artists have limited visibility into these problems and lack portable, affordable tools to diagnose HDMI issues in the field. This project aims to develop open-source, cost-accessible hardware tools to monitor and diagnose HDMI connections.

## Proposed Solution

Two complementary devices built from off-the-shelf components:

### Device 1: EDID Reader/Manager
**Purpose**: Pre-connection diagnostics and EDID management  
**Function**: Plugs into venue HDMI equipment (projectors, routers, etc.) to read and display supported EDID modes, with capability to inject custom EDID for compatibility  
**Use Case**: Allows VJs to verify compatibility, configure output settings appropriately, and work around problematic venue EDIDs  
**Inspiration**: HDMI dummy plug reprogramming via Raspberry Pi I2C

### Device 2: Inline Connection Monitor  
**Purpose**: Real-time connection monitoring and protocol analysis  
**Function**: Sits inline between video source and display to monitor HDMI signal health, DDC/I2C traffic, and protocol-level events during performance  
**Use Case**: Logs timing, sync, retraining, HDCP handshake events, and CEC traffic to diagnose intermittent connection issues  
**Inspiration**: HDMI-over-IP extenders (Lenkeng), CEC sniffers, DDC traffic analysis tools

## R&D Areas

### Hardware Architecture
- [ ] **EDID Reader/Manager**: Raspberry Pi with I2C access (address 0x50) for reading/writing EDID data
  - Evaluate HDMI dummy plug modification approach (reprogrammable EEPROM)
  - Consider EDID injection/override capabilities for compatibility fixes
  - Research multi-bank EDID storage (see commercial devices: 5-40 banks)
- [ ] **Inline Monitor**: Research HDMI pass-through or splitter-based solutions
  - Investigate Lenkeng HDMI-over-IP extenders as potential monitoring platform
  - Evaluate CEC-Tiny-Pro approach for DDC/CEC sniffer (zero additional hardware)
  - Consider HDMI breakout cables for I2C/CEC access without signal interruption
- [ ] Evaluate whether devices should be separate or combinable (leaning toward separate due to multi-connection monitoring needs)
- [ ] Component sourcing to maintain cost accessibility

### EDID Reading and Management
- [ ] I2C-based EDID reading using i2c-tools (read-edid, get-edid)
- [ ] EDID parsing and validation (parse-edid, edid-decode)
- [ ] EDID generation for custom resolutions/timings (edid-generate)
- [ ] Safe EDID writing with verification (edid-checked-writer approach)
- [ ] Display supported resolutions, refresh rates, timings, and vendor info
- [ ] EDID override/injection for problematic displays

### Display & Interface
- [ ] **EDID Reader**: On-screen display showing supported modes, timings, manufacturer data
- [ ] **Inline Monitor**: On-screen display + data logging capability (SD card, USB, or network)
- [ ] Design for both technical and non-technical users with minimal learning curve
- [ ] Standalone operation (no computer required for monitoring)
- [ ] Consider USB interface for configuration and firmware updates

### HDMI Protocol Analysis
- [ ] **DDC (Display Data Channel)**: I2C bus monitoring (address 0x50 for EDID, 0x37 for DDC/CI)
  - Monitor VCP (VESA Control Protocol) feature codes
  - Log DDC/CI transactions for debugging
  - Detect EDID read failures or corruption
- [ ] **CEC (Consumer Electronics Control)**: Pin 13 monitoring
  - Passive sniffing of CEC commands
  - Log device discovery and control messages
- [ ] **TMDS (Transition-Minimized Differential Signaling)**: Signal quality monitoring
  - Clock signal synchronization and drift detection
  - Signal integrity and jitter analysis
  - Pixel clock frequency verification (25.2MHz to 297MHz range)
- [ ] **HDCP (High-bandwidth Digital Content Protection)**:
  - Handshake monitoring and failure detection
  - Authentication event logging (without key extraction)
- [ ] **Hot-plug detect (HPD)** event tracking
- [ ] Signal retraining event detection and logging
- [ ] Cable integrity and signal quality metrics
- [ ] InfoFrame packet analysis (AVI, Audio, Vendor-Specific)

### Common Failure Modes to Address
- [ ] Resolution negotiation failures (EDID incompatibilities)
- [ ] HDCP handshake issues
- [ ] Signal degradation over cable length
- [ ] Timing/sync drift during extended operation
- [ ] Vendor-specific projector/router compatibility issues
- [ ] Intermittent connection dropouts
- [ ] DDC/I2C communication failures
- [ ] CEC conflicts between devices
- [ ] Duplicate serial numbers for sinks confusing MadMapper

### Data Logging & Analysis
- [ ] What metrics to log for the inline monitor
  - HPD events with timestamps
  - HDCP authentication attempts/failures
  - Signal retraining events
  - DDC/I2C transaction logs
  - CEC command sequences
  - Resolution/timing changes
- [ ] Log format: structured text (CSV/JSON) or binary
- [ ] Storage: SD card or USB flash drive
- [ ] Real-time vs. post-performance analysis tools
- [ ] Visualization of connection health over time
- [ ] Export compatibility with standard analysis tools

## Reading EDID from Command Line

These are existing methods to read EDID information on different platforms. Our project aims to provide a portable, standalone alternative for field use.

### Linux

**Using read-edid (recommended):**
```bash
# Install tools
sudo apt-get install read-edid

# Read and parse EDID
sudo get-edid | parse-edid
```

**Using ddcutil:**
```bash
# Install
sudo apt-get install ddcutil

# Detect monitors
ddcutil detect

# Read EDID
ddcutil --display 1 getvcp known
```

**Using xrandr (for connected displays):**
```bash
# List connected displays
xrandr --verbose

# Get detailed output
xrandr --prop
```

**Direct from sysfs:**
```bash
# Find EDID in sysfs
find /sys/devices -name edid

# Read and decode EDID
cat /sys/class/drm/card0-HDMI-A-1/edid | edid-decode
```

#### Raspberry Pi Specific

The Raspberry Pi has special HDMI configuration capabilities through its boot configuration system.

**Reading EDID on Raspberry Pi:**
```bash
# Get current EDID
tvservice -d /tmp/edid.dat
edidparser /tmp/edid.dat

# Or use standard Linux tools
sudo get-edid > edid.bin
parse-edid < edid.bin
```

**Boot Configuration (`/boot/firmware/config.txt` or `/boot/config.txt`):**

```ini
# Force specific HDMI mode
hdmi_group=2        # 1=CEA (TV), 2=DMT (Monitor)
hdmi_mode=82        # 1080p 60Hz

# Force HDMI output even if no display detected
hdmi_force_hotplug=1

# Boost HDMI signal (0-11, default 5)
config_hdmi_boost=7

# Ignore EDID and use safe mode
hdmi_safe=1

# Custom EDID file override
hdmi_edid_file=1
hdmi_edid_filename=custom_edid.dat
```

**Common HDMI modes:**
- Mode 4: 720p 60Hz
- Mode 16: 1080p 60Hz
- Mode 82: 1080p 60Hz (DMT)
- Mode 97: 1080p 60Hz (reduced blanking)

**Useful Raspberry Pi commands:**
```bash
# Show current HDMI status
tvservice -s

# List supported modes
tvservice -m CEA
tvservice -m DMT

# Set specific mode
tvservice -e "CEA 16"  # 1080p 60Hz

# Read I2C EDID directly (requires i2c-tools)
sudo i2cdetect -y 2
sudo i2cdump -y 2 0x50
```

**I2C Access on Raspberry Pi:**

The Raspberry Pi exposes I2C buses that can be used to read EDID:
- I2C-0, I2C-1: Internal (used for HATs, camera, etc.)
- I2C-2: HDMI DDC (EDID at address 0x50)

Enable I2C in `/boot/firmware/config.txt`:
```ini
dtparam=i2c_arm=on
```

### macOS

**Using system_profiler:**
```bash
# Get display information
system_profiler SPDisplaysDataType

# More detailed output
ioreg -lw0 | grep IODisplayEDID
```

**Extracting and decoding EDID:**
```bash
# Get raw EDID (requires parsing ioreg output)
ioreg -lw0 -r -c "IODisplayConnect" | \
  grep -A 5 "IODisplayEDID" | \
  grep "IODisplayEDID" | \
  sed 's/.*<\(.*\)>/\1/' | \
  xxd -r -p > edid.bin

# Decode with edid-decode (install via Homebrew)
brew install edid-decode
edid-decode edid.bin
```

**Using third-party tools:**  
BetterDisplay - https://github.com/waydabber/BetterDisplay  
DisplayMenu - https://apps.apple.com/us/app/display-menu/id549083868  
SwitchResX - https://www.madrau.com/  


### Windows

**Using PowerShell:**
```powershell
# Get monitor information
Get-WmiObject WmiMonitorID -Namespace root\wmi | 
  Format-List *

# Get EDID data
Get-WmiObject -Namespace root\wmi -Class WmiMonitorDescriptorMethods |
  ForEach-Object {
    $_.WmiGetMonitorRawEEdidV1Block(0)
  }
```

**Using Command Prompt with WMIC:**
```cmd
wmic desktopmonitor get /format:list
wmic path WmiMonitorID get /format:list
```

**Registry method:**
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\DISPLAY\
```
EDID is stored as binary data under each display's `Device Parameters` key.

### IOS
This app may have some utility - https://apps.apple.com/us/app/genr8/id6751822886  
IOS UIKit provides very limited display enumearaton capabilities. - https://developer.apple.com/documentation/uikit/uiscreen  



**Third-party tools:**
- MonitorInfoView (NirSoft) - Free utility
- Custom Resolution Utility (CRU) - Includes EDID editor
- EDID Reader - Simple EDID extraction tool

## Project Status

Currently in R&D phase. This repository will document research findings, prototype designs, and eventually provide hardware schematics and software for building these tools.

## Resources and References

### Existing Tools and Projects

**Hardware EDID Tools:**
- [Raspberry Pi EDID modification tutorial](https://www.downtowndougbrown.com/2025/06/modifying-an-hdmi-dummy-plugs-edid-using-a-raspberry-pi/) - Reprogramming HDMI dummy plug I2C EEPROM
- [CEC-Tiny-Pro](https://github.com/SzymonSlupik/CEC-Tiny-Pro) - Arduino HDMI CEC sniffer requiring zero additional hardware
- [hdmi-sniff](https://github.com/AdamLaurie/hdmi-sniff) - Python DDC traffic sniffer using Bus Pirate

**Software Utilities:**
- [read-edid](http://www.polypux.org/projects/read-edid/) - get-edid and parse-edid tools for Linux
- [ddcutil](https://github.com/rockowitz/ddcutil) - Comprehensive DDC/CI monitor control utility
- [edid-checked-writer](https://github.com/galkinvv/edid-checked-writer) - Python I2C EDID writer with verification
- [edid-generate](https://github.com/librerpi/rpi-tools/blob/master/edid/edid-generate.cpp) - Programmatic EDID generation
- [linuxhw/EDID](https://github.com/linuxhw/EDID) - Database of 141,906+ decoded EDIDs

**Commercial Reference Devices:**
- HDFury Dr HDMI (1080p) and Dr HDMI 4K - Multi-bank EDID managers
- Geffen DisplayPort Detective Plus - DisplayPort EDID manager with sink sniffing

**Protocol Documentation:**
- [HDMI Protocols Demystified](https://www.prodigitalweb.com/hdmi-protocols-tmds-cec-ddc-frl-explained/) - Comprehensive technical overview
- [hdl-util/hdmi](https://github.com/hdl-util/hdmi) - FPGA HDMI implementation with pixel clock tables
- [EDID Fundamentals](https://edidcraft.com/learn) - Explanation of EDID fields


**Interesting Approaches:**
- [Lenkeng HDMI-over-IP reverse engineering](https://blog.danman.eu/reverse-engineering-lenkeng-hdmi-over-ip-extender/) - Potential platform for inline monitoring
- [Cheap HDMI capture for Linux](https://blog.benjojo.co.uk/post/cheap-hdmi-capture-for-linux) - Using Lenkeng devices with libpcap

### Key Technical Details

**I2C Addresses:**
- 0x50: EDID EEPROM
- 0x37: DDC/CI (Monitor Control)

**HDMI Pins:**
- Pin 13: CEC (Consumer Electronics Control)
- Pin 15: SCL (I2C Clock)
- Pin 16: SDA (I2C Data)
- Pin 19: HPD (Hot Plug Detect)

**Pixel Clock Frequencies:**
- 640×480 @ 60Hz: 25.2 MHz
- 1920×1080 @ 60Hz: 148.5 MHz
- 3840×2160 @ 60Hz: 297 MHz

## Contributing

This is an early-stage research project. Feedback, suggestions, and contributions are welcome as we explore solutions to HDMI connectivity challenges in live performance contexts.

---

**Target Users**: Video artists, VJs, live visual performers, AV technicians  
**Design Principles**: Cost-accessible, portable, user-friendly, open-source
