# syntax=docker/dockerfile:1.8
# check=error=true

## ============================================================
## Pinned tool versions
## To update individual versions, change the ARG defaults below.
## Automated updates are handled by .github/workflows/update-tool-versions.yml
## ============================================================

# Base image versions (must appear before FROM instructions)
ARG ECHIDNA_VERSION=v2.3.2
ARG PYTHON_VERSION=3.12.13-slim-bookworm

## Multi-stage build!
# Pull pinned prebuilt Echidna binary.
FROM --platform=linux/amd64 ghcr.io/crytic/echidna/echidna:${ECHIDNA_VERSION} AS echidna

# Grab pinned python
FROM python:${PYTHON_VERSION} AS python-base

# Base ubuntu devcontainer image (pinned to ubuntu-24.04)
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

# Switch to root (the default might be root anyway)
USER root

COPY --from=python-base /usr/local /usr/local

# Super basic stuff to get everything started
RUN apt-get update -y && apt-get install -y \
    python3-dev libpython3-dev build-essential vim curl git sudo pkg-config xz-utils \
    --no-install-recommends

# The base container usually has a "vscode" user. If not, create one here.
RUN echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Switch to vscode (drop privs)
USER vscode
WORKDIR /home/vscode
ENV HOME=/home/vscode

# Set needed paths (for python, uv, pnpm)
ENV USR_LOCAL_BIN=/usr/local/bin
ENV LOCAL_BIN=${HOME}/.local/bin
ENV PNPM_HOME=${HOME}/.local/share/pnpm
ENV PATH=${PATH}:${USR_LOCAL_BIN}:${LOCAL_BIN}:${PNPM_HOME}

# Tool version ARGs (declared in final stage so they can be used in RUN commands)
ARG UV_VERSION=0.11.14
ARG NVM_VERSION=0.40.4
ARG NODE_VERSION=24.15.0
ARG PNPM_VERSION=11.1.2
ARG HARDHAT_VERSION=3.4.5
ARG SLITHER_VERSION=0.11.5
ARG CRYTIC_COMPILE_VERSION=0.3.11
ARG VYPER_VERSION=0.4.3
ARG SLITHER_LSP_VERSION=0.0.1
ARG SEMGREP_VERSION=1.163.0
ARG SLITHERIN_VERSION=0.7.2
ARG SOLC_SELECT_VERSION=1.2.0
ARG HALMOS_VERSION=0.3.3
ARG FOUNDRY_VERSION=v1.7.1
ARG ADERYN_VERSION=0.6.8
# ityfuzz only publishes nightly builds identified by commit hash
ARG ITYFUZZ_NIGHTLY=nightly-35b7f08962fdd0c2e02df7ef8a43164913d514d9

# Install uv at pinned version
RUN python3 -m pip install --no-cache-dir uv==${UV_VERSION}

ENV SHELL=/usr/bin/bash
SHELL ["/usr/bin/bash", "-ic"]

## Install nvm at pinned version, then install pinned node + pnpm
USER root
RUN curl -o- "https://raw.githubusercontent.com/nvm-sh/nvm/v${NVM_VERSION}/install.sh" | bash && \
    . ${HOME}/.nvm/nvm.sh && \
    nvm install ${NODE_VERSION} && \
    nvm alias default ${NODE_VERSION} && \
    npm install -g pnpm@${PNPM_VERSION}
RUN chown -R vscode:vscode ${HOME}/.nvm ${HOME}/.npm ${HOME}/.local/share/pnpm 2>/dev/null || true
USER vscode

# Expose node/npm/pnpm on PATH without relying solely on nvm interactive-shell init
ENV NVM_DIR=${HOME}/.nvm
ENV PATH="${HOME}/.nvm/versions/node/v${NODE_VERSION}/bin:${PATH}"

RUN pnpm install hardhat@${HARDHAT_VERSION} -g

# Python installations
# Install slither, crytic-compile, solc, vyper, panoramix, slither-lsp (needed for contract explorer).
RUN uv tool install slither-analyzer==${SLITHER_VERSION} && \
    uv tool install crytic-compile==${CRYTIC_COMPILE_VERSION} && \
    uv tool install vyper==${VYPER_VERSION} && \
    uv tool install slither-lsp==${SLITHER_LSP_VERSION} && \
    uv tool install semgrep==${SEMGREP_VERSION} && \
    uv tool install slitherin==${SLITHERIN_VERSION} && \
    uv tool install solc-select==${SOLC_SELECT_VERSION} && \
    solc-select install 0.4.26 0.5.17 0.6.12 0.7.6 0.8.10 latest && solc-select use latest

# Halmos
RUN uv tool install halmos==${HALMOS_VERSION}

## ityfuzz - direct binary download (only nightly builds available upstream)
RUN curl -fsSLo /tmp/ityfuzz.tar.gz \
        "https://github.com/fuzzland/ityfuzz/releases/download/${ITYFUZZ_NIGHTLY}/ityfuzz_nightly_linux_amd64.tar.gz" && \
    mkdir -p ${HOME}/.ityfuzz/bin && \
    tar -xzf /tmp/ityfuzz.tar.gz -C ${HOME}/.ityfuzz/bin && \
    rm /tmp/ityfuzz.tar.gz
ENV PATH="${HOME}/.ityfuzz/bin:${PATH}"

## Foundry framework - direct binary download
RUN curl -fsSLo /tmp/foundry.tar.gz \
        "https://github.com/foundry-rs/foundry/releases/download/${FOUNDRY_VERSION}/foundry_${FOUNDRY_VERSION}_linux_amd64.tar.gz" && \
    mkdir -p ${HOME}/.foundry/bin && \
    tar -xzf /tmp/foundry.tar.gz -C ${HOME}/.foundry/bin && \
    rm /tmp/foundry.tar.gz
ENV PATH="${HOME}/.foundry/bin:${PATH}"

## Aderyn - direct binary download
RUN curl -fsSLo /tmp/aderyn.tar.xz \
        "https://github.com/Cyfrin/aderyn/releases/download/aderyn-v${ADERYN_VERSION}/aderyn-x86_64-unknown-linux-gnu.tar.xz" && \
    mkdir -p ${HOME}/.aderyn/bin && \
    tar -xJf /tmp/aderyn.tar.xz -C ${HOME}/.aderyn/bin && \
    rm /tmp/aderyn.tar.xz
ENV PATH="${HOME}/.aderyn/bin:${PATH}"

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
