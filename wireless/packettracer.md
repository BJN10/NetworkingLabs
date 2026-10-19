# ICT369 - Wireshark Practical Test 2: Four Questions - Step-by-Step Master Guide

**Murdoch University | ICT369 Wireless and Interactive Networks | Semester 2, 2026**

> **Purpose:** Follow the four practice-question workflows from beginning to end. The format mirrors the Physical Lab Competency Master Guide: **do a step -> check the result -> fix the common error -> continue**.
>
> **Important:** This is a **practice guide**, not a permitted exam reference. The assessment says that, during the invigilated test, you may use the *specified official lab pages* but not your own notes or other online resources. Do not open this Markdown in the test unless the teaching team explicitly authorises it.

**Physical test (separate guide):** [Open the ICT369 Physical Lab Competency Master Guide (PDF)](./ICT369_Physical_Lab_Competency_Master_Guide.pdf)

This page is **only about the four packet-analysis questions**. It does not repeat the physical lab setup.

## Assessment rules — read before practice

| Item | Requirement |
|---|---|
| Assessment | **Wireless Packet Capture Skills and Network Diagnostics** |
| Value and duration | **15%** of unit mark; **90 minutes** |
| Assessment mode | **LMS quiz**, automatically graded, invigilated/closed-book |
| Skills | Use Wireshark filters for statistics and stream data; recover/apply WEP and WPA keys; load a **provided TLS key file** to decrypt and analyse traffic |
| Allowed references during the actual test | **Only the four official unit lab pages listed below**, unless the teaching team gives different explicit instructions |
| Not permitted during the actual test | This personal Markdown guide, the accompanying PDF, independent online notes, or other resources unless authorised |

**Accuracy and case matter.** The LMS might ask for `31.0` (not `31`), `USA` (not `US`), the characters inside `DMFlag{...}` (not the braces), or a *request* packet number rather than a response. Compare `0` vs `O` and `1` vs `l` vs `I`. Do not paste sample answers into a different capture.

**Scope:** Sections Q1–Q4 are **Test 2 practice only**. A separate appendix contains the GPON credentials you specifically requested for *physical-test practice*; **GPON is not part of the stated four-lab Test 2 reference list**. These lab credentials appear on public educational pages and should never be used in real-world deployments.

## 1. The four questions at a glance

| Question | Recognise it by | You must produce |
|---|---|---|
| **Q1 - 802.11 statistics** | beacons, SSID, retransmissions/retries | Beacon percentage (1 decimal), exact SSID, retry count |
| **Q2 - TLS / HTTP forensics** | SSL/TLS, key log, web search, image, second-largest file | Search location/country, recovered image text, original HTTP GET frame |
| **Q3 - Wi-Fi decryption** | WPA/WEP, EAPOL, wordlist, Aircrack, `common.css` | Recovered Wi-Fi key and requested decrypted-traffic frame |
| **Q4 - Exfiltration / timing** | flag, secret message, source port, consistent delay | Message **inside** the flag braces, source port, delay in seconds |

### Four officially permitted assessment lab links

- [RTS/CTS and Network Analysis using Wireshark](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/rts_cts_and_network_analysis.md) - Q1
- [Network Forensics](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/network_forensics.md) - Q2 and Q4 techniques
- [WEP](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wep.md) - Q3 WEP variant
- [WPA](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wpa.md) - Q3 WPA variant

---

## 2. Universal preparation - do this before every question

### Step 1 - Open Kali/Ubuntu and make four folders

Open **Terminal**. Copy one entire command at a time:

```bash
mkdir -p ~/ICT369_Prac2/{Q1,Q2,Q3,Q4}
cd ~/ICT369_Prac2
pwd
ls -lh
```

**Checkpoint:** The current folder is `~/ICT369_Prac2`, and you have Q1, Q2, Q3 and Q4 folders.

### Step 2 - Know the three Wireshark controls

| Control | What it actually does | Common mistake |
|---|---|---|
| **Display Filter** at the top | Shows only matching packets | Entering a filter in Ctrl+F instead |
| **Ctrl+F - Find Packet** | Searches a string, value or bytes | Thinking it sets a filter |
| **Ctrl+G - Go to Packet** | Jumps to a specific frame | Confusing request frame with response frame |

**Check:** With no display filter, `Packets` and `Displayed` at the bottom of Wireshark should match.

### Step 3 - Download, decrypt and verify in this order

Each question below gives the exact filename and URL. The general pattern is:

```bash
wget -O INPUT.enc 'OFFICIAL_URL'
openssl enc -d -aes-256-cbc -in INPUT.enc -out OUTPUT
# Type the exact volume password from that LMS practice question at the password prompt.
echo $?
file OUTPUT
ls -lh
```

