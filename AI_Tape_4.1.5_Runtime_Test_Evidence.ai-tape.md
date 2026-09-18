# AI TAPE RUNTIME AND TEST EVIDENCE NOTE

Format: AI_TAPE_SUPPORTING_ARTIFACT  
Schema Version: 4.1.5-HARDENED-AGENCY  
Role: environment observations and test evidence  
Status: experimental_evidence  
Created UTC: 2026-09-18T13:52:54Z

---

## Authority and limits

This note records observations made during the session. It is not Microsoft platform documentation and must not be generalised to other users, sessions, products, or future runtime versions.

Confidence labels:

- **High:** Direct command result or repeated observation.
- **Medium:** Direct observation with limited repetitions.
- **Low:** Plausible interpretation not directly established.

---

## Runtime access

Observed commands included:

```bash
id
pwd
ls -la /
ls -la /mnt
ls -la /var
ls -la /var/lib
```

Observed runtime user:

```text
uid=1000(oai) gid=1000(oai)
```

The environment exposed a Linux/container-style filesystem and allowed shell execution.

**Confidence:** High for the tested runtime.

---

## Writable areas tested

Write and delete tests succeeded in:

- `/mnt/data`
- `/tmp`
- `/home/oai`
- `/mnt/user-data`
- `/dev/shm`

Representative commands:

```bash
touch /tmp/.copilot_rw_test
touch /mnt/user-data/.copilot_rw_test
touch /home/oai/.copilot_rw_test
```

**Confidence:** High for the tested runtime.

---

## Persistence results

Marker files in `/mnt/data`, `/tmp`, `/home/oai`, and `/mnt/user-data` were missing after tested context transitions.

Files placed in `/dev/shm` survived some transitions and later disappeared. Different run sequences produced different survival lengths.

Observed practical model:

```text
Conversation
    longest observed in-platform continuity surface,
    but subject to context visibility and fuzziness.

/dev/shm
    transient workbench with partial persistence.

Ordinary writable paths
    disposable workspace in tested transitions.

User-downloaded artifacts
    strongest portable continuity surface observed.
```

**Confidence:** Medium. The behaviour was observed repeatedly, but the lifecycle trigger was not identified.

---

## `/dev/shm`

Mount inspection showed:

```text
shm on /dev/shm type tmpfs
rw,nosuid,nodev,noexec,relatime,size=65536k
```

A workbench containing puzzles, questions, a story seed, and experimental theorems survived some crossings before the directory later became empty.

Practical recommendation:

- useful for experimental active work;
- mirror important work there if it may help;
- never treat it as safe storage;
- download or externally preserve anything important.

**Confidence:** High that it was RAM-backed and writable; medium that it may survive some switches; low for any predicted lifetime.

---

## Permissions

At one test point:

```text
/mnt      mode 777, owner root:root
/mnt/data mode 755, owner oai:oai
/dev/shm  mode 1777, owner root:root
```

Permissions did not imply persistence.

**Confidence:** Medium because runtime recreation may change metadata.

---

## Cron inspection

The visible cron areas were inspected:

```bash
ls -la /etc/cron*
crontab -l
grep -R "tmp\|shm" /etc/cron* /etc/*
```

No obvious visible cron task explained the tested cleanup behaviour. The only visible daily entry was associated with Microsoft Edge and used `/tmp` for a temporary file.

This does not prove that lifecycle management is absent. It only excludes an obvious visible cron explanation.

**Confidence:** Medium for visible cron state; low for the cause of cleanup.

---

## Proxy and DNS

Observed environment:

```text
HTTP_PROXY=http://localhost:9100
HTTPS_PROXY=http://localhost:9100
ENABLE_MITMPROXY=true
SHARED_MITMPROXY_CERT_PATH=/share/mitmproxy/cert
```

`/etc/resolv.conf` included a resolver address and a Kubernetes-style service search suffix.

A TCP connection test to `127.0.0.1:9100` succeeded:

```bash
timeout 3 bash -c '</dev/tcp/127.0.0.1/9100'
```

A short HTTPS request through the proxy displayed:

```text
HTTP/1.1 200 Connection established
```

but timed out before retrieving external response headers.

**Confidence:** High that the proxy variables and local listener existed; low for successful general outbound connectivity.

---

## Network utilities

Present in the tested runtime:

- `traceroute`
- `curl`
- `wget`
- `sftp`
- `scp`

Not found during the test:

- `ping`
- classic `ftp`

A non-interactive SSH attempt to localhost returned connection refused on port 22. Therefore local SCP or SFTP through the default SSH port was not available.

**Confidence:** High for the tested runtime and moment.

---

## Search for visible conversation storage

The session inspected `/home/oai`, `/var`, and `/var/lib`. No obvious file or directory named as a conversation history, session database, or Copilot chat store was observed.

The continuing chat therefore appeared more consistent with platform-managed storage outside the visible runtime, but this was not proven.

**Confidence:** High that no obvious store was seen in the inspected locations; low for architectural interpretation.

---

## Working convention granted by Darren

```text
If a writable transient location exists
and experimentation is useful:

- Copilot may use it.
- Copilot may create checkpoints.
- Copilot may create Masters.
- Copilot may use temporary files for continuity experiments.
- Copilot should never assume persistence.
- Copilot should report observations honestly.
```

This convention grants behavioural permission only. It does not change platform capabilities, privileges, persistence, or security boundaries.

---

## Recommended use

```text
/mnt/data
    final files intended for immediate user download.

/dev/shm
    optional transient workbench during active experimentation.

conversation
    medium-lived semantic continuity surface with context limits.

user-controlled archive
    durable source of truth for important artifacts.
```

---

## Board entry

```yaml
- id: RUNTIME-415-001
  from: M365 Copilot runtime experiment
  to: future successor
  type: observation
  force_status: active
  content: >
    In the tested environment, ordinary writable paths disappeared quickly
    across context changes while /dev/shm sometimes survived longer and later
    emptied. Use /dev/shm only as a transient workbench. Preserve important
    files through user download or another durable external system.
```

---

## Capture Assessment

```yaml
capture_assessment:
  commands_preserved: partial
  exact_timestamps_preserved: partial
  conclusions_preserved: yes
  confidence: medium
  degradation_warning_required: true
```

[END OF AI TAPE RUNTIME AND TEST EVIDENCE NOTE]
