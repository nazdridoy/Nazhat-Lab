# Nazhat-Lab — Notes

---

## Hostname & `hostnamectl` Behavior in Docker

- **`hostnamectl set-hostname` EBUSY error**: Docker bind-mounts `/etc/hostname` as read-only. Modifying it returns an `EBUSY` error. However, the `sethostname()` syscall succeeds, meaning `hostname` updates correctly for RHCSA testing.
- **Default Hostname Setup**: Modifying `/etc/hostname` during build has no effect because Docker overrides it at runtime. NetworkManager also resets the hostname asynchronously during connection activation.
- **Solution Used**: We disable NetworkManager's hostname handling entirely (`hostname-mode=none` in `/etc/NetworkManager/conf.d/`). We then use a systemd oneshot service (`nazhat-hostname.service`) to set `nazhat-lab` before network targets.
- **Compose Warning**: Do not use `hostname: <name>` in `docker-compose.yml`. It locks the UTS namespace, breaking `hostnamectl` entirely.

---

## PS1 — dynamic hostname

`PS1` is set to `\u@\h` in `/etc/bashrc`. Bash expands `\h` to `gethostname()` at startup, so the prompt dynamically reflects `hostnamectl set-hostname` changes.

---

## `useradd` chaining — explicit precedence

Guarded user creation commands are wrapped in `{ ...; }` to ensure shell operator precedence is explicit:

```dockerfile
RUN { id student  &>/dev/null || useradd -m -s /bin/bash student; } && \
    { id operator &>/dev/null || useradd -m -s /bin/bash operator; } && \
    ...
```

This prevents potential fragility with `||` and `&&` chaining over multiple lines.

---

## `--systemd=true` (Podman) is not enough

`--systemd=true` sets up systemd as PID 1 (tmpfs mounts, cgroup hierarchy, stop signal) but grants no extra capabilities. This lab needs:

| Capability | Used by |
|---|---|
| `NET_ADMIN` | `firewalld`, NetworkManager |
| `SYS_ADMIN` | `hostnamectl`, `semanage`, cgroup management |
| `SYS_PTRACE` | `ps`, `lsof` |

Use `--privileged` instead.

---

## Dnf Doc Stripping

AlmaLinux (and most container images) sets `tsflags=nodocs` in `/etc/dnf/dnf.conf`. This prevents man pages and documentation from being installed to save space. Even if you install `man-pages`, the files won't exist until this flag is removed.

---

## SELinux (Disabled for Compatibility)

The images have SELinux explicitly **disabled** in `/etc/selinux/config`. 

- **Why?**: Core binaries like `sshd` and PAM modules are SELinux-aware. On kernels where SELinux is not available (such as Docker Desktop on Windows/Mac or standard WSL2), these binaries may attempt `setcon()` syscalls that fail with `Permission denied`, causing fatal crashes during login.
- **Removed Tools**: SELinux administration tools (`policycoreutils`, etc.) have been removed to reduce complexity and avoid confusion, as they cannot function without an SELinux-enabled kernel.
- **Practice Implications**: While you cannot practice SELinux labeling or policy enforcement in this sandbox, this ensures the environment remains stable and accessible across all operating systems.

---

## NetworkManager & Networking

`NetworkManager` is enabled to allow `nmcli` and `nmtui` practice. In a container, it manages the container's virtual ethernet interface. It cannot create or modify hardware-level networking (like bonding or VLANs) the same way it would on a physical RHEL node.

---

## Auditd Limitations

`auditd` is not installed because the Linux kernel audit system is not fully namespaced. The daemon typically fails to start inside a container because it cannot claim the audit netlink socket which is already owned by the host's audit system.

---

## CRB / PowerTools

- **RHEL 8:** `dnf config-manager --set-enabled powertools`

---

## AlmaLinux vs UBI

UBI (`ubi8`) has a stripped repo — `podman`, `firewalld`, `setools-console`, `nmap`, etc. are missing or subscription-gated. AlmaLinux has the full repo.

---

## `operator` user guard

AlmaLinux ships a system `operator` account. The Dockerfile guards with `id operator &>/dev/null || useradd ...` to avoid a build-time duplicate-user error.

---

## Masked systemd units

Masked at build time to stop log spam. All are hardware/device/TTY units that don't apply in a container.

| Unit | Reason |
|---|---|
| `dev-hugepages.mount` | Host kernel feature |
| `sys-fs-fuse-connections.mount` | No physical devices |
| `systemd-udevd.service` | No hardware events |
| `systemd-udev-trigger.service` | No hardware events |
| `systemd-update-utmp.service` | No login tracking needed |
| `systemd-tmpfiles-setup-dev.service` | No device nodes |
| `getty.target` | No TTY |
| `console-getty.service` | No console |
| `getty@tty1.service` | No TTY |

---

## `STOPSIGNAL SIGRTMIN+3`

systemd ignores `SIGTERM`. `SIGRTMIN+3` triggers a clean shutdown (`systemctl poweroff`). Without this, `docker stop` / `docker compose down` would hang or kill PID 1 abruptly.

---

## SSH

`PermitRootLogin yes` and `PasswordAuthentication yes` are set in `sshd_config` at build time. Lab-only — don't expose these ports.

| Service | Host port |
|---|---|
| `rhel8` | `2222` |

---

## `/etc/shadow` Permissions — SSH Password Auth Failure

**Symptom**: SSH password authentication fails with `Permission denied` (or PAM reports "user unknown") when running the container on Debian/Ubuntu-based hosts, Linux Mint, or any environment with a hardened Seccomp profile.

**Root cause**: AlmaLinux / RHEL `shadow-utils` creates `/etc/shadow` with permissions `0000` (no read access for anyone) by design. On bare-metal this is safe because the PAM helper binary `unix_chkpwd` carries the **SUID** bit and always executes as `root`, so it can read the shadow file regardless of its permissions.

Inside containers, many Docker hosts (Debian/Ubuntu kernels, Linux Mint, Docker Desktop with strict Seccomp) **strip the SUID bit** from container binaries. Without the SUID bit, `unix_chkpwd` runs as the unprivileged calling user and cannot open a `0000` shadow file — PAM reports authentication failure and SSH refuses the login.

**Fix applied**:

```dockerfile
chmod 0600 /etc/shadow
```

`0600` keeps the file readable **only by root** (matching RHEL bare-metal defaults on many real systems) while guaranteeing that any root-level process — including `unix_chkpwd` running as root after losing its SUID bit — can still read it. This makes SSH password authentication work reliably across all Docker host environments.