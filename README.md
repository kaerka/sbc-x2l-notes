# Radxa X2L – Windows 11 on an SBC  
## Field Notes, Failure Analysis, and Thermal Postmortem

> **TL;DR**  
> Windows 11 *does* run on the Radxa X2L (Intel Celeron J4125).  
> Early installer failures were **not firmware, storage, or USB issues** — they were **thermal lockups**.  
> Once thermals were addressed, the system became stable enough to run real workloads (Fusion, Steam, slicers).

---

## Why This Repository Exists

This repository documents an experiment that was *not* supposed to work:

- Installing **Windows 11** on a **Radxa X2L SBC**
- Using it as a **portable x86_64 Windows node**
- Running **real software**, not just benchmarks:
  - Autodesk Fusion
  - Bambu Studio
  - Steam (including local-network game transfer)
  - GitHub Desktop
  - Syncthing
  - Remote desktop (NoMachine)

This is **not** about performance.  
It is about **viability, stability, and understanding failure modes** on small x86 SBCs.

---

## Hardware Overview

| Component | Details |
|---------|--------|
| Board | Radxa X2L |
| CPU | Intel Celeron J4125 (4 cores / 4 threads, 2.0 GHz base) |
| RAM | 8 GB |
| GPU | Intel UHD Graphics 600 (DVMT, ~128 MB reported) |
| Storage | NVMe SSD |
| Cooling | Heatsink + fan (stock configuration initially) |
| OS | Windows 11 (24H2 / 2025-era installer) |

---

## Initial Symptoms (Misleading but Real)

During early installation attempts, the system exhibited:

- Windows installer hanging consistently at **6–7%**
- EFI shell appearing after reboot
- USB devices disappearing after hangs
- NVMe disks briefly reporting **0 bytes**
- Successful boots after long power-off periods
- Different behavior depending on USB port used

These symptoms strongly suggested:
- USB controller instability
- Firmware / BIOS issues
- fTPM or Secure Boot interactions
- NVMe enclosure or bridge failures

**All of these appeared plausible — and all were incorrect as root causes.**

---

## Final Root Cause

### 🔥 Thermal runaway causing hard system lockups

The board was **overheating under sustained load**, causing:

- Complete system freezes (no BSOD, no panic)
- USB controller wedging
- Apparent installer “hangs”
- Firmware fallback into EFI shell
- Recovery only after cooling down

In short:

> The system was not crashing — it was **thermally locking up**.

---

## How the Root Cause Was Identified

### 1. Time-based failures, not task-based
- Lockups occurred **2–3 minutes after boot**
- Same behavior during:
  - Windows installer
  - First boot
  - Login screen
  - Idle desktop

### 2. Fan behavior exposed the truth
- BIOS reported CPU ~32°C
- Default fan ramp was ~60°C
- Fan suddenly ramped **immediately before lockup**
- Lowering fan ramp to ~33°C caused:
  - Audible fan spike
  - Immediate freeze

➡️ On-die hotspots were significantly hotter than BIOS telemetry suggested.

### 3. Disabling CPU features restored stability
Temporary mitigations that allowed installation to complete:

- Disable Intel Turbo / Boost
- Disable Intel SpeedStep
- Disable two CPU cores
- Force aggressive fan behavior

With these changes:
- Windows 11 installation completed
- OOBE completed
- Desktop became usable

Re-enabling features too early reintroduced instability.

### 4. Wi-Fi card was a secondary amplifier
- Radxa A8 PCIe Wi-Fi card worsened instability
- Removing Wi-Fi allowed early boot
- Reinstalling Wi-Fi **after thermal fixes** worked normally

➡️ Wi-Fi increased thermal load but was not the root cause.

---

## Why the Installer Always Hung at ~7%

That phase of Windows setup involves:

- Sustained CPU usage
- Memory pressure
- Heavy NVMe writes
- USB I/O
- iGPU activity (1080p output)

In other words:

> **Maximum sustained thermal load**

The installer was not hung — the system was **thermally locked**.

---

## Validation After Thermal Stabilization

Once thermals were controlled:

- System uptime exceeded **1 hour**
- Wi-Fi stable
- Steam installed successfully
- Steam hardware survey correctly reported:
  - Intel Celeron J4125
  - 4 cores / 4 threads
  - Intel UHD 600
  - 8 GB RAM
- Applications successfully installed and launched:
  - Bambu Studio (usable)
  - GitHub Desktop
  - Syncthing
  - Autodesk Fusion (surprisingly usable)
- Steam local-network game transfer:
  - ~250–275 Mbps sustained
  - No instability during 30+ GB transfer

At this point the system crossed from **fragile** to **predictable**.

---

## What Actually Caused the Overheating

Likely contributors:

- Marginal thermal pad contact
- Insufficient heatsink mounting pressure
- Conservative fan curve defaults
- Sustained 1080p output increasing iGPU load
- Small SBC thermal mass
- Burst-heavy modern installers

Nothing was defective — the system was simply **under-cooled**.

---

## Lessons Learned

### The Big One
> Many SBC “firmware” and “installer” failures are actually **thermal failures**.

### Practical Takeaways
- BIOS temperature readings can be misleading
- Windows setup is an excellent thermal stress test
- USB instability can be a downstream thermal symptom
- CPU, GPU, Wi-Fi, and I/O stack heat very quickly
- Cold power cycles “fix” things because they cool the board

---

## Recommended Configuration

### Hardware
- Replace thermal pad with **high-quality thermal paste**
- Ensure even heatsink pressure
- Consider always-on fan or aggressive fan curve

### BIOS
- Enable SpeedStep
- Disable Turbo Boost (or cap clocks)
- Use aggressive fan ramp
- Re-enable CPU cores gradually while testing

### OS
- Reduce display resolution where possible
- Use balanced or capped power plans
- Expect UHD 600 + low VRAM to be the primary limitation

---

## Final Verdict

**Windows 11 on the Radxa X2L is viable**, with constraints:

- ✅ Stable once thermals are addressed
- ✅ Works as a portable Windows node
- ✅ Runs real productivity software
- ❌ Not a gaming machine
- ❌ Not thermally forgiving

Most importantly:

> **This validated the architecture, not the performance.**

Which means a future drop-in board upgrade (better CPU, GPU, RAM, cooling) is not just justified — it is *proven*.

---

## Status

✔ Root cause identified  
✔ Reproducible  
✔ Documented  
✔ Lessons extracted  

This repository exists so the next person does not spend two days chasing ghosts that were really just heat.


