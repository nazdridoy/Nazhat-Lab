RH124 practice environment provided as a Docker/Podman sandbox. Emulates the Red Hat System Administration I (RH124) lab environment for RHEL 8.

---

## Project Structure

```text
nazhat-lab/
├── RHEL8/
│   └── Dockerfile          # AlmaLinux 8 (RHEL 8 binary-compatible)
└── docker-compose.yml      # Recommended run configuration
```

---

## Quick Start

### 1. Pull

```bash
# Latest
docker pull nazdridoy/nazhat-lab:rhel8

# Specific version
docker pull nazdridoy/nazhat-lab:rhel8-v1.0
```

### 2. Build & Push

Builds apply two tags: a floating `:rhel8` and a pinned `:rhel8-vX.Y`. Update the `VER` variable when modifying the Dockerfile.

```bash
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
Containers are assigned sequential SSH ports starting at `2222` (supports up to 100 simultaneous containers).

**Single container:**

```bash
docker compose up -d
```

**Multiple containers (e.g. 3 × RHEL 8):**

```bash
docker compose up -d --scale rhel8=3
```

**List containers and their SSH ports:**

```bash
docker compose ps
```

Example output:

```
NAME                  STATUS    PORTS
nazhat-lab-rhel8-1    Up        127.0.0.1:2222->22/tcp
nazhat-lab-rhel8-2    Up        127.0.0.1:2223->22/tcp
nazhat-lab-rhel8-3    Up        127.0.0.1:2224->22/tcp
```

**Stop and remove all containers:**

```bash
docker compose down
```

### Alternative: Plain Docker

```bash
# RHEL 8 (single, fixed port)
docker run -d --privileged -p 2222:22 --name nazhat8 nazdridoy/nazhat-lab:rhel8

# RHEL 8 (dynamic port — lettng Docker pick)
docker run -d --privileged -p 127.0.0.1::22 nazdridoy/nazhat-lab:rhel8
```

### Alternative: Podman

Podman includes native systemd support (`--systemd=true`).

```bash
podman run -d --privileged -p 127.0.0.1::22 nazdridoy/nazhat-lab:rhel8
```

### Accessing the Container

**Via SSH** — first container is always `2222`, second is `2223`, and so on:

```bash
ssh student@localhost -p 2222   # 1st container — Password: student
ssh root@localhost    -p 2223   # 2nd container — Password: redhat
```

**Via Shell** — use the container name shown in `docker compose ps`:

```bash
docker exec -it <container-name> bash
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
timedatectl set-timezone Asia/Dhaka
timedatectl set-ntp true
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
| Init & Logging | `systemd`, `rsyslog`, `chrony` |
| Firewall | `firewalld` |
| Emulation Base | `podman` |
| Diagnostics | `hostname`, `which`, `bash-completion`, `time`, `sos` |

> [!NOTE] 
> SELinux is explicitly **disabled** in these images to ensure compatibility with non-SELinux kernels (like Docker Desktop on Windows). See [NOTES.md](./NOTES.md) for details.

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
# Stop and remove a container (use the name from `docker ps`)
docker stop <container-name> && docker rm <container-name>
# or
podman stop <container-name> && podman rm <container-name>

# Remove images
docker rmi nazdridoy/nazhat-lab:rhel8
# or
podman rmi nazdridoy/nazhat-lab:rhel8
```

---

## License

MIT © 2026 nazDridoy
