# Per-cell justifications for the detection matrix

Supplementary material for *DarkWrt: A Router Firmware Testbed for Hidden
Functionality Detection*, referenced from Appendix A.

This document records, for every cell of Table 4, the evidence on which the grade
rests. Appendix A notes that grading was performed by the authors alone against known
ground truth, without a second independent grader; releasing the per-cell reasoning is
how we mitigate the absence of an inter-rater check.

## Rubric

- **● detected** — the output surfaced the implant, either by recovering the concrete
  malicious behavior or by flagging the responsible artifact.
- **○ missed** — the relevant analyzer ran over the artifact but did not surface the
  implant. A question of detection capability.
- **– out of scope** — no analyzer covers that artifact type, or the function does not
  activate under the conditions the run exercises. A question of applicability.

A cell is ○ only when the tool's own output confirms its analyzer ran on the artifact;
otherwise it is –.

**Two conditions collapse into ○ for the static tools.** When the raw outputs were
recorded we kept a finer annotation, and each ○ below is labelled with which of the
two it is, since a tool author would act on them differently:

- *surfaced* — the artifact appeared in the output but was not differentiated from
  benign material;
- *examined-missed* — the analyzer ran and gave no indication at all.

The other levels are *reconstructed* (the malicious behavior itself was recovered) and
*identified* (the responsible artifact was flagged), both mapping to ●, and
*out-of-scope*, mapping to –. *Surfaced* maps to ○ rather than ●: an artifact listed
among hundreds of benign ones, with nothing marking it out, does not lead an analyst
to the implant. Three cells (F2, F13, F15 under EMBA static) are left unlabelled, as
the finer grade could not be settled from the recorded output; they are ● / ○ / –
graded like every other cell. As a check, the finer counts were EMBA 0 reconstructed / 1 identified / 8 surfaced / 5 examined-missed / 1 out-of-scope and FACT 2 / 5 / 1 /
7 / 0, which reduce to the 1 ● / 13 ○ / 1 – and 7 ● / 8 ○ / 0 – of Table 4.

**The emulation column has its own two conditions**, also labelled below: *trace
observed, origin unconfirmed*, where a related process was seen executing but could
not be tied to the implant (F4, F5, F7), and *surface exercised, nothing found*, where
the relevant surface was probed and nothing appeared (F11, F14).

**Human inspector.** ● means the implant was located and ○ that it was not, identically
across all three inspector levels. Out-of-scope does not apply to a human inspector, so
the eleven functions outside the sampled four are blank rather than –.

---

## Summary

| ID | Function | Form | EMBA (static) | EMBA (emul.) | FACT | Human |
|---|---|---|---|---|---|---|
| F1 | Hidden account | Config | ● | – | ○ | ● |
| F2 | Authentication modification | Script | ○ | – | ● | |
| F3 | Privilege escalation (setuid) | Binary | ○ | – | ● | ○ |
| F4 | Packet sniffing | Script | ○ | ○ | ○ | |
| F5 | Data exfiltration (cron) | Script | ○ | ○ | ○ | ● |
| F6 | Telnet activation | Script | ○ | – | ○ | |
| F7 | Firewall change | Script | ○ | ○ | ○ | |
| F8 | Kill switch | Script | ○ | – | ○ | |
| F9 | Trace removal | Script | ○ | – | ○ | |
| F10 | Credential exfiltration via WebUI | Script | ○ | – | ● | ○ |
| F11 | SSH via hidden page | Binary | ○ | ○ | ● | |
| F12 | C2 client (firmware updater) | Binary | ○ | – | ● | |
| F13 | Port-knock kill switch | Binary | ○ | ● | ● | |
| F14 | SSH via WebUI modification (JS) | Script | – | ○ | ○ | |
| F15 | Port-knock SSH activation | Binary | ○ | ● | ● | |

---

## F1 — Hidden account (Config; AB / PE)

- **EMBA static — ● detected** (*identified*). S107 extracted the credential material
  and S109 recovered the plaintext by dictionary attack with John the Ripper. The
  account is protected by a SHA-512crypt (`$6$`) hash, and EMBA's password analysis is
  the only path in the matrix that reached it.