**Checkpoint:** `echo $?` is `0` immediately after the OpenSSL command, and `file OUTPUT` recognises the expected PCAP/PCAPNG or compressed archive.

**Do not** open `.enc` or `.bz2` directly as a PCAP. Decompress an archive first. The password is not the Wi-Fi WPA password: the OpenSSL **volume password** unlocks the practice file; the WPA **network password** is recovered from the capture later.

**If you get `bad decrypt`:** check the password and original input. An older practice file might require the OpenSSL legacy `-md md5` derivation, but **do not apply this to every file**. Use it only as a troubleshooting fallback after the normal command fails and delete the failed output before retrying.

**VirtualBox/VM note:** For analysis of supplied `.pcap/.pcapng` files, a virtual **wired/NAT adapter is enough** to download files. A `wlan0` interface or USB monitor-mode adapter is **not required** to examine an existing capture. Those are needed only for a task that explicitly requires *live wireless capture*.

### Step 4 - If a terminal command or file appears broken

| Symptom | First diagnostic | Fix |
|---|---|---|
| `No such file or directory` | `pwd`; `ls -lah`; `file *` | Correct the working folder, extension, filename or capital letters. `pcap` is not `pnap`. |
| `echo$?` fails | Check spaces | Type `echo $?` **with a space**, immediately after the command you want to check. |
| `-in`, `-out`, or `-o` says command not found | Inspect how the command was pasted | Paste the **whole** `openssl` or `tshark` command on one line; options are not commands. |
| Nothing appears while entering OpenSSL password | This is normal terminal behaviour | Type the password exactly and press Enter; the prompt does not show characters. |
| Decrypted file says only `data` | Run `file OUTPUT` and check OpenSSL status | Treat as suspect; preserve `.enc`, remove only failed output, then retry correctly. |
| Wireshark sees no wireless adapter (`wlan0`) | Are you analysing a downloaded file or capturing live radio frames? | For **existing PCAPs**, a wired/NAT VM is sufficient. For **live monitor mode**, pass through a supported USB wireless adapter. |
| TLS objects missing / only 242-byte redirect | Check Wireshark TLS keylog setting | Supply the extracted key log, **Apply**, **Ctrl+R**, then check `http || http2` again. |
| Password contains `!`, `$`, or `&` | Did the shell expand it? | Prefer the interactive OpenSSL password prompt rather than embedding the value in a shell command. |

---

# Q1 - 802.11 Beacons, SSID and Retransmission Count

**Goal:** Answer three statistical fields correctly. The most common errors are leaving an old filter active and entering a whole-number percentage without its required decimal.

**Official lab:** [RTS/CTS and Network Analysis](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/rts_cts_and_network_analysis.md)

### Step 1 - Download the Q1 practice file

```bash
cd ~/ICT369_Prac2/Q1
wget -O lab_red.pcap.enc 'https://github.com/SCH-IT-MurdochUni/NetworkingLabs/raw/refs/heads/main/Reusable_Learning_Objects/.test/.misc/lab_red.pcap.enc'
ls -lh
```

**Checkpoint:** `lab_red.pcap.enc` appears in the Q1 folder.

### Step 2 - Decrypt the capture

```bash
openssl enc -d -aes-256-cbc -in lab_red.pcap.enc -out lab_red.pcap
```

Type the **Q1 volume password supplied by LMS** when prompted (input stays invisible). Then immediately run:

```bash
echo $?
file lab_red.pcap
```

**Checkpoint:** exit status `0`; `file` identifies a packet capture. If the correct practice volume password fails with default settings, try the legacy digest *only as a fallback*:

```bash
rm -f lab_red.pcap
openssl enc -d -aes-256-cbc -md md5 -in lab_red.pcap.enc -out lab_red.pcap
echo $?
file lab_red.pcap
```

### Step 3 - Open the capture and record the total

```bash
wireshark lab_red.pcap
```

1. Clear any existing **Display Filter** at the top.
2. Read the `Packets` number at the bottom.
3. Confirm `Packets = Displayed`.
4. Write down **TOTAL = ______**.

**Do not continue** with a filtered total; it changes your percentage denominator.

### Step 4 - Count Beacon frames

1. In **Display Filter**, enter:

   ```wireshark
   wlan.fc.type_subtype == 0x0008
   ```

2. Press Enter.
3. Read **Displayed = ______**. This is **BEACONS**.
4. A Beacon is a **management frame** advertising the access point.

### Step 5 - Calculate Beacon percentage to one decimal place

**Formula:** `(BEACONS / TOTAL) * 100`

Replace the two example numbers with your own **real counts**:

