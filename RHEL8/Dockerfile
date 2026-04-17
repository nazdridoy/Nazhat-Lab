# ============================================================
# Nazhat-Lab – RedHat System Administration I (RH124) Sandbox
# Base: AlmaLinux 8 — free, 100% RHEL 8 binary-compatible
#       (UBI8 has a stripped repo set that blocks many packages)
# ============================================================


# MIT License
# 
# Copyright (c) 2026 nazDridoy
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

FROM almalinux:8

LABEL maintainer="nazhat-lab"
LABEL description="All tools needed for the Red Hat System Administration I (RH124) course — RHEL 8"

# ── 1. Enable EPEL 8 + CodeReady Builder (PowerTools) ───────────────────────
RUN dnf install -y epel-release \
    && dnf config-manager --set-enabled powertools \
    && dnf clean all

# ── 2. Core system utilities ─────────────────────────────────────────────────
RUN dnf install -y --allowerasing \
    # ── Shell & text editors ──────────────────────────────────────────────
    bash \
    vim-enhanced \
    nano \
    less \
    # ── Manual pages ──────────────────────────────────────────────────────
    man-db \
    man-pages \
    pinfo \
    # ── File & directory management ───────────────────────────────────────
    coreutils \
    findutils \
    diffutils \
    file \
    tree \
    # ── Archiving & compression ───────────────────────────────────────────
    tar \
    gzip \
    bzip2 \
    xz \
    zip \
    unzip \
    rsync \
    # ── Text processing ───────────────────────────────────────────────────
    grep \
    sed \
    gawk \
    util-linux \
    # ── Process management ────────────────────────────────────────────────
    procps-ng \
    psmisc \
    # ── Networking ───────────────────────────────────────────────────────
    iproute \
    iputils \
    net-tools \
    bind-utils \
    traceroute \
    nmap \
    nmap-ncat \
    curl \
    wget \
    NetworkManager \
    # ── Package management utilities ──────────────────────────────────────
    dnf-utils \
    rpm \
    # ── User & group management ───────────────────────────────────────────
    shadow-utils \
    sudo \
    passwd \
    # ── SSH ───────────────────────────────────────────────────────────────
    openssh-server \
    openssh-clients \
    # ── Storage & filesystem ──────────────────────────────────────────────
    parted \
    lsof \
    # ── systemd & Logging ─────────────────────────────────────────────────
    systemd \
    rsyslog \
    logrotate \
    chrony \
    # ── Firewall ──────────────────────────────────────────────────────────
    firewalld \
    # ── Containers (introductory chapter) ────────────────────────────────
    podman \
    # ── Misc helpers ──────────────────────────────────────────────────────
    hostname \
    which \
    bash-completion \
    time \
    sos \
    && dnf clean all

# ── 3. Disable SELinux (not functional in containers; causes sshd/PAM failures
#       on non-SELinux kernels such as Docker Desktop on Windows) ─────────────
RUN sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

# ── 4. Configure SSH ─────────────────────────────────────────────────────────
RUN ssh-keygen -A && \
    sed -i \
    -e 's/^#PermitRootLogin.*/PermitRootLogin yes/' \
    -e 's/^#PasswordAuthentication.*/PasswordAuthentication yes/' \
    /etc/ssh/sshd_config && \
    systemctl enable sshd

# ── 5. Create course users (mirrors RH124 lab accounts) ──────────────────────
RUN { id student  &>/dev/null || useradd -m -s /bin/bash student; } && \
    { id operator &>/dev/null || useradd -m -s /bin/bash operator; } && \
    usermod -s /bin/bash operator && \
    usermod -aG wheel student && \
    echo "root:redhat"         | chpasswd && \
    echo "student:student"     | chpasswd && \
    echo "operator:operator"   | chpasswd && \
    echo "%wheel ALL=(ALL) ALL" >> /etc/sudoers && \
    # AlmaLinux/RHEL shadow-utils sets /etc/shadow to 0000 by default.
    # On bare-metal this is fine because unix_chkpwd has the SUID bit and
    # always runs as root. Inside containers, however, many Docker hosts
    # (Debian/Ubuntu-based, Linux Mint, hardened Seccomp profiles) strip the
    # SUID bit — so unix_chkpwd cannot read a 0000 shadow file and PAM
    # reports "user unknown", causing SSH password auth to fail.
    # Setting 0600 keeps the file root-only while guaranteeing root-level
    # processes (like unix_chkpwd running as root) can always read it.
    chmod 0600 /etc/shadow

# ── 6. Mask systemd units that don't apply inside a container ────────────────
# These units fail or spam logs in containerized environments.
RUN systemctl mask \
    dev-hugepages.mount \
    sys-fs-fuse-connections.mount \
    systemd-update-utmp.service \
    systemd-tmpfiles-setup-dev.service \
    systemd-udev-trigger.service \
    systemd-udevd.service \
    getty.target \
    console-getty.service \
    getty@tty1.service

# ── 7. Enable system services & pre-create standard log files ───────────────
# rsyslog only appends to files that already exist; pre-touch them so
# /var/log/secure, /var/log/messages, etc. are present from first boot.
RUN systemctl enable rsyslog chronyd && \
    touch /var/log/messages \
          /var/log/secure \
          /var/log/maillog \
          /var/log/cron \
          /var/log/boot.log && \
    chmod 600 /var/log/secure

# ── 8. Default hostname ──────────────────────────────────────────────────────
# Disable NM hostname management and use a oneshot service to set the 
# kernel hostname, overriding Docker's /etc/hostname bind-mount at boot.
RUN mkdir -p /etc/NetworkManager/conf.d && \
    printf '[main]\nhostname-mode=none\n' \
    > /etc/NetworkManager/conf.d/99-no-hostname.conf
RUN printf '[Unit]\nDescription=Set default hostname\nAfter=sysinit.target\nBefore=network-pre.target\n\n[Service]\nType=oneshot\nExecStart=/bin/hostname nazhat-lab\nRemainAfterExit=yes\n\n[Install]\nWantedBy=basic.target\n' \
    > /etc/systemd/system/nazhat-hostname.service \
    && systemctl enable nazhat-hostname.service

# ── 8. Shell environment ──────────────────────────────────────────────────────
RUN echo 'PS1="[\u@\h \W]\\$ "' >> /etc/bashrc && \
    echo 'alias ll="ls -lah --color=auto"' >> /etc/bashrc && \
    echo 'alias grep="grep --color=auto"'  >> /etc/bashrc

# ── 9. Container settings ─────────────────────────────────────────────────────
EXPOSE 22
WORKDIR /home/student

# ── 10. Entrypoint ────────────────────────────────────────────────────────────
# Run systemd as PID 1 — enables systemctl, hostnamectl, firewall-cmd, etc.
# sshd starts automatically via the enabled unit (see step 3).
STOPSIGNAL SIGRTMIN+3
CMD ["/sbin/init"]
