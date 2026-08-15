# Case collection: procedure and per-case sources

Supplementary material for *DarkWrt: A Router Firmware Testbed for Hidden
Functionality Detection*, referenced from Section 3.1 and the caption of Table 1.

This document gives the collection procedure and the public sources for each of the 23
cases. **Entries 1–23 follow the row order of Table 1**, so the numbering can be used
directly to cross-reference the two. Categories and tiers are as defined in the paper:
triggers AB, US, UC, TI; capabilities PE, DB, CC, KS; tiers T1 (IoT vendor), T2 (ODM
supplier), T3 (OSS developer), `--` (not confidently attributable).

## Procedure

**Sources searched.** News reports, vendor advisories, technical blogs, and the Japan
Vulnerability Notes (JVN) database.

**Keywords.** English and Japanese combinations including "backdoor," "kill switch,"
"information leakage," "hidden functionality," and "supply chain attack."

**Search period.** Searches were conducted between April and August 2024. They were
not run to a predefined stopping rule such as saturation; collection ended once the set
was judged sufficient to ground the testbed, which is the purpose the case study serves
here. The set is therefore a grounding sample rather than an exhaustive census, and
cases disclosed after August 2024 are not represented.

**Inclusion criterion.** A case was included when at least one of its trigger or
capability could be identified from public reporting. Intentionality was not required:
cases where the vendor disputed intent, such as ADUPS or hardcoded credentials later
described as debug artifacts, remain in the set. Case 20 (ASML) shows why the criterion
is stated over the two axes — no trigger could be identified, and the case is admitted
on its capability alone.

**Exclusion criteria.** Two classes are excluded. Tier 4 (chip-vendor) embeddings in
SDKs or baseband firmware, which are out of scope for the paper; and post-shipment
additions by parties outside the supply chain, such as malware infection of deployed
devices, which fall under a different threat model. Tier 4 is therefore absent from the
set by construction, not by search.

**Trigger and capability assignment.** The categories are mutually exclusive within
each axis at the level of an individual hidden function, but a single case can contain
several distinct hidden functions. Where it does, all applicable categories are marked:
case 5 (D-Link) contains separate AB, US, and UC functions.

**Tier assignment.** Cases were attributed by who supplied the affected firmware
component, not by who shipped the finished device. Where public reporting did not
clearly establish this, the case is marked `--`.

**Coverage.** The set reflects publicly disclosed cases only, and underrepresents
embeddings never publicly identified. Within the tiers collected, the bias is most
pronounced for T3, where embeddings are harder to attribute and rarely surface in
public reporting.

---

## 1. LG mobile Wi-Fi router — 2017, T1, AB / PE, DB

Undocumented administrative interface exposing the device to unauthorized access
(Wi-Fi STATION L-02F).

- <https://jvndb.jvn.jp/ja/contents/2017/JVNDB-2017-000217.html> (JVNDB-2017-000217)
- <https://nvd.nist.gov/vuln/detail/CVE-2017-10845>
- <https://nvd.nist.gov/vuln/detail/CVE-2017-10846>

## 2. Jetstream / Wavlink router — 2020, T2, AB, US, UC / PE, DB

Backdoor scripts and hardcoded credentials shipped in firmware sold under multiple
brands.

- <https://cybernews.com/security/walmart-exclusive-routers-others-made-in-china-contain-backdoors-to-control-devices/>

## 3. Dahua router — 2017, T1, AB / PE, DB

Hardcoded credentials enabling unauthenticated access; also affects Dahua IP cameras.

- <https://github.com/mcw0/PoC/blob/master/dahua-backdoor.txt> (PoC by bashis)
- <https://nvd.nist.gov/vuln/detail/CVE-2017-7925>

## 4. Netis router — 2014, T1, US / PE

UDP port left open for remote administrative access.

- <https://securityaffairs.com/27816/hacking/netis-routers-backdoor.html>

## 5. D-Link router — 2016, T1, AB, US, UC / PE, CC

Authentication bypass via a specific User-Agent string, together with hardcoded
credentials.

- <https://threatpost.com/backdoored-d-link-router-should-be-trashed-researcher-says/120979/>
- <https://nvd.nist.gov/vuln/detail/CVE-2016-10177>

## 6. Sercomm-firmware router — 2014, T2, US / PE

Network listener providing backdoor access, in firmware used by Linksys, Netgear,
Cisco, and other vendors.

- <https://arstechnica.com/information-technology/2014/01/backdoor-in-wireless-dsl-routers-lets-attacker-reset-router-get-admin/>
- <https://arstechnica.com/information-technology/2014/04/easter-egg-dsl-router-patch-merely-hides-backdoor-instead-of-closing-it/>

## 7. TP-Link router — 2013, T1, UC / PE, DB, CC, KS