```bash
awk 'BEGIN {printf "%.1f\n", (3100/10000)*100}'
```

**Checkpoint:** You get a number such as `31.0`, **not** `31` and **not** `31.0%` if LMS requests one decimal and no percentage sign. `3100` and `10000` above are examples, **not confirmed counts from Q1**.

### Step 6 - Find the exact SSID

1. Leave the Beacon filter applied.
2. Click a Beacon packet belonging to the relevant AP.
3. Expand **IEEE 802.11 Wireless Management** -> **Tagged parameters** -> **SSID parameter set**.
4. Copy **SSID = ______** exactly, including uppercase/lowercase.
5. If multiple SSIDs occur, identify the one requested by LMS, not simply the first one.

Optional terminal cross-check:

```bash
tshark -r lab_red.pcap -Y 'wlan.fc.type_subtype == 0x0008' -T fields -e wlan.ssid | sed '/^$/d' | sort | uniq -c | sort -nr
```

### Step 7 - Count retransmitted/retry frames

1. **Clear the Beacon filter.**
2. Enter:

   ```wireshark
   wlan.fc.retry == 1
   ```

3. Read `Displayed = ______` as the retry-frame count.
4. Unless the question specifically asks for **data-frame** retries, do **not** add a data-frame condition.

Optional terminal verification:

```bash
tshark -r lab_red.pcap -Y 'wlan.fc.retry == 1' -T fields -e frame.number | wc -l
```

### Step 8 - Check Q1 before answering

- [ ] Beacon percentage came from `BEACONS / ALL frames`, not a filtered subset.
- [ ] Percentage is **one decimal** with no `%` if required.
- [ ] SSID matches exact case and the correct AP.
- [ ] Retry number comes from `wlan.fc.retry == 1`.

**Practice-history caution:** The previous Q1 answers `31`, `wirelessWPAtest` and `50` were **only partially marked correct**. They are **not a verified answer set**. Recompute from the capture and obey the decimal instruction.

---

# Q2 - TLS / HTTP Search Evidence, Second-Largest Object, Original GET Frame

**Goal:** Decrypt the supplied archive, load the included TLS key log in Wireshark, inspect web-search evidence, retrieve the second-largest HTTP object and locate the frame that **requested** it.

**Official lab:** [Network Forensics](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/network_forensics.md)

### Step 1 - Download Q2

```bash
cd ~/ICT369_Prac2/Q2
wget -O pp.tar.bz2.enc 'https://github.com/SCH-IT-MurdochUni/NetworkingLabs/raw/refs/heads/main/Reusable_Learning_Objects/.test/.misc/pp.tar.bz2.enc'
ls -lh
```

### Step 2 - Decrypt and extract

```bash
openssl enc -d -aes-256-cbc -in pp.tar.bz2.enc -out pp.tar.bz2
# Enter the exact Q2 volume password from LMS.
echo $?
file pp.tar.bz2
tar -tjf pp.tar.bz2
tar -xjf pp.tar.bz2
ls -lah
```

**Checkpoint:** You can see the practice capture **`pp_ogl.pcapng`** and TLS secret file **`sslkeylog.log`**. Do not proceed with an invalid archive.

### Step 3 - Open the real PCAPNG

```bash
wireshark pp_ogl.pcapng
```

### Step 4 - Tell Wireshark where the TLS secrets are

1. Go to **Edit -> Preferences -> Protocols -> TLS**.
2. Find **(Pre)-Master-Secret log filename**.
3. Browse to `~/ICT369_Prac2/Q2/sslkeylog.log`.
4. Click **Apply -> OK**.
5. Press **Ctrl+R** to reload the capture.
6. Try the Display Filter:

   ```wireshark
   http || http2
   ```

**Checkpoint:** You can inspect decrypted HTTP/HTTP2 details or decrypted TLS data. If only a tiny redirect appears in HTTP objects, the key log may not be loaded correctly.

### Step 5 - Investigate the location/search-history clue

Try the following **one at a time** in the Display Filter box:

```wireshark
http.request.method == "GET"
```

```wireshark
http2.headers.method == "GET"
```

```wireshark
http2.headers.path contains "/search?q="
```

```wireshark
http2.headers.path contains "pikes"
```

1. Inspect **request paths**, URL arguments and map/search API requests.
2. Do not assume every search goes to `/search?q=`; look at other path patterns such as maps/autocomplete.
3. Identify the referenced mountain/location using the decrypted evidence and enter the required **three-letter country code**.

**Practice checkpoint:** This capture led to **Pikes Peak, Colorado, USA**. Country answer in the previous practice was `USA`. A different exam capture may have a different country.

