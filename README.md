# SyncOrSwim
HDMI troubleshooting tools for VJs and Video Artists# HDMI Sniffer Project

## Problem Statement

Traveling video artists and VJs frequently encounter HDMI connectivity issues at venues that are difficult to diagnose and troubleshoot in real-time. These issues manifest in two critical ways:

1. **Initial Connection Failures**: Unable to establish a stable HDMI connection with venue equipment (projectors, video routers, switchers) due to EDID incompatibilities or negotiation failures
2. **Connection Dropouts During Performance**: HDMI connections that fail mid-performance due to timing sync issues, signal retraining, or other protocol-level problems

Currently, video artists have limited visibility into these problems and lack portable, affordable tools to diagnose HDMI issues in the field. This project aims to develop open-source, cost-accessible hardware tools to monitor and diagnose HDMI connections.

## Proposed Solution

Two complementary devices built from off-the-shelf components:

### Device 1: EDID Reader
**Purpose**: Pre-connection diagnostics  
**Function**: Plugs into venue HDMI equipment (projectors, routers, etc.) to read and display supported EDID modes before the performance  
**Use Case**: Allows VJs to verify compatibility and configure their output settings, or copy settings to an EDID Dummy

### Device 2: Inline Connection Monitor  
**Purpose**: Real-time connection monitoring  
**Function**: Sits inline between video source and display to monitor HDMI signal health during performance. Passes logs to external device. 
**Use Case**: Logs timing, sync, and retraining events to diagnose intermittent connection issues

## R&D Areas

### EDID Software
- Linux
- Mac
- Windows

### Hardware Architecture
- [ ] **EDID Reader**: Raspberry Pi with I2C access for reading EDID data
- [ ] **Inline Monitor**: Research HDMI pass-through or splitter-based solutions for signal monitoring
- [ ] Evaluate whether devices should be separate or combinable (leaning toward separate, because may need to monitor more than 1 HDMI connection
- [ ] Component sourcing to maintain cost accessibility

### Display & Interface
- [ ] **EDID Reader**: On-screen display showing supported resolutions, refresh rates, and modes
- [ ] **Inline Monitor**: On-screen display + data logging capability (SD card, USB, or network)
- [ ] Standalone operation (no computer required for monitoring)

### HDMI Protocol Analysis
- [ ] Clock signal synchronization and drift detection
- [ ] Signal retraining event detection and logging
- [ ] HDCP handshake monitoring
- [ ] Hot-plug detect (HPD) event tracking
- [ ] DDC/EDID transaction monitoring
- [ ] Cable integrity and signal quality metrics

### Common Failure Modes to Address
- [ ] Resolution negotiation failures
- [ ] HDCP handshake issues
- [ ] Signal degradation over cable length
- [ ] Timing/sync drift during extended operation
- [ ] Vendor-specific projector/router compatibility issues
- [ ] Intermittent connection dropouts

### Data Logging & Analysis
- [ ] What metrics to log for the inline monitor
- [ ] Log format and storage requirements
- [ ] Visualization of connection health over time

## Project Status

Currently in R&D phase. This repository will document research findings, prototype designs, and eventually provide hardware schematics and software for building these tools.

---

**Target Users**: Video artists, VJs, live visual performers, AV technicians  
**Design Principles**: Cost-accessible, portable, user-friendly, open-source
