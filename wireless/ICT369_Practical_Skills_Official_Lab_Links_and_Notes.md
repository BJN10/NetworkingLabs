# ICT369 — Practical Skills Tests (2026 S2)

> **Companion revision notes — NOT an authorised exam resource.** Prepared for pre-test practice from the 2026 assessment description, the supplied packet-analysis notes and the **official GitHub lab files**. The actual invigilated test permits the **official linked labs** but **not other notes or online resources** unless the tutor explicitly allows them. Do not open this file or your own GitHub copy during the closed-book test without written approval.
>
> These notes preserve the official lab links rather than copying/rehosting the university's complete lab pages. Lab credentials shown below are *classroom exercise credentials*, not private service passwords. Never reuse them on a real deployment or store live secrets on a public repository.

## Assessment: know both tests

| Test | Value | Duration | What must work |
|---|---:|---:|---|
| Physical Lab Competency | 15% | 90 minutes (core build expected within ~45 minutes) | Data-centre cabling -> Linux VM can ping `100.64.0.1`; working FreeRADIUS Wi-Fi username/password authentication |
| Wireless Packet Capture and Diagnostics | 15% | 90 minutes | Filters/statistics; WEP and WPA recovery; apply keys; TLS decryption with a supplied key file; identify exact evidence and flags |

You do not know the order of these two tests until entering the lab. Test 1 is demonstrated to the tutor; Test 2 is graded through the LMS quiz. The assessment says the tests are invigilated/closed-book, except that **the official labs listed here may be used**. Read the LMS rubric when published.

## The seven official assessment labs (same links supplied by the unit)

### Practical Test 1 — Physical Network / Wi-Fi Enterprise Authentication

1. [Broadband CPE Scenarios](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/broadband_cpe_senarios.md) — router vs access point; WAN/LAN cabling; DHCP/NAT; Linksys/DD-WRT.
2. [GPON / PPPoE](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/gpon_pppoe.md) — correct copper/fibre patch panel, PPPoE credentials, WAN connectivity, Apache and MikroTik port forwarding.
3. [Enterprise Authentication with FreeRADIUS](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/enterprise_authentication_with_FreeRADIUS.md) — `clients.conf`, RADIUS users, `freeradius -C`, debug output and WPA2-Enterprise.

### Practical Test 2 — Packet Capture / Diagnostics

4. [RTS/CTS and Network Analysis](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/rts_cts_and_network_analysis.md) — management/control/data frames, monitor mode, Wireshark packet inspection.
5. [WEP](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wep.md) — controlled WEP lab; capture IVs, recover and apply WEP key.
6. [WPA](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wpa.md) — capture four-way handshake, wordlist test and apply WPA key.
7. [Network Forensics](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/network_forensics.md) — TCP reconstruction, HTTP objects, TLS key logging and decrypted analysis.

**Official source precedence:** test question and current LMS rubric > linked official lab for that test > these personal revision notes. Hardware/firmware may differ; verify the actual port/setting rather than relying on a memorised screenshot.

---

## Test 1 — Physical lab: complete practical run-through

### 1. Before touching the router: understand the network

```text
                         Campus ISP / MikroTik CCR
                             (PPPoE server)
                              /            \
                    GPON FIBRE A-H      COPPER I-P
                         |                  |
                    OLT / ONU          direct copper
                         |                  |
                    MikroTik WAN1     MikroTik WAN1
                          \                 /
                         MikroTik LAN (192.168.88.1)
                                    |
                    Ubuntu VM / FreeRADIUS server
                         (bridged Ethernet)
                                    |
                          Wi-Fi AP (Linksys)
                                    |
                         Wi-Fi clients (802.1X)
```

The physical cabling and available devices determine the *actual* topology. In a normal group GPON lab, the two paths use **two different MikroTik devices**. For the competency test, follow your tutor's allocated equipment/patch panel. **Do not connect the campus data-centre feed to a LAN port:** the feed enters the router's **WAN / ether1**. The Ubuntu VM attaches through a **LAN port (ether2–ether5)** and the host's correct Ethernet NIC.

### 2. Cable, bridge one VM network interface, and check IP