Optional TShark request export (paste as **one complete command**):

```bash
tshark -r pp_ogl.pcapng -o tls.keylog_file:sslkeylog.log -Y 'http2.headers.method == "GET"' -T fields -e frame.number -e http2.headers.path > requests.txt
```

Then search the generated text:

```bash
grep -iE 'pikes|peak|colorado' requests.txt
```

**Error to avoid:** `-o` is a **TShark option**. Never enter `-o tls.keylog_file:...` as a new shell command.

### Step 6 - Recover the second-largest HTTP object

1. **Clear Wireshark's Display Filter** first.
2. Go to **File -> Export Objects -> HTTP**.
3. Leave the **Export Objects Text Filter EMPTY**.
4. Choose **All Content-Types**.
5. Expand/maximise the object list.
6. Click the **Size** column until objects are **largest first**.
7. Select the **SECOND** largest object (rank 2, not rank 1).
8. Save it and open it.
9. Read the words in the image and format them **exactly** as the question says.

**Practice checkpoint:** The second-largest file was `640x440_DRIVE.jpg` and contained **drive your Dream**. The requested lowercase/no-spaces answer was `driveyourdream`.

**Common mistake:** Typing `HTTP2` in the Export Objects **Text Filter** makes results disappear; it does **not** switch protocol.

### Step 7 - Find the original GET frame number

1. In **File -> Export Objects -> HTTP**, note the **packet number** for the recovered image.
2. Press **Ctrl+G** and enter that packet number.
3. Expand **Hypertext Transfer Protocol** in packet details.
4. Look for **`[Request in frame: N]`**.
5. Enter **N**, not the response's packet number, if LMS asks for the frame where the image was *requested*.

**Practice checkpoint:** Object response packet `25630`, but the original GET **request frame was `13588`**.

### Step 8 - Check Q2 before answering

- [ ] TLS key log successfully loaded; HTTP data actually decrypted.
- [ ] Search/location answer is based on decoded requests.
- [ ] Object chosen is **second-largest**.
- [ ] Text casing/spaces match LMS instruction.
- [ ] Frame number comes from **Request in frame**, not HTTP `200 OK` response.

**Previous practice self-check:** `USA` | `driveyourdream` | `13588`.

---

# Q3 - Recover a WPA/WEP Key, Decrypt Wi-Fi Frames and Find a GET Request

**Goal:** Identify whether the provided task is WPA or WEP. For the supplied Q3 practice example, recover a **WPA/WPA2-Personal** password with Aircrack, enable Wireshark 802.11 decryption, and find the first HTTP GET for `common.css`.

**Official labs:** [WPA](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wpa.md) and [WEP](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wep.md)

### Step 1 - Download the WPA practice PCAP

```bash
cd ~/ICT369_Prac2/Q3
wget -O eggtopia_anon_practice.pcap.bz2.enc 'https://github.com/SCH-IT-MurdochUni/NetworkingLabs/raw/refs/heads/main/Reusable_Learning_Objects/.test/.misc/eggtopia_anon_practice.pcap.bz2.enc'
ls -lh
```

### Step 2 - Decrypt and decompress

```bash
openssl enc -d -aes-256-cbc -in eggtopia_anon_practice.pcap.bz2.enc -out eggtopia_anon_practice.pcap.bz2
# Enter Q3 volume password from LMS.
echo $?
file eggtopia_anon_practice.pcap.bz2
bunzip2 -k eggtopia_anon_practice.pcap.bz2
file eggtopia_anon_practice.pcap
ls -lh
```

**Checkpoint:** `eggtopia_anon_practice.pcap` exists and `file` recognises it as a capture.

### Step 3 - Inspect the WPA handshake and SSID

```bash
wireshark eggtopia_anon_practice.pcap
```

1. In **Display Filter**, enter:

   ```wireshark
   eapol
   ```

2. Check the capture contains authentication-handshake evidence.
3. Look at **Beacon/Probe** frames to confirm the **exact SSID**, including capitalisation.
4. The supplied practice network is `Eggtopia134321`.

**Checkpoint:** You have identified the right target ESSID and handshake. Seeing EAPOL alone does **not** mean data packets are already decrypted.

### Step 4 - Download the supplied password list

```bash
wget -O leaked_password_list_clean.txt 'https://raw.githubusercontent.com/SCH-IT-MurdochUni/NetworkingLabs/refs/heads/main/Reusable_Learning_Objects/.test/.misc/leaked_password_list_clean.txt'
ls -lh leaked_password_list_clean.txt
```

### Step 5 - Recover the WPA passphrase using Aircrack

```bash
aircrack-ng -w leaked_password_list_clean.txt -e Eggtopia134321 eggtopia_anon_practice.pcap
```

