
# LTESniffer - An Open-source LTE Downlink/Uplink Eavesdropper

**LTESniffer** is An Open-source LTE Downlink/Uplink Eavesdropper 

It first decodes the Physical Downlink Control Channel (PDCCH) to obtain the Downlink Control Informations (DCIs) and Radio Network Temporary Identifiers (RNTIs) of all active users. Using decoded DCIs and RNTIs, LTESniffer further decodes the Physical Downlink Shared Channel (PDSCH) and Physical Uplink Shared Channel (PUSCH) to retrieve uplink and downlink data traffic.

LTESniffer supports an API with three functions for security applications and research. Many LTE security research assumes
a passive sniffer that can capture privacy-related packets on the air. However, non of the current open-source sniffers satisfy their requirements as they cannot decode protocol packets in PDSCH and PUSCH. We developed a proof-of-concept security API that supports three tasks that were proposed by previous works: 1) Identity mapping, 2) IMSI collecting, and 3) Capability profiling.

Please refer to our [paper][paper] for more details.

## LTESniffer in layman's terms
LTESniffer is a tool that can capture the LTE wireless messages that are sent between a cell tower and smartphones connected to it. LTESniffer supports capturing the messages in both directions, from the tower to the smartphones, and from the smartphones back to the cell tower.

LTESniffer **CANNOT DECRYPT** encrypted messages between the cell tower and smartphones. It can be used for analyzing unencrypted parts of the communication between the cell tower and smartphones. For example, for encrypted messages, it can allow the user to analyze unencrypted parts, such as headers in MAC and physical layers. However, those messages sent in plaintext can be completely analyzable. For example, the broadcast messages sent by the cell tower, or the messages at the beginning of the connection are completely visible.

## Ethical Consideration

The main purpose of LTESniffer is to support security and analysis research on the cellular network. Due to the collection of uplink-downlink user data, any use of LTESniffer must follow the local regulations on sniffing the LTE traffic. We are not responsible for any illegal purposes such as intentionally collecting user privacy-related information.

## Features
### New Update v2.1.0
- Supports recording IQ raw data of subframes to file. Please refer to `LTESniffer-record-subframe` branch and its [README][capture-readme] for more details.
- Supports offline decoding using recorded files ([README][capture-readme]).
- Enable API in the downlink mode (only apply for identity collecting and mapping API)
### New Update v2.0.0
- Supports two USRP B-series for uplink sniffing mode. Please refer to `LTESniffer-multi-usrp` branch and its [README][multi-readme] for more details.
- Fixed some bugs.

LTESniffer is implemented on top of [FALCON][falcon] with the help of [srsRAN][srsran] library. LTESniffer supports:
- Real-time decoding LTE uplink-downlink control-data channels: PDCCH, PDSCH, PUSCH
- LTE Advanced and LTE Advanced Pro, up to 256QAM in both uplink and downlink
- DCI formats: 0, 1A, 1, 1B, 1C, 2, 2A, 2B
- Transmission modes: 1, 2, 3, 4
- FDD only
- Maximum 20 MHz base station. 
- Automatically detect maximum UL/DL modulation schemes of smartphones (64QAM/256QAM on DL and 16QAM/64QAM/256QAM on UL)
- Automatically detect physical layer configuration per UE.
- LTE Security API: RNTI-TMSI mapping, IMSI collecting, UECapability Profiling.

