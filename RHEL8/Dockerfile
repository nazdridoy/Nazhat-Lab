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
    # ── systemd ───────────────────────────────────────────────────────────
    systemd \
    # ── SELinux tools ─────────────────────────────────────────────────────
    policycoreutils \
    policycoreutils-python-utils \
    setools-console \
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

# ── 3. Configure SSH ─────────────────────────────────────────────────────────
RUN ssh-keygen -A && \
    sed -i \
    -e 's/^#PermitRootLogin.*/PermitRootLogin yes/' \
    -e 's/^#PasswordAuthentication.*/PasswordAuthentication yes/' \
    /etc/ssh/sshd_config && \
    systemctl enable sshd

# ── 4. Create course users (mirrors RH124 lab accounts) ──────────────────────
RUN { id student  &>/dev/null || useradd -m -s /bin/bash student; } && \
    { id operator &>/dev/null || useradd -m -s /bin/bash operator; } && \
    usermod -s /bin/bash operator && \
    usermod -aG wheel student && \
    echo "root:redhat"         | chpasswd && \
    echo "student:student"     | chpasswd && \
    echo "operator:operator"   | chpasswd && \
    echo "%wheel ALL=(ALL) ALL" >> /etc/sudoers

# ── 5. Mask systemd units that don't apply inside a container ────────────────
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

# ── 6. Default hostname ──────────────────────────────────────────────────────
# Disable NM hostname management and use a oneshot service to set the 
# kernel hostname, overriding Docker's /etc/hostname bind-mount at boot.
RUN mkdir -p /etc/NetworkManager/conf.d && \
    printf '[main]\nhostname-mode=none\n' \
    > /etc/NetworkManager/conf.d/99-no-hostname.conf
RUN printf '[Unit]\nDescription=Set default hostname\nAfter=sysinit.target\nBefore=network-pre.target\n\n[Service]\nType=oneshot\nExecStart=/bin/hostname nazhat-lab\nRemainAfterExit=yes\n\n[Install]\nWantedBy=basic.target\n' \
    > /etc/systemd/system/nazhat-hostname.service \
    && systemctl enable nazhat-hostname.service

# ── 7. Shell environment ──────────────────────────────────────────────────────
RUN echo 'PS1="[\u@\h \W]\\$ "' >> /etc/bashrc && \
    echo 'alias ll="ls -lah --color=auto"' >> /etc/bashrc && \
    echo 'alias grep="grep --color=auto"'  >> /etc/bashrc

# ── 7. Container settings ─────────────────────────────────────────────────────
EXPOSE 22
WORKDIR /home/student

# ── 8. Entrypoint ─────────────────────────────────────────────────────────────
# Run systemd as PID 1 — enables systemctl, hostnamectl, firewall-cmd, etc.
# sshd starts automatically via the enabled unit (see step 3).
STOPSIGNAL SIGRTMIN+3
CMD ["/sbin/init"]
