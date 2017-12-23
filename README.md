![NexMon logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/nexmon.png)

# Nexmon Debugger

To analyze the FullMAC firmware running on the ARM microcontroller in the BCM4339 Wi-Fi chip,
we created a debugger in software that directly accesses the hardware registers of the ARM
Debugging core. As we do not have access to the JTAG port, we generate exceptions whenever a
breakpoint or watchpoint is triggered. We handle those exceptions in our firmware patch and
can then continue with the execution of the firmware code.

In our example, we set a breakpoint at the `printf` function:
```
dbg_set_breakpoint_for_addr_match(0, 0x126f0); // trigger on printf
```
When it triggers, the function `handle_pref_abort_exception` is called. There, we check if 
register `R1` points to the string `sdpcmd_dpc`. If the condition matches, we print a debug 
message and activate single-step debugging. To this end, we set the breakpoint to trigger on
a mismatch of the address of the current program counter. Then we can decide to either trigger 
on a mismatch of the current program counter address again for single stepping or reset the 
breakpoint to the original address of the `printf` funciton.

Additionally, we set a memory watchpoint on the address of the string 
`%s: Broadcom SDPCMD CDC driver`:
```
dbg_set_watchpoint_for_addr_match(0, 0x1FC2A4); // trigger on "%s: Broadcom SDPCMD CDC driver" string
```
Whenever, code reads or writes to this address, a data abort exception is triggered that is
handled by the `handle_data_abort_exception` function. In this function, we print the address
of the instruction accessing monitored memory location. We also disable the watchpoint and set
a new breakpoint to trigger on an address mismatch of the current program counter to advance 
by one instruction. The breakpoint handler reactivates the watchpoint.

The output observed on the console is as follows:
```
start=0x001eb5d8 len=0x00000800

RTE (USB-SDIO-CDC) 6.37.32.RC23.34.43 (r639704) on BCM4339 r1 @ 37.4/161.3/161.3MHz
000000.010    WP hit pc=00012b3a
000000.013    WP hit pc=00012b3a
000000.010 sdpcmdcdc0: Broadcom SDPCMD CDC driver
000000.141 reclaim section 0: Returned 31688 bytes to the heap
000000.189 nexmon_ver: 63fb-dirty-14
000000.192 wl_nd_ra_filter_init: Enter..
000000.196 TCAM: 256 used: 198 exceed:0
000000.200    WP hit pc=000126c2
000000.203 reclaim section 1: Returned 71844 bytes to the heap
000000.208    BP0 step 0: pc=000126f0 *r1=sdpcmd_dpc
000000.213    BP0 step 1: pc=000126f2
000000.216    BP0 step 2: pc=000126f4
000000.219    BP0 step 3: pc=000126f6
000000.223    BP0 step 4: pc=000126fa
000000.226    BP0 single-stepping done
000000.229 sdpcmd_dpc: Enable
000000.234 wl0: wlc_bmac_ucodembss_hwcap: Insuff mem for MBSS: templ memblks 192 fifo memblks 259
000000.249 wl0: wlc_enable_probe_req: state down, deferring setting of host flags
000000.295 wl0: wlc_enable_probe_req: state down, deferring setting of host flags
000000.303 wl0: wlc_enable_probe_req: state down, deferring setting of host flags
```

`0x12b3a` is an instruction in the `vsnprintf` function called during the `printf` call writing to 
the chip's console. `0x126c2` is an instruction in the `memset` function, called after initialization
when parts of the code used during initialization is freed and assigned to the heap.

# Extract from our License

Any use of the Software which results in an academic publication or
other publication which includes a bibliography must include
citations to the nexmon project (1):

1. "Matthias Schulz, Daniel Wegemer and Matthias Hollick. Nexmon:
    The C-based Firmware Patching Framework. https://nexmon.org"

# Getting Started

To compile the source code, you are required to first checkout a copy of the original nexmon
repository that contains our C-based patching framework for Wi-Fi firmwares. That you checkout
this repository as one of the sub-projects in the corresponding patches sub-directory. This 
allows you to build and compile all the firmware patches required to repeat our experiments.
The following steps will get you started on Xubuntu 16.04 LTS:

1. Install some dependencies: `sudo apt-get install git gawk qpdf adb`
2. **Only necessary for x86_64 systems**, install i386 libs: 

  ```
  sudo dpkg --add-architecture i386
  sudo apt-get update
  sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386
  ```
3. Clone the nexmon base repository: `git clone https://github.com/seemoo-lab/nexmon.git`.
4. Download and extract Android NDK r11c (use exactly this version!).
5. Export the NDK_ROOT environment variable pointing to the location where you extracted the ndk so that it can be found by our build environment.
6. Navigate to the previously cloned nexmon directory and execute `source setup_env.sh` to set a couple of environment variables.
7. Run `make` to extract ucode, templateram and flashpatches from the original firmwares.
8. Navigate to utilities and run `make` to build all utilities such as nexmon.
9. Attach your rooted Nexus 5 smartphone running stock firmware version 6.0.1 (M4B30Z, Dec 2016).
10. Run `make install` to install all the built utilities on your phone.
11. Navigate to patches/bcm4339/6_37_34_43/ and clone this repository: `git clone https://github.com/seemoo-lab/wintech2017_nexmon_ping_offloading.git`
12. Enter the created subdirectory wisec2017_nexmon_jammer and run `make install-firmware` to compile our firmware patch and install it on the attached Nexus 5 smartphone.

# References

* Matthias Schulz, Daniel Wegemer and Matthias Hollick. **Nexmon: The C-based Firmware Patching Framework**. https://nexmon.org

[Get references as bibtex file](https://nexmon.org/bib)

# Contact

* [Matthias Schulz](https://seemoo.tu-darmstadt.de/mschulz) <mschulz@seemoo.tu-darmstadt.de>

# Powered By

## Secure Mobile Networking Lab (SEEMOO)
<a href="https://www.seemoo.tu-darmstadt.de">![SEEMOO logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/seemoo.png)</a>
## Networked Infrastructureless Cooperation for Emergency Response (NICER)
<a href="https://www.nicer.tu-darmstadt.de">![NICER logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/nicer.png)</a>
## Multi-Mechanisms Adaptation for the Future Internet (MAKI)
<a href="http://www.maki.tu-darmstadt.de/">![MAKI logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/maki.png)</a>
## Technische Universität Darmstadt
<a href="https://www.tu-darmstadt.de/index.en.jsp">![TU Darmstadt logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/tudarmstadt.png)</a>
