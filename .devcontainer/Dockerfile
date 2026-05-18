# syntax=docker/dockerfile:1.8
# check=error=true

## Multi-stage build!
# Pull prebuilt Echidna binary (pinned release — update ECHIDNA_VERSION to upgrade).
# See https://github.com/crytic/echidna/releases for available versions.
FROM --platform=linux/amd64 ghcr.io/crytic/echidna/echidna:v2.2.4 AS echidna

# Grab python 3.12 (pinned to specific patch release).
FROM python:3.12.7-slim AS python-base

# Base ubuntu devcontainer (pinned to ubuntu-22.04 definition series 0).
FROM mcr.microsoft.com/vscode/devcontainers/base:0-ubuntu-22.04

# Switch to root (the default might be root anyway)
USER root

COPY --from=python-base /usr/local /usr/local

# Super basic stuff to get everything started
RUN apt-get update -y && apt-get install -y \
    python3-dev libpython3-dev build-essential vim curl git sudo pkg-config \
    --no-install-recommends

# The base container usually has a "vscode" user. If not, create one here.
RUN echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Switch to vscode (drop privs)
USER vscode
WORKDIR /home/vscode
ENV HOME=/home/vscode

# Set needed paths (for python, pip, pnpm)
ENV USR_LOCAL_BIN=/usr/local/bin
ENV LOCAL_BIN=${HOME}/.local/bin
ENV PNPM_HOME=${HOME}/.local/share/pnpm
ENV PATH=${PATH}:${USR_LOCAL_BIN}:${LOCAL_BIN}:${PNPM_HOME}

# Pinned installer versions — update these ENV vars to upgrade the corresponding tool.
# uv: https://github.com/astral-sh/uv/releases
ENV UV_VERSION="0.4.20"
# nvm: https://github.com/nvm-sh/nvm/releases
ENV NVM_VERSION="0.40.1"
# Node.js LTS: https://nodejs.org/en/download/releases
ENV NODE_VERSION="22.9.0"
# Foundry: https://github.com/foundry-rs/foundry/releases
# Use a nightly hash tag (e.g. "nightly-abc1234") or stable tag for full pinning.
ENV FOUNDRY_VERSION="stable"
# Aderyn (via cyfrinup): https://github.com/Cyfrin/aderyn/releases
ENV ADERYN_VERSION="v0.4.0"
# ityfuzz: https://github.com/fuzzland/ityfuzz/releases
ENV ITYFUZZ_VERSION="v0.8.1"

# Install uv (pinned version)
RUN python3 -m pip install --no-cache-dir uv==${UV_VERSION}

ENV SHELL=/usr/bin/bash
SHELL ["/usr/bin/bash", "-ic"]

USER root
## Install nvm (pinned) + Node.js (pinned) + pnpm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v${NVM_VERSION}/install.sh | bash && \
    export NVM_DIR="${HOME}/.nvm" && \
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" && \
    nvm install ${NODE_VERSION} && \
    nvm use ${NODE_VERSION} && \
    nvm alias default ${NODE_VERSION} && \
    npm install -g pnpm && \
    chown -R vscode:vscode ${HOME}/.npm ${HOME}/.nvm
USER vscode

# Install hardhat at the pinned version from package.json.
# When Dependabot bumps package.json, update the version here too.
RUN pnpm install --global hardhat@2.22.12

# Python security tool installations.
# Versions are managed in requirements.txt (Dependabot tracks that file).
# When Dependabot opens a PR bumping a version in requirements.txt, the same
# version must be applied here in the xargs command (both files stay in sync).
COPY .devcontainer/requirements.txt /tmp/tool-requirements.txt
RUN grep -v '^#' /tmp/tool-requirements.txt | grep -v '^[[:space:]]*$' | \
    xargs -I {} uv tool install {} && \
    solc-select install 0.4.26 0.5.17 0.6.12 0.7.6 0.8.10 0.8.26 && \
    solc-select use 0.8.26

# Fetch and install setups
## ityfuzz (version pinned via ITYFUZZ_VERSION)
RUN curl -fsSL https://ity.fuzz.land/ | bash
RUN ityfuzzup --version ${ITYFUZZ_VERSION}

## Foundry framework (version pinned via FOUNDRY_VERSION)
RUN curl -fsSL https://foundry.paradigm.xyz | bash
RUN foundryup --version ${FOUNDRY_VERSION}

## Aderyn (version pinned via ADERYN_VERSION)
RUN curl -fsSL https://raw.githubusercontent.com/Cyfrin/up/main/install | bash
RUN cyfrinup --version ${ADERYN_VERSION}

# Copy prebuilt Echidna binary
COPY --chown=vscode:vscode --from=echidna /usr/local/bin/echidna ${HOME}/.local/bin/echidna
RUN chmod 755 ${HOME}/.local/bin/echidna

# Back to home in case we want to do something later.
WORKDIR ${HOME}


## Add completions for anvil, cast, forge.
RUN mkdir -p ${HOME}/.local/share/bash-completion/completions && \
    forge completions bash > ${HOME}/.local/share/bash-completion/completions/forge && \
    cast completions bash > ${HOME}/.local/share/bash-completion/completions/cast && \
    anvil completions bash > ${HOME}/.local/share/bash-completion/completions/anvil

USER root

## Clean
RUN apt-get autoremove -y && apt-get clean -y

## back to user!
USER vscode

# Example HEALTHCHECK, we don't need once since we're not using services. If you add services in the future, you would need to add "something" like this:
HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 CMD \
  bash -c 'command -v echidna && command -v slither && command -v solc && echo "OK" || exit 1'
