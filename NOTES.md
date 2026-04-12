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

## SELinux Reality

The images include `policycoreutils` and `setools-console` for practicing commands (`semanage`, `getsebool`, etc.). However, SELinux is a kernel feature. Enforcement happens at the host level, not within the container's isolated view. Tools can manipulate policy files, but the "Enforcing" status reflects the host's actual state.

---

## NetworkManager & Networking

`NetworkManager` is enabled to allow `nmcli` and `nmtui` practice. In a container, it manages the container's virtual ethernet interface. It cannot create or modify hardware-level networking (like bonding or VLANs) the same way it would on a physical RHEL node.

---

## Auditd Limitations

`auditd` is not installed because the Linux kernel audit system is not fully namespaced. The daemon typically fails to start inside a container because it cannot claim the audit netlink socket which is already owned by the host's audit system.

---

## CRB / PowerTools — name differs by version

- **RHEL 8:** `dnf config-manager --set-enabled powertools`
- **RHEL 9:** `/usr/bin/crb enable` (ships with `epel-release`)

---

## AlmaLinux vs UBI

UBI (`ubi8`/`ubi9`) has a stripped repo — `podman`, `firewalld`, `setools-console`, `nmap`, etc. are missing or subscription-gated. AlmaLinux has the full repo.

---

## `operator` user guard

AlmaLinux 9 may ship a system `operator` account. Both Dockerfiles guard with `id operator &>/dev/null || useradd ...` to avoid a build-time duplicate-user error.

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
| `rhel9` | `2222` |
| `rhel8` | `3333` |