Hidden HTTP request handler granting access to system functionality.

- <https://sekurak.pl/tp-link-httptftp-backdoor/>

## 8. Uniway router — 2023, `--`, UC / KS

Hidden component file allowing remote disabling of the device.

- <https://nvd.nist.gov/vuln/detail/CVE-2023-7209>

## 9. Trendnet IP camera — 2012, T1, UC / DB

Undocumented URL exposing the video stream without authentication.

- <https://console-cowboys.blogspot.com/2012/01/trendnet-cameras-i-always-feel-like.html>

## 10. Xiongmai camera — 2020, T1, US, UC / PE, CC

Backdoor providing administrative access on networked video recorders.

- <https://www.theregister.com/2020/02/04/dvr_nvr_backdoor/>

## 11. ADUPS smartphone firmware — 2016, T2, TI / DB

Firmware-over-the-air update component periodically transmitting user data without
disclosure.

- <https://arstechnica.com/information-technology/2016/11/chinese-company-installed-secret-backdoor-on-hundreds-of-thousands-of-phones/>

## 12. Samsung smartphone — 2014, T1, UC / DB, CC

Modem-side interface that can be invoked remotely to read or modify device state
(Galaxy series).

- <https://redmine.replicant.us/projects/replicant/wiki/SamsungGalaxyBackdoor>

## 13. Qihoo360 smartwatch — 2020, T2, UC / DB

Embedded surveillance functionality in a children's smartwatch.

- <https://www.mnemonic.io/resources/blog/exposing-backdoor-consumer-products>

## 14. Harman AMX product — 2016, T1, AB, UC / PE, DB

Hardcoded administrative account intentionally hidden in firmware.

- <https://seclists.org/fulldisclosure/2016/Jan/63> (SEC Consult advisory)

## 15. Huawei equipment — 2019, T1, AB / PE

Large-scale audit identifying numerous potential backdoors and hardcoded credentials.

- <https://www.securityweek.com/many-potential-backdoors-found-huawei-equipment-study/>
- <https://finitestate.io/resources/huawei-supply-chain-assessment>

## 16. Dbltek VoIP gateway — 2017, T1, AB / PE

Hidden Telnet administration interface accessible with vendor-known credentials.

- <https://www.techradar.com/news/dangerous-backdoor-exploit-found-on-popular-iot-devices>

## 17. Seagate wireless HDD — 2015, T1, US / PE

Undocumented Telnet service providing root access on wireless storage products; also
affects LaCie products.

- <https://www.kb.cert.org/vuls/id/903500>

## 18. Western Digital NAS — 2018, T1, AB, UC / PE, DB

Hardcoded administrative credentials and undocumented endpoints (My Cloud series).

- <https://thehackernews.com/2018/01/western-digital-mycloud.html>

## 19. Zyxel device — 2020, T1, AB / PE

Hardcoded administrative account affecting more than 100,000 firewall and VPN gateway
devices, with the credentials hidden in the user interface.

- <https://www.zdnet.com/article/backdoor-account-discovered-in-more-than-100000-zyxel-firewalls-vpn-gateways/>
- <https://www.zyxel.com/global/en/support/security-advisories/zyxel-security-advisory-for-hardcoded-credential-vulnerability>
- <https://nvd.nist.gov/vuln/detail/CVE-2020-29583>

## 20. ASML EUV machine — 2024, T1, `--` / KS

Reported remote kill switch capability on extreme ultraviolet lithography equipment
delivered to overseas facilities. No trigger could be identified from public reporting.

- <https://www.datacenterdynamics.com/en/news/asml-adds-remote-kill-switch-to-tsmcs-euv-machines-in-case-china-invades-taiwan-report/>

## 21. Juniper ScreenOS — 2015, T1, AB, US / PE, DB

Two distinct backdoors in ScreenOS firmware: an authentication bypass, and a weakened
VPN random number generator allowing decryption.

- <https://www.rapid7.com/blog/post/2015/12/20/cve-2015-7755-juniper-screenos-authentication-backdoor/>
- <https://nvd.nist.gov/vuln/detail/CVE-2015-7755>
- <https://nvd.nist.gov/vuln/detail/CVE-2015-7756>

## 22. XZ Utils — 2024, T3, AB / PE

Multi-year supply chain attack in which malicious code was introduced into the upstream
open-source project by a maintainer-level contributor, designed to bypass SSH
authentication.

- <https://tukaani.org/xz-backdoor/>
- <https://nvd.nist.gov/vuln/detail/CVE-2024-3094>

## 23. SSH Decorator — 2018, `--`, UC / PE, DB

Malicious release of a Python library disguised as a legitimate update, exfiltrating
SSH credentials.

- <https://www.bleepingcomputer.com/news/security/backdoored-python-library-caught-stealing-ssh-credentials/>
