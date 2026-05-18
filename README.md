# Solar Space Devcontainer

The Solar Space Devcontainer provides a pre-configured development environment for
Ethereum projects.

## What's included

### Base environment
- Ubuntu-based VS Code devcontainer image (`ghcr.io/solarspace-dev/solar-blank:latest`)
- Python 3.12 with `uv`
- Node.js toolchain (npm, pnpm, yarn, nvm)
- Common build tooling (`build-essential`, `git`, `curl`, `vim`, etc.)

### Smart contract tooling
- Hardhat (global install)
- Foundry (`forge`, `cast`, `anvil`) with bash completions
- Echidna
- Slither ecosystem tools:
  - `slither-analyzer`
  - `slither-lsp`
  - `slitherin`
  - `crytic-compile`
- `solc-select` with multiple compiler versions preinstalled
- `vyper`
- `semgrep`
- `ityfuzz`
- `aderyn` (via `cyfrinup`)
- `halmos`

### VS Code extensions
- `solarspace.solarspace`
- `juanblanco.solidity`
- `trailofbits.weaudit`
- `runtimeverification.simbolik`
