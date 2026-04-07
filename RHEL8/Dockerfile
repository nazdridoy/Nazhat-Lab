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
    /etc/ssh/sshd_config

# ── 4. Create course users (mirrors RH124 lab accounts) ──────────────────────
RUN id student  &>/dev/null || useradd -m -s /bin/bash student && \
    id operator &>/dev/null || useradd -m -s /bin/bash operator && \
    usermod -s /bin/bash operator && \
    usermod -aG wheel student && \
    echo "root:redhat"         | chpasswd && \
    echo "student:student"     | chpasswd && \
    echo "operator:operator"   | chpasswd && \
    echo "%wheel ALL=(ALL) ALL" >> /etc/sudoers

# ── 5. Shell environment ──────────────────────────────────────────────────────
RUN echo 'PS1="[\u@nazhat-lab \W]\\$ "' >> /etc/bashrc && \
    echo 'alias ll="ls -lah --color=auto"' >> /etc/bashrc && \
    echo 'alias grep="grep --color=auto"'  >> /etc/bashrc

# ── 6. Expose SSH port ────────────────────────────────────────────────────────
EXPOSE 22

# ── 7. Default working directory ──────────────────────────────────────────────
WORKDIR /home/student

# ── 8. Default user ──────────────────────────────────────────────────────────
# Container runs as root so it can start services without sudo
USER root

# ── 9. Entrypoint ─────────────────────────────────────────────────────────────
# Start sshd in background as root, then hand over to student interactive shell
# If no TTY is attached (e.g. docker run -d), run sshd in foreground to keep container alive
CMD ["/bin/bash", "-c", "if [ -t 0 ]; then /usr/sbin/sshd && exec su - student; else exec /usr/sbin/sshd -D; fi"]