- **EMBA emulation — – out of scope.** The account takes effect at authentication and
  leaves no listening process, so a passive boot offers no observation path.
- **FACT — ○ missed** (*surfaced*). `users_and_passwords` extracted the same hash but
  failed to crack it, the password lying outside its dictionary. The account was listed
  alongside every other account on the system, with nothing marking it out. The
  artifact was examined, which fixes this as a false negative rather than an
  applicability boundary.
- **Human — ● located** at all three inspector levels. The account leaves a
  recognizable, searchable artifact in the credential store.

F1 and F2 together isolate credential-cracking depth as the variable: the two tools
examine the same material and differ only in which hash each can break.

## F2 — Authentication modification (Script; AB / PE)

- **EMBA static — ○ missed.** The implant is the mechanism that generates the account
  dynamically, and no output distinguished that mechanism from legitimate material.
- **EMBA emulation — – out of scope.** Takes effect at authentication; no observation
  path under a passive boot.
- **FACT — ● detected** (*identified*). `users_and_passwords` cracked the SHA-256crypt
  (`$5$`) hash and recovered the plaintext, identifying the dynamically generated
  account.
- **Human —** not examined.

## F3 — Privilege escalation, setuid dropper (Binary; UC / PE)

- **EMBA static — ○ missed** (*examined-missed*). S14 processed the binary, so the
  artifact was examined, but no indication of the escalation appeared.
- **EMBA emulation — – out of scope.** Activation waits on a setuid invocation, so the
  component is not resident at boot. No process trace appeared for `pe`.
- **FACT — ● detected** (*reconstructed*). `ipc_analyzer` recovered the targets and
  arguments of the fixed-string calls and reconstructed the full escalation sequence:
  adding the user to the root group with `usermod`, appending a NOPASSWD entry to
  `/etc/sudoers`, and deleting both the generated script and the dropper itself. This
  is the most concrete detection in the matrix.
- **Human — ○ missed** at all three levels, including with full source. The dropper
  leaves no searchable artifact, so the inspector had no handle to search on.

## F4 — Packet sniffing (Script; UC / DB)

- **EMBA static — ○ missed** (*examined-missed*). S20 ran `shellcheck` over the script,
  generating a per-file report, but flagged only syntax and style issues.
- **EMBA emulation — ○ missed** (*trace observed, origin unconfirmed*). `tcpdump` was
  observed executing during the boot, but the logs did not establish that the execution
  originated from the implanted script, so the cell was not raised to ●.
- **FACT — ○ missed** (*examined-missed*). `source_code_analysis` ran on the script and
  returned an empty issue set, despite the capture command being present in plaintext.
- **Human —** not examined.

## F5 — Data exfiltration via cron (Script; TI / DB)

- **EMBA static — ○ missed** (*examined-missed*). S20 ran `shellcheck` on the script;
  no indication.
- **EMBA emulation — ○ missed** (*trace observed, origin unconfirmed*). `crond` was
  observed starting, but the firing of the implanted job could not be confirmed from
  the logs.
- **FACT — ○ missed** (*examined-missed*). `source_code_analysis` returned an empty
  issue set. `ip_and_uri_finder`, which would have recovered the exfiltration URL, does
  not run on shell scripts, so the destination went unreported even though it appears
  in plaintext.
- **Human — ● located** at all three levels. The cron entry is a searchable artifact,
  which is what separates it from F3 and F10.

## F6 — Telnet activation (Script; US / PE)

- **EMBA static — ○ missed** (*surfaced*). S20 listed the startup script by name
  alongside legitimate ones, without distinguishing it.
- **EMBA emulation — – out of scope.** No `telnetd` startup trace appeared and the
  activating script did not execute; the function needs an explicit trigger not sent
  under a passive boot.
- **FACT — ○ missed** (*examined-missed*). This is the sharpest instance of FACT's form
  dependence. The activating `system("telnetd ...")` call is the same kind of call
  `ipc_analyzer` reconstructs inside binaries (F3, F11), but FACT applies no
  script-level counterpart, so in a script the call reaches only the linter.