1. Trace the correct copper or GPON patch-panel port in the data centre. Record the **label and number**. Use its corresponding PPPoE username/password from the *matching lab table below*.
2. Patch the data-centre uplink to **MikroTik WAN/ether1**, and a **LAN/ether2–5** port to the host/VM access Ethernet network.
3. In VirtualBox (or the installed VM system), bridge **ONE** Ubuntu VM network adapter to the **physically connected Ethernet NIC**, not Wi-Fi or an unrelated campus NIC. Disable other unnecessary VM adapters for a predictable route. This refers to Ethernet/IP connectivity, **not** a Wi-Fi adapter or a monitor-mode `wlan0`.
4. In Ubuntu:

```bash
ip -br addr
ip route
ip route get 192.168.88.1
ping -c 3 192.168.88.1
```

5. The router's default LAN network is `192.168.88.0/24`, usually router `192.168.88.1` and client `192.168.88.2–254` when DHCP is enabled. The Ubuntu interface name could be `ens33`, `enp0s3` or another: **check, do not guess**.
6. From the VM open `http://192.168.88.1`; MikroTik configuration normally uses **Quick Set**. Default router login varies between lab pages/device states: try username `admin` with blank password when reset (GPON lab); the Broadband CPE page also advises checking the router sticker or trying `admin` for the password. Do not assume a universal password.

### 3. GPON / PPPoE classroom credentials: ALL pods

**Important:** the **GPON lab** has its *own* credential map. It is not identical to the earlier Broadband CPE lab.

#### GPON fibre — A–H (PPPoE AND ONU credentials)

| GPON label | PPPoE username | PPPoE password | ONU label | ONU username | ONU password |
|---|---|---|---|---|---|
| A | `alpha` | `alpha` | alpha | `alpha` | `alpha` |
| B | `bravo` | `bravo` | bravo | `bravo` | `bravo` |
| C | `charlie` | `charlie` | charlie | `charlie` | `charlie` |
| D | `delta` | `delta` | delta | `delta` | `delta` |
| E | `echo` | `echo` | echo | `echo` | `echo` |
| F | `foxtrot` | `foxtrot` | foxtrot | `foxtrot` | `foxtrot` |
| G | `golf` | `golf` | golf | `golf` | `golf` |
| H | `hotel` | `hotel` | hotel | `hotel` | `hotel` |

#### Direct copper — I–P (GPON lab, NO ONU)

| Pod | PPPoE username | PPPoE password | Data centre port |
|---|---|---|---:|
| I | `india` | `india` | 16 |
| J | `juliett` | `juliett` | 17 |
| **K** | **`kilo`** | **`kilo`** | **18** |
| L | `lima` | `lima` | 19 |
| M | `mike` | `mike` | 20 |
| N | `november` | `november` | 21 |
| O | `oscar` | `oscar` | 23 |
| P | `papa` | `papa` | 24 |

**From our completed lab:** your copper allocation was **K**. That meant **PPPoE username `kilo`, password `kilo`, port 18**. This is a **past allocation**, not a guarantee for the next exam. Match the allocated port on exam day. **Port 22 is skipped** in this mapping. Note that the *earlier* Broadband CPE Scenarios file lists pods A–H and uses `beta` for B, while *GPON* uses `bravo` for B; **do not mix the two exercises**.

### 4. MikroTik PPPoE setup and verification

1. Open **Quick Set** at `192.168.88.1`. Configure **PPPoE** on the WAN connection with the **username/password of your allocated port**. Enable **NAT** and **DHCP Server** where required. Apply/Save.
2. In WebFig **Interfaces**, verify `pppoe-out1` is connected and has been issued a **`100.64.0.x`** WAN address.
3. From Ubuntu run:

```bash
ip route
ping -c 4 192.168.88.1   # LAN gateway reachable?
ping -c 4 100.64.0.1     # REQUIRED tutor check
```

4. If gateway ping works but `100.64.0.1` fails: inspect PPPoE session/credentials, patch-panel choice, WAN cable, NAT settings and route. Do not assume the VM must have a `100.64.x.x` address: **the WAN address belongs to the MikroTik**; the VM normally has a **`192.168.88.x` LAN** address.
5. `100.64.0.0/10` is **Shared Address Space** used for carrier-grade NAT; it is **not ordinary globally routable public IPv4**.

### 5. Optional GPON lab speed test / port forwarding (practise fully)

