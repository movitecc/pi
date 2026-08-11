# Offline `pi-subagents` bundle

This directory contains the npm packages required to install `pi-subagents` into a Pi Agent portable bundle without access to the public npm registry.

## Contents

- `pi-subagents-0.46.0.tgz` — Pi extension, skills, prompts, and builtin agents
- `jiti-2.7.0.tgz` — runtime dependency
- `typebox-1.1.38.tgz` — runtime dependency
- `yaml-2.8.3.tgz` — runtime dependency
- `SHA256SUMS` — integrity checksums

The package versions are pinned intentionally. Do not replace them with an untested version while offline.

## Linux portable Pi

Assume the portable bundle is extracted to `~/pi-portable/pi-agent-portable-linux-x64` and this directory is copied to `~/pi-subagents-offline`.

```bash
cd ~/pi-portable/pi-agent-portable-linux-x64
mkdir -p data/npm-cache data/pi-home

for f in ~/pi-subagents-offline/*.tgz; do
  ./runtime/bin/npm --cache "$PWD/data/npm-cache" cache add "$f"
done

PI_CODING_AGENT_DIR="$PWD/data/pi-home" \
npm_config_cache="$PWD/data/npm-cache" \
npm_config_offline=true \
./pi install ~/pi-subagents-offline/pi-subagents-0.46.0.tgz --no-approve

PI_CODING_AGENT_DIR="$PWD/data/pi-home" ./pi list
```

Start Pi with the same portable configuration directory:

```bash
cd ~/pi-portable/pi-agent-portable-linux-x64
PI_CODING_AGENT_DIR="$PWD/data/pi-home" ./pi
```

## Windows portable Pi

Assume the portable bundle is extracted to `C:\PiAgentPortable\pi-agent-portable-windows-x64` and this directory is copied to `C:\pi-subagents-offline`.

Open PowerShell:

```powershell
$PiRoot = 'C:\PiAgentPortable\pi-agent-portable-windows-x64'
$Offline = 'C:\pi-subagents-offline'
$env:PI_CODING_AGENT_DIR = "$PiRoot\data\pi-home"
$env:npm_config_cache = "$PiRoot\data\npm-cache"
$env:npm_config_offline = 'true'

New-Item -ItemType Directory -Force "$PiRoot\data\npm-cache" | Out-Null
New-Item -ItemType Directory -Force "$PiRoot\data\pi-home" | Out-Null

Get-ChildItem "$Offline\*.tgz" | ForEach-Object {
  & "$PiRoot\runtime\node\npm.cmd" --cache $env:npm_config_cache cache add $_.FullName
}

& "$PiRoot\pi.cmd" install "$Offline\pi-subagents-0.46.0.tgz" --no-approve
& "$PiRoot\pi.cmd" list
```

Start it later with the same environment variables:

```powershell
$env:PI_CODING_AGENT_DIR = "$PiRoot\data\pi-home"
& "$PiRoot\pi.cmd"
```

## Automatic instructions for Pi Agent

After installation, send one of these prompts to Pi:

```text
Show me the available subagents.
```

```text
Use scout to inspect this codebase before planning.
```

```text
Use reviewer to review the current diff.
```

```text
Check whether subagents and intercom are set up correctly.
```

The last prompt should invoke the package doctor. The explicit command is:

```text
/subagents-doctor
```

## Verify integrity before installing

Linux/macOS:

```bash
cd ~/pi-subagents-offline
sha256sum -c SHA256SUMS
```

PowerShell:

```powershell
Get-FileHash .\*.tgz -Algorithm SHA256
```

Compare the result with `SHA256SUMS`.

## Important notes

- Do not run `npm install -g pi-subagents`; use Pi's package installer.
- The Pi portable bundle must match the package's supported Pi API. This bundle was prepared for Pi `0.83.x` and `pi-subagents 0.46.0`.
- Keep `PI_CODING_AGENT_DIR` pointed at the portable `data\pi-home` or `data/pi-home`; otherwise Pi may modify the host user's normal `~/.pi/agent` configuration.
- The subagent child processes still need model credentials and any required network access when they actually run. This bundle only makes package installation offline.
- Review third-party package source before enabling it. Pi packages can execute code and change agent behavior.

## Source

- Package page: https://pi.dev/packages/pi-subagents
- npm package: `pi-subagents@0.46.0`
