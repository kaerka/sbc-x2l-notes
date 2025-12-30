# Radxa X2L – Windows 11 Installation Notes & Root Cause Summary

## Goal
Install Windows 11 on a Radxa X2L (Alder Lake-N) as a **portable x86_64 Windows appliance** for:
- Fusion 360
- Bambu Studio / slicers
- Windows-only tools
- Remote desktop / remote gaming

Not intended as a primary workstation or gaming PC.

---

## Initial State
- NVMe SSD previously used with:
  - Ubuntu
  - ZFS
  - Encrypted root
- Windows 11 installer USB created normally
- BIOS firmware had **never been updated** (pre-v14)

---

## Symptoms Observed

### During Windows Setup on X2L
- Installer booted inconsistently
- Frequent hangs at **6–7% “Installing Windows”**
- USB installer sometimes disappeared after a hang
- EFI shell appeared due to missing boot entries
- Warm reboot often failed to detect USB devices
- Cold power cycle restored USB visibility

### After BIOS Update
- Windows kernel booted further
- Installer UI reached disk selection reliably
- Still hung consistently during **image expansion phase (6–7%)**
- Hangs occurred:
  - After deleting partitions
  - After selecting unallocated space
- Partition wipes *did succeed* despite UI hang

### NVMe in USB Enclosure
- Occasionally showed as **0 B / 0 B**
- `diskpart convert gpt` failed with:
  > fatal device hardware error
- Full power removal restored correct disk size
- Disk later showed healthy GPT, EFI, and NTFS partitions

---

## Root Causes Identified

### 1. Windows PE + SBC Firmware Instability
- Windows Setup (WinPE) is fragile on x86 SBC firmware
- The **6–7% hang** corresponds to:
  - Expanding `install.wim`
  - Sustained I/O + DMA
  - ACPI / storage re-enumeration
- Reproducible and deterministic on this platform

### 2. USB Controller / USB–NVMe Bridge State Issues
- USB devices (installer or NVMe enclosure) can wedge
- Warm reboot does **not** reset USB controller
- Cold power cycle **does**
- Enclosure firmware is stateful and can report `0 B` until reset

### 3. fTPM (Firmware TPM) Interaction
- fTPM caused:
  - Early Windows boot hangs
  - Installer instability
- Disabling fTPM improved boot reliability
- Windows 11 does **not** require TPM after bypassed install

---

## What Did *Not* Cause the Issue
- ZFS remnants
- Disk corruption
- BitLocker encryption
- Bad NVMe SSD
- User error

---

## Key Lessons Learned

- **Never trust Windows Setup GUI disk operations** on SBCs
- Avoid deleting partitions via installer UI
- DiskPart is safer, but still triggers heavy firmware interaction
- Windows PE is the weakest link on this platform
- USB bridges and SBC firmware often require **cold power resets**

---

## Final Working Strategy (Recommended)

### Install Windows *Outside* the X2L

#### Rationale
- Avoids Windows PE entirely
- Avoids USB installer instability
- Uses full Windows kernel + drivers
- Proven, deterministic method for SBCs

---

## Final Installation Method (Clean & Reliable)

### 1. Remove NVMe from X2L
- Install NVMe into a USB enclosure **or**
- Preferably into an internal M.2 slot on another PC

### 2. On a Stable Windows Machine
Option A (preferred): **Manual image application**

1. Mount Windows 11 ISO
2. Identify image index:
   ```cmd
   dism /get-wiminfo /wimfile:D:\sources\install.wim

3. dism /apply-image /imagefile:D:\sources\install.wim /index:1 /applydir:G:\

4. bcdboot G:\Windows /s S: /f UEFI



### 3. Reinstall NVMe into X2L
Boot X2L
Select Windows Boot Manager if needed
Windows performs hardware re-enumeration
Expect:
Black screen pauses
1–2 reboots
Normal OOBE behavior