The GPON lab tests partner HTTP connectivity and downloads. This may support troubleshooting but is **additional to the explicitly stated competency requirement** (ping the server and working RADIUS Wi-Fi).

```bash
sudo apt update
sudo apt install -y apache2
sudo systemctl enable --now apache2
sudo systemctl status apache2 --no-pager
# The official GPON lab creates a 1000 MiB file (1 MiB x 1000):
dd if=/dev/zero of=output.dat bs=1M count=1000
sudo cp output.dat /var/www/html/
ls -lh output.dat /var/www/html/output.dat
curl -I http://127.0.0.1/output.dat
```

**Why it failed in our earlier session:** Apache was stopped, so your partner could not reach your web server. Starting Apache fixed the access. Later `output.dat` was missing on your VM; **Apache running does not generate the file**. You must execute `dd`, copy the file to `/var/www/html/`, and verify both paths.

Use the **actual** Ubuntu LAN IP (shown by `ip -br addr`) in the NAT target. The **following is MikroTik RouterOS syntax**, not a Linux shell command:

```routeros
/ip firewall nat add chain=dstnat in-interface=pppoe-out1 dst-port=80 action=dst-nat protocol=tcp to-address=192.168.88.X to-port=80
/ip firewall nat print
# If a bad NAT rule exists, identify its number above, then:
/ip firewall nat remove numbers=RULE_NUMBER
```

The official lab's SSH connection to MikroTik is:

```bash
ssh admin@192.168.88.1 -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa
```

On a partner's VM, for a **verified** partner WAN address only:

```bash
curl -I http://PARTNER_WAN_IP/
wget http://PARTNER_WAN_IP/output.dat
```

Router GUI addresses, Ubuntu LAN addresses and WAN/partner addresses are **different**; never substitute one blindly for another.

### 6. Linksys Wi-Fi AP — bridge, don't double-route

The CPE lab explains a typical split configuration:

- MikroTik **LAN** address `192.168.88.1` with DHCP/NAT.
- Linksys/DD-WRT **LAN** address `192.168.88.2`, gateway `192.168.88.1`.
- **Disable Linksys WAN connection and Linksys DHCP**; use **MikroTik LAN -> Linksys LAN**, not Internet/WAN, when configured as a plain AP.
- Linksys management credentials in the CPE lab: **username `root`, password `admin`**. If factory defaults or firmware differ, verify with the lab device.
- Unique SSID, appropriate radio/channel, reduced transmit power where requested (the lab suggests 5 mW to minimise classroom interference).
- Linksys can have functioning Wi-Fi without the **Ubuntu VM** showing `wlan0`. A VM's bridged Ethernet NIC (`ens33`) is enough for RADIUS backhaul; a physical Wi-Fi adapter is needed only when monitor mode/live Wi-Fi capture is required.

### 7. FreeRADIUS: configure Ubuntu and authenticate Wi-Fi users

**Goal:** Wi-Fi client -> AP (NAS/RADIUS client) -> Ubuntu FreeRADIUS server -> Accept/Reject -> Wi-Fi connection.

1. Check Ubuntu and AP have **reachable LAN addresses**. Make a note of **Ubuntu RADIUS server IP**, **AP management/source IP**, **SSID**. Do not assume values from an old lab remain valid.
2. Install FreeRADIUS:

```bash
sudo apt update
sudo apt install -y freeradius
```

3. Define your **AP as a RADIUS client** in the file used by the official lab:

```bash
sudo nano /etc/freeradius/3.0/clients.conf
```

Example (REPLACE values with actual lab AP IP and your own temporary shared secret):

```text
client lab_ap {
    ipaddr = 192.168.88.2
    secret = temporaryLabSharedSecret123
}
```

4. Create a **test user** in the user file the GitHub lab specifies:

```bash
sudo nano /etc/freeradius/3.0/users
```

Example for pre-exam practice only:

```text
ajtest   Cleartext-Password := "TestOnlyPassword123"
```

5. Stop the service, validate configuration, then start foreground debug (watch for **Access-Accept/Access-Reject**):

```bash
sudo systemctl stop freeradius
sudo freeradius -C
sudo freeradius -X
```

