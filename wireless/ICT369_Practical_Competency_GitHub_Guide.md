# ICT369 — Practical Skills Tests: Lab Companion & Troubleshooting

> **Study / practice companion** · Murdoch University · Semester 2, 2026  
> **Physical network test:** 15%, 90 minutes · **Packet diagnostics test:** 15%, 90 minutes.  
> **Assessment rule:** Both tests are invigilated/closed-book. The LMS permits the **specified official lab pages** during the test, but **not personal notes, this guide, other web resources, or unrelated GitHub pages**. Use this document **before the exam for practice only**; do not consult it in the assessment unless the unit coordinator explicitly authorises it.
>
> Passwords below are **published lab/example credentials**, not universally valid exam credentials. Use only the values associated with **your assigned port**. Do not publish personal, university-account, or private device passwords.

## Table of contents
1. [Exact permitted lab links](#1-exact-permitted-lab-links)
2. [Physical lab network design](#2-physical-lab-network-design)
3. [GPON and copper credentials](#3-gpon-and-copper-credentials)
4. [MikroTik PPPoE setup](#4-mikrotik-pppoe-setup)
5. [VMware and Ubuntu troubleshooting](#5-vmware-and-ubuntu-troubleshooting)
6. [Linksys AP-only conversion](#6-linksys-ap-only-conversion)
7. [FreeRADIUS and enterprise Wi-Fi](#7-freeradius-and-enterprise-wi-fi)
8. [SSH public-key setup](#8-ssh-public-key-setup)
9. [Physical assessment evidence and reset](#9-physical-assessment-evidence-and-reset)
10. [Packet-analysis test: methods](#10-packet-analysis-test-methods)
11. [Error-to-fix matrix](#11-error-to-fix-matrix)
12. [Practice-only self-checks](#12-practice-only-self-checks)

## 1. Exact permitted lab links

These are the **seven specified official GitHub lab exercises** in the assessment description. Use these official links during the test, subject to the LMS rules—not this companion file:

### Physical Lab Competency Test
- [Broadband CPE Scenarios](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/broadband_cpe_senarios.md)
- [GPON PPPoE](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/gpon_pppoe.md)
- [Enterprise Authentication with FreeRADIUS](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/enterprise_authentication_with_FreeRADIUS.md)

### Wireless Packet Capture Skills and Network Diagnostics
- [RTS/CTS and Network Analysis](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/rts_cts_and_network_analysis.md)
- [WEP](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wep.md)
- [WPA](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wpa.md)
- [Network Forensics](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/network_forensics.md)

**Scope:** This file links to (but does not copy) the original lab instructions. The commentary and troubleshooting are derived from our practice conversation and the two uploaded practical-note PDFs; confirm differences against the current LMS and assigned hardware.

## 2. Physical lab network design

```text
Data centre / ISP (PPPoE, gateway 100.64.0.1)
           |
       GPON or copper
           |
  ether1 / WAN  — MikroTik hEX PoE
       WAN: 100.64.0.x (assigned)
       LAN: 192.168.88.1/24
       DHCP: ON; NAT: ON
           |
       MikroTik LAN ports (ether2-ether5)
          / \
         /   \---- Linksys DD-WRT LAN port (NOT WAN)
        |           IP: 192.168.88.2/24
        |           WAN: disabled; DHCP: disabled
        |           WPA2 Enterprise AP ---- wireless phone
        |
   Windows physical LAN + Ubuntu bridged NIC
   Ubuntu: 192.168.88.x (FreeRADIUS server)
   RADIUS UDP 1812: Linksys -> Ubuntu
```

**Checkpoint order:** PPPoE connected → VM pings `192.168.88.1` → VM pings `100.64.0.1` → Linksys can ping / be pinged at `192.168.88.2` → local RADIUS test → phone RADIUS `Access-Accept` → phone DHCP address → passwordless SSH. A successful ping does **not** prove wireless authentication, and `Access-Accept` does **not** prove DHCP has succeeded.

## 3. GPON and copper credentials

The **GPON PPPoE** GitHub lab assigns different port ranges and credentials. **Never interchange GPON fibre and copper port assignments.**

### 3.1 GPON fibre side — ports A–H

| Fibre pod | Data-centre GPON port | PPPoE username | PPPoE password | ONU label | ONU username | ONU password |
|---|---|---|---|---|---|---|
| A | A | `alpha` | `alpha` | alpha | `alpha` | `alpha` |
| B | B | `bravo` | `bravo` | bravo | `bravo` | `bravo` |
| C | C | `charlie` | `charlie` | charlie | `charlie` | `charlie` |
| D | D | `delta` | `delta` | delta | `delta` | `delta` |
| E | E | `echo` | `echo` | echo | `echo` | `echo` |
| F | F | `foxtrot` | `foxtrot` | foxtrot | `foxtrot` | `foxtrot` |
| G | G | `golf` | `golf` | golf | `golf` | `golf` |
| H | H | `hotel` | `hotel` | hotel | `hotel` | `hotel` |

### 3.2 Router / copper side — pods I–P

| Copper pod | Data-centre copper port | PPPoE username | PPPoE password |
|---|---:|---|---|
| I | 16 | `india` | `india` |
| J | 17 | `juliett` | `juliett` |
| K | 18 | `kilo` | `kilo` |
| L | 19 | `lima` | `lima` |
| M | 20 | `mike` | `mike` |
| N | 21 | `november` | `november` |
| O | 23 | `oscar` | `oscar` |
| P | 24 | `papa` | `papa` |

**Important discrepancy in official materials:** The older **Broadband CPE Scenarios** lab has an A–H table for **copper ports 16–21, 23–24** (including `beta` for B); the newer **GPON PPPoE** lab uses **fibre A–H (`bravo` for B)** and **copper I–P**. Follow the table for the **actual lab and patch panel** you are using. Ask the tutor if labels differ. In an earlier separate practice scenario, GPON F used `foxtrot/foxtrot`; this is **not** a universal credential.

## 4. MikroTik PPPoE setup

1. Confirm the physical patch-panel letter/number and whether it is **GPON** or **copper**. For the practice topology, connect **data-centre line → MikroTik ether1/WAN**, and **MikroTik LAN → host PC**.
2. Start with MikroTik only; leave Linksys unplugged for now.
3. On the directly connected host, open `http://192.168.88.1`.
4. Log in using the reset-state credentials appropriate to the hardware. **Do not assume one universal password**: the GPON lab says `admin` with a blank password; the Broadband CPE lab says the device sticker or `admin`. Use the current lab/tutor instructions.
5. In **Quick Set**, select router configuration, **Address Acquisition = PPPoE**, enter **assigned** PPPoE username/password, leave service name blank unless directed, set LAN `192.168.88.1/24`, **DHCP ON**, **NAT ON**.
6. Click **Apply Configuration**; do not substitute the Reconnect button for Apply.
7. Verify PPPoE status **connected**, WAN `100.64.0.x`, gateway commonly `100.64.0.1`, and LAN `192.168.88.1`.
8. MikroTik terminal: `/ping 100.64.0.1`. This tests the uplink independently of Windows/Ubuntu routing.

**Remember:** The old failed command was `ping 100.64.88.1`; the test asks for **`100.64.0.1`**. Private `192.168.88.1` is the MikroTik **LAN** address, not the data-centre gateway.

### GPON speed-test extension (weekly lab practice)

The linked GPON lab also covers **two routers (fibre + copper)**, Apache file transfer and port forwarding. Those are useful practice but **do not replace** the specific physical-test requirements. From Ubuntu:

```bash
sudo apt update
sudo apt install apache2
sudo systemctl enable --now apache2
# Example test file; creates ~1000 MiB, ensure adequate disk space:
dd if=/dev/zero of=output.dat bs=1M count=1000
sudo cp output.dat /var/www/html/
ls -lh /var/www/html/output.dat
```

If a partner can download your file only after Apache starts, the route may have been correct but **the web server was off**. If `output.dat` is missing, create/copy it before testing HTTP. Configure port forwarding according to the **official lab** and your actual VM IP; do not assume a friend's WAN IP is your LAN host IP.

## 5. VMware and Ubuntu troubleshooting

### 5.1 What actually happened during practice

Windows showed two physical networks:

```text
Ethernet (university) : 134.115.148.13/24; gateway 134.115.148.1
Ethernet 2 (MikroTik): 192.168.88.251/24; gateway 192.168.88.1
```

Ubuntu initially used `ens38 = 134.115.148.38/24`, with no MikroTik IPv4 address. Later it had `ens33 = 192.168.88.250/24` and `ens39 = 192.168.88.249/24` simultaneously. **These are practice values, not fixed IPs**. Several VMware NICs were bridged automatically; a custom VMnet2 was also present. The campus PC had no administrative rights to change the VMware Virtual Network Editor.

### 5.2 Inspect before changing

```bash
ip -br addr
ip route
ip route get 100.64.0.1
ip route get 192.168.88.2
ping -c 4 192.168.88.1
ping -c 4 100.64.0.1
```

Expected **route** to GPON should use the MikroTik LAN interface (`via 192.168.88.1`) and an Ubuntu `192.168.88.x` source IP.

### 5.3 Wrong default route / metric

The practice result was:

```text
default via 134.115.148.1 dev ens38 metric 101  # unwanted for lab path
default via 192.168.88.1 dev ens33 metric 102 # valid MikroTik path
```

Linux prefers the lower metric, therefore a ping to `100.64.0.1` may leave via the campus network. **Temporary correction**, replacing `ens33` with the actual MikroTik-facing interface:

```bash
sudo ip route replace default via 192.168.88.1 dev ens33 metric 50
ip route get 100.64.0.1
ping -c 4 100.64.0.1
```

**Safer when multiple NICs exist:** disconnect the campus VM adapter during the lab, or explicitly select the right VMware bridge. **Important**: interface state and routing changes may revert after reboot. `ip link set <interface> down` does not always remove assigned addresses; review routes afterward. Avoid disabling the host's university Ethernet without approval.

### 5.4 Bring an interface up, obtain DHCP

```bash
sudo ip link set ens33 up
ip -br addr
sudo dhclient -v ens33  # if dhclient is installed; not all Ubuntu builds include it
ip route
```

`UP` means the interface is enabled, **not** that it has received a usable IPv4 address. `NO-CARRIER` means the virtual cable or underlying network is disconnected. If your VM has multiple `192.168.88.x` NICs, prefer **one** bridged adapter for predictable RADIUS traffic. When changing bridged interfaces, verify `ip route get 192.168.88.2` and choose the source IP shown there **only after confirming Linksys can reach it**.

### 5.5 Windows diagnostic checkpoints

```cmd
ipconfig
ping 192.168.88.1
ping 192.168.88.2
arp -a
```

`Media disconnected` on Ethernet 2 indicates no cable/link. After Linksys reset, Windows briefly received `192.168.1.144` with gateway `192.168.1.1`; this meant it was talking to Linksys's default subnet (or retained an old lease), **not** that the MikroTik network had broken. Reconnect cables after AP configuration and renew DHCP as appropriate.

## 6. Linksys AP-only conversion

1. **Isolate Linksys from MikroTik initially.** Connect host PC to **Linksys LAN** and browse to `http://192.168.1.1` if at factory settings. If already reconfigured, try its known new IP `http://192.168.88.2`; do not reset before diagnosing.
2. **Setup → Basic Setup**:
   - WAN Connection Type: **Disabled**
   - Local IP Address: **`192.168.88.2`**
   - Subnet Mask: `255.255.255.0`
   - Gateway: `192.168.88.1`
   - DHCP Server: **Disabled**
3. **Save → Apply Settings**. The browser may disconnect when the management IP moves to `192.168.88.2`.
4. Cable **MikroTik LAN → Linksys LAN** (**not** Linksys WAN). Host should receive DHCP `192.168.88.x` from MikroTik.
5. Verify `ping -c 4 192.168.88.2` from Ubuntu before setting RADIUS.

### 6.1 Dual-band trap encountered

DD-WRT exposed:
- `ath0`: **5 GHz**, SSID `Anuj`
- `ath1`: **2.4 GHz**, also SSID `Anuj`

Initially **WPA2 Enterprise was enabled on one radio**, but **`ath1` showed Security Mode Disabled**. The phone could connect to an unintended/open SSID and fail to send an enterprise authentication request. For testing, rename radios distinctly (`Anuj-5G` / `Anuj-2G`) and configure WPA2 Enterprise on the exact SSID/radio being tested. This isolates the failing path; names are examples, not requirements.

## 7. FreeRADIUS and enterprise Wi-Fi

### 7.1 Install, locate, edit

```bash
sudo apt update
sudo apt install freeradius freeradius-utils
freeradius -v
hostname -I
ip route get 192.168.88.2
```

FreeRADIUS v3 on this Ubuntu used:

```text
/etc/freeradius/3.0/clients.conf
/etc/freeradius/3.0/mods-config/files/authorize
```

The legacy `/etc/freeradius/3.0/users` path may be a symlink on some distributions; the actual parsing error in the practice environment specifically referenced `mods-config/files/authorize`.

### 7.2 Authorise Linksys as a RADIUS client

```bash
sudo nano /etc/freeradius/3.0/clients.conf
```

Add **outside** other client blocks:

```conf
client linksys {
    ipaddr = 192.168.88.2
    secret = radiussecret
}
```

**Actual practice errors:**
- Nano opened `etc/freeradius/...` **without leading `/`** → failed writing `No such file or directory`.
- `secret - radiussecret` instead of `secret = radiussecret` → invalid configuration.
- If editing as non-root → `Permission denied`; use `sudo nano`.
- To save: **Ctrl+O → Enter → Ctrl+X**.

### 7.3 Create a test user

```bash
sudo nano /etc/freeradius/3.0/mods-config/files/authorize
```

Near the top, add:

```conf
aj Cleartext-Password := "student123"
```

The real practice error was `Parse error (check) for entry aj: Expecting operator` because the password attribute/operator was typed incorrectly. The correct operator is **`:=`** (colon + equals). Don't duplicate the same `aj` entry without checking existing rules.

### 7.4 Validate, debug and test

```bash
sudo freeradius -C                 # Syntax test; should say Configuration appears to be OK
sudo systemctl stop freeradius     # Avoid two processes fighting for ports
sudo freeradius -X                # Keep this terminal open: Ready to process requests
```

In **another** Ubuntu terminal (assuming stock `client localhost` with `testing123`):

```bash
radtest aj student123 127.0.0.1 0 testing123
```

**Interpretation:**
- `Ready to process requests` = daemon listening, **not** proof of a Wi-Fi login.
- `Received Access-Request` = AP reached FreeRADIUS.
- `Access-Reject` = failed policy/credentials/EAP negotiation.
- `Sent Access-Accept` = RADIUS authentication accepted, **not** proof the phone obtained DHCP/IP.
- **No request shown** = first check wireless security on the SSID actually joined, server IP, UDP 1812, reachability and shared secret.
- `Address already in use` / `18120` = another process is running; stop background service before `-X`.

### 7.5 Configure DD-WRT enterprise authentication

**Wireless → Wireless Security**, for the appropriate active radio:

| Field | Value |
|---|---|
| Security mode | **WPA2 Enterprise** |
| RADIUS authentication server IP | **Current Ubuntu MikroTik-subnet address** (observed `192.168.88.249` in final practice) |
| RADIUS port | `1812/UDP` |
| Shared secret | `radiussecret` (must equal `clients.conf`) |

**Practice IP confusion:** Ubuntu had **both** `192.168.88.250` (`ens33`) and `192.168.88.249` (`ens39`). The command `ip route get 192.168.88.2` ultimately returned `src 192.168.88.249`, so DD-WRT's server IP `192.168.88.249` matched the chosen path. **A source-route output is not itself proof of inbound UDP reachability.** Avoid two VM interfaces on the same subnet if possible and verify with real RADIUS requests.

### 7.6 Android phone settings

| Field | Isolated practice example |
|---|---|
| EAP method | `TTLS` |
| Phase 2 | `MSCHAPV2` |
| Identity | `aj` |
| Password | `student123` |
| Anonymous identity | blank |
| IP settings | DHCP |
| CA certificate | Lab guide says **Don't validate** |

**Security warning:** “Don't validate” disables verification of the RADIUS server's certificate and is **unsafe on production/campus enterprise networks**. Use a properly installed trusted CA and server name in real deployments.

### 7.7 If phone says “Couldn't get IP address”

Diagnose in **two phases**:

1. **Authentication**: Forget SSID on phone; join the correct protected radio; watch `sudo freeradius -X`. Was there `Access-Request` and `Access-Accept`?
2. **DHCP only after authentication succeeds**: MikroTik DHCP ON, Linksys DHCP OFF, **LAN-to-LAN** cable, SSID bridged to LAN (not an isolated guest VLAN), phone DHCP enabled. Confirm phone receives `192.168.88.x`; check MikroTik leases and DHCP logs. If there is no `Access-Request`, don't start by adjusting DHCP.

For deeper path investigation **in the authorised lab**, compare authentication packets to your actual NIC:

```bash
sudo tcpdump -ni ens39 udp port 1812
```

Use `ens33` rather than `ens39` if that is your chosen FreeRADIUS NIC. **No packets** suggests AP/server/path mismatch; packets arriving but no debug events suggest client identification, destination or service binding problems.

## 8. SSH public-key setup

From Ubuntu:

```bash
ssh-keygen -t rsa
ls -la ~/.ssh
cat ~/.ssh/id_rsa.pub
```

Use the required lab prompts. A blank passphrase was used in practice for the demonstration, but for real deployments protect your private key with a passphrase and never share `~/.ssh/id_rsa`.

Transfer **public** key only:

```bash
scp -O -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa ~/.ssh/id_rsa.pub admin@192.168.88.1:/
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa admin@192.168.88.1
```

Inside the **MikroTik RouterOS prompt** (`[admin@...] >`), not inside Ubuntu:

```routeros
/file print
/user ssh-keys import public-key-file=id_rsa.pub user=admin
/user ssh-keys print
```

Exit and reconnect over SSH. The practice legacy `ssh-rsa` options may be needed on old MikroTik firmware; they weaken compatibility/security preferences and must not be copied to modern infrastructure unnecessarily.

**Practice mistakes:** `id_rsa.pub` is case-sensitive; typing Linux `/user` commands in the Ubuntu shell fails; do not type the next command into an unfinished `ssh-keygen` prompt.

## 9. Physical assessment evidence and reset

Before asking the tutor to assess, demonstrate:

1. Ubuntu ping to **`100.64.0.1`**, preferably **0% loss**.
2. MikroTik PPPoE **connected**; correct WAN; LAN `192.168.88.1`; DHCP/NAT enabled.
3. Linksys **AP-only**, WAN disabled, DHCP disabled, address `192.168.88.2`.
4. Phone authenticated to the correct enterprise SSID; FreeRADIUS log with **`Access-Accept`**; phone received DHCP `192.168.88.x`.
5. Passwordless SSH public-key login to MikroTik and `/user ssh-keys print`.
6. **Reset hardware only when the tutor permits/after verification.**

Keep evidence screenshots free of private keys, account passwords and RADIUS shared secrets when sharing them.

## 10. Packet-analysis test: methods

> Adapted from the uploaded **ICT369 Practical Test 2 — One-Stop Notes** (10 pages). The examination can use **different captures, filenames, passwords, flags and packet numbers**. Reproduce every answer from the supplied evidence, not these practice examples.

### 10.1 Identify the question type

| Type | Clue | Start here |
|---|---|---|
| A — 802.11 statistics | Beacon %, SSID, retransmits, RTS/CTS | Clear filters; record full packet count; apply specific filter |
| B — TLS / HTTP | GET, images/objects, HTTPS, search terms | Check archive for TLS keylog, import, reload, filter HTTP |
| C — WEP / WPA key recovery | `.ivs`, password list, handshake | Identify cipher; confirm captures/handshake; Aircrack then Wireshark decryption |
| D — exfiltration / timing | flag, UDP ports, periodic packets | Protocol Hierarchy → Conversations → isolate suspicious flow |

### 10.2 Verify provided encrypted captures

```bash
pwd
ls -lah
file *
openssl enc -d -aes-256-cbc -in INPUT.enc -out OUTPUT
# Immediately after decryption, before any other command:
echo $?
file OUTPUT
# If output is archive:
tar -tjf archive.tar.bz2
tar -xjf archive.tar.bz2
# If bzip2-compressed PCAP:
bunzip2 -k capture.pcap.bz2
```

Enter the provided AES password at the **OpenSSL prompt** rather than as a shell argument, especially if it contains `$`, `&`, `!` or `#`. Do not assume `-md md5`; it was only a **legacy fallback** for one practice file. Wrong working folder, opening `.enc`/`.bz2` in Wireshark, and relying on a `file` result of just `data` caused practice errors.

### 10.3 Type A — WLAN statistics

| Wireshark display filter | Purpose |
|---|---|
| `wlan.fc.type_subtype == 0x0008` | Beacon frames |
| `wlan.fc.retry == 1` | Retry flag |
| `wlan.fc.type == 2` | Data frames |
| `wlan.fc.protected == 1` | Protected frames |
| `eapol` | EAPOL/handshake packets |

`Beacon percentage = (beacon count / unfiltered total count) × 100`. Check whether the LMS wants **one decimal without `%`** (`31.0` rather than `31` or `31.0%`). Read SSID exactly from beacon tagged parameters. When counting retries, include the exact requested frame type rather than narrowing unintentionally.

**RTS/CTS/ACK diagnostics** (official RTS/CTS lab): compare control/data frames, Block ACK ratios, retry rates, and frame size bins **under 500**, **under 1000**, **at least 1000** as specified; use correct non-overlapping groups when aggregating sizes, and answer percentages with the denominator requested by the question.

### 10.4 Type B — TLS / HTTP and object recovery

1. Locate `.pcapng` and `.log`/TLS secret key file in the **supplied authorised test material**.
2. Wireshark → **Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log filename** → select supplied log → reload (`Ctrl+R`).
3. Try `http || http2`, `http.request.method == "GET"`, `http2.headers.method == "GET"`.
4. **File → Export Objects → HTTP**, keep the object-window **Text Filter empty**, select all content types and sort by size. If question says *second-largest*, pick rank 2.
5. Export Objects may show **response** packet number; to answer the **GET request frame**, open its response and inspect `[Request in frame: N]`.
6. Search/map queries may use API or autocomplete paths, not always `/search?q=`.

`Ctrl+F` is **Find Packet**, not the top **Display Filter** bar. An Export Objects list containing only a tiny redirect while content should be much larger suggests TLS decryption secrets were not loaded.

### 10.5 Type C — offline WEP and WPA recovery (authorised captures only)

```bash
# WEP: supplied .ivs or capture
 aircrack-ng capture.ivs
# WPA/WPA2-Personal: supplied full handshake and wordlist
 aircrack-ng -w WORDLIST.txt -e ESSID capture.pcap
```

For WPA, check `eapol` and correct SSID/BSSID, look for **KEY FOUND!**, then Wireshark → **IEEE 802.11 → Enable decryption → Decryption Keys → `wpa-pwd` → `PASSWORD:ESSID`**. For WEP add a `wep` key. After reloading, verify higher-layer decoded packets (e.g. `ip`, `http`) before answering. A visible 4-way handshake **does not** mean decrypted application traffic is available. The prior practice typo `drpeeperdrinker` versus `drpepperdrinker` broke decryption.

The official WEP/WPA labs also describe live wireless capture with `airodump-ng`; practise live attack techniques **only on your own or explicitly authorised test wireless network**, never on university production Wi-Fi. For a supplied offline PCAP/IVS assessment, don't waste time setting up live monitor mode.

### 10.6 Type D — secret message and timing

1. Use **Statistics → Protocol Hierarchy / Conversations**; note low-volume repeated flows.
2. Filter **only** suspicious UDP/TCP endpoint or port and inspect payload bytes in capture order.
3. Example reusable syntax:

```bash
tshark -r CAPTURE.pcap -Y 'udp.dstport == PORT' -T fields -e data
# Only if each payload is exactly one ASCII byte plus trailing 0a:
tshark -r CAPTURE.pcap -Y 'udp.dstport == PORT' -T fields -e data | cut -c1-2 | tr -d '\n' | xxd -r -p; echo
```

4. For time intervals: **filter the suspicious flow FIRST**, then Wireshark → **View → Time Display Format → Seconds Since Previous Displayed Packet**.
5. Follow exact flag format: if form already prints `DMFlag{ [answer] }`, enter only the inside characters; preserve case, `%`, `0/O`, `1/l/I`, exact source versus destination port.

### 10.7 Fast Wireshark filter sheet

```text
wlan.fc.type_subtype == 0x0008  # beacon
wlan.fc.retry == 1              # retransmission flag
wlan.fc.type == 2               # Wi-Fi data
eapol                           # EAPOL
ip                              # decoded IP
http || http2                   # decoded web traffic
http.request.method == "GET"
http2.headers.method == "GET"
http.request.uri contains "FILE"
http2.headers.path contains "TERM"
udp.dstport == 8673             # EXAMPLE ONLY
tcp.port == 443                 # EXAMPLE ONLY
dns
tls
frame.number == 123            # EXAMPLE ONLY
```

## 11. Error-to-fix matrix

| Symptom from our actual practice | Cause / next action |
|---|---|
| PPPoE connected, but `ping 100.64.88.1` timed out | Wrong target; gateway in task is **`100.64.0.1`** |
| Ubuntu `134.115.148.x` while Windows has MikroTik `192.168.88.x` | VM bridged to **campus** NIC instead of MikroTik-connected NIC |
| VMware Virtual Network Editor requires admin | Don't override permissions; use permitted VM per-adapter settings or ask tutor to map correct bridge |
| VM `ens33 DOWN` | `sudo ip link set ens33 up`; then check carrier/DHCP |
| Ubuntu two defaults; campus metric 101, MikroTik 102 | `ip route get`; prefer MikroTik for lab target; temporary metric route as above |
| Ubuntu multiple `192.168.88.x` interfaces | Choose one VM adapter for stable return path and RADIUS server IP |
| Windows has `192.168.1.144`, Linksys page unreachable | Linksys factory DHCP/old lease or management IP changed; check `192.168.88.2`, cables, `ipconfig` |
| Linksys and MikroTik both DHCP | Disable Linksys DHCP; use **LAN-to-LAN** |
| Nano `Error writing etc/freeradius/...` | Missing leading `/` in absolute file path |
| FreeRADIUS `Expecting operator` for `aj` | Fix line to `aj Cleartext-Password := "student123"` |
| FreeRADIUS `Ready to process requests` but phone fails | Debug does not prove login; look for `Access-Request` and `Access-Accept` |
| Phone doesn't prompt credentials | Wrong/open radio (`ath1` security Disabled) or cached wrong SSID; split SSID and configure the active radio |
| Linksys RADIUS IP `.249` vs Ubuntu `.250` | VM had both addresses; check routing and packet receipt; don't guess or change at random |
| Phone `Couldn't get IP address` | First check RADIUS success; **only then** debug MikroTik DHCP, AP bridge, LAN cables |
| `Address already in use` FreeRADIUS | Stop the existing service/debug process before `freeradius -X` |
| SSH command `/user` fails in Ubuntu shell | RouterOS commands require `[admin@MikroTik] >` prompt |
| WPA key found but `ip` filter empty | Enable Wireshark 802.11 decryption; exact `PASSWORD:ESSID`, confirm handshake |
| TLS HTTP objects mostly absent | Load supplied TLS key log and reload Wireshark |
| Wrong packet number / flag | Read question precisely: GET request vs response, braces/case/units/source port |

## 12. Practice-only self-checks

> **These are from the uploaded practice notes, NOT verified answers for the real test.** Recalculate from the provided authorised practice PCAP each time.

| Practice task | Self-check / caution |
|---|---|
| Type A beacon / SSID / retries | Previous submitted values `31`, `wirelessWPAtest`, `50` were **only partially correct**. Recompute; format decimal as requested. |
| Type B TLS/HTTP | `USA`; `driveyourdream`; original GET request **13588** |
| Type C WPA (practice capture) | Recovered example key `drpepperdrinker`; first `common.css` GET **2466** |
| Type D exfil | Inside `DMFlag{}`: `10gQAZ%qgXX5`; source port `51822`; interval `20` seconds |

### Final practice checklist

- [ ] Can build physical PPPoE topology, choose assigned **fibre/copper** credentials, ping `100.64.0.1` from Linux VM.
- [ ] Can distinguish campus from MikroTik NICs using `ip route get` and avoid ambiguous multiple VM bridges.
- [ ] Can configure Linksys as LAN-only AP, find both 2.4/5 GHz radios, and protect the SSID actually joined.
- [ ] Can validate FreeRADIUS configuration, test local user and show real Wi-Fi `Access-Accept`.
- [ ] Can identify authentication failure versus post-authentication DHCP failure.
- [ ] Can import and demonstrate MikroTik SSH public key and reset devices after tutor approval.
- [ ] Can verify/decompress PCAP, identify question type, use precise filters, decrypt authorised WEP/WPA or TLS keys, and copy case-sensitive answers accurately.

**Use the official lab exercises for the invigilated assessment. This repository copy is for pretest preparation and review only.**