## Hardware and Software Requirement
### OS Requirement
Currently, LTESniffer works stably on Ubuntu 18.04/20.04/22.04.
It also builds and runs on **macOS (Apple Silicon)** for downlink sniffing with
SoapySDR devices such as HackRF and PlutoSDR — see
[Building on macOS (Apple Silicon)](#building-on-macos-apple-silicon).

### Hardware Requirement
Achieving real-time decoding of LTE traffic requires a high-performance CPU with multiple physical cores, especially during peak hours when the base station has many active users. LTESniffer successfully achieved real-time decoding when deployed on an Intel i7-9700K PC, decoding traffic from a base station with 150 active users.

**The following hardware is recommended**
- Intel i7 CPU with at least 8 physical cores
- At least 16Gb RAM
- 256 Gb SSD storage
### SDR
LTESniffer requires different SDR for its uplink and downlink sniffing modes.

To sniff only downlink traffic from the base station, LTESniffer is compatible with most SDRs that are supported by the srsRAN library (for example, USRP or BladeRF). 
The SDR should be connected to the PC via a USB 3.0 port. Additionally, it should be equipped with two RX antennas to decode downlink messages in transmission modes 3 and 4. 
If your SDR only has one RX antenna, LTESniffer will only decode downlink messages in transmission mode 1. Note that GPSDO is optional for downlink sniffing; it will help improve synchronization but is not mandatory.

On the other hand, to sniff uplink traffic from smartphones to base stations, LTESniffer needs to listen to two different frequencies (Uplink and Downlink) concurrently. To solve this problem, LTESniffer supports two options:
- Using a single USRP X310. USRP X310 has two Local Oscillators (LOs) for 2 RX channels, which can turn each RX channel to a distinct Uplink/Downlink frequency. Similar to Downlink Sniffing, GPSDO is optional for this option. To use this option, please refer to the `main` branch of LTESniffer.
- Using 2 USRP B-Series. LTESniffer utilizes 2 USRP B-series (B210/B200) for uplink and downlink separately. It achieves synchronization between 2 USRPs by using GPSDO for clock source and time reference. GPSDO is mandatory for this option. To use this option, please refer to the `LTESniffer-multi-usrp` branch of LTESniffer and its [README][multi-readme].

## Installation
**Important note: To avoid unexpected errors, please follow the following steps on Ubuntu 18.04/20.04/22.04.**

**Dependencies**
- **Important dependency**: [UHD][uhd] library version >= 4.0 must be installed in advance (recommend building from source). The following steps can be used on Ubuntu 18.04. Refer to UHD Manual for full installation guidance. 

UHD dependencies:
```bash
sudo apt update
sudo apt-get install autoconf automake build-essential ccache cmake cpufrequtils doxygen ethtool \
g++ git inetutils-tools libboost-all-dev libncurses5 libncurses5-dev libusb-1.0-0 libusb-1.0-0-dev \
libusb-dev python3-dev python3-mako python3-numpy python3-requests python3-scipy python3-setuptools \
python3-ruamel.yaml
```
Clone and build UHD from source (make sure that the current branch is higher than 4.0)
```bash
git clone https://github.com/EttusResearch/uhd.git
cd <uhd-repo-path>/host
mkdir build
cd build
cmake ../
make -j 4
make test
sudo make install
sudo ldconfig
```
Download firmwares for USRPs:
```bash
sudo uhd_images_downloader
```
We use a [10Gb card](https://www.ettus.com/all-products/10gige-kit/) to connect USRP X310 to PC, refer to UHD Manual [[1]](https://files.ettus.com/manual/page_usrp_x3x0.html), [[2]](https://files.ettus.com/manual/page_usrp_x3x0_config.html) to configure USRP X310 and 10Gb card interface. For USRP B210, it should be connected to PC via a USB 3.0 port.

Test the connection and firmware (for USRP X310 only):
```bash
sudo sysctl -w net.core.rmem_max=33554432
sudo sysctl -w net.core.wmem_max=33554432
sudo ifconfig <10Gb card interface> mtu 9000
sudo uhd_usrp_probe
```

- srsRAN dependencies:
```bash
sudo apt-get install build-essential git cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev
```

- LTESniffer dependencies:
```bash
sudo apt-get install libglib2.0-dev libudev-dev libcurl4-gnutls-dev libboost-all-dev qtdeclarative5-dev libqt5charts5-dev
```

**Build LTESniffer from source:**
```bash
git clone https://github.com/SysSec-KAIST/LTESniffer.git
cd LTESniffer
mkdir build
cd build
cmake ../
make -j 4 (use 4 threads)
```

## Building on macOS (Apple Silicon)

LTESniffer also builds and runs on macOS on Apple Silicon (tested on an M-series
Mac with Homebrew). The downlink path works with any SoapySDR-supported SDR; it
has been used with a **HackRF One** and is prepared for **PlutoSDR** (both go
through the same SoapySDR driver, so a single binary drives either device).

> Note: macOS support targets the **downlink** sniffing mode. Uplink mode needs
> a USRP X310 (or two synchronized USRP B2xx) and UHD, which is outside this
> macOS/HackRF/Pluto setup.

**1. Install the dependencies with Homebrew**
```bash
# Toolchain + core libraries
brew install cmake fftw boost glib libconfig

# mbedTLS: the 2.x API is required (the 4.x default is incompatible)
brew install mbedtls@2

# SoapySDR and the HackRF driver module
brew install soapysdr soapyhackrf
```

**2. (Optional) PlutoSDR support**

PlutoSDR (ADALM-PLUTO) is reached through `SoapyPlutoSDR`, which is not in
Homebrew and must be built from source together with Analog Devices'
`libiio`/`libad9361`. Once its module is present under
`/opt/homebrew/lib/SoapySDR/modules0.8/`, the same `LTESniffer` binary can use it
with `-a "driver=plutosdr"` — no rebuild needed. Build the stack into the
Homebrew prefix (no sudo needed on Apple Silicon):
```bash
# libiio (v0.25; OSX_FRAMEWORK=OFF installs a plain dylib, not a framework)
git clone -b v0.25 https://github.com/analogdevicesinc/libiio
cmake -S libiio -B libiio/build -DCMAKE_INSTALL_PREFIX=/opt/homebrew \
  -DOSX_FRAMEWORK=OFF -DCMAKE_INSTALL_NAME_DIR=/opt/homebrew/lib -DWITH_LOCAL_BACKEND=OFF -DWITH_TESTS=OFF \
  -DWITH_DOC=OFF -DWITH_IIOD=OFF -DHAVE_DNS_SD=OFF
cmake --build libiio/build -j4 && cmake --install libiio/build

# libad9361
git clone https://github.com/analogdevicesinc/libad9361-iio
cmake -S libad9361-iio -B libad9361-iio/build \
  -DCMAKE_INSTALL_PREFIX=/opt/homebrew -DOSX_FRAMEWORK=OFF -DCMAKE_INSTALL_NAME_DIR=/opt/homebrew/lib
cmake --build libad9361-iio/build -j4 && cmake --install libad9361-iio/build

# SoapyPlutoSDR (CMAKE_FIND_FRAMEWORK=NEVER avoids a stale /Library/Frameworks copy)
git clone https://github.com/pothosware/SoapyPlutoSDR
cmake -S SoapyPlutoSDR -B SoapyPlutoSDR/build \
  -DCMAKE_INSTALL_PREFIX=/opt/homebrew -DCMAKE_FIND_FRAMEWORK=NEVER \
  -DCMAKE_PREFIX_PATH=/opt/homebrew
cmake --build SoapyPlutoSDR/build -j4 && cmake --install SoapyPlutoSDR/build

SoapySDRUtil --find    # should now list plutosdr (and hackrf)
```
PlutoSDR tunes 325 MHz – 3.8 GHz, so use it only within that range.

**3. Build**
```bash
git clone https://github.com/SysSec-KAIST/LTESniffer.git
cd LTESniffer
mkdir build && cd build
cmake ..
make -j4
```
The build downloads and, on macOS, automatically patches the srsRAN2 sub-project
(`external/patches/srsRAN2-macos.patch`) so it compiles with the Apple clang
toolchain. The resulting binary is `build/src/LTESniffer`.

To run `LTESniffer` from any directory, install it onto your PATH (no sudo when
using the Homebrew prefix on Apple Silicon):
```bash
cmake .. -DCMAKE_INSTALL_PREFIX=/opt/homebrew
make -j4
make install          # installs LTESniffer into /opt/homebrew/bin
```
Keep the `build/` directory in place: `LTESniffer` loads `libsrsran_rf.dylib`
from the build tree via an embedded path. If `/opt/homebrew/bin` is not on your
PATH, add it: `echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc`.

**4. Verify the SDR is detected**
```bash
SoapySDRUtil --info                 # should list the hackrf factory/module
SoapySDRUtil --probe=driver=hackrf  # should open your HackRF
```

**5. Run (HackRF, downlink)**

Pass the SoapySDR device through `-a`. HackRF has a single RX antenna, so use one
antenna (`-A 1`, transmission mode 1):
```bash
# Cell search + downlink sniffing on a HackRF
./src/LTESniffer -A 1 -W 4 -f <DL Freq> -C -m 0 -a "driver=hackrf"
example: ./src/LTESniffer -A 1 -W 4 -f 1840e6 -C -m 0 -a "driver=hackrf"
```
For PlutoSDR, use `-a "driver=plutosdr"` (over USB it is auto-detected; append
`,uri=ip:192.168.2.1` to force the network interface). PlutoSDR also has a single
RX channel, so use `-A 1`, and tune within its 325 MHz – 3.8 GHz range:
```bash
./src/LTESniffer -A 1 -W 4 -f 1840e6 -C -m 0 -a "driver=plutosdr"
```

> macOS notes:
> - `sudo` is generally not required for HackRF/Pluto over USB on macOS.
> - **Gain:** prefer the default AGC — just omit `-g`. A fixed `-g` value that is
>   too low/high can make the cell search sync to garbage ("Invalid number of
>   PRB ..."). With AGC, PlutoSDR reliably decodes the MIB here (verified on a
>   live Band 28 cell at 763 MHz).
> - PlutoSDR / HackRF frequency error is corrected automatically (CFO), so no
>   ppm option is needed, unlike CellSearch in LTE-Cell-Scanner.
> - `--cpu_affinity` is ignored on macOS (no thread-affinity API); real-time
>   performance depends on your Mac's CPU.

### Watching the traffic live in Wireshark (macOS)

LTESniffer writes a MAC-LTE pcap (DLT 147 / `DLT_USER0`). To view packets live
instead of opening a file afterwards, stream the pcap into Wireshark through a
named pipe (FIFO).

1. Install Wireshark and configure the dissector (one-time):
   ```bash
   brew install --cask wireshark
   ```
   In Wireshark: **Preferences → Protocols → DLT_USER → Edit** and add a row
   `User 0 (DLT=147)` with payload protocol `mac-lte-framed`. (This is the same
   mapping described in the pcap configuration guide linked below.)

2. Create a FIFO and point LTESniffer's pcap output at it with `-F`:
   ```bash
   mkfifo /tmp/lte.pcap
   # Start Wireshark reading the FIFO FIRST (it blocks waiting for the writer):
   /Applications/Wireshark.app/Contents/MacOS/Wireshark -k -i /tmp/lte.pcap &
   # Then start LTESniffer, writing the pcap to the same FIFO:
   LTESniffer -A 1 -W 4 -f 763e6 -C -m 0 -a "driver=plutosdr" -F /tmp/lte.pcap
   ```
   Packets now appear in Wireshark in real time. Useful display filters:
   `mac-lte` (all), `mac-lte.direction == 1` (downlink), `== 0` (uplink).

   Order matters: open the Wireshark reader before LTESniffer, because writing a
   FIFO blocks until a reader is attached. The `-F` flag sets the pcap filename
   for any mode; without it, LTESniffer writes `ltesniffer_dl_mode.pcap` /
   `ltesniffer_ul_mode.pcap` in the current directory as before.

## Usage
LTESniffer has 3 main functions: 
- Sniffing LTE downlink traffic from the base station
- Sniffing LTE uplink traffic from smartphones
- Security API

After building from source, ``LTESniffer`` is located in ``<build-dir>/src/LTESniffer``

Note that before using LTESniffer on the commercial, one should have to check the local regulations on sniffing LTE traffic, as we explained in the **Ethical Consideration**.

To figure out the base station and Uplink-Downlink band the test smartphone is connected to, install [Cellular-Z][app] app on the test smartphone (the app only supports Android). It will show the cell ID and Uplink-Downlink band/frequency to which the test smartphone is connected. Make sure that LTESniffer also connects to the same cell and frequency.
### General downlink sniffing

<p align="center">
  <img src="png/dl_mode_png.png" alt="LTESniffer Downlink Mode">
</p>

```bash
sudo ./<build-dir>/src/LTESniffer -A 2 -W <number of threads> -f <DL Freq> -C -m 0
example: sudo ./src/LTESniffer -A 2 -W 4 -f 1840e6 -C -m 0
-A: number of antennas
-W: number of threads
-f: downlink frequency
-C: turn on cell search
-m: sniffer mode, 0 for downlink sniffing and 1 for uplink sniffing
```
Note: to run ``LTESniffer`` with USRP B210 in the downlink mode, add option ``-a "num_recv_frames=512" `` to the command line.
This option extends the receiving buffer for USRP B210 to achieve better synchronization.

```bash
sudo ./<build-dir>/src/LTESniffer -A 2 -W <number of threads> -f <DL Freq> -C -m 0 -a "num_recv_frames=512"
example: sudo ./src/LTESniffer -A 2 -W 4 -f 1840e6 -C -m 0 -a "num_recv_frames=512"
```

### General uplink sniffing
Note: In the uplink sniffing mode, the test smartphones should be located nearby the sniffer, because the uplink signal power from UE is significantly weaker compared to the downlink signal from the base station.

<p align="center">
  <img src="png/ul_mode_png.png" alt="LTESniffer Uplink Mode">
</p>

```bash
sudo ./<build-dir>/src/LTESniffer -A 2 -W <number of threads> -f <DL Freq> -u <UL Freq> -C -m 1
example: sudo ./src/LTESniffer -A 2 -W 4 -f 1840e6 -u 1745e6 -C -m 1
-u: uplink frequency
```

### Security API

<p align="center">
  <img src="png/api_png.png" alt="LTESniffer API Mode">
</p>

```bash
sudo ./<build-dir>/src/LTESniffer -A 2 -W <number of threads> -f <DL Freq> -u <UL Freq> -C -m 1 -z 3
example: sudo ./src/LTESniffer -A 2 -W 4 -f 1840e6 -u 1745e6 -C -m 1 -z 3
-z: 3 for turnning on 3 functions of sniffer, which are identity mapping, IMSI collecting, and UECapability profiling.
    2 for UECapability profiling
    1 for IMSI collecting
    0 for identity mapping
```
### Specify a base station

LTESniffer can sniff on a specific base station by using options ``-I <Phycial Cell ID (PCI)> -p <number of Physical Resource Block (PRB)>``. In this case, LTESniffer does not do the cell search but connects directly to the specified cell.
```bash
sudo ./<build-dir>/src/LTESniffer -A 2 -W <number of threads> -f <DL Freq> -I <PCI> -p <PRB> -m 0
sudo ./<build-dir>/src/LTESniffer -A 2 -W <number of threads> -f <DL Freq> -u <UL Freq> -I <PCI> -p <PRB> -m 1
example: sudo ./src/LTESniffer -A 2 -W 4 -f 1840e6 -u 1745e6 -I 379 -p 100 -m 1
```
The debug mode can be enabled by using option ``-d``. In this case, the debug messages will be printed on the terminal.

### Output of LTESniffer

LTESniffer provides pcap files in the output. The pcap file can be opened by WireShark for further analysis and packet trace.
The name of downlink pcap file: ``sniffer_dl_mode.pcap``, uplink pcap file: ``sniffer_ul_mode.pcap``, and API pcap file: ``api_collector.pcap``.
The pcap files are located in the same directory ``LTESniffer`` has been executed.
To enable the WireShark to analyze the decoded packets correctly, please refer to the WireShark configuration guide [here][pcap]. There are also some examples of pcap files in the link.\
**Note:** The uplink pcap file contains both uplink and downlink messages. On the WireShark, use this filter to monitor only uplink messages: ``mac-lte.direction == 0``; or this filter to monitor only downlink messages: ``mac-lte.direction == 1``.

## Application Note
### Distance for uplink sniffing
The effective range for sniffing uplink is limited in LTESniffer due to the capability of the RF front-end of the hardware (i.e. SDR). The uplink signal power from UE is significantly weaker compared to the downlink signal because UE is a handheld device that optimizes battery usage, while the eNB uses sufficient power to cover a large area. To successfully capture the uplink traffic, LTESniffer can increase the strength of the signal power by i) being physically close to the UE, or ii) improving the signal reception capability with specialized hardware, such as a directional antenna, dedicated RF front-end, and signal amplifier.
### The information displayed on the terminal
**Downlink Sniffing Mode** 