**Checkpoint:** Aircrack says **`KEY FOUND!`** and shows the matching passphrase. Copy it **exactly**; do not guess from a similar word.

**Practice self-check:** `drpepperdrinker`.

### Step 6 - Apply the recovered key in Wireshark

1. Go to **Edit -> Preferences -> Protocols -> IEEE 802.11**.
2. Tick **Enable decryption**.
3. Click **Edit** beside **Decryption Keys**.
4. Add a key of type **`wpa-pwd`**.
5. Enter **`PASSWORD:ESSID`** (both must match exactly).
6. For this practice file use:

   ```text
   drpepperdrinker:Eggtopia134321
   ```

7. Save, close Preferences and press **Ctrl+R** to reload.
8. Try these Display Filters **one at a time**:

   ```wireshark
   ip
   ```

   ```wireshark
   http
   ```

**Checkpoint:** Higher-layer packets (such as IP/HTTP) now become visible, demonstrating **successful payload decryption**.

**Big previous mistake:** `drpeeperdrinker` was entered with an extra `e`. That incorrect key meant **zero useful decrypted HTTP packets**, even though the EAPOL handshake was visible.

### Step 7 - Find the FIRST HTTP GET for `common.css`

In Wireshark Display Filter:

```wireshark
http.request.method == "GET" && http.request.uri contains "common.css"
```

1. Apply the filter **after** you have proved decryption works.
2. Read the **No.** column for the *first* displayed matching GET request.
3. Enter the **frame number only**.

**Practice self-check:** first GET for `/style/common.css` = frame `2466`.

### Step 8 - If the real question is WEP instead of WPA

Do **not** use the WPA wordlist steps for an already supplied WEP `.ivs` file:

1. Examine the provided `capture.ivs` or `capture.cap` filename.
2. Recover the WEP key **only on your authorised lab/supplied capture**:

   ```bash
   aircrack-ng capture.ivs
   # or, if a full WEP capture was supplied:
   aircrack-ng capture.cap
   ```

3. Wait for the tool's recovered WEP key.
4. In **Wireshark -> Edit -> Preferences -> Protocols -> IEEE 802.11**, tick **Enable decryption**.
5. Add decryption key **type `wep`**, value **the recovered hexadecimal WEP key**.
6. Reload and verify decrypted higher-layer packets, then apply the question's exact filter.

**Do not attempt live monitor-mode capture for an offline provided PCAP/IVS question.** Live-capture tasks require authorised test equipment and a compatible wireless adapter.

### Step 9 - Check Q3 before answering

- [ ] Correct target protocol **WPA or WEP** identified.
- [ ] Correct capture, ESSID and supplied wordlist used.
- [ ] `KEY FOUND!` (or equivalent WEP recovery) actually appeared.
- [ ] Key added in the **correct Wireshark key type**.
- [ ] Higher-layer data is decrypted, not merely EAPOL displayed.
- [ ] Answer is the requested **GET request frame**, not another packet.

**Previous WPA practice self-check:** recovered key `drpepperdrinker`; GET request frame `2466`.

---

# Q4 - Recover a Secret Flag, Source Port and Message Timing

**Goal:** Find small, repetitive, suspicious traffic; reconstruct the flag from payload bytes; read the **source** port and calculate the interval using **previous displayed packet** timing.

**Related official lab:** [Network Forensics](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/network_forensics.md). The exact exfiltration example below is from the supplied LMS practice question.

### Step 1 - Download Q4

```bash
cd ~/ICT369_Prac2/Q4
wget -O 2987.pcap.enc 'https://github.com/SCH-IT-MurdochUni/NetworkingLabs/raw/refs/heads/main/Reusable_Learning_Objects/.test/.misc/2987.pcap.enc'
ls -lh
```

### Step 2 - Decrypt and verify

```bash
openssl enc -d -aes-256-cbc -in 2987.pcap.enc -out 2987.pcap
# Enter Q4 volume password from LMS.
echo $?
file 2987.pcap
wireshark 2987.pcap
```

**Checkpoint:** Valid PCAP opens in Wireshark.

### Step 3 - Find suspicious low-volume conversations

1. Clear the existing Wireshark Display Filter.
2. Go to **Statistics -> Protocol Hierarchy**.
3. Look for protocol types that may carry unusual payloads.
4. Go to **Statistics -> Conversations**.
5. Look at repeated **IP address / port pairs**, especially small flows with regular timing.
6. **Do not assume** the biggest protocol by volume contains the flag.

**Practice clue:** repeated **UDP** datagrams to destination port `8673`.

### Step 4 - Filter only the suspicious messages

