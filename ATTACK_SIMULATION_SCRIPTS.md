# ⚔️ FIRM-LOCK Attack Simulation Scripts

> **Live Demo Attack Scenarios for Hackathon Presentation**

---

## 🎯 Overview

These scripts simulate real-world firmware attacks against the FIRM-LOCK device. Use them during your live demo to show:
1. **The Attack** - How easy it is to compromise firmware
2. **The Detection** - How FIRM-LOCK immediately identifies tampering
3. **The Recovery** - How the system automatically restores trust

---

## 📁 Script Collection

### Script 1: Firmware Modification Attack

**File:** `attack_firmware_modify.sh`

```bash
#!/bin/bash
# =============================================================================
# FIRM-LOCK Attack Simulation: Firmware Modification
# =============================================================================
# This script simulates an attacker with physical access modifying device
# firmware through the SWD debug interface.
# =============================================================================

set -e

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

echo -e "${RED}"
echo "╔══════════════════════════════════════════════════════════════════════╗"
echo "║              ⚠️  FIRMWARE ATTACK SIMULATION  ⚠️                       ║"
echo "║         For Educational/Demonstration Purposes Only                  ║"
echo "╚══════════════════════════════════════════════════════════════════════╝"
echo -e "${NC}"
echo ""

# Configuration
DEVICE_ID="FL-2847-AF"
TARGET_MCU="STM32U585"
FLASH_ADDR="0x08000000"

echo -e "${BLUE}[INFO]${NC} Target Device: $DEVICE_ID"
echo -e "${BLUE}[INFO]${NC} Target MCU: $TARGET_MCU"
echo -e "${BLUE}[INFO]${NC} Flash Address: $FLASH_ADDR"
echo ""

# Step 1: Connect to device
echo -e "${YELLOW}[STEP 1]${NC} Connecting to device via SWD..."
sleep 1
echo -e "${GREEN}[OK]${NC} Debugger connected"
echo -e "${GREEN}[OK]${NC} Device ID: $DEVICE_ID"
echo -e "${GREEN}[OK]${NC} Flash size: 2MB"
echo ""

# Step 2: Read original firmware
echo -e "${YELLOW}[STEP 2]${NC} Reading original firmware..."
sleep 1
echo -e "${BLUE}[PROGRESS]${NC} Reading sector 0/256..."
sleep 0.5
echo -e "${BLUE}[PROGRESS]${NC} Reading sector 128/256..."
sleep 0.5
echo -e "${GREEN}[OK]${NC} Firmware dumped: original_$(date +%s).bin (128KB)"
echo ""

# Step 3: Analyze firmware
echo -e "${YELLOW}[STEP 3]${NC} Analyzing firmware structure..."
sleep 1
echo -e "${BLUE}[INFO]${NC} Bootloader: 0x08060000 (128KB)"
echo -e "${BLUE}[INFO]${NC} Application: 0x08000000 (128KB)"
echo -e "${BLUE}[INFO]${NC} Entry point: 0x08000200"
echo -e "${BLUE}[INFO]${NC} Signature valid: ECDSA P-256"
echo ""

# Step 4: Inject malicious payload
echo -e "${YELLOW}[STEP 4]${NC} Injecting malicious payload..."
sleep 1
echo -e "${BLUE}[INFO]${NC} Target: Application code at offset 0x4000"
echo -e "${BLUE}[INFO]${NC} Payload: Reverse shell (backdoor.bin)"
sleep 0.5
echo -e "${BLUE}[PROGRESS]${NC} Patching function at 0x08004200..."
sleep 0.5
echo -e "${BLUE}[PROGRESS]${NC} Injecting shellcode..."
sleep 0.5
echo -e "${GREEN}[OK]${NC} Payload injected successfully"
echo ""

# Step 5: Calculate new checksums
echo -e "${YELLOW}[STEP 5]${NC} Calculating new checksums..."
sleep 1
echo -e "${BLUE}[INFO]${NC} Original SHA-256: a1b2c3d4..."
echo -e "${BLUE}[INFO]${NC} Modified SHA-256: e5f6a7b8..."
echo -e "${YELLOW}[WARNING]${NC} Signature will be INVALID after flashing"
echo ""

# Step 6: Flash malicious firmware
echo -e "${YELLOW}[STEP 6]${NC} Flashing modified firmware..."
sleep 1
echo -e "${BLUE}[PROGRESS]${NC} Erasing flash sector..."
sleep 0.5
echo -e "${BLUE}[PROGRESS]${NC} Writing 128KB..."
sleep 1
echo -e "${GREEN}[OK]${NC} Flash complete"
echo ""

# Step 7: Verify and reboot
echo -e "${YELLOW}[STEP 7]${NC} Verifying and rebooting..."
sleep 1
echo -e "${GREEN}[OK]${NC} Flash verification passed"
echo -e "${GREEN}[OK]${NC} Device rebooting..."
sleep 1
echo ""

# Final status
echo -e "${RED}╔══════════════════════════════════════════════════════════════════════╗${NC}"
echo -e "${RED}║                    ⚠️  ATTACK SUCCESSFUL  ⚠️                          ║${NC}"
echo -e "${RED}╠══════════════════════════════════════════════════════════════════════╣${NC}"
echo -e "${RED}║  Device: $DEVICE_ID compromised                                      ║${NC}"
echo -e "${RED}║  Payload: Reverse shell backdoor installed                           ║${NC}"
echo -e "${RED}║  PCR[1] changed: Application measurement differs                     ║${NC}"
echo -e "${RED}║  Status: Device will FAIL next attestation                           ║${NC}"
echo -e "${RED}╚══════════════════════════════════════════════════════════════════════╝${NC}"
echo ""
echo -e "${YELLOW}[NEXT]${NC} Run attestation to detect compromise"
```

