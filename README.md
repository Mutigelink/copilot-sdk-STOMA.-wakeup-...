# === Replace these placeholders before running ===
UPSTREAM_OWNER="Mutigelink"               # upstream owner
UPSTREAM_REPO="copilot-sdk-STOMA-wakeup"  # upstream repo (replace with exact)
REPO_NAME="$UPSTREAM_REPO"                # local folder name after clone
FORK_OWNER="stomde"                       # your GitHub username
BRANCH="stoma-woken-up"
# =================================================

# 1) Fork upstream to your account and clone (uses gh)
gh repo fork "${UPSTREAM_OWNER}/${UPSTREAM_REPO}" --clone --remote=true

# if you already own the repo and just need to clone:
# gh repo clone "${FORK_OWNER}/${REPO_NAME}"

cd "${REPO_NAME}" || { echo "cd failed; check REPO_NAME"; exit 1; }

# 2) Create the branch
git switch -c "${BRANCH}"

# 3) Create workflow + script + Makefile + README addition
mkdir -p .github/workflows .github/scripts

cat > .github/workflows/ci.yml <<'YML'
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.20'

      - name: Make scripts executable
        run: chmod +x .github/scripts/test.sh || true

      - name: Run repository checks
        run: .github/scripts/test.sh
YML

cat > .github/scripts/test.sh <<'BASH'
#!/usr/bin/env bash
set -euo pipefail

echo "Detecting project type and running checks..."

# Go checks
if [[ -f go.mod ]]; then
  echo "==> Detected Go project (go.mod found). Running go checks..."
  gofmt -l .
  go vet ./...
  go test ./... -v -race -coverprofile=coverage.out
  echo "Go checks completed."
fi

# Python checks
if [[ -f requirements.txt ]] || [[ -f pyproject.toml ]] || [[ -f setup.py ]]; then
  echo "==> Detected Python project. Running Python checks..."
  python -m pip install --upgrade pip setuptools wheel || true
  if [[ -f requirements.txt ]]; then
    python -m pip install -r requirements.txt || true/n# GitHub Copilot CLI SDKs

![GitHub Copilot SDK](./assets/RepoHeader_01.png)

Agents for every app.

Embed Copilot's agentic workflows in your application—now available in Technical preview as a programmable SDK for Python, TypeScript, Go, and .NET.

The GitHub Copilot SDK exposes the same engine behind Copilot CLI: a production-tested agent runtime you can invoke programmatically. No need to build your own orchestration—you define agent behavior, Copilot handles planning, tool invocation, file edits, and more.

## Available SDKs

| SDK                      | Location                                          | Installation                              |
| ------------------------ | ------------------------------------------------- | ----------------------------------------- |
| **Node.js / TypeScript** | [`cookbook/nodejs/`](./cookbook/nodejs/README.md) | `npm install @github/copilot-sdk`         |
| **Python**               | [`cookbook/python/`](./cookbook/python/README.md) | `pip install github-copilot-sdk`          |
| **Go**                   | [`cookbook/go/`](./cookbook/go/README.md)         | `go get github.com/github/copilot-sdk/go` |
| **.NET**                 | [`cookbook/dotnet/`](./cookbook/dotnet/README.md) | `dotnet add package GitHub.Copilot.SDK`   |

See the individual SDK READMEs for installation, usage examples, and API reference.

## Getting Started

For a complete walkthrough, see the **[Getting Started Guide](./docs/getting-started.md)**.

Quick steps:

1. **Install the Copilot CLI:**

   Follow the [Copilot CLI installation guide](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli) to install the CLI, or ensure `copilot` is available in your PATH.

2. **Install your preferred SDK** using the commands above.

3. **See the SDK README** for usage examples and API documentation.

## Architecture

All SDKs communicate with the Copilot CLI server via JSON-RPC:

```
Your Application
       ↓
  SDK Client
       ↓ JSON-RPC
  Copilot CLI (server mode)
```

The SDK manages the CLI process lifecycle automatically. You can also connect to an external CLI server—see the [Getting Started Guide](./docs/getting-started.md#connecting-to-an-external-cli-server) for details on running the CLI in server mode.

## FAQ

### Do I need a GitHub Copilot subscription to use the SDK?

Yes, a GitHub Copilot subscription is required to use the GitHub Copilot SDK. Refer to the [GitHub Copilot pricing page](https://github.com/features/copilot#pricing). You can use the free tier of the Copilot CLI, which includes limited usage.

### How does billing work for SDK usage?

Billing for the GitHub Copilot SDK is based on the same model as the Copilot CLI, with each prompt being counted towards your premium request quota. For more information on premium requests, see [Requests in GitHub Copilot](https://docs.github.com/en/copilot/concepts/billing/copilot-requests).

### Does it support BYOK (Bring Your Own Key)?

Yes, the GitHub Copilot SDK supports BYOK. You can configure the SDK to use your own encryption keys for data security. Refer to the individual SDK documentation for instructions on setting up BYOK.

### Do I need to install the Copilot CLI separately?

Yes, the Copilot CLI must be installed separately. The SDKs communicate with the Copilot CLI in server mode to provide agent capabilities.

### What tools are enabled by default?

By default, the SDK will operate the Copilot CLI in the equivalent of `--allow-all` being passed to the CLI, enabling all first-party tools, which means that the agents can perform a wide range of actions, including file system operations, Git operations, and web requests. You can customize tool availability by configuring the SDK client options to enable and disable specific tools. Refer to the individual SDK documentation for details on tool configuration and Copilot CLI for the list of tools available.

### Can I use custom agents, skills or tools?

Yes, the GitHub Copilot SDK allows you to define custom agents, skills, and tools. You can extend the functionality of the agents by implementing your own logic and integrating additional tools as needed. Refer to the SDK documentation of your preferred language for more details.

### Are there instructions for Copilot to speed up development with the SDK?

Yes, check out the custom instructions at [`github/awesome-copilot`](https://github.com/github/awesome-copilot/blob/main/collections/copilot-sdk.md).

### What models are supported?

All models available via Copilot CLI are supported in the SDK. The SDK also exposes a method which will return the models available so they can be accessed at runtime.

### Is the SDK production-ready?

The GitHub Copilot SDK is currently in Technical Preview. While it is functional and can be used for development and testing, it may not yet be suitable for production use.

### How do I report issues or request features?

Please use the [GitHub Issues](https://github.com/github/copilot-sdk/issues) page to report bugs or request new features. We welcome your feedback to help improve the SDK.

## Quick Links

- **[Getting Started](./docs/getting-started.md)** – Tutorial to get up and running
- **[Cookbook](./cookbook/README.md)** – Practical recipes for common tasks across all languages
- **[Samples](./samples/README.md)** – Video walkthroughs and sample projects

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for contribution guidelines.

## License

MIT