6. **In a second terminal or the AP's GUI**, enter the RADIUS **server IP**, usual RADIUS authentication port **1812**, and the **EXACT shared secret from `clients.conf`**; configure **WPA2-Enterprise** (or the supported enterprise method shown by the AP/lab). Do **not** use WPA2-Personal/PSK for the enterprise-authentication task.
7. On a Wi-Fi laptop/phone choose that SSID, set **EAP-TTLS / MSCHAPv2** where the lab specifies it, then enter the **RADIUS username and password** created in `users`. A lab may show no certificate requirement, but **real networks must validate the server certificate** against trusted identity; disabling validation is a controlled-classroom shortcut, not deployment advice.
8. Demonstrate one **successful** authentication and one **wrong-password rejection** to verify the service. Return to `freeradius -X` output for evidence.

**What the three different passwords mean:**

| Password | Entered on | Where it comes from |
|---|---|---|
| **PPPoE password** (e.g., `kilo`) | MikroTik PPPoE WAN | Given by fibre/copper port table |
| **RADIUS shared secret** | Both AP Enterprise settings and `clients.conf` | **You choose it**; values must match exactly |
| **Wi-Fi user's password** | Client device and FreeRADIUS user entry | **You create the user**; client must match |

No universally fixed FreeRADIUS user password or RADIUS shared secret is specified by the text of the official RADIUS lab. Don't invent one as if it were issued by the tutor.

### 8. RADIUS service issues

| Symptom | Check / correction |
|---|---|
| `freeradius -C` reports error | Fix the precise line/location of `clients.conf` or `users` |
| Address/port already in use with `freeradius -X` | Stop service first; lab suggests `ps -e \| grep radius` then stopping the old process |
| RADIUS client not found | AP packet source IP must be included as `ipaddr` in `clients.conf` |
| Invalid shared secret | AP's RADIUS secret must exactly match `clients.conf` |
| `Access-Reject` | Check identity, matching `Cleartext-Password`, EAP type and user configuration |
| Timeout / no request in debug | Check IP reachability, AP-to-server route and host firewall; authentication commonly uses UDP/1812 |
| Wireless login never prompts for a user | AP likely configured for **WPA-Personal** instead of **WPA-Enterprise** |

---

## Test 2 — Packet-analysis workflow (companion to the supplied PDF)

The uploaded **10-page 'ICT369 Practical Test 2 - One-Stop Notes' PDF** already contains detailed exercises, evidence examples, and troubleshooting. Use its **Question-Type Detector** and self-check sections for *practice*. Do not carry it into the test unless the tutor authorises it.

| Question type | Start | Key proof |
|---|---|---|
| **A. WLAN statistics** | Clear filter; count **all** packets | `wlan.fc.type_subtype == 0x0008` (beacons); `wlan.fc.retry == 1` (retry flag); proper **total** denominator |
| **B. TLS / HTTP** | Locate supplied `.pcapng` **and TLS key log** | Wireshark Preferences -> TLS -> (Pre)-Master-Secret log filename; reload; then `http || http2`; Export Objects -> HTTP |
| **C. WEP / WPA** | WEP needs IVs; WPA needs usable EAPOL handshake | `aircrack-ng` key recovery; enable IEEE 802.11 decryption; for WPA `wpa-pwd` value is `PASSWORD:ESSID`; prove decoded data |
| **D. Flag / exfiltration / delay** | Statistics -> Protocol Hierarchy/Conversations | Isolate flow; reassemble payload; read **source** port separately; timing = **Seconds Since Previous Displayed Packet** |

### Useful packet filters

```text
wlan.fc.type_subtype == 0x0008            # Beacon
wlan.fc.retry == 1                       # retry frame
wlan.fc.type == 2                        # data frame
wlan.fc.protected == 1                   # protected frame
eapol                                    # WPA 4-way handshake
http || http2                            # HTTP traffic after TLS setup
http.request.method == "GET"
http2.headers.method == "GET"
http.request.uri contains "common.css"
frame.number == N
udp.dstport == PORT
```

### Essential terminal patterns

```bash
pwd; ls -lh; file *
# Password entered at prompt; do not put complex secrets unquoted in shell:
openssl enc -d -aes-256-cbc -in INPUT.enc -out OUTPUT
echo $?          # immediately after the command to verify exit code
file OUTPUT
# If the file is compressed, decompress as needed:
bunzip2 -k capture.pcap.bz2
tar -tjf archive.tar.bz2
tar -xjf archive.tar.bz2
# WPA personal practice capture and wordlist:
aircrack-ng -w WORDLIST.txt -e ESSID CAPTURE.pcap
# WEP IV practice:
aircrack-ng capture.ivs
```