---

### Script 2: Rollback Attack

**File:** `attack_rollback.sh`

```bash
#!/bin/bash
# =============================================================================
# FIRM-LOCK Attack Simulation: Rollback Attack
# =============================================================================
# This script attempts to downgrade firmware to a vulnerable version.
# Demonstrates anti-rollback protection.
# =============================================================================

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${RED}"
echo "╔══════════════════════════════════════════════════════════════════════╗"
echo "║              ⚠️  ROLLBACK ATTACK SIMULATION  ⚠️                       ║"
echo "╚══════════════════════════════════════════════════════════════════════╝"
echo -e "${NC}"
echo ""

DEVICE_ID="FL-2847-AF"
VULNERABLE_VERSION="1.0.0"
CURRENT_VERSION="2.1.0"

echo -e "${BLUE}[INFO]${NC} Device: $DEVICE_ID"
echo -e "${BLUE}[INFO]${NC} Current Version: $CURRENT_VERSION"
echo -e "${BLUE}[INFO]${NC} Target Version: $VULNERABLE_VERSION (has known CVE)"
echo ""

# Step 1: Check anti-rollback counter
echo -e "${YELLOW}[STEP 1]${NC} Reading device security counter..."
sleep 1
echo -e "${BLUE}[INFO]${NC} Device monotonic counter: 0x00000005"
echo -e "${BLUE}[INFO]${NC} Attempting to flash version: 0x00000001"
echo ""

# Step 2: Attempt flash
echo -e "${YELLOW}[STEP 2]${NC} Attempting to flash vulnerable firmware..."
sleep 1
echo -e "${BLUE}[PROGRESS]${NC} Erasing flash..."
sleep 0.5
echo -e "${BLUE}[PROGRESS]${NC} Writing firmware_v1.0.0.bin..."
sleep 1

# Simulate detection
echo -e "${RED}[BLOCKED]${NC} Rollback detected!"
echo -e "${RED}[BLOCKED]${NC} Image security counter (0x01) < Device counter (0x05)"
echo ""

# Step 3: Show bootloader response
echo -e "${YELLOW}[STEP 3]${NC} Bootloader security response:"
echo ""
echo -e "${RED}╔══════════════════════════════════════════════════════════════════════╗${NC}"
echo -e "${RED}║  SECURITY ALERT: ROLLBACK ATTACK DETECTED                            ║${NC}"
echo -e "${RED}╠══════════════════════════════════════════════════════════════════════╣${NC}"
echo -e "${RED}║  Attempted version: 1.0.0 (security_counter=1)                       ║${NC}"
echo -e "${RED}║  Minimum allowed:   2.0.0 (security_counter=5)                       ║${NC}"
echo -e "${RED}║  Action: FLASH WRITE BLOCKED                                         ║${NC}"
echo -e "${RED}║  Event: Logged to security audit log                                 ║${NC}"
echo -e "${RED}╚══════════════════════════════════════════════════════════════════════╝${NC}"
echo ""

echo -e "${GREEN}[SUCCESS]${NC} Anti-rollback protection working correctly!"
echo -e "${BLUE}[INFO]${NC} Device remains on secure version $CURRENT_VERSION"
```

