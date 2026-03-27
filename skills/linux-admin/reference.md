## Reference: safe playbooks (Ubuntu/Debian)

This file contains safe, reproducible playbooks for user/SSH/sudo operations. Keep the defaults conservative. If something is unclear, stop and ask for details rather than guessing.

### Global safety gates (always)
- Keep an existing privileged session open while making access changes.
- Prefer having a second active session (separate SSH connection) before risky steps.
- Do not disable password auth until key-based login is verified for the target user.
- Any risky change must have: verification command + rollback step.

---

## Playbook 1: create user + SSH key + sudo

### What this does
- Create a new user.
- Install an SSH public key.
- Grant sudo access.
- Verify login and sudo.

### Commands (example skeleton)
> Replace placeholders: `<user>`, `<ssh_public_key>`.

```bash
# 0) Pre-checks
id "<user>" || true
getent passwd "<user>" || true

# 1) Create user (no password login by default)
sudo adduser --disabled-password --gecos "" "<user>"

# 2) Create .ssh and set permissions
sudo install -d -m 700 -o "<user>" -g "<user>" "/home/<user>/.ssh"

# 3) Add authorized_keys
sudo tee "/home/<user>/.ssh/authorized_keys" > /dev/null <<'EOF'
<ssh_public_key>
EOF
sudo chown "<user>:<user>" "/home/<user>/.ssh/authorized_keys"
sudo chmod 600 "/home/<user>/.ssh/authorized_keys"

# 4) Grant sudo (prefer group on Ubuntu/Debian)
sudo usermod -aG sudo "<user>"
```

### Verify
From your workstation:
- `ssh -i <keyfile> <user>@<host> 'id && groups && sudo -n true && echo OK'`

On the server (optional):
- `getent group sudo | grep -F "<user>" || true`

### Rollback (safe)
- Remove user from sudo: `sudo deluser "<user>" sudo`
- Remove key: `sudo truncate -s 0 "/home/<user>/.ssh/authorized_keys"` (or delete specific lines)
- Remove user (only if safe and confirmed): `sudo deluser --remove-home "<user>"`

Stop conditions:
- If you cannot verify key-based login, do not proceed to any step that reduces access.

---

## Playbook 2: disable password SSH login safely

### Preconditions (must-have)
- You have at least one working SSH key-based login path (preferably two).
- You can access the server via console / out-of-band access if something goes wrong (if applicable).

### Strategy
1) Validate key-based login for the target user(s).
2) Apply SSH config change in a minimal, reversible way.
3) Reload sshd (not restart if avoidable).
4) Verify in a new session.
5) Only then consider further hardening.

### Commands (example skeleton)
```bash
# 0) Inspect current sshd effective config
sudo sshd -T | egrep -i 'passwordauthentication|kbdinteractiveauthentication|challengeresponseauthentication|pubkeyauthentication' || true

# 1) Edit config carefully (choose the project standard; common options below)
# Prefer adding a drop-in under /etc/ssh/sshd_config.d/ if supported.

# Example: create a drop-in (Debian/Ubuntu modern OpenSSH)
sudo install -d -m 755 /etc/ssh/sshd_config.d
sudo tee /etc/ssh/sshd_config.d/99-no-password.conf > /dev/null <<'EOF'
PasswordAuthentication no
KbdInteractiveAuthentication no
EOF

# 2) Validate config before applying
sudo sshd -t

# 3) Reload (preferred) or restart (only if required)
sudo systemctl reload ssh || sudo systemctl reload sshd
```

### Verify (required)
In a NEW terminal session:
- Key login works: `ssh <user>@<host> 'echo key_ok'`
- Password login fails (do not lock yourself out while testing):
  - attempt password-based login and confirm it’s rejected

### Rollback
```bash
sudo rm -f /etc/ssh/sshd_config.d/99-no-password.conf
sudo sshd -t
sudo systemctl reload ssh || sudo systemctl reload sshd
```

Stop conditions:
- If `sshd -t` fails, do not reload/restart.
- If key login is not verified in a new session, do not proceed with further restrictions.

---

## Notes on sudo

### Prefer group-based sudo
On Ubuntu/Debian, prefer adding users to `sudo` group instead of custom sudoers lines.

### If sudoers changes are required
- Always use `visudo` or validate with `visudo -c`.
- Keep changes minimal and reversible.

