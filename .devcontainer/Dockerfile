# syntax=docker/dockerfile:1

# Define build arguments
ARG BASE_IMAGE_TAG=12.9-slim
ARG BASE_OS=debian

# Stage: PowerShell Downloader
FROM --platform=$BUILDPLATFORM alpine:3.20 AS pwsh_downloader
ARG PWSH_VERSION
WORKDIR /app
COPY get-pwsh.sh .
RUN chmod +x get-pwsh.sh
RUN ./get-pwsh.sh $PWSH_VERSION

# Stage: Final Image
FROM ${BASE_OS}:${BASE_IMAGE_TAG}
ARG TARGETARCH
ARG PWSH_VERSION
ENV PWSH_VERSION=$PWSH_VERSION
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary dependencies and copy PowerShell binaries
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && dpkg --configure -a || true \
    && apt-get install --no-install-recommends  libicu-dev ca-certificates -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/ /var/cache/apt/archives /var/log/apt/ /tmp/*

# Copy PowerShell from the previous stage
COPY --from=pwsh_downloader /powershell/7/$TARGETARCH /opt/microsoft/powershell/7/
RUN ln -s /opt/microsoft/powershell/7/pwsh /usr/bin/pwsh && \
    chmod +x /usr/bin/pwsh

ARG USERNAME=vscode

# Create a non-root user and set permissions
RUN useradd -ms /bin/bash $USERNAME && \
    mkdir -p /home/$USERNAME/.local/share/powershell/Modules && \
    chown -R $USERNAME:$USERNAME /home/$USERNAME


# Setup a neat little PowerShell experience
RUN pwsh -Command Install-Module posh-git -Scope AllUsers -Force; \
    pwsh -Command Install-Module z -Scope AllUsers -Force; \
    pwsh -Command Install-Module PSFzf -Scope AllUsers -Force; \
    pwsh -Command Install-Module Terminal-Icons -Scope AllUsers -Force;

# Install oh-my-posh with debugging
RUN curl https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O /usr/local/bin/oh-my-posh && \
    ls -l /usr/local/bin/oh-my-posh && \
    file /usr/local/bin/oh-my-posh && \
    chmod +x /usr/local/bin/oh-my-posh && \
    /usr/local/bin/oh-my-posh --version


# Set the user to vscode
USER $USERNAME

# Add oh-my-posh to the PowerShell profile
RUN echo 'oh-my-posh init pwsh --config ~/.poshthemes/jandedobbeleer.omp.json | Invoke-Expression' >> /home/$USERNAME/.config/powershell/Microsoft.PowerShell_profile.ps1

ENV PATH=$PATH:/usr/local/pwsh

#CMD ["pwsh"]