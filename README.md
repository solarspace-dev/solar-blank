# Solar Space Devcontainer

The Solar Space Devcontainer provides a pre-configured development environment for
Ethereum projects.

## Claude Code

[Claude Code](https://claude.ai/code) is included in this devcontainer. It is installed automatically after the container is created and is available as the `claude` CLI and as a VS Code extension (`anthropic.claude-code`).

### Authentication

**GitHub Codespaces (recommended):** Add a [Codespaces secret](https://docs.github.com/en/codespaces/managing-your-codespaces/managing-secrets-for-your-codespaces) named `ANTHROPIC_API_KEY` and grant it access to this repository. The secret is automatically injected into the container — no manual login required.

**Local devcontainer:** Set `ANTHROPIC_API_KEY` in your local environment before starting the container. The devcontainer will pick it up automatically.

**Without an API key:** Claude Code will still install, but you will need to run `claude login` inside the terminal to authenticate interactively.

### Usage

Once authenticated, open a terminal in the devcontainer and run:

```bash
# Start an interactive session
claude

# Ask a one-off question
claude "Explain what this contract does"

# Work on a specific file
claude --file src/MyContract.sol "Review this for security issues"
```

The VS Code extension provides the same capabilities directly from the editor sidebar.
