# Base image
FROM debian:bookworm-slim AS base

# Arguments
ARG USER=vscode
ARG DEBIAN_FRONTEND=noninteractive

# Install sudo and other dependencies
RUN apt update && apt install -y --no-install-recommends \
    sudo \
    ca-certificates \
    curl \
    git \
    unzip \
    gnupg \
    && apt autoremove -y \
    && rm -rf /var/lib/apt/lists/* \
    && useradd -m -s /usr/bin/bash ${USER} \
    && echo "${USER} ALL=(ALL) NOPASSWD: ALL" >/etc/sudoers.d/${USER} \
    && chmod 0440 /etc/sudoers.d/${USER}

# PowerShell installation stage
FROM base AS powershell

# Install PowerShell based on architecture
RUN ARCH=$(dpkg --print-architecture) && \
    if [ "$ARCH" = "amd64" ]; then \
    curl -sSL https://github.com/PowerShell/PowerShell/releases/download/v7.5.0/powershell-7.5.0-linux-x64.tar.gz -o /tmp/powershell.tar.gz; \
    elif [ "$ARCH" = "arm64" ]; then \
    curl -sSL https://github.com/PowerShell/PowerShell/releases/download/v7.5.0/powershell-7.5.0-linux-arm64.tar.gz -o /tmp/powershell.tar.gz; \
    else \
    echo "Unsupported architecture: $ARCH" && exit 1; \
    fi && \
    mkdir -p /opt/microsoft/powershell/7 && \
    tar -xzf /tmp/powershell.tar.gz -C /opt/microsoft/powershell/7 && \
    ln -s /opt/microsoft/powershell/7/pwsh /usr/bin/pwsh && \
    rm /tmp/powershell.tar.gz

# Ensure PowerShell has the correct permissions
RUN chmod +x /usr/bin/pwsh

# Set PowerShell as the default shell
SHELL ["/usr/bin/pwsh", "-c"]

# Oh My Posh installation stage
FROM powershell AS ohmyposh

# Run as non-root user
USER ${USER}

# Install Oh My Posh
RUN curl -s https://ohmyposh.dev/install.sh | bash

# Download Oh My Posh themes
RUN mkdir -p /home/${USER}/.poshthemes && \
    curl -s https://ohmyposh.dev/themes | tar -xz -C /home/${USER}/.poshthemes

# Configure Oh My Posh in PowerShell profile
RUN mkdir -p /home/${USER}/.config/powershell/ && \
    echo 'oh-my-posh init pwsh --config /home/${USER}/.poshthemes/paradox.omp.json | Invoke-Expression' >> /home/${USER}/.config/powershell/Microsoft.PowerShell_profile.ps1



# Final stage
FROM ohmyposh AS final