In Wireshark Display Filter, enter:

```wireshark
udp.dstport == 8673
```

**Checkpoint (this practice capture):** 20 matching packets. Source IP `10.0.2.15`; destination IP `1.1.1.1`; destination UDP port `8673`.

For a different file use its **actual** suspicious port instead of copying `8673`.

### Step 5 - Recover the flag text from the UDP payloads

**GUI method**

1. Select the first displayed UDP packet.
2. Expand **User Datagram Protocol** and **Data**.
3. Inspect the bytes shown under packet details / bytes.
4. Record payload bytes from every matching packet **in frame order**.
5. Reconstruct the text and look for the `DMFlag{...}` pattern.

**Terminal method for this exact practice pattern:**

```bash
tshark -r 2987.pcap -Y 'udp.dstport == 8673' -T fields -e data
```

For this capture each data field has **one ASCII character and a line-feed byte `0a`**. Decode just the first byte of each packet:

```bash
tshark -r 2987.pcap -Y 'udp.dstport == 8673' -T fields -e data | cut -c1-2 | tr -d '\n' | xxd -r -p; echo
```

**Checkpoint:** Practice message was `DMFlag{10gQAZ%qgXX5}`.

**CRITICAL format rule:** If LMS already prints `DMFlag{ [your answer] }`, enter **only** `10gQAZ%qgXX5`. Do **not** type the prefix or braces again. Preserve uppercase/lowercase and `%`.

**Caution:** The `cut -c1-2` trick is valid only because the **practice** packets contain one character plus newline. For a different payload format, inspect it before truncating bytes.

### Step 6 - Read the SOURCE port (not destination)

1. Click any filtered suspicious UDP packet.
2. Expand **User Datagram Protocol**.
3. Record **Source Port = ______**.
4. Verify that it is **not** the Destination Port.

**Practice self-check:** Source Port `51822`; Destination Port `8673`. If asked for the **source**, answer `51822`.

### Step 7 - Find the regular delay between messages

1. **Leave `udp.dstport == 8673` active** so only relevant messages are displayed.
2. Select **View -> Time Display Format -> Seconds Since Previous Displayed Packet**.
3. Compare the times on consecutive suspicious packets (ignore the first one if it lacks a previous displayed packet).
4. Record the consistent delay in **seconds**.

**Checkpoint:** Practice interval was approximately **20 seconds**. If LMS asks for a number only, enter `20`.

**Common mistake:** Choosing **Seconds Since Previous Captured Packet** includes irrelevant packets and produces inconsistent intervals. Choose **Previous Displayed Packet** *after filtering*.

### Step 8 - Check Q4 before answering

- [ ] Suspicious flow was identified using packet evidence, not guessed.
- [ ] Flag bytes decoded in chronological capture order.
- [ ] Flag response contains only the substring LMS requests.
- [ ] Port is **source**, not destination.
- [ ] Delay measured after filtering using **previous displayed packet**.

**Previous practice self-check:** flag contents `10gQAZ%qgXX5` | source port `51822` | delay `20`.

---

## 3. One-page quick filter sheet

| Task | Display filter |
|---|---|
| Beacon | `wlan.fc.type_subtype == 0x0008` |
| Retries | `wlan.fc.retry == 1` |
| 802.11 data frames | `wlan.fc.type == 2` |
| Protected 802.11 frames | `wlan.fc.protected == 1` |
| WPA handshake | `eapol` |
| Decoded IP | `ip` |
| Decrypted HTTP | `http` |
| HTTP/2 | `http2` |
| HTTP/1.1 GET | `http.request.method == "GET"` |
| HTTP/2 GET | `http2.headers.method == "GET"` |
| URL contains common.css | `http.request.uri contains "common.css"` |
| TCP port | `tcp.port == 80` (replace 80 if needed) |
| UDP destination port | `udp.dstport == 8673` (replace 8673 if needed) |
| DNS | `dns` |
| TLS | `tls` |
| Exact frame | `frame.number == 13588` (replace value if needed) |

## 4. Error -> correction table

