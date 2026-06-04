---
title: "Crazyrouter Codex CLI: Use Codex with One API Key and an OpenAI-Compatible Gatew"
emoji: "⚡"
type: "tech"
topics: [codex, ai, cli, api]
published: true
---

# Crazyrouter Codex CLI: Use Codex with One API Key and an OpenAI-Compatible Gateway

OpenAI Codex CLI is useful when you want an AI coding agent directly inside your terminal. The painful part is not the idea — it is the setup: API keys, base URLs, model names, Windows environment variables, macOS shell profiles, Linux config files, and different providers for different models.

The new [`crazyrouter-codex-cli`](https://github.com/xujfcn/crazyrouter-codex-cli?utm_source=blog&utm_medium=article&utm_campaign=codex_cli) repo solves one specific problem:

> connect Codex CLI to Crazyrouter with one API key and an OpenAI-compatible API endpoint.

Repository:

```text
https://github.com/xujfcn/crazyrouter-codex-cli
```

## What this repo does

The repo provides simple install scripts for:

- Windows PowerShell
- Windows batch file
- macOS
- Linux

It configures Codex CLI to use:

```bash
OPENAI_BASE_URL=https://cn.crazyrouter.com/v1
OPENAI_API_KEY=your-crazyrouter-key
```

That means Codex CLI can talk to Crazyrouter through an OpenAI-compatible interface, while Crazyrouter handles the model/provider side.

Important rule:

> Do not add UTM parameters to API endpoints. UTM belongs on human-clickable website links, not `OPENAI_BASE_URL`.

Correct:

```bash
export OPENAI_BASE_URL=https://cn.crazyrouter.com/v1
```

Wrong:

```bash
export OPENAI_BASE_URL=https://cn.crazyrouter.com/v1?utm_source=...
```

## Why use Codex CLI through Crazyrouter?

A terminal coding agent is most useful when it can become part of your normal development loop:

1. open a project directory;
2. ask the agent to inspect code;
3. let it patch files;
4. run tests;
5. review the diff;
6. repeat.

But real developer teams often want more than one model. Some tasks need a fast low-cost model. Some need a stronger reasoning model. Some need Claude-style code review. Some need Gemini-style long-context analysis. Some teams also need a more stable route from regions where direct access is unreliable.

Crazyrouter gives Codex CLI a single OpenAI-compatible gateway:

- one API key;
- one base URL;
- multiple model choices;
- OpenAI-compatible client configuration;
- easier switching between coding models.

## One-command install

### Windows PowerShell

Open PowerShell as a normal user and run:

```powershell
iwr -UseB https://raw.githubusercontent.com/xujfcn/crazyrouter-codex-cli/main/install-crazyrouter-codex.ps1 | iex
```

Or download and run:

```text
install-crazyrouter-codex.bat
```

### macOS / Linux

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-codex-cli/main/install-crazyrouter-codex.sh | bash
```

The script asks for your Crazyrouter API key, writes the needed environment variables, and backs up existing Codex configuration when needed.

## Manual setup

If you prefer to configure it yourself, install Codex CLI first:

```bash
npm install -g @openai/codex
```

Node.js 22+ is recommended.

Then set the environment variables.

### macOS / Linux

```bash
export OPENAI_API_KEY=sk-your-crazyrouter-key
export OPENAI_BASE_URL=https://cn.crazyrouter.com/v1
```

### Windows PowerShell

```powershell
setx OPENAI_API_KEY "sk-your-crazyrouter-key"
setx OPENAI_BASE_URL "https://cn.crazyrouter.com/v1"
```

After `setx`, reopen your terminal.

Then start Codex:

```bash
codex
```

## Codex `config.toml` example

Some Codex CLI versions support provider configuration in:

- Windows: `%USERPROFILE%\.codex\config.toml`
- macOS / Linux: `~/.codex/config.toml`

Example:

```toml
model = "gpt-5.5"
model_provider = "crazyrouter"

[model_providers.crazyrouter]
name = "Crazyrouter"
base_url = "https://cn.crazyrouter.com/v1"
env_key = "OPENAI_API_KEY"
wire_api = "responses"

[model_providers.crazyrouter.query_params]
```

If you see this error:

```text
wire_api = "chat" is no longer supported
```

change:

```toml
wire_api = "chat"
```

to:

```toml
wire_api = "responses"
```

## Model selection

Once the gateway is configured, you can start Codex with the default model or specify one explicitly:

```bash
codex
codex --model gpt-5.5
codex --model gpt-4o-mini
codex --model claude-sonnet-4-6
```

Model availability can vary by account, provider route, and current upstream status. Check the current model list here:

[Crazyrouter model list](https://crazyrouter.com/models?utm_source=blog&utm_medium=article&utm_campaign=codex_cli)

## Practical workflow example

A simple Codex CLI coding loop:

```bash
cd your-project
codex
```

Then ask:

```text
Inspect this repo and explain the architecture. Do not edit files yet.
```

After the summary:

```text
Find the smallest safe fix for the failing login test. Make the change, then run the relevant test only.
```

Then review:

```bash
git diff
npm test -- login
```

The gateway setup does not replace careful review. It makes the model/provider configuration less annoying so you can focus on the actual engineering loop.

## Troubleshooting

### `codex: command not found`

Install Codex globally:

```bash
npm install -g @openai/codex
```

Then check:

```bash
which codex
codex --help
```

On Windows, reopen the terminal after installation.

### API key not found

Check whether the environment variable exists.

macOS / Linux:

```bash
echo $OPENAI_API_KEY
```

Windows PowerShell:

```powershell
$env:OPENAI_API_KEY
```

If empty, set it again.

### Wrong base URL

The base URL should be exactly:

```text
https://cn.crazyrouter.com/v1
```

Do not add `/chat/completions`, `/responses`, or UTM parameters. Client libraries append the final API path themselves.

### Existing Codex config conflict

If you had another provider configured before, check:

```bash
cat ~/.codex/config.toml
```

Make sure the selected `model_provider` points to the Crazyrouter provider block.

## When this setup is useful

This repo is especially useful for developers who:

- use Codex CLI but want an OpenAI-compatible gateway;
- want to try multiple models from one CLI setup;
- work across Windows, macOS, and Linux machines;
- need a repeatable install script for teammates;
- want a simpler path for AI coding workflows in regions where direct provider access can be unstable.

## Links

- GitHub repo: [xujfcn/crazyrouter-codex-cli](https://github.com/xujfcn/crazyrouter-codex-cli?utm_source=blog&utm_medium=article&utm_campaign=codex_cli)
- Crazyrouter: [crazyrouter.com](https://crazyrouter.com?utm_source=blog&utm_medium=article&utm_campaign=codex_cli)
- Model list: [crazyrouter.com/models](https://crazyrouter.com/models?utm_source=blog&utm_medium=article&utm_campaign=codex_cli)
- Docs: [crazyrouter.com/docs](https://crazyrouter.com/docs?utm_source=blog&utm_medium=article&utm_campaign=codex_cli)

## Bottom line

`crazyrouter-codex-cli` is a small repo, but it removes a common setup tax: configuring Codex CLI to use an OpenAI-compatible gateway correctly.

If you want Codex CLI with one key, one base URL, and easier model routing, start here:

```text
https://github.com/xujfcn/crazyrouter-codex-cli
```

---

Canonical: https://crazyrouter.com/blog/crazyrouter-codex-cli-one-command-setup?utm_source=zenn&utm_medium=article&utm_campaign=codex_cli
