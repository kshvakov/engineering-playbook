## Examples (Linux admin / access hardening)

These examples show the expected output shape: what will be done → commands → verification → rollback.

---

## Example 1: create user + SSH key + sudo

### What will be done
- Create user `deploy`.
- Add SSH public key to `deploy`.
- Grant sudo via `sudo` group.

### Commands
```bash
sudo adduser --disabled-password --gecos "" deploy
sudo install -d -m 700 -o deploy -g deploy /home/deploy/.ssh
sudo tee /home/deploy/.ssh/authorized_keys > /dev/null <<'EOF'
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@laptop
EOF
sudo chown deploy:deploy /home/deploy/.ssh/authorized_keys
sudo chmod 600 /home/deploy/.ssh/authorized_keys
sudo usermod -aG sudo deploy
```

### Verify
From workstation:
```bash
ssh -i ~/.ssh/deploy_key deploy@HOST 'id && groups && sudo -n true && echo OK'
```

### Rollback
```bash
sudo deluser deploy sudo
sudo deluser --remove-home deploy
```

---

## Example 2: disable password SSH login (safe)

### What will be done
- Confirm key-based login works in a new session.
- Disable password authentication via `sshd_config.d` drop-in.
- Reload sshd and verify.

### Commands
```bash
sudo install -d -m 755 /etc/ssh/sshd_config.d
sudo tee /etc/ssh/sshd_config.d/99-no-password.conf > /dev/null <<'EOF'
PasswordAuthentication no
KbdInteractiveAuthentication no
EOF
sudo sshd -t
sudo systemctl reload ssh || sudo systemctl reload sshd
```

### Verify
- In a NEW session:
  - `ssh USER@HOST 'echo key_ok'`
- Attempt password login and confirm it’s rejected.

### Rollback
```bash
sudo rm -f /etc/ssh/sshd_config.d/99-no-password.conf
sudo sshd -t
sudo systemctl reload ssh || sudo systemctl reload sshd
```

