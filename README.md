# ICT369 · Find the right lab quickly

[Official labs](#official-labs) · [Wireshark practice](#wireshark-practice) · [Physical lab practice](#physical-lab-practice) · [Troubleshooting](#troubleshooting) · [Full guides](#full-guides)

> **Assessment access:** Your assessment brief permits the specified official lab pages. This README, personal guides and companion website are preparation resources unless your tutor explicitly authorises them. Before the test, bookmark the official pages below directly.

## Official labs

### Packet capture and diagnostics

| I need to… | Open the official lab |
|---|---|
| Analyse wireless frames, statistics, RTS and CTS | [RTS/CTS and network analysis](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/rts_cts_and_network_analysis.md) |
| Recover a WEP key | [WEP](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wep.md) |
| Recover a WPA password using a wordlist | [WPA](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/wpa.md) |
| Investigate streams, web content and TLS | [Network forensics](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/network_forensics.md) |

### Physical network and enterprise Wi-Fi

| I need to… | Open the official lab |
|---|---|
| Cable and configure the router / access point | [Broadband CPE scenarios](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/broadband_cpe_senarios.md) |
| Check GPON/copper assignments and configure PPPoE | [GPON PPPoE](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/gpon_pppoe.md) |
| Set up FreeRADIUS and enterprise authentication | [Enterprise authentication with FreeRADIUS](https://github.com/SCH-IT-MurdochUni/NetworkingLabs/blob/main/Wireless/enterprise_authentication_with_FreeRADIUS.md) |

[Back to top](#ict369--find-the-right-lab-quickly)

## Wireshark practice

**Choose the wording closest to your practice question.** Each link jumps directly into the relevant guide section.

| Question or task | Jump to the walkthrough |
|---|---|
| “Download and decrypt this file” / `.enc`, `.bz2`, archive | [Open and verify the capture](wireless/ICT369_Practical_Competency_GitHub_Guide.md#102-verify-provided-encrypted-captures) |
| “What percentage are beacons?” / “What is the SSID?” / retransmissions | [Wireless statistics](wireless/ICT369_Practical_Competency_GitHub_Guide.md#103-type-a--wlan-statistics) |
| “Load a TLS key log” / “Find the HTTP GET” / “Recover the image” | [TLS, HTTP and object recovery](wireless/ICT369_Practical_Competency_GitHub_Guide.md#104-type-b--tls--http-and-object-recovery) |
| “Find the Wi-Fi key” / supplied password list / WEP | [WEP and WPA recovery](wireless/ICT369_Practical_Competency_GitHub_Guide.md#105-type-c--offline-wep-and-wpa-recovery-authorised-captures-only) |
| “Find the secret message” / source port / delay in seconds | [Message recovery and timing](wireless/ICT369_Practical_Competency_GitHub_Guide.md#106-type-d--secret-message-and-timing) |
| “Which filter do I use?” | [Filter reference](wireless/ICT369_Practical_Competency_GitHub_Guide.md#107-fast-wireshark-filter-sheet) |
| Unsure where to begin | [Identify the question type](wireless/ICT369_Practical_Competency_GitHub_Guide.md#101-identify-the-question-type) |

[Interactive LMS practice lessons](https://ict369-lab-companion-ranji.bonnycove128.chatgpt.site/wireshark.html#step-17) · [Back to top](#ict369--find-the-right-lab-quickly)

## Physical lab practice

**Follow this order during preparation.** Open a step to see its instructions and checks.

| Step | Task |
|---|---|
| 1 | [Understand the cabling and network layout](wireless/ICT369_Practical_Competency_GitHub_Guide.md#2-physical-lab-network-design) |
| 2 | [Match GPON/copper credentials to the assigned port](wireless/ICT369_Practical_Competency_GitHub_Guide.md#3-gpon-and-copper-credentials) |
| 3 | [Configure MikroTik PPPoE](wireless/ICT369_Practical_Competency_GitHub_Guide.md#4-mikrotik-pppoe-setup) |
| 4 | [Check the Ubuntu VM, IP address and route](wireless/ICT369_Practical_Competency_GitHub_Guide.md#5-vmware-and-ubuntu-troubleshooting) |
| 5 | [Convert Linksys to AP-only mode](wireless/ICT369_Practical_Competency_GitHub_Guide.md#6-linksys-ap-only-conversion) |
| 6 | [Install FreeRADIUS](wireless/ICT369_Practical_Competency_GitHub_Guide.md#71-install-locate-edit) |
| 7 | [Add Linksys as a RADIUS client](wireless/ICT369_Practical_Competency_GitHub_Guide.md#72-authorise-linksys-as-a-radius-client) |
| 8 | [Create the Wi-Fi user](wireless/ICT369_Practical_Competency_GitHub_Guide.md#73-create-a-test-user) |
| 9 | [Validate configuration and start debug mode](wireless/ICT369_Practical_Competency_GitHub_Guide.md#74-validate-debug-and-test) |
| 10 | [Configure enterprise Wi-Fi on Linksys](wireless/ICT369_Practical_Competency_GitHub_Guide.md#75-configure-dd-wrt-enterprise-authentication) |
| 11 | [Connect the Android phone](wireless/ICT369_Practical_Competency_GitHub_Guide.md#76-android-phone-settings) |
| 12 | [Practise SSH public-key setup](wireless/ICT369_Practical_Competency_GitHub_Guide.md#8-ssh-public-key-setup) |
| 13 | [Review demonstration evidence and reset instructions](wireless/ICT369_Practical_Competency_GitHub_Guide.md#9-physical-assessment-evidence-and-reset) |

Use current assigned addresses and credentials; values from past practice may differ. Check the rubric for required demonstrations, including whether SSH is assessed.

[Back to top](#ict369--find-the-right-lab-quickly)

## Troubleshooting

| What went wrong? | Go straight here |
|---|---|
| VM uses the wrong network or gateway | [Default route and metrics](wireless/ICT369_Practical_Competency_GitHub_Guide.md#53-wrong-default-route--metric) |
| VM interface has no usable address | [Interface state and DHCP](wireless/ICT369_Practical_Competency_GitHub_Guide.md#54-bring-an-interface-up-obtain-dhcp) |
| Phone connects to the wrong/open radio | [Dual-band configuration](wireless/ICT369_Practical_Competency_GitHub_Guide.md#61-dual-band-trap-encountered) |
| FreeRADIUS reports a syntax error or port conflict | [Validate, debug and test](wireless/ICT369_Practical_Competency_GitHub_Guide.md#74-validate-debug-and-test) |
| Phone cannot obtain an IP address | [Authentication first, then DHCP](wireless/ICT369_Practical_Competency_GitHub_Guide.md#77-if-phone-says-couldnt-get-ip-address) |
| HTTP remains encrypted, wrong frame, or wrong timing | [Packet-analysis fixes](wireless/ICT369_Practical_Skills_Official_Lab_Links_and_Notes.md#packet-analysis-fixes-we-encountered) |
| Another error | [Error-to-fix matrix](wireless/ICT369_Practical_Competency_GitHub_Guide.md#11-error-to-fix-matrix) |

[Back to top](#ict369--find-the-right-lab-quickly)

## Full guides

- [Wireshark: four practice questions, step by step](wireless/packettracer.md)

- [Practical competency guide and troubleshooting](wireless/ICT369_Practical_Competency_GitHub_Guide.md)
- [Official lab links and additional practice notes](wireless/ICT369_Practical_Skills_Official_Lab_Links_and_Notes.md)
- [Interactive lab companion](https://ict369-lab-companion-ranji.bonnycove128.chatgpt.site/)
- [Final practice checklist](wireless/ICT369_Practical_Competency_GitHub_Guide.md#final-practice-checklist)

**Navigation tip:** Use the tables above to jump to a section, then your browser's Back button to return. Within a guide, press **Ctrl+F** to find an exact error message or term.

