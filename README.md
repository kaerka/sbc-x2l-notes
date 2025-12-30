# Radxa X2L – Windows 11 Notes

This repository contains notes from experimenting with running **Windows 11** on a **Radxa X2L** (Intel Celeron J4125).

The goal was not to build a high-performance system, but to see whether the X2L could function as a **portable x86_64 Windows node** for occasional Windows-only applications and general experimentation.

These notes document what I observed during installation, the issues I ran into, and what ultimately made the system usable.

---

## Motivation

I wanted a small, modular Windows system that could:

- Run Windows-only applications when needed
- Be swapped in and out of a larger workflow
- Coexist alongside Linux-based SBC systems
- Serve as a portable or secondary Windows environment

This was not intended to replace a workstation or gaming system.

---

## Hardware Used

- **Board:** Radxa X2L  
- **CPU:** Intel Celeron J4125 (4 cores / 4 threads)  
- **Memory:** 8 GB  
- **Graphics:** Intel UHD Graphics 600  
- **Storage:** NVMe SSD  
- **Cooling:** Stock heatsink and fan  
- **OS:** Windows 11 (current installer at time of testing)

---

## Initial Installation Experience

During early attempts to install Windows 11, I observed a number of behaviors that made the process confusing:

- The Windows installer would consistently stop progressing around **6–7%**
- The system would sometimes reboot into the EFI shell
- USB devices would occasionally not appear after a reboot
- NVMe storage would sometimes appear unavailable until power was removed for a while
- Behavior varied depending on USB port and boot order

At the time, this made it difficult to tell whether the issue was related to firmware, storage, USB, or the installer itself.

---

## What Eventually Became Clear

After multiple attempts and incremental testing, it became clear that the system behavior was strongly correlated with **time under load**, rather than with a specific installer step.

A few observations stood out:

- Lockups would occur a few minutes after boot, regardless of what the system was doing
- The same behavior occurred during installation, first boot, login, and even idle periods
- Powering the system off for a while often allowed it to boot again

This suggested that the system was reaching a limit over time, rather than failing at a specific configuration step.

---

## Cooling and CPU Configuration

While observing fan behavior in the BIOS, I noticed that:

- Reported CPU temperatures were relatively low
- The fan would suddenly ramp up shortly before the system became unresponsive

I experimented with several BIOS settings to reduce sustained load, including:

- Disabling turbo/boost behavior
- Temporarily disabling Intel SpeedStep
- Reducing the number of active CPU cores
- Making the fan respond earlier

With these changes in place, the Windows installer was able to complete, and the system could finish setup and boot into the desktop.

Over time, features were re-enabled gradually while watching for stability.

---

## Wi-Fi Observation

At one point, removing the PCIe Wi-Fi card made early boot behavior more predictable.  
After thermal behavior was better understood and controlled, the Wi-Fi card was reinstalled and worked normally.

This appeared to be related to overall system load rather than a problem with the Wi-Fi hardware itself.

---

## System Behavior After Setup

Once the system was able to remain stable:

- Windows 11 could stay running for extended periods
- Networking functioned normally
- Steam installed successfully and reported hardware correctly
- Local Steam transfers worked at expected speeds
- Several applications installed and launched, including:
  - Bambu Studio
  - GitHub Desktop
  - Syncthing
  - Autodesk Fusion (usable, with expected performance limits)

At this point the system became predictable and usable within the constraints of the hardware.

---

## Observations and Takeaways

A few general observations from this process:

- Windows installation places sustained load on small systems
- Thermal behavior can influence symptoms that initially look unrelated
- USB and storage behavior can be affected indirectly by system load
- Small x86 SBCs are capable of running modern software when expectations are set appropriately

Nothing observed suggested faulty hardware — the system simply needed to be configured with its thermal and performance envelope in mind.

---

## Current Use

The X2L is now usable as:

- A portable Windows environment
- A companion system alongside Linux SBCs
- A way to run Windows-only tools when needed

It is not intended for heavy workloads, but it has proven useful within its intended scope.

---

### Note on CPU C-state Configuration

During testing, I observed that system stability was sensitive to the
`Max Core C State` setting in the BIOS.

The default value (`Fused`) allows the firmware to select the deepest
supported core C-states dynamically. In my case, this resulted in
repeatable system lockups shortly after boot under Windows 11.

Setting `Max Core C State` explicitly to **Core C6** eliminated these
lockups and allowed the system to run stably for extended periods,
including under sustained CPU and GPU load.

This appears to be related to idle-to-load power state transitions rather
than sustained thermal limits.

---

## Closing Notes

These notes are primarily here so that:

- I can reproduce the setup later
- Others experimenting with similar hardware have a reference
- The reasoning behind certain configuration choices is documented

This was a useful exercise in understanding how modern software behaves on small, modular systems.