---

### Script 3: Replay Attack

**File:** `attack_replay.py`

```python
#!/usr/bin/env python3
"""
FIRM-LOCK Attack Simulation: Replay Attack

This script attempts to replay a previously captured attestation
response to fool the verifier.
"""

import requests
import json
import time
from datetime import datetime

# Colors for terminal output
RED = '\033[91m'
GREEN = '\033[92m'
YELLOW = '\033[93m'
BLUE = '\033[94m'
NC = '\033[0m'

def log_info(msg):
    print(f"{BLUE}[INFO]{NC} {msg}")

def log_warning(msg):
    print(f"{YELLOW}[WARNING]{NC} {msg}")

def log_error(msg):
    print(f"{RED}[ERROR]{NC} {msg}")

def log_success(msg):
    print(f"{GREEN}[SUCCESS]{NC} {msg}")

def main():
    print(f"{RED}")
    print("╔══════════════════════════════════════════════════════════════════════╗")
    print("║              ⚠️  REPLAY ATTACK SIMULATION  ⚠️                        ║")
    print("╚══════════════════════════════════════════════════════════════════════╝")
    print(f"{NC}\n")
    
    device_id = "FL-2847-AF"
    verifier_url = "http://verifier.local/api/v1/attest/evidence"
    
    log_info(f"Target Device: {device_id}")
    log_info(f"Verifier: {verifier_url}")
    print()
    
    # Step 1: Show captured evidence
    log_warning("STEP 1: Using previously captured attestation evidence")
    captured_evidence = {
        "device_id": device_id,
        "nonce": "0x8f3a9e2d7c1b4f6a5e8d3c9b0a2f7e4d1c6b8a3f5e9d0c7b2a4f8e1d5c9b0a3f7",
        "timestamp": 1704067200,  # Old timestamp
        "pcrs": [
            "0x7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b",
            "0xa1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2",
            "0xd4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5",
            "0x9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b"
        ],
        "signature": "0x...valid_signature...",
        "certificate": "0x...device_cert..."
    }
    
    log_info(f"Captured at: {datetime.fromtimestamp(captured_evidence['timestamp'])}")
    log_info(f"Nonce: {captured_evidence['nonce'][:20]}...")
    print()
    
    # Step 2: Attempt replay
    log_warning("STEP 2: Sending captured evidence to verifier...")
    time.sleep(1)
    
    # Simulate verifier response
    print()
    log_error("REPLAY DETECTED!")
    log_error("Nonce has already been used")
    log_error(f"Timestamp expired: {datetime.fromtimestamp(captured_evidence['timestamp'])}")
    print()
    
    # Show verifier rejection
    print(f"{RED}╔══════════════════════════════════════════════════════════════════════╗{NC}")
    print(f"{RED}║  VERIFIER RESPONSE: ATTESTATION REJECTED                             ║{NC}")
    print(f"{RED}╠══════════════════════════════════════════════════════════════════════╣{NC}")
    print(f"{RED}║  Status: FAILED                                                      ║{NC}")
    print(f"{RED}║  Reason: FRESHNESS_VIOLATION                                         ║{NC}")
    print(f"{RED}║  Details: Nonce has expired (max age: 60 seconds)                    ║{NC}")
    print(f"{RED}║  Action: Device status unchanged                                     ║{NC}")
    print(f"{RED}╚══════════════════════════════════════════════════════════════════════╝{NC}")
    print()
    
    log_success("Replay attack prevented by freshness check!")
    log_info("Each attestation requires a unique, time-bound nonce")

if __name__ == "__main__":
    main()
```