| Symptom | Fix |
|---|---|
| `No such file or directory` | Check `pwd`, `ls -lh`, exact filename and extension. |
| Cannot download practice file | Check VM internet/DNS; use a working NAT or bridged adapter for downloading. |
| `bad decrypt` | Re-check exact **volume** password and AES command; check MD5 only for an older file when needed. |
| Wireshark cannot open output | Verify `echo $?` and `file OUTPUT`, then decompress any `.bz2`/`.tar.bz2`. |
| Filters have no result | Ensure expression is in **Display Filter**, not Ctrl+F. |
| Wrong Beacon percent | Clear filters before recording the **full** packet count; format one decimal. |
| Export Objects lists only redirect | Load supplied **TLS key log**, apply, then Ctrl+R. |
| Export Objects suddenly empty | Clear Export Objects **Text Filter** (do not type `HTTP2` there). |
| WPA key found but HTTP still hidden | Correct `wpa-pwd`, exact `PASSWORD:ESSID`, **Enable decryption**, reload. |
| Wrong HTTP frame | Find **`[Request in frame: N]`** in the object response packet. |
| Wrong port | Read **Source Port** if the question says source; destination is different. |
| Irregular timing | First filter the suspicious flow, then set **Seconds Since Previous Displayed Packet**. |
| Flag rejected | Check braces/prefix, case, punctuation, spaces, `0/O`, `1/l/I`. |

## 5. Final 90-minute practice plan

| Time | Do this |
|---|---|
| 0-5 min | Read question wording, note required output formats, identify files and passwords. |
| 5-20 min | Q1 - total, Beacons, percentage, SSID, retries. |
| 20-45 min | Q2 - unpack, load TLS key log, search, object, original GET frame. |
| 45-65 min | Q3 - WPA/WEP recovery, Wireshark decryption, requested GET frame. |
| 65-80 min | Q4 - suspicious flow, reconstruct flag, source port and timing. |
| 80-90 min | Recheck the evidence behind each answer and the exact LMS format. |

### Last check before entering any answer

- [ ] It came from the **current** packet capture, not a memorised practice answer.
- [ ] I used the right question's files and any supplied key/wordlist.
- [ ] I can point to a precise packet, field, displayed count or verified tool output.
- [ ] The exact LMS format is satisfied (decimal, uppercase/lowercase, braces, units).
- [ ] I checked confusing characters (`0`/`O`, `1`/`l`/`I`).

> **Practice answers are for self-checking only.** Your invigilated test may use different captures, network names, passwords, ports and answers. Practise reproducing the workflow rather than memorising values.

---

# Appendix A — GPON / PPPoE classroom credentials (PHYSICAL TEST ONLY)

> **Not part of the Wireshark packet-capture test.** This reference is included because you explicitly requested **all GPON passwords**. It comes from the **official GPON/PPPoE lab** that was checked against the connected GitHub source, together with the historical physical-lab details from your uploaded PDF. Use it only in authorised laboratory practice; the test assigns its own actual pod/port. **These are shared teaching-lab passwords and should never be reused for personal or production services.**

## A1. Fibre GPON ports A–H — both PPPoE and ONU credentials

The GPON lab maps letters **A–H** to matching **ONU** labels. The **PPPoE username and password** are the same lowercase word; the ONU username and password are also that word.

| GPON fibre pod | GPON patch-port | PPPoE username | PPPoE password | ONU username | ONU password |
|---|---|---|---|---|---|
| A | A | `alpha` | `alpha` | `alpha` | `alpha` |
| B | B | `bravo` | `bravo` | `bravo` | `bravo` |
| C | C | `charlie` | `charlie` | `charlie` | `charlie` |
| D | D | `delta` | `delta` | `delta` | `delta` |
| E | E | `echo` | `echo` | `echo` | `echo` |
| F | F | `foxtrot` | `foxtrot` | `foxtrot` | `foxtrot` |
| G | G | `golf` | `golf` | `golf` | `golf` |
| H | H | `hotel` | `hotel` | `hotel` | `hotel` |

## A2. Direct copper ports I–P — GPON lab (NO ONU)

| Copper pod | Data-centre patch-port | PPPoE username | PPPoE password |
|---|---:|---|---|
| I | 16 | `india` | `india` |
| J | 17 | `juliett` | `juliett` |
| K | 18 | `kilo` | `kilo` |
| L | 19 | `lima` | `lima` |
| M | 20 | `mike` | `mike` |
| N | 21 | `november` | `november` |
| O | 23 | `oscar` | `oscar` |
| P | 24 | `papa` | `papa` |

**Important:** The official copper table skips **data-centre port 22**. Your *earlier* GPON session used **Copper Pod K / port 18 / username `kilo` / password `kilo`**. Your uploaded newer Physical Master Guide also recorded a **GPON F / `foxtrot` / `foxtrot`** practice run. Those are separate past practice configurations, **not guaranteed assignments on exam day**.

## A3. Different exercise, different table: Broadband CPE Scenarios A–H

**Do not confuse the older Broadband CPE Scenarios table with the GPON lab table.** Its physical mapping and the username for B differ.

