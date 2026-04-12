# Nazhat-Lab

RH124 practice environment provided as a Docker/Podman sandbox. Emulates the Red Hat System Administration I (RH124) lab environment for both RHEL 8 and RHEL 9.

The containers run systemd as PID 1. Tools like `systemctl`, `hostnamectl`, `firewall-cmd`, and `journalctl` function as they would on a standard RHEL system.

---

## Project Structure

```text
nazhat-lab/
├── RHEL8/
│   └── Dockerfile          # AlmaLinux 8 (RHEL 8 binary-compatible)
├── RHEL9/
│   └── Dockerfile          # AlmaLinux 9 (RHEL 9 binary-compatible)
└── docker-compose.yml      # Recommended run configuration
```

---

## Quick Start

### 1. Pull

```bash
# Latest
docker pull nazdridoy/nazhat-lab:rhel8
docker pull nazdridoy/nazhat-lab:rhel9

# Specific version
docker pull nazdridoy/nazhat-lab:rhel8-v1.0
docker pull nazdridoy/nazhat-lab:rhel9-v1.0
```

### 2. Build & Push

Builds apply two tags: a floating `:rhelX` and a pinned `:rhelX-vX.Y`. Update the `VER` variable when modifying the Dockerfile.

```bash
# RHEL 9
VER=1.0
docker build \
  -t nazdridoy/nazhat-lab:rhel9 \
  -t nazdridoy/nazhat-lab:rhel9-v${VER} \
  ./RHEL9
docker push nazdridoy/nazhat-lab:rhel9
docker push nazdridoy/nazhat-lab:rhel9-v${VER}

# RHEL 8
VER=1.0
docker build \
  -t nazdridoy/nazhat-lab:rhel8 \
  -t nazdridoy/nazhat-lab:rhel8-v${VER} \
  ./RHEL8
docker push nazdridoy/nazhat-lab:rhel8
docker push nazdridoy/nazhat-lab:rhel8-v${VER}
```

---

## Running the Container

Running systemd as PID 1 requires specific flags for cgroup management. Docker Compose is recommended for convenience.

### Recommended: Docker Compose

`docker-compose.yml` provides the intended configuration out-of-the-box.

```bash
# Start RHEL 9
docker compose up -d rhel9

# Start RHEL 8
docker compose up -d rhel8

# Start both
docker compose up -d

# Stop and remove
docker compose down
```

| Service | SSH port |
|---------|----------|
| `rhel9` | `2222`   |
| `rhel8` | `3333`   |

### Alternative: Plain Docker

```bash
# RHEL 9
docker run -d --privileged -p 2222:22 --name nazhat9 nazdridoy/nazhat-lab:rhel9

# RHEL 8
docker run -d --privileged -p 3333:22 --name nazhat8 nazdridoy/nazhat-lab:rhel8
```

### Alternative: Podman

Podman includes native systemd support (`--systemd=true`).

```bash
podman run -d --systemd=true --cap-add NET_ADMIN -p 2222:22 --name nazhat9 nazdridoy/nazhat-lab:rhel9
podman run -d --systemd=true --cap-add NET_ADMIN -p 3333:22 --name nazhat8 nazdridoy/nazhat-lab:rhel8
```

### Accessing the Container

**Via SSH:**

```bash
ssh student@localhost -p 2222   # Password: student
ssh root@localhost    -p 2222   # Password: redhat
```

**Via Shell:**

```bash
docker exec -it nazhat9 bash
# or
podman exec -it nazhat9 bash
```

---

## Environment Configuration

### Accounts

| User       | Password   | Groups       |
|------------|------------|--------------|
| `root`     | `redhat`   | root         |
| `student`  | `student`  | wheel (sudo) |
| `operator` | `operator` | —            |

---

### systemd Initialization

Having systemd configured as PID 1 allows native services to operate normally:

```bash
hostnamectl set-hostname nazhat-lab.example.com
systemctl start firewalld
systemctl enable --now sshd
firewall-cmd --add-service=http --permanent
journalctl -xe
```

The following systemd units are masked during build to suppress container-irrelevant errors:

| Masked Unit | Reason |
|---|---|
| `dev-hugepages.mount` | Host kernel feature |
| `sys-fs-fuse-connections.mount` | FUSE / No physical filesystem |
| `systemd-udevd.service` | No hardware events |
| `systemd-udev-trigger.service` | No hardware events |
| `systemd-update-utmp.service` | Login tracking |
| `systemd-tmpfiles-setup-dev.service` | Device node setup |
| `getty.target` / `console-getty.service` / `getty@tty1.service` | Virtual terminals |

---

### Installed Packages

| Functional Area | Key Packages |
|--|--|
| Shell / Editors | `bash`, `vim-enhanced`, `nano`, `less` |
| Manuals | `man-db`, `man-pages` |
| File Mgmt | `coreutils`, `findutils`, `tree`, `diffutils`, `file` |
| Archiving | `tar`, `gzip`, `bzip2`, `xz`, `zip`, `unzip`, `rsync` |
| Text Processing | `grep`, `sed`, `gawk`, `util-linux` |
| Process Mgmt | `procps-ng`, `psmisc` |
| Networking | `iproute`, `iputils`, `net-tools`, `bind-utils`, `traceroute`, `nmap`, `nmap-ncat`, `curl`, `wget`, `NetworkManager` |
| Package Mgmt | `dnf-utils`, `rpm` |
| Users | `shadow-utils`, `sudo`, `passwd` |
| Remote Access | `openssh-server`, `openssh-clients` |
| Storage | `parted`, `util-linux`, `lsof` |
| Init System | `systemd` |
| Security Contexts | `policycoreutils`, `policycoreutils-python-utils`, `setools-console` |
| Firewall | `firewalld` |
| Emulation Base | `podman` |
| Diagnostics | `hostname`, `which`, `bash-completion`, `time`, `sos` |

---

## Teardown

Depending on how you started the environment, use one of the following methods to clean up:

**If using Docker Compose (Recommended):**

```bash
# Stop and remove containers and networks
docker compose down

# To also remove the built local images
docker compose down --rmi local
```

**If using Plain Docker or Podman:**

```bash
# Stop and remove containers
docker stop nazhat9 nazhat8 && docker rm nazhat9 nazhat8
# or
podman stop nazhat9 nazhat8 && podman rm nazhat9 nazhat8

# Remove images
docker rmi nazdridoy/nazhat-lab:rhel8 nazdridoy/nazhat-lab:rhel9
# or
podman rmi nazdridoy/nazhat-lab:rhel8 nazdridoy/nazhat-lab:rhel9
```

---

## License

MIT © 2026 nazDridoy