---

## 🎬 Demo Execution Flow

### Phase 1: Normal Operation (30 seconds)

```bash
# Terminal 1: Start dashboard
cd firmlock-dashboard
python3 -m http.server 8080
# Open browser to http://localhost:8080

# Terminal 2: Show device is healthy
echo "Device FL-2847-AF is running normally"
echo "All PCRs match golden values"
echo "Last attestation: PASSED"
```

### Phase 2: The Attack (45 seconds)

```bash
# Run attack script
./attack_firmware_modify.sh

# Dashboard should show:
# - Red LED indicator
# - PCR[1] mismatch
# - Device status: COMPROMISED
```

### Phase 3: Detection (30 seconds)

```bash
# Trigger attestation from dashboard
# Or run:
curl -X POST http://device.local/api/attest

# Expected output:
{
  "result": "FAIL",
  "reason": "PCR_MISMATCH",
  "details": {
    "pcr_index": 1,
    "expected": "0xa1b2...",
    "received": "0xe5f6..."
  },
  "action": "QUARANTINE"
}
```

### Phase 4: Recovery (45 seconds)

```bash
# Trigger recovery from dashboard
# Or run:
curl -X POST http://device.local/api/recover

# Expected output:
{
  "result": "SUCCESS",
  "action": "GOLDEN_IMAGE_RESTORED",
  "new_firmware": "v2.1.0-factory",
  "status": "HEALTHY"
}
```

---

## 📊 Expected Dashboard States

### State 1: Healthy

```
┌─────────────────────────────────────────┐
│  🟢 HEALTHY                             │
│                                         │
│  PCR[0]: ✓ Match (Bootloader)           │
│  PCR[1]: ✓ Match (Application)          │
│  PCR[2]: ✓ Match (Configuration)        │
│  PCR[3]: ✓ Match (Identity)             │
│                                         │
│  Last Attestation: PASS                 │
└─────────────────────────────────────────┘
```

### State 2: Compromised

```
┌─────────────────────────────────────────┐
│  🔴 COMPROMISED                         │
│                                         │
│  PCR[0]: ✓ Match (Bootloader)           │
│  PCR[1]: ✗ MISMATCH! (Application)      │
│  PCR[2]: ✓ Match (Configuration)        │
│  PCR[3]: ✓ Match (Identity)             │
│                                         │
│  Last Attestation: FAIL                 │
│  Action: QUARANTINED                    │
└─────────────────────────────────────────┘
```

### State 3: Recovered

```
┌─────────────────────────────────────────┐
│  🟢 RESTORED                            │
│                                         │
│  PCR[0]: ✓ Match (Bootloader)           │
│  PCR[1]: ✓ Match (Application)          │
│  PCR[2]: ✓ Match (Configuration)        │
│  PCR[3]: ✓ Match (Identity)             │
│                                         │
│  Recovery: SUCCESS                      │
│  Source: Golden Image                   │
└─────────────────────────────────────────┘
```

---

## 🎤 Presenter Script Integration

### When Running Attack Script:

> "This is an attacker with physical access. They're using a standard SWD debugger - the same tool developers use every day. Watch as they read the firmware, inject malicious code, and flash it back to the device."

### When Detection Occurs:

> "The device has been compromised. But here's the key - the secure element still has the original private key. When we trigger attestation, the device signs the NEW PCR values. The verifier immediately sees the mismatch."

### When Recovery Completes:

> "And now for the magic - automatic recovery. The device has a factory-fresh 'golden image' stored in protected flash. We restore from that, and within 15 seconds, the device is back to a trusted state."

---

**Remember: Practice these scripts until they're muscle memory. The demo should feel effortless.**

---

*Attack simulations for educational purposes only. Always follow responsible disclosure.*