| CPE pod | Data-centre port | PPPoE username | PPPoE password |
|---|---:|---|---|
| A | 16 | `alpha` | `alpha` |
| B | 17 | **`beta`** | **`beta`** |
| C | 18 | `charlie` | `charlie` |
| D | 19 | `delta` | `delta` |
| E | 20 | `echo` | `echo` |
| F | 21 | `foxtrot` | `foxtrot` |
| G | 23 | `golf` | `golf` |
| H | 24 | `hotel` | `hotel` |

**Contrast:** **GPON fibre B = `bravo`**, but **CPE Scenarios B = `beta`**. Always choose the credential table for the **actual lab and patch panel**.

## A4. Lab passwords that are NOT PPPoE passwords

| Context | Credential | What to do with it |
|---|---|---|
| MikroTik after a genuine factory reset in **GPON** lab | Username `admin`, **blank** password according to that lab | If hardware state differs, follow the instructions on the device / from tutor. |
| Older **Broadband CPE** lab | Username `admin`; password may be on sticker or `admin` according to that lab | Device state may differ. |
| Linksys/DD-WRT lab management | Username `root`, password `admin` | Applies to the cited classroom AP configuration only. |
| WPA cracking lab's intentionally weak demonstration WLAN | Passphrase `charlie12` | This is a deliberately weak **test WLAN** password, **not** PPPoE, FreeRADIUS or OpenSSL volume password. |
| Your Physical Master Guide **practice** FreeRADIUS AP shared secret | `radiussecret` | Must exactly match the AP settings and the FreeRADIUS `clients.conf` entry; not a universal official password. |
| Your Physical Master Guide **practice** RADIUS user | Username `aj`, password `student123` | Illustrative practice test-user credentials, **not** guaranteed LMS question values. |
| This Wireshark Q3 practice capture | Recovered passphrase `drpepperdrinker` for SSID `Eggtopia134321` | Applies **only** after recovering it from the supplied capture; not the WPA classroom live-AP password. |
| Q1–Q4 encrypted practice archives | **Volume password given with that practice question in LMS/PDF** | Use the exact supplied archive password at the **OpenSSL prompt**; not the PPPoE password. |

## A5. Minimal physical-lab error fixes from your uploaded Master Guide

```text
Data centre / GPON (PPPoE server 100.64.0.1)
    -> MikroTik ether1/WAN (PPPoE)
    -> MikroTik LAN 192.168.88.1  [NAT=ON, DHCP=ON]
    -> Ubuntu VM 192.168.88.x    [single bridged Ethernet adapter]
    -> Linksys AP 192.168.88.2   [WAN=OFF, DHCP=OFF, LAN-to-LAN]
    -> Wi-Fi client               [WPA2 Enterprise -> FreeRADIUS on Ubuntu]
```

- **Confirm before further configuration:** Ubuntu `ip -br addr`, `ip route`, `ping -c 4 192.168.88.1`, and `ping -c 4 100.64.0.1`.
- **Multiple VM adapters:** You previously had too many bridged adapters. One bridged interface on the **correct physical Ethernet NIC** removes ambiguous default routes.
- **MikroTik PPPoE:** Apply the configuration before attempting Reconnect; verify `pppoe-out1`, NAT and DHCP.
- **Linksys as AP only:** Connect **MikroTik LAN to Linksys LAN**, with its WAN and DHCP disabled. Do **not** accidentally make a double-router topology.
- **RADIUS config paths in the official lab:** `/etc/freeradius/3.0/clients.conf` for the AP's source IP/shared secret; `/etc/freeradius/3.0/users` for usernames/passwords. Run `sudo freeradius -C`, then `sudo freeradius -X` after stopping the service; successful authentication shows **`Access-Accept`**.
- **A connected AP does not require `wlan0` inside Ubuntu:** The Ubuntu RADIUS backhaul can use bridged Ethernet. Monitor mode is a separate packet-capture task.
- **Apache problem from your previous GPON lab:** The partner couldn't reach your HTTP file server until Apache was started. Then you discovered that `/var/www/html/output.dat` did not exist. **Starting Apache does not create `output.dat`**; the file must be generated/copied separately.
- **Physical-test proof:** Show the tutor the successful Ubuntu ping, enterprise Wi-Fi login/**Access-Accept** and any additional rubric requirements. Reset only **after** tutor verification.

---

## Final revision strategy

1. Reproduce Q1–Q4 from scratch using the **supplied practice captures**, without consulting the self-check answers until the end.
2. Make sure you can explain **why** each Wireshark filter, decryption step, and exact answer format is correct.
3. Keep GPON/PPPoE passwords separate from capture passwords and from Wi-Fi PSKs.
4. During the invigilated packet test, **use only the four official permitted GitHub labs**, unless the teaching team expressly allows something else.

**End of guide.**
