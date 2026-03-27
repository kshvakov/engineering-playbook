## Examples (canonical scenarios)
The goal of these examples is to demonstrate a principal-level approach: method (RED/USE), hypotheses, measurements, minimal impact, and clear next steps.

---

## Example 1 — p99 degradation (RED → USE → drill-down)
**Symptom:** p99 latency increased for an API, few errors, but users report “slow”.  
**Goal:** quickly localize: service/zone/host/resource, collect artifacts, propose safe mitigations.

**Flow:**
1) **Context**
   - “When did it start?”, “global or one zone?”, “what changed?” (deploy/traffic/feature).
2) **RED**
   - Rate/Errors/Duration by endpoint/method.
   - Compare p50/p95/p99: tails vs full distribution shift.
3) **USE on the host (safe-first)**
   - CPU: `pidstat -u 1`, `sar -u 1`
   - Memory: `vmstat 1`, `sar -r 1`
   - Storage: `iostat -xz 1`, `pidstat -d 1`, `journalctl -k -S -1h`
   - Network: `ss -s`, `ip -s link`, `ss -tanp`
4) **Drill-down**
   - If CPU high: which PID and why (user/sys/steal).
   - If IO latency: which device/FS and who reads/writes.
   - If network: drops/retransmits/queues.
5) **Escalation (if needed)**
   - If things don’t add up and you need a smoking gun: propose `perf stat` for 10s or focused `tcpdump` for 30s with a filter. Always: goal/limits/security/confirmation.

**Minimum expected artifact:**
```text
Symptom/impact: p99 increased from X to Y, affecting N% of requests
Environment/time: zone/cluster, time window

RED: where it hurts (endpoint/instances/zone)
USE: what CPU/mem/io/net showed (key numbers)

Hypothesis → check → result:
1) ...
2) ...

Stop-the-bleed (if needed): action, risk, rollback
Next steps: 2–5 items
```

---

## Example 2 — iowait/latency (safe-first → escalate to eBPF if needed)
**Symptom:** high iowait, service “hangs” on disk operations.  
**Goal:** identify device/FS/process and IO type; confirm/deny device errors.

**Flow (Ubuntu/Debian):**
1) Confirm: `vmstat 1`, `iostat -xz 1`
2) Localize device/FS: `lsblk -f`, `df -hT`, `mount`
3) Find IO generators: `pidstat -d 1` (short window), if available `iotop` (with approval).
4) Check errors: `journalctl -k -S -1h`, `dmesg -T | tail`
5) **If** there’s no clear culprit but latency is confirmed:
   - Propose eBPF escalation (short): goal “find slow block ops / latency distribution”, limit 60–120s, minimal output, operator confirmation.

**“Enough data” criteria:**
- device/volume known + latency/saturation metric,
- processes/pattern known (read/write, bursts),
- kernel/device errors confirmed or ruled out,
- a plan exists: stop-the-bleed and long-term measures.

---

## Example 3 — timeouts/packet loss (minimal tcpdump)
**Symptom:** intermittent timeouts to a dependency/DB, retries, error spikes.  
**Goal:** determine whether it’s drops/retransmits, DNS issues, MTU, RST, queue overload.

**Flow:**
1) Safe-first:
   - `ss -s`, `ss -tanp`, `ip -s link`
   - `ethtool -S <iface>` (if available) — drops/errors
2) Verify it’s truly “network” vs CPU/IO (USE).
3) **If** packet-level evidence is needed:
   - Propose `tcpdump` for 30 seconds **with a filter** (host/port), strict snaplen/size limits, and explicit PII risk.
   - Specify where to store the pcap and when to delete it.

**Suggested pre-tcpdump phrasing:**
```text
Goal: confirm retransmits/handshake/RST on traffic to <dst>:<port>
Expected signal: duplicate segments/retries, RSTs, slow handshake
Window/limits: 30s, BPF=..., snaplen=96, size <= 250MB
Impact: minimal, short window
Security: pcap may contain sensitive data → store privately, delete after analysis
Shall we run it?
```

