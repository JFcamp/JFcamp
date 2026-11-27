# Agent bridge (Claude Code ⇄ Kiro CLI)

Este projeto é desenvolvido com o [agent-bridge](https://github.com/Kjanray/agent-bridge), um servidor MCP que permite ao Claude Code delegar tarefas ao `kiro-cli` (e vice-versa).

Nada desta configuração fica no repositório: ela vive na máquina de cada desenvolvedor. Este arquivo documenta como reproduzi-la.

## Pré-requisitos

- git, Python 3.10+, Claude Code (`claude`)
- Kiro CLI (Windows 11, PowerShell): `irm 'https://cli.kiro.dev/install.ps1' | iex`
- Login gratuito no Kiro: `kiro-cli login --license free`

## Instalação

```powershell
git clone https://github.com/Kjanray/agent-bridge "$HOME/tools/agent-bridge"

# Claude Code
claude mcp add --scope user agent-bridge -- python ~/tools/agent-bridge/server.py

# Kiro: agentes e regras
Copy-Item ~/tools/agent-bridge/kiro/worker.json, ~/tools/agent-bridge/kiro/trusted-dev.json ~/.kiro/agents/
Copy-Item ~/tools/agent-bridge/rules.md ~/.kiro/steering/agent-bridge.md   # ajuste o caminho ~/agent-bridge -> ~/tools/agent-bridge
```

Em `~/.kiro/settings/mcp.json`, dentro de `mcpServers`, sem remover entradas existentes:

```json
"agent-bridge": {
  "command": "python",
  "args": ["C:/Users/<usuario>/tools/agent-bridge/server.py"]
}
```

Faça backup de `~/.kiro/settings/mcp.json`, `~/.kiro/agents/` e `~/.kiro/steering/` antes de alterar.

## Regras de uso neste projeto

- Delegações ao Kiro usam `mode="read_only"` (padrão) ou `mode="write"`. **Não usar `mode="auto"`.**
- `trusted-dev` é usado só por sessão: `kiro-cli chat --agent trusted-dev`. **Não** defini-lo como padrão global (`kiro-cli agent set-default`).
- Nunca versionar `~/.kiro`, `.env`, tokens, chaves ou credenciais (ver `.gitignore`).

## Verificação

1. `claude mcp get agent-bridge` → `Status: ✔ Connected`
2. No Claude Code (após reiniciá-lo): `ask_kiro` com `mode="read_only"` e um prompt simples, depois `check_task(task_id)` → `status: "ok"`.
3. `kiro-cli agent list` → `trusted-dev` e `worker` listados como Global; `kiro_default` continua marcado como padrão.