``Processed 1000/1000 subframes``: Number of subframes was processed by LTESniffer last 1 second. There are 1000 LTE subframes per second by design. \
``RNTI``: Radio Network Temporary Identifier of UEs. \
``Table``: The maximum modulation scheme that is used by smartphones in downlink. LTESniffer supports up to 256QAM in the downlink. Refer to our [paper][paper] for more details. \
``Active``: Number of detected messages of RNTIs. \
``Success``: Number of successfully decoded messages over number of detected messages (``Active``). \
``New TX, ReTX, HARQ, Normal``: Statistic of new messages and retransmitted messages. This function is in development. \
``W_MIMO, W_pinfor, Other``: Number of messages with wrong radio configuration, only for debugging. 

**Uplink Sniffing Mode** 

``Max Mod``: The maximum modulation scheme that is used by smartphones in uplink. It can be 16/64/256QAM depending on the support of smartphones and the configuration of the network. Refer to our [paper][paper] for more details. \
``SNR``: Signal-to-noise ratio (dB). Low SNR means the uplink signal quality from the smartphone is bad. One possible reason is the smartphone is far from the sniffer. \
``DL-UL_delay``: The average of time delay between downlink signal from the base station and uplink signal from the smartphone. \
``Other Info``: Information only for debugging. 

