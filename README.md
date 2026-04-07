# 🧪 Nazhat-Lab — RH124 Practice Environment

A Docker-based sandbox mirroring the **Red Hat System Administration I (RH124)** lab environment. Available for both **RHEL 8** and **RHEL 9**.

---

## 📁 Project Structure

```
nazhat-lab/
├── RHEL8/
│   └── Dockerfile      # AlmaLinux 8 — RHEL 8 environment
└── RHEL9/
    └── Dockerfile      # AlmaLinux 9 — RHEL 9 environment
```

---

## 🚀 Quick Start

### 1. Pull
```bash
docker pull nazdridoy/nazhat-lab:rhel8
docker pull nazdridoy/nazhat-lab:rhel9
```

### 2. Build & Push (Local)
```bash
# RHEL 8
docker build -t nazdridoy/nazhat-lab:rhel8 ./RHEL8
docker push nazdridoy/nazhat-lab:rhel8

# RHEL 9
docker build -t nazdridoy/nazhat-lab:rhel9 ./RHEL9
docker push nazdridoy/nazhat-lab:rhel9
```

### Run — interactive shell

```bash
docker run -it --rm nazdridoy/nazhat-lab:rhel8
# or
docker run -it --rm nazdridoy/nazhat-lab:rhel9
```

### Run — with SSH access (mirrors real lab workflow)

```bash
docker run -d -p 2222:22 --name rh124-lab nazdridoy/nazhat-lab:rhel8
ssh student@localhost -p 2222          # password: student
ssh root@localhost    -p 2222          # password: redhat
```

---

## 👤 Pre-created Users

| User       | Password   | Groups        |
|------------|------------|---------------|
| `root`     | `redhat`   | root          |
| `student`  | `student`  | wheel (sudo)  |
| `operator` | `operator` | —             |

---

## 🛠️ Installed Tools by Topic

| RH124 Chapter Area       | Key Packages                                              |
|--------------------------|-----------------------------------------------------------|
| Shell & text editors     | `bash`, `vim-enhanced`, `nano`, `less`                    |
| File & directory mgmt    | `coreutils`, `findutils`, `tree`, `diffutils`, `file`    |
| Archiving & compression  | `tar`, `gzip`, `bzip2`, `xz`, `zip`, `unzip`, `rsync`             |
| Text processing          | `grep`, `sed`, `gawk`                                     |
| Process management       | `procps-ng` (ps/top/kill), `psmisc` (killall/pstree)     |
| Networking               | `iproute`, `iputils`, `net-tools`, `bind-utils`, `traceroute`, `nmap` (`nmap-ncat`), `curl`, `wget`, `NetworkManager` |
| Package management       | `dnf-utils`, `rpm`                                        |
| User & group management  | `shadow-utils`, `sudo`, `passwd`                          |
| SSH                      | `openssh-server`, `openssh-clients`                       |
| Storage & filesystem     | `parted`, `util-linux` (lsblk/mount/fdisk), `lsof`       |
| systemd                  | `systemd` (systemctl, journalctl)                         |
| SELinux                  | `policycoreutils`, `policycoreutils-python-utils`, `setools-console` |
| Firewall                 | `firewalld` (firewall-cmd)                                |
| Containers (intro)       | `podman`                                                  |
| Manual pages             | `man-db`, `man-pages`                                     |
| Misc helpers             | `hostname`, `which`, `bash-completion`, `time`, `sos`     |

---

## 🧹 Cleanup

```bash
docker stop rh124-lab && docker rm rh124-lab
docker rmi nazdridoy/nazhat-lab:rhel8 nazdridoy/nazhat-lab:rhel9
```
