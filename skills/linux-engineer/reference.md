## Reference (Brendan Gregg / principal style)
This file contains deeper playbooks and advanced tooling. Use it after basic localization (RED/USE) and only with the safety/impact constraints in mind.

### Escalation rule (mandatory)
Before using `perf`/`eBPF`/`tcpdump`, always record: **goal**, **expected signal**, **window**, **limits**, **impact**, **security (PII/secrets)**, **storage/cleanup**, and get **explicit operator confirmation**.

---

## CPU profiling (Ubuntu/Debian)
### Quick check
- `pidstat -u 1` — who burns CPU; user/sys/steal breakdown.
- `sar -u 1` — overall CPU picture.

### perf (short windows only)
Goals: identify what burns CPU (functions/stack), or estimate CPI/branches/cache-misses.

- **Lightweight stats** (usually safer):
  - `perf stat -p <pid> -- sleep 10`
  - `perf stat -- sleep 10`

- **Hot functions** (short):
  - `perf top -p <pid>` (run briefly, then stop)

- **Sampling record** (strict limits):
  - `perf record -F 99 -p <pid> -- sleep 15`
  - then `perf report`

Notes:
- Avoid long `perf record` on production.
- If permissions/kernel restrictions block perf, don’t bypass them; propose alternatives (metrics/logs/tighter windows).

---

## Off-CPU / latency investigation (eBPF)
Goals: find latency that isn’t explained by CPU utilization (I/O wait, run queue, scheduler delays).

Approach:
1) Ensure baseline picture is collected (RED/USE).
2) Ask a concrete question: “where is the latency?” and “what type of latency?”.
3) Pick the least invasive eBPF tool and a short window.

Example task types (conceptual):
- **Block I/O latency**: identify slow block ops and latency distributions.
- **TCP retransmits**: confirm retransmits exist and how often.
- **Run queue latency / scheduler**: how long threads wait for CPU.

Constraints:
- eBPF may require packages (bpftrace/BCC) and privileges. If not installed, first provide justification + risk assessment (or propose non-install alternatives).
- Always set strict limits (e.g. 30–120 seconds) and bound output volume.

---

## Network: safe tcpdump/pcap
### When it’s appropriate
- You have network symptoms (timeouts/packet loss/retransmits) and baseline counters/`ss` don’t explain them.
- You need to confirm a specific scenario (RST, handshake, DNS, TLS, MTU).

### Minimal safety rules
- **Filter is mandatory** (BPF): host/port/protocol; avoid “capture everything”.
- **Limits**: time, file size, rotation/ring buffer.
- **PII/secrets**: pcap can contain sensitive data. Don’t upload to public places.
- **Storage**: predefine where it goes, who can access it, retention, and deletion procedure.

### Baseline templates (minimally invasive)
- Capture headers without payload:
  - `tcpdump -i <iface> -s 96 -w capture.pcap '<bpf>'`
- Time limit:
  - `timeout 30s tcpdump ...`
- Size/rotation limit:
  - `tcpdump -C 50 -W 5 -w capture.pcap ...`

Example BPF filters (adapt):
- `host 10.0.0.10 and tcp and port 443`
- `udp and port 53`

---

## Storage/IO: common patterns (interpretation help)
### What to look for
- **Latency**: increased `await`/`r_await`/`w_await`, long tails.
- **Saturation**: increased `aqu-sz`/queues, `%util`.
- **Errors**: I/O error, timeout, reset in `dmesg`/`journalctl -k`.

### Common causes
- **Writeback pressure**: dirty pages, background flush stalls on flash/RAID.
- **FS/journal**: ext4/jbd2, locks, sync writes.
- **Device timeouts**: NVMe/SATA resets, controller errors, array degradation.

### Safe-first commands
- `iostat -xz 1`
- `pidstat -d 1`
- `journalctl -k -S -1h`
- `lsblk -f`, `df -hT`, `mount`