**API Mode** 

``Detected Identity``: The name of detected identity. \
``Value``: The value of detected identity. \
``From Message``: The name of the message that contains the detected identity. 

<!-- ## FAQ
**Q:** Is it possible to capture and see the phone call content using LTESniffer? \
**A:** No. LTE traffic including phone call traffic is encrypted, so you cannot use LTESniffer to know the content of phone calls of someone. Moreover, it is important to note that sniffing phone calls in the commercial network is illegal in most countries. -->
## Credits
We sincerely appreciate the [FALCON][falcon] and [SRS team][srsran] for making their great softwares available.

## Contributor
Special thanks to all the contributors who helped us to fix bugs and improve LTESniffer

1. [cellular777][cellular77]
2. [Cemaxecuter][Cemaxecuter]
3. [Ksk190809][Ksk190809]

## BibTex
Please refer to our [paper][paper] for more details.

```bibtex
@inproceedings{hoang:ltesniffer,
  title = {{LTESniffer: An Open-source LTE Downlink/Uplink Eavesdropper}},
  author = {Hoang, Dinh Tuan and Park, CheolJun and Son, Mincheol and Oh, Taekkyung and Bae, Sangwook and Ahn, Junho and Oh, BeomSeok and Kim, Yongdae},
  booktitle = {16th ACM Conference on Security and Privacy in Wireless and Mobile Networks (WiSec '23)},
  year = {2023}
}
```

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=SysSec-KAIST/LTESniffer&type=Date)](https://star-history.com/#SysSec-KAIST/LTESniffer&Date)

## FAQ

<!-- **Q:** What kind of SDRs I can use to run LTESniffer? \
**A:** To sniff only downlink traffic from the base station, LTESniffer works well with USRP B210 with 2 RX antennas.
To sniff the uplink traffic, LTESniffer requires USRP X310 with 2 daughterboards. There are two reasons for this. First, sniffing the uplink traffic requires precise time synchronization between uplink and downlink subframes, which can be simply achieved by using two daughterboards with the same clock source from a single motherboard of USRP X310. Second, the "srsran_rf_set_rx_freq" function used by LTESniffer seems to only support the USRP X310 with 2 daughterboards for simultaneous reception of signals at two different frequencies. -->

**Q:** Is it mandatory to use GPSDO with the USRP in order to run LTESniffer? \
**A:** GPSDO is useful for more stable synchronization. However, for downlink sniffing mode, LTESniffer still can synchronize with the LTE signal to decode the packets without GPSDO. For uplink sniffing mode, GPSDO is only required when using 2 USRP B-series, as it is the time and clock reference sources for synchrozation between uplink and downlink channels. Another uplink SDR option, using a single USRP X310, does not require GPSDO.

**Q:** For downlink traffic, can I use a cheaper SDR? \
**A:** Technically, any SDRs supported by srsRAN library such as Blade RF can be used to run LTESniffer in the downlink sniffing mode. However, we only tested the downlink sniffing function of LTESniffer with USRP B210 and X310. 

**Q:** Is it illegal to use LTESniffer to sniff the LTE traffic? \
**A:** You should have to check the local regulations on sniffing (unencrypted) LTE traffic. Another way to test LTESniffer is setting up a personal LTE network by using [srsRAN][srsran] - an open-source LTE implementation in a Faraday cage. 

**Q:** Can LTESniffer be used to view the content of messages between two users? \
**A:** One can see only the "unencrypted" part of the messages. Note that the air traffic between the base station and users is mostly encrypted.

**Q:** Is there any device identity exposed in plaintext in the LTE network? \
**A:** Yes, literature shows that there are multiple identities exposed, such as TMSI, GUTI, IMSI, and RNTI. Please refer to the academic literature for more details. e.g. [Watching the Watchers: Practical Video Identification Attack in LTE Networks][watching]

[falcon]: https://github.com/falkenber9/falcon
[srsran]: https://github.com/srsran/srsRAN_4G
[uhd]:    https://github.com/EttusResearch/uhd
[paper]:  https://syssec.kaist.ac.kr/pub/2023/wisec2023_tuan.pdf
[pcap]:   pcap_file_example/README.md
[app]:    https://apkpure.com/cellular-z/make.more.r2d2.cellular_z
[watching]: https://syssec.kaist.ac.kr/pub/2022/sec22summer_bae.pdf
[multi-readme]: https://github.com/SysSec-KAIST/LTESniffer/tree/LTESniffer-multi-usrp
[capture-readme]: https://github.com/SysSec-KAIST/LTESniffer/tree/LTESniffer-record-subframe
[cellular77]: https://github.com/cellular777
[Cemaxecuter]: https://www.youtube.com/@cemaxecuter7783
[Ksk190809]: https://github.com/Ksk190809