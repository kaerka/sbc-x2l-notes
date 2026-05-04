# Radxa X2L — Windows 11 field notes

Hands-on notes from running **Windows 11** on a **Radxa X2L** (Intel Celeron **J4125**, 8 GB RAM, Intel UHD Graphics 600, NVMe). The aim was a **small, standard x86_64 Windows node** for occasional Windows-only applications, light CAD/slicer work, and remote use — not a primary workstation.

**Audience:** Technical readers evaluating similar hardware for labs, edge prototypes, or companion systems next to Linux SBCs.

**Disclaimer:** These are individual observations on one board and BIOS revision. They are not a Radxa or Microsoft endorsement, and your firmware, peripherals, or workload may behave differently.

---

## Scope of this document

| In scope | Out of scope (for now) |
|----------|-------------------------|
| Windows 11 installation pitfalls, thermal/power behavior, BIOS-related mitigations | Step-by-step OEM imaging or enterprise deployment guides |
| Practical workload evidence (screenshots below) | Formal benchmarking or certification |

**Related work:** A separate write-up may cover **RHEL** or **ArgusOS** on a second unit later; this README is intentionally focused on **Windows 11** only.

---

## Motivation

- Run Windows-only tools when needed without dedicating a full desktop  
- Swap a compact module in and out of a larger workflow  
- Sit beside Linux-based boards while staying on a mainstream Windows stack  
- Optional remote access (e.g. NoMachine) for operations from another machine  

---

## Hardware configuration

| Component | Detail |
|-----------|--------|
| Board | Radxa X2L |
| CPU | Intel Celeron J4125 (4 cores / 4 threads, no hyper-threading) |
| Memory | 8 GB |
| Graphics | Intel UHD Graphics 600 |
| Storage | NVMe SSD |
| Cooling | Stock heatsink and fan |
| OS tested | Windows 11 (installer current at time of testing) |

---

## Installation experience (summary)

Early Windows setup was unreliable:

- Installer often stalled around **6–7%**
- Occasional reboot into **EFI shell**
- **USB** devices sometimes missing after reboot until a cold power cycle
- **NVMe** occasionally not visible until power had been removed for a period  
- Behavior varied with **USB port** and **boot order**

Initially it was unclear whether firmware, storage, USB, or the installer was at fault.

---

## Root cause direction: time under load

Repeated trials suggested correlation with **time under load**, not a single wizard step:

- Lockups appeared minutes after boot during install, first boot, login, or idle stretches  
- Cooling adjustments changed outcomes more than rearranging installer steps  
- A longer powered-off interval often allowed another boot attempt  

That pointed to **thermal / power-management envelope** symptoms rather than a single “bad ISO” failure mode.

---

## BIOS-oriented mitigations (no photos in this set)

In firmware, CPU temperature readings stayed modest while the fan sometimes **ramped sharply shortly before** an apparent hang — consistent with hitting a limit indirectly rather than a steady high temperature readout.

Experiments included reducing sustained CPU demand and changing fan responsiveness (e.g. turbo/boost tweaks, SpeedStep trials, fewer active cores, earlier fan ramps). With those in place, setup **completed** and the desktop became reachable; settings were **relaxed gradually** while watching stability.

**Wi‑Fi:** Removing the PCIe Wi‑Fi card briefly made early boot more predictable; after thermal behavior was understood, the card was reinstalled and worked normally — suggesting **overall platform stress**, not necessarily defective Wi‑Fi hardware.

---

## Critical BIOS finding: Max Core C-State

Stability under Windows 11 was sensitive to **`Max Core C State`**:

- Default (**`Fused`**) allowed deep dynamic C-states and produced **repeatable lockups shortly after boot**.  
- Setting **`Max Core C State`** explicitly to **`Core C6`** removed those lockups and allowed **extended runs**, including sustained CPU and GPU load.

This aligns better with **idle ↔ active power-state transitions** than with sustained overheating alone.

---

## Verified behavior after stabilization

With BIOS tuned (including **Max Core C State → Core C6**, turbo enabled per notes), the system became **predictable**:

- Extended uptime on Windows 11  
- Networking normal  
- **Steam** recognized hardware correctly  
- Local Steam transfers behaved as expected on Wi‑Fi (see screenshots)  
- Examples that ran acceptably within hardware limits: **Bambu Studio**, **GitHub Desktop**, **Syncthing**, **Autodesk Fusion** (usable with realistic expectations)

---

## Practical roles (aligned with enterprise-style use)

- Portable or secondary Windows environment  
- Companion to Linux SBCs  
- Light CAD, slicing, scripting, documentation, or remote-first workflows  

**Differentiator:** **GPIO on x86** is relatively uncommon; for teams that want x86 compatibility and tooling without ARM BSP coupling, platforms like this can be interesting for **prototyping and field kits** (Linux side not covered here).

---

## Multi-unit strategy

Keeping **one unit** as a known-good **Windows** baseline and **another** on **Fedora** (or later **RHEL / ArgusOS**) avoids configuration churn and preserves apples-to-apples comparisons.

---

## Screenshots and figures

Unless noted, captures were taken over **NoMachine** from a host named **Radxa-x2l-win11**. Paths use URL-encoded spaces (`%20`) so links work in GitHub-style viewers.

### Figure 1 — Heavy installer load (Autodesk + Task Manager)

Typical sustained CPU use during a large installer on the J4125 (~96% CPU in this moment).

![Autodesk installer at ~20% with Task Manager showing high CPU](./Screenshot%202025-12-30%20131223.png)

### Figure 2 — Autodesk Fusion installer initializing

Fusion installer splash with Task Manager (~86% CPU, ~72% RAM).

![Fusion initializing with Performance tab](./Screenshot%202025-12-30%20131614.png)

### Figure 3 — Autodesk Fusion session (CAD workload)

Demonstrates interactive Fusion use on the board after stabilization (design session; UI shows normal Windows 11 desktop integration).

![Fusion 3D modeling session](./Screenshot%202025-12-30%20132306.png)

### Figures 4–5 — Installer progression and settled CPU

Mid-install (~69%) and later (~88%) with detailed CPU specification visible in Task Manager.

![Autodesk installer ~69% with Task Manager](./Screenshot%202025-12-30%20132415.png)

![Autodesk installer ~88% with Task Manager](./Screenshot%202025-12-30%20132737.png)

### Figures 6–7 — Developer toolchain install (VS Code)

Shows VS Code setup extracting files alongside Task Manager — representative of software rollout stress on 8 GB RAM.

![VS Code setup ~45% with Task Manager](./Screenshot%202025-12-30%20133413.png)

![VS Code setup ~80% with Task Manager](./Screenshot%202025-12-30%20133435.png)

### Figure 8 — Documentation / IDE workload

VS Code editing Markdown project notes over remote desktop — illustrative of light technical writing or repo maintenance on the same hardware.

![VS Code remote session](./Screenshot%202025-12-30%20134210.png)

### Figures 9–12 — Steam “System Information”: X2L vs reference workstation

The following images are **composite Steam hardware summaries**. In each pair, **the left window is the Radxa X2L (this Windows 11 build)**; **the right window is a separate high-end workstation** captured for **expectations calibration** (throughput, RAM, GPU features — not a performance contest).

**Figure 9 — Storage footprint**

![Steam storage summary: X2L left vs workstation right](./Screenshot%202025-12-30%20135428.png)

**Figure 10 — Display and system RAM**

![Steam video/RAM summary: X2L left vs workstation right](./Screenshot%202025-12-30%20135500.png)

**Figure 11 — CPU capability flags**

Left: **J4125** (no AVX / AVX2). Right: desktop-class CPU with wider SIMD support — relevant when selecting binaries or ML tooling.

![Steam CPU summary: J4125 left vs Core i9 right](./Screenshot%202025-12-30%20135700.png)

**Figure 12 — Integrated graphics vs discrete GPU**

Left: **Intel UHD Graphics 600** at modest resolution. Right: discrete NVIDIA GPU — sets realistic bounds for local rendering vs remote/streamed workloads.

![Steam GPU summary: UHD 600 left vs RTX class right](./Screenshot%202025-12-30%20135753.png)

### Figures 13–14 — Local Steam transfer performance (Wi‑Fi)

Steam **local network** game transfer (~250 Mbps class in these snapshots) with Task Manager showing concurrent CPU load — useful evidence that **Wi‑Fi and disk path** behave sanely for large payloads once the platform is stable.

![Steam LAN transfer with Task Manager](./Screenshot%202025-12-30%20141054.png)

![Steam LAN transfer sustained](./Screenshot%202025-12-30%20141355.png)

---

## Supporting artifacts in this folder

- **`Radxa X2L – Windows 11 Installation.md`** — Longer symptom timeline and BIOS-update narrative (companion to this README).  
- **`HWMonitor-X2L.txt`**, **`RADXA-X2L-WIN11.txt`** — Raw sensor / text dumps from testing if you need numeric traces beyond the screenshots.

---

## Takeaways for readers standardizing on this class of hardware

1. Treat **installation stalls** as potential **platform envelope** issues — validate cooling, fan curves, and **core C-state** limits early.  
2. Expect **AVX‑lite** CPUs; prefer stacks that do not assume AVX2 everywhere.  
3. Use **remote desktop** when appropriate — integrated graphics is sufficient for tooling, not for workstation-class 3D.  
4. Keep **golden images per role** (Windows vs Linux) when comparing boards across a team.

---

## Closing

These notes exist so setups can be **reproduced**, choices **explained**, and others experimenting with similar kits have a **reference path**. Feedback and divergent BIOS revisions are welcome — capture your own screenshots against this outline if you extend the doc for **RHEL**, **ArgusOS**, or fleet deployment later.
