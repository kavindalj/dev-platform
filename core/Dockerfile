# -----------------------------------------------------------------------------
# Dev Base Container
# -----------------------------------------------------------------------------
# Base: Debian Trixie
#
# Purpose:
# - Portable development environment
# - Neovim + tmux + terminal tooling
# - Suitable as a reusable base image
#
# -----------------------------------------------------------------------------

FROM debian:trixie

ENV DEBIAN_FRONTEND=noninteractive
ENV TERM=xterm-256color

# -----------------------------------------------------------------------------
# Install core development tools
# -----------------------------------------------------------------------------

RUN apt-get update && apt-get install -y \
    sudo \
    git \
    tmux \
    zsh \
    curl \
    wget \
    unzip \
    zip \
    tar \
    gzip \
    gpg \
    bat \
    eza \
    starship \
    openssh-client \
    ripgrep \
    fd-find \
    fzf \
    tree \
    less \
    jq \
    yq \
    btop \
    xz-utils \
    ca-certificates \
    inetutils-ping \
    dnsutils \
    traceroute \
    tcpdump \
    net-tools \
    iproute2 \
    procps \
    lsof \
    strace \
    build-essential \
    make \
    pkg-config \
    man-db \
    manpages \
    locales \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


# -----------------------------------------------------------------------------
# Install latest stable Neovim
# -----------------------------------------------------------------------------

ARG NVIM_VERSION=0.12.5

RUN ARCH="$(dpkg --print-architecture)" && \
    case "$ARCH" in \
        amd64) NVIM_ARCH="x86_64" ;; \
        arm64) NVIM_ARCH="arm64" ;; \
        *) echo "Unsupported architecture: $ARCH" >&2; exit 1 ;; \
    esac && \
    curl -fL \
        "https://github.com/neovim/neovim-releases/releases/download/v${NVIM_VERSION}/nvim-linux-${NVIM_ARCH}.tar.gz" \
        -o /tmp/nvim.tar.gz && \
    tar -xzf /tmp/nvim.tar.gz -C /opt && \
    rm /tmp/nvim.tar.gz

ENV PATH="/opt/nvim-linux-x86_64/bin:/opt/nvim-linux-arm64/bin:${PATH}"


# -----------------------------------------------------------------------------
# Configure locale
# -----------------------------------------------------------------------------

RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && \
    locale-gen

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# -----------------------------------------------------------------------------
# Create non-root user
# -----------------------------------------------------------------------------

ARG USERNAME=dev
ARG UID=1000
ARG GID=1000

RUN groupadd -g ${GID} ${USERNAME} && \
    useradd -m -u ${UID} -g ${GID} -s /usr/bin/zsh ${USERNAME}

RUN echo "${USERNAME} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# -----------------------------------------------------------------------------
# Switch to user
# -----------------------------------------------------------------------------

USER ${USERNAME}
WORKDIR /home/${USERNAME}

# -----------------------------------------------------------------------------
# Install zsh plugins
# -----------------------------------------------------------------------------

RUN git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.config/zsh/zsh-syntax-highlighting

RUN git clone https://github.com/zsh-users/zsh-autosuggestions ~/.config/zsh/zsh-autosuggestions

# -----------------------------------------------------------------------------
# copy configs
# -----------------------------------------------------------------------------

COPY --chown=${USERNAME}:${USERNAME} config /home/${USERNAME}/.config
COPY --chown=${USERNAME}:${USERNAME} config/zsh/zshrc /home/${USERNAME}/.zshrc

# -----------------------------------------------------------------------------
# Install posting
# -----------------------------------------------------------------------------

RUN curl -LsSf https://astral.sh/uv/install.sh | sh
RUN . "$HOME/.local/bin/env" && uv tool install --python 3.12 posting

# -----------------------------------------------------------------------------
# Install opencode
# -----------------------------------------------------------------------------

RUN curl -L \
    https://github.com/anomalyco/opencode/releases/latest/download/opencode-linux-x64.tar.gz \
    -o /tmp/opencode.tar.gz && \
    tar -xzf /tmp/opencode.tar.gz -C /tmp && \
    mv /tmp/opencode /home/${USERNAME}/.local/bin/opencode && \
    chmod +x /home/${USERNAME}/.local/bin/opencode

ENV PATH="/home/${USERNAME}/.local/bin:$PATH"

# -----------------------------------------------------------------------------
# Cleanup
# -----------------------------------------------------------------------------

RUN rm -rf /tmp/opencode.tar.gz
RUN rm -rf ~/.cache

# -----------------------------------------------------------------------------
# Default shell
# -----------------------------------------------------------------------------

CMD ["/usr/bin/zsh"]
