#### Intel ME TXE Firmware Cleaner

A Python script able to modify an Intel ME firmware image with the final purpose of reducing its ability to interact with the system.

#### Solus Usage

### 1. **Hardware Exposure**
- IME operates independently of OS (Linux) and boots before the main CPU.
- Claims about capabilities (TCP/IP connections, hardware access) are technically accurate but require context:
  - IME *can* access hardware at firmware level, but **practical exploitation requires physical access or high-level firmware vulnerabilities**.

### 2. **Can You Be Protected?**
#### **Disabling IME: Limited Options**
- **Full disable is impossible** on consumer hardware
- **Partial mitigations exist** (with trade-offs):
  - **"Neutering"**: Using tools like this repo to remove IME functionality from firmware.
    → *Risks*: May brick your system, void warranty, and break features (Thunderbolt, Wake-on-LAN, etc).
  - **OS-level disable**:  
    ```bash
    # In Linux (Solus), block IME interface:
    echo "blacklist intel_mei" | sudo tee /etc/modprobe.d/blacklist-ime.conf
    ```
    → *Limitation*: Only stops OS communication with IME; IME still runs at hardware level.
  - **BIOS settings**: Some enterprise boards allow disabling IME (e.g., "Intel ME Mode" → "Disabled").
    → *Reality*: Consumer-grade laptops **lacks this option** (common in gaming/workstation devices).

#### **AMD PSP Comparison**
- AMD PSP has similar architecture but **even fewer disable options** on consumer hardware.
- BIOS "disable PSP" settings (on some motherboards) often only disable *communication* with PSP, not the processor itself.

### 3. **Your Actual Risk Level**
- **Low for average users**: IME exploits require:
  - Physical access *or*  
  - Pre-existing root-level compromise *or*  
  - Rare firmware vulnerabilities (e.g., [CVE-2017-5705](https://nvd.nist.gov/vuln/detail/CVE-2017-5705)).
- **High-risk scenarios**: Only relevant if targeted by nation-state actors (e.g., journalists, activists).
- **Linux advantage**: Solus reduces attack surface vs. Windows/Mac, but **does not bypass IME's hardware access**.

### 4. **Practical Recommendations**
- **For most users**:  
  - Keep firmware updated (check [Intel's downloads](https://www.intel.com/content/www/us/en/download/682431/intel-management-engine-drivers-for-windows-10-and-windows-11.html)). 
  - Disable unused features (e.g., Intel AMT in BIOS if available).  
  - Use hardware kill switches for wi-fi, bluetooth, gps/location, and mic/camera (physical mitigation).  
- **For high-security needs**:  
  - Consider **Libreboot**-compatible hardware (e.g., older ThinkPads with IME physically removed).  
  - **Purism Librem laptops** with deblobbed firmware.  
  - Avoid Intel/AMD consumer hardware entirely (opt for RISC-V or ARM-based systems).  

## Intel ME

Intel ME is a co-processor integrated in all post-2006 Intel boards, which is
the base hardware for many Intel features like Intel AMT, Intel Boot Guard,
Intel PAVP and many others. To provide such features, it requires full access to
the system, including memory (through DMA) and network access (transparent to
the user).

Unlike many other firmware components, the Intel ME firmware can't be neither
disabled nor reimplemented, as it is tightly integrated in the boot process and
it is signed.

This poses an issue both to the free firmware implementations like [coreboot](
https://www.coreboot.org/), which are forced to rely on a proprietary, obscure
and always-on blob, and to the privacy-aware users, who are reasonably worried
about such firmware, running on the lowest privilege ring on x86.

## What can be done

Before Nehalem (ME version 6, 2008/2009) the ME firmware could be removed
completely from the flash chip by setting a couple of bits inside the flash
descriptor, effectively disabling it.

Starting from Nehalem the Intel ME firmware can't be removed anymore: without a
valid firmware the PC shuts off forcefully after 30 minutes, probably as an
attempt to enforce the Intel Anti-Theft policies.

However, while Intel ME can't be turned off completely, it is still possible to
modify its firmware up to a point where Intel ME is active only during the boot
process, effectively disabling it during the normal operation, which is what
_me\_cleaner_ tries to accomplish.

## Platform support

_me\_cleaner_ currently works on [most of the Intel platforms](
https://github.com/corna/me_cleaner/wiki/me_cleaner-status); while this doesn't
mean it works on all the boards (due to the different firmware implementations),
it has been proven quite reliable on a great number of them.

## Usage

_me\_cleaner_ should handle all the steps necessary to the modification of an
Intel ME firmware with the command:

      $ python me_cleaner.py -S -O modified_image.bin original_dump.bin

However, obtaining the original firmware and flashing back the modified one is
usually not trivial, as the Intel ME firmware region is often non-writable from
the OS (and it's not a safe option anyways), requiring the use of an external
SPI programmer.

## Results

For generation 1 (before Nehalem, ME version <= 5) this tool removes the whole
ME firmware and disables it completely.

For generation 2 (Nehalem-Broadwell, ME version between 6 and 10) this tool
removes almost everything, leaving only the two fundamental modules needed for
the correct boot, `ROMP` and `BUP`. The firmware size is reduced from 1.5 MB
(non-AMT firmware) or 5 MB (AMT firmware) to ~90 kB.

For generation 3 (from Skylake onwards, ME version >= 11) the ME subsystem and
the firmware structure have changed, requiring substantial changes
in _me\_cleaner_. The fundamental modules required for the correct boot are now
four (`rbe`,  `kernel`, `syslib` and `bup`) and the minimum firmware size is
~300 kB (from the 2 MB of the non-AMT firmware and the 7 MB of the AMT one).

On some boards the OEM firmware fails to boot without a valid Intel ME firmware;
in the other cases the system should work with minor inconveniences (like longer
boot times or warning messages) or without issues at all.

Obviously, the features provided by Intel ME won't be functional anymore after
the modifications.

## Documentation

The detailed documentation about the working of _me\_cleaner_ can be found on
the page ["How does it work?" page](
https://github.com/corna/me_cleaner/wiki/How-does-it-work%3F).

Various guides and tutorials are available on the Internet, however a good
starting point is the ["How to apply me_cleaner" guide](
https://github.com/corna/me_cleaner/wiki/How-to-apply-me_cleaner).

