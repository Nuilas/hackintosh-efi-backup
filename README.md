# Gigabyte Z370 AORUS Gaming 7 + RTX 3060 Hackintosh OpenCore EFI

Bu depo, **macOS 15 Sequoia** çalıştıran aşağıdaki donanım konfigürasyonunun çalışan, test edilmiş tam OpenCore EFI önyükleyicisini ve otonom yapay zeka ajanları (Hermes, Claude Code vb.) için sıfırdan kurulum reçetesini içerir.

## 🖥️ Donanım Özellikleri
- **Anakart:** Gigabyte Z370 AORUS Gaming 7 (Rev 1.0)
- **BIOS Sürümü:** F15 (28 Kasım 2021 - Resizable BAR destekli)
- **İşlemci:** Intel Core i7-8700K (Coffee Lake 6C/12T @ 3.7 - 4.7 GHz)
- **Ekran Kartı:** NVIDIA GeForce RTX 3060 12GB (Ampere GA106)
- **Monitör:** Samsung C27HG7x 27" 2560x1440 @ 144Hz (DisplayPort)
- **Hedef Disk:** Samsung 970 EVO 250GB NVMe SSD (macOS)
- **İkincil Disk:** KIOXIA Exceria Plus G3 NVMe SSD (Windows 11)

---

## ⚙️ Zorunlu BIOS Ayarları (F15)
Anakart BIOS'u fabrika ayarlarına sıfırlandıktan (Clear CMOS) sonra şu ayarlar yapılmalıdır:
1. **Peripherals:**
   - `Above 4G Decoding`: **Enabled** (RTX 3060 GOP görüntü çıkışı için zorunlu)
   - `Re-Size BAR Support`: **Auto**
   - `Initial Display Output`: **PCIe 1 Slot**
   - `XHCI Hand-off`: **Enabled**
   - `SATA Mode`: **AHCI**
2. **Chipset:**
   - `Internal Graphics` (Intel iGPU): **Disabled** (iMacPro1,1 SMBIOS ile PCI0 çakışmasını önler)
   - `VT-d`: **Disabled**
3. **BIOS Features:**
   - `CSM Support`: **Disabled** (Saf UEFI zorunlu)
4. **M.I.T.:**
   - `Extreme Memory Profile (X.M.P.)`: **Profile 1**

---

## 🧠 Kritik OpenCore Parametreleri ve Tuzaklar (Agent Playbook)

1. **SSDT-AWAC.aml Zorunluluğu (RTC Donanımsal Saat Fixi):**
   - Gigabyte F15 BIOS'unda ACPI DSDT tablosu `STAS = 0` yapar ve `Device (AWAC)` devreye girerken eski `Device (RTC)` (`PNP0B00`) kapatılır.
   - macOS Sequoia'da yerel AWAC sürücüsü olmadığı için sistem saatsiz kalır ve kernel `[AHCI] start::514:Successfully initialized AHCI controller` satırında sonsuz döngüde donar.
   - Bu repodaki derlenmiş `EFI/OC/ACPI/SSDT-AWAC.aml` tablosu `STAS = One` yaparak legacy RTC'yi açar.
   - Ayrıca `Kernel -> Quirks -> DisableRtcChecksum = True` ayarı Gigabyte anakartlarda CMOS resetlenmesini engeller.

2. **Resizable BAR & Kernel PCI Uyumluluğu:**
   - `Booter -> Quirks -> ResizeAppleGpuBars`: **0** (macOS erken kernelinin 1GB üzeri BAR adreslemesinde `PCI configuration PCI0` satırında donmasını önler).
   - `UEFI -> Quirks -> ResizeGpuBars`: **-1** (OpenCore'un UEFI açılışında donmasını önler).

3. **Yerel USB Desteği (Kext Çakışma Engelleme):**
   - Z370 yonga seti macOS Sequoia tarafından `AppleUSBXHCIPCI` ile doğal desteklenir.
   - `USBToolBox.kext`, `UTBDefault.kext` ve `XHCI-unsupported.kext` kextleri `config.plist` içinde **Disabled** kalmalıdır (aksi halde `waitForMatchingService` döngüsünde kilitlenir).

4. **Ekran Kartı Sürücüsü (NullMoth v1.15.0 Metal 3):**
   - SMBIOS: `iMacPro1,1` (Apple dahili grafik aramayı bırakıp yalnızca PCIe GPU'ya bağlanır).
   - Sürücü: NullMoth Metal 3 Stack (`NVRM.kext`, `NVAccel.kext`, `NVRMFB.kext`, `NVRMAGDC.kext` + `/Library/GPUBundles/NVMTLDriver.bundle`).
   - 144Hz Modu: `NVRMFB.kext` içindeki 22-baytlık pixel clock assembly kontrolü korunmuştur.

---

## 🤖 Otonom Ajan Sıfırdan Kurulum Prosedürü

### Adım 1: USB / EFI Hazırlama
FAT32 formatlı GPT USB diske bu depodaki `EFI` klasörü kopyalanır:
```bash
cp -R EFI /Volumes/<USB_DRIVE>/
```

### Adım 2: macOS Sequoia Kurulumu
macOS 15 Sequoia Recovery (`BaseSystem.dmg`) üzerinden hedef Samsung 970 EVO diskine:
- Format: `APFS`
- Scheme: `GUID Partition Map`
- İsim: `macOS`
seçilerek işletim sistemi kurulur.

### Adım 3: Dahili NVMe Diske EFI Kopyalama
Kurulum tamamlandıktan sonra dahili diskin EFI bölümü bağlanır ve USB'deki EFI içeriye aktarılır:
```bash
sudo diskutil mount /dev/disk0s1
sudo cp -R /Volumes/<USB_DRIVE>/EFI /Volumes/EFI/
sudo diskutil unmount /Volumes/EFI
```

### Adım 4: NullMoth Sürücüsünü Devreye Alma
Masaüstü açıldıktan sonra NullMoth 1.15.0 kextleri sisteme işlenir (`./install.sh`) ve sistem yeniden başlatılır.

---
*Yedek Tarihi: 13 Ekim 2026 - Test Edildi & Çalışır Durumda.*