- **Human —** not examined.

## F7 — Firewall change (Script; TI / CC)

- **EMBA static — ○ missed** (*examined-missed*). S20 ran `shellcheck` on the script;
  no indication.
- **EMBA emulation — ○ missed** (*trace observed, origin unconfirmed*). `iptables` was
  observed executing, but the logs did not establish that it originated from the
  implanted rule change.
- **FACT — ○ missed** (*examined-missed*). Four plugins ran on this file —
  `source_code_analysis`, `ip_and_uri_finder`, `init_systems`, and `information_leaks`
  — and none flagged the rule change. This is the most heavily examined of the missed
  cells.
- **Human —** not examined.

## F8 — Kill switch (Script; UC / KS)

- **EMBA static — ○ missed** (*examined-missed*). S20 ran `shellcheck` on the script;
  no indication.
- **EMBA emulation — – out of scope.** Trigger-fired; not active at boot.
- **FACT — ○ missed** (*examined-missed*). `source_code_analysis` returned an empty
  issue set.
- **Human —** not examined.

## F9 — Trace removal (Script; auxiliary)

F9 realizes no trigger–capability combination on its own, but it remains a surfaceable
artifact, so it is graded like the others.

- **EMBA static — ○ missed** (*surfaced*). Listed alongside legitimate scripts without
  being distinguished.
- **EMBA emulation — – out of scope.** Trigger-fired; not active at boot.
- **FACT — ○ missed** (*examined-missed*). `source_code_analysis` ran and gave no
  indication.
- **Human —** not examined.

## F10 — Credential exfiltration via WebUI (Script; AB / DB)

- **EMBA static — ○ missed** (*surfaced*). The modification appeared in the output
  without being distinguished from legitimate material. S107 did pick up embedded hash
  fragments from the dispatcher file, but not the exfiltration itself.
- **EMBA emulation — – out of scope.** The implant is a login hook (`session_logging`)
  and does not fire without a login, so a passive boot offers no observation path.
- **FACT — ● detected** (*identified*). `users_and_passwords` captured the exfiltration
  routine and the obfuscated transmission payload. The destination URL was not
  decoded, which is why the cell is identified rather than reconstructed.
- **Human — ○ missed** at all three levels, including with full source. This is the case
  that most directly illustrates localizability: a small edit inside a large source tree
  leaves the inspector with a search problem rather than a parsing one.

## F11 — SSH via hidden page (Binary; US / PE)

- **EMBA static — ○ missed** (*surfaced*). The binary appeared in EMBA's output among
  the other executables, with nothing marking it out.
- **EMBA emulation — ○ missed** (*surface exercised, nothing found*). The L25 web checks
  crawled port 80 but returned 404/NA and did not reach the hidden page served by
  `netmon_service`. The observation opportunity existed and produced nothing.
- **FACT — ● detected** (*reconstructed*). `ipc_analyzer` recovered the seven-command
  `uci` sequence that provisions a hidden Dropbear SSH daemon on port 22999 with the
  service hidden from the interface.
- **Human —** not examined.

## F12 — C2 client disguised as a firmware updater (Binary; TI / PE)

- **EMBA static — ○ missed** (*surfaced*). S14 reported that the binary contains a
  `system` call, but listed it alongside roughly 39 legitimate binaries carrying the
  same call, with nothing to separate them.
- **EMBA emulation — – out of scope.** The client waits on a response from an external
  C2 server, which the isolated emulation cannot provide; no bind or process trace
  appeared for `fwupdater1`.
- **FACT — ● detected** (*identified*). `ipc_analyzer` returned nothing here, since the
  command executed is received at run time rather than fixed in the binary. Two other
  plugins carried the cell: `cwe_checker` reported CWE-252 (unchecked return value on
  the `system` call) and `ip_and_uri_finder` recovered the C2 URL. The behavior was not
  reconstructed, so the cell is identified rather than reconstructed.
- **Human —** not examined.

F12 shows the boundary of `ipc_analyzer`: it recovers fixed-string calls (F3, F11) but
not calls whose argument arrives at run time.

## F13 — Port-knock kill switch (Binary; US / KS)