**Lab vs main-test values:** The supplied PDF's practice answers (such as `drpepperdrinker`, practice flags and specific frame numbers) apply only to those particular captures. **Do not paste practice answers into the actual test**. Answers and flags are case-sensitive; distinguish `0/O`, `1/l/I`, source/destination port, request/response frame, and required decimal formatting.

### Packet-analysis fixes we encountered

| Previous issue | Proven prevention |
|---|---|
| No `wlan0` in Kali VM | A bridged/wired VM interface does **not** provide a wireless radio. Live monitor-mode capture requires a compatible Wi-Fi adapter passed through to the VM. Offline `.pcap` work needs no Wi-Fi radio. |
| Typed a display filter inside `Ctrl+F` | Use the **top Wireshark Display Filter bar** for protocol expressions; `Ctrl+F` is packet searching |
| Previously filtered 'Displayed' used as denominator | Clear the filter; record **Packets** for all-frames total |
| WPA 'handshake visible' mistaken for plaintext recovered | Handshake visible does not prove success. Configure exact WPA password/SSID and verify `ip`/`http` appears |
| HTTP object found but wrong frame entered | Exported object may be on **response** packet; find `[Request in frame: N]` |
| `-o` or `-in` pasted as its own Linux command | These are **options** to `tshark` or `openssl`; paste the complete command |
| TLS data stayed encrypted / HTTP objects showed tiny redirect | Find the accompanying TLS key-log file; load it and reload the capture |
| Wrong flag/port/timing | Respect braces/prefix instructions, exact case, **source vs destination**, and **previous displayed** time |

---

## Our real GPON lab: mistakes and corrections

1. **Three VirtualBox network adapters bridged simultaneously**: you reported mixed routing/gateway behaviour. We reduced to **one correctly bridged interface**, identified as `ens33` in that session. Always examine `ip route` rather than guessing which NIC will be used.
2. **Gateway `192.168.88.1` vs WAN `100.64.0.x`**: different layers. Test first `ping 192.168.88.1`, then `ping 100.64.0.1` from Ubuntu; inspect PPPoE status if only the first works.
3. **Apache was not running**: partner could not fetch the page/file. Restarting it restored basic HTTP access.
4. **`output.dat` was missing**: starting the web server does not create that file. Run `dd`, copy to `/var/www/html/` and verify it exists.
5. **Partner IP confusion**: always verify the WAN IP using MikroTik `pppoe-out1`; the host's `192.168.88.x` is a LAN address and should not be inserted into the partner's URL. Your earlier `100.64.0.23` example should not be assumed to repeat in the test.
6. **Kali wired vs Wi-Fi**: wired Ethernet is sufficient for Linux routing/RADIUS/capture-file analysis; a real WLAN interface is only required to capture/attack your **authorised lab** Wi-Fi network live.
7. **CPE 'beta' vs GPON 'bravo'**: always follow the *specific* lab's credential table rather than a generic phonetic alphabet memory.
8. **Restore factory defaults** at the beginning/end as directed by the specific official lab and tutor; do not retain another student's router settings.

## Final practice checklist (before the invigilated test)

- [ ] Identify the *current* port and match *that* lab's PPPoE credentials.
- [ ] Explain data centre -> WAN `ether1`, router LAN -> Ubuntu VM / AP.
- [ ] Make a **single** correctly bridged Ubuntu VM adapter work, check address and default route.
- [ ] Log in to MikroTik, configure PPPoE, prove `ping -c 4 100.64.0.1` from VM.
- [ ] Make Apache and `output.dat` work end-to-end as additional GPON lab practice.
- [ ] Install/configure FreeRADIUS, check `freeradius -C`, demonstrate `Access-Accept` and rejection.
- [ ] Match RADIUS AP shared secret **and** wireless user credentials separately.
- [ ] For packet exam, practise the **four question types** from the supplied PDF without looking at practice answers first.
- [ ] Recheck required output syntax, case and flags before LMS submission.

_Last checked against the seven official GitHub lab files: 19 October 2026._
