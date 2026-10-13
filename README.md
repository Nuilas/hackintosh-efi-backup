# Hackintosh EFI Backup

**Date:** October 13, 2026  
**BIOS Version:** F15 (Gigabyte Z390 AORUS PRO)  
**OpenCore:** Latest with NullMoth 1.15.0  

## Hardware
- **CPU:** Intel Core i9-9900K (Coffee Lake)
- **GPU:** NVIDIA RTX (with NullMoth for Sequoia)
- **RAM:** 32GB DDR4
- **Storage:** NVMe SSD

## Boot Performance
- BIOS POST → OpenCore: ~8-12 seconds (F15 memory training)
- OpenCore → macOS kernel handoff: ~180 seconds
- Total boot time: ~3 minutes

## Key Features
- **NullMoth 1.15.0:** NVIDIA driver compatibility for macOS Sequoia
- **ReBAR Support:** ResizeAppleGpuBars = 0, ResizeGpuBars = -1
- **Fast Boot:** Can be enabled in BIOS to skip memory training
- **AMFI Pass:** Disabled for kext loading

## Boot Arguments
```
-v debug=0x100 keepsyms=1 nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth amfi_get_out_of_my_way=0x1 amfi=0x80
```

## Restore Instructions
1. Format USB as FAT32 with GPT
2. Copy EFI folder to USB root
3. Boot from USB
4. Copy EFI to internal drive's EFI partition

## Notes
- This backup includes the working configuration after F15 BIOS update
- Memory training adds ~8-12s to POST time (can be reduced with Fast Boot)
- All kexts and ACPI patches are included