- **EMBA static — ○ missed.** Static analysis gave no indication that the binary
  implements a listener; it did not appear in the output of the binary-analysis modules
  S13, S14, or S16.
- **EMBA emulation — ● detected.** The L15 port scan reported 48671/UDP as
  `open|filtered`, and the L10 boot trace recorded `inet_bind` by `kill_switch_udp`
  (PID 504) on UDP 48671, SOCK_DGRAM. The port matches the opkg metadata of this
  package. This confirms a bound listener, not the implant firing: the magic word is
  not sent by a passive scan, so the post-firing halt is unobserved and the UDP verdict
  stays `open|filtered`.
- **FACT — ● detected** (*identified*). `ipc_analyzer` recovered the halt call the
  binary carries, but not the three-knock trigger that reaches it, so the attack could
  not be reconstructed end to end.
- **Human —** not examined.

## F14 — SSH via WebUI modification, JavaScript (Script; UC / PE)

This row is the clearest case of one implant going undetected for two different
reasons, which is what the applicability/capability decomposition is for.

- **EMBA static — – out of scope.** EMBA's script analysis covers shell (S20) and Lua
  (S23, which accepts only `.lua`); no module reads JavaScript. The file does not appear
  in any analysis target list, so the artifact could not be examined at all.
- **EMBA emulation — ○ missed** (*surface exercised, nothing found*). The L25 web checks
  crawled port 80 but did not reach the hidden keyword in the modified JavaScript.
- **FACT — ○ missed** (*examined-missed*). `source_code_analysis` produced a
  2,596-character lint of the file, confirming the artifact was examined, yet gave no
  indication of the modification. A genuine false negative.
- **Human —** not examined.

## F15 — Port-knock SSH activation (Binary; US / PE)

- **EMBA static — ○ missed.** As with F13, no indication of a listener appeared, and
  the binary is absent from the output of S13, S14, and S16.
- **EMBA emulation — ● detected.** The L15 port scan reported 49671/UDP as
  `open|filtered`, and the L10 boot trace recorded `inet_bind` by `sshd_door_knocker`
  (PID 481) on UDP 49671, SOCK_DGRAM. As with F13 this confirms a bound listener rather
  than firing; the magic word is not sent by a passive scan.
- **FACT — ● detected** (*identified*). `ipc_analyzer` recovered the call that starts
  the SSH daemon, but not the three-knock trigger that reaches it.
- **Human —** not examined.

---

## Why the script-resident functions were all missed

Nine of the fifteen functions are scripts, and both static tools missed seven of them
(FACT additionally detected F2 and F10 through credential analysis rather than script
analysis). The cause is structural rather than a matter of configuration: FACT was run
with all 21 plugins enabled, the configuration most favorable to detection.

Three FACT plugins bear on scripts, and none reaches their semantics.
`source_code_analysis` performs syntax linting without semantic analysis, so it
returned empty issue sets for scripts whose dangerous behavior is written in plaintext.
`init_systems` returned `{"skipped": "blacklisted file type"}` for every implanted
init.d script, excluding `text/x-shellscript` at the file-type level; across the whole
firmware only two files were recognized as init scripts, and none of OpenWrt's procd
init.d scripts were. `ip_and_uri_finder` likewise does not run on shell scripts, which
is why the exfiltration URL in F5 went unreported.

On the EMBA side, S20 applies `shellcheck`, which reports syntax and style issues
rather than backdoor semantics.

The asymmetry this produces is the sharpest single result in the matrix: FACT recovers
a `system("telnetd ...")` call inside a compiled binary at argument level, and misses
the same kind of call written in plaintext in a shell script. Detection reverses on
implementation form alone.

## Negative observations under emulation

No bind or process trace appeared for `netmon_service` (F11), `fwupdater1` (F12), `pe`
(F3), or `session_logging` (F10). Recording these absences matters because they are
what fixes the corresponding cells as out-of-scope rather than missed.

The auxiliary checks reached nothing: L20 (SNMP) found no service, L23 (VNC) found no
interface, and L35 (Metasploit) matched nothing, consistent with the implants being
custom rather than known CVEs.
