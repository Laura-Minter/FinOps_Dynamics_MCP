# Dynamics MCP — local client

The machine-side half of [Dynamics MCP](https://dynamics-mcp.com), for Dynamics
365 Finance & Operations development.

It writes AOT objects, runs the compiler, synchronises the database, and indexes
your models. It does not decide *what* to write — that comes from the hosted
service, which resolves against a real D365FO metadata index.

## Why it exists

Two things can only happen on the machine that holds your `PackagesLocalDirectory`:

**Encoding.** D365FO reads AOT XML as UTF-8 **with** a byte order mark. Most
editors and file tools write UTF-8 *without* one, and the resulting file fails at
build time with a unicode-substitution error that names no file — so the symptom
appears far from the write that caused it. This client always writes the BOM.

**Everything downstream of it.** `xppc`, `SyncEngine`, the best-practice checker,
the test console and your `.rnrproj` all live here.

## Requirements

- Windows with .NET Framework 4.8 — included in Windows 10 1903+ and Windows
  Server 2019+, so there is normally nothing to install
- A D365FO development environment (`PackagesLocalDirectory`)
- An active Dynamics MCP subscription

No Node.js, no runtime download, no build step.

## Install

Download the contents of `bin/` and put them in a folder of your choice, for
example `C:\Tools\dynamics-mcp\`. Keep the files together — the `.dll`s beside
the `.exe` are its dependencies.

## Configure

Add it to your MCP client alongside the hosted server. In VS Code, `mcp.json`:

```json
{
  "servers": {
    "dynamics-local": {
      "type": "stdio",
      "command": "C:\\Tools\\dynamics-mcp\\dynamics-mcp.exe",
      "env": {
        "DYNAMICS_MCP_API_KEY": "<your API key>"
      }
    },
    "dynamics-cloud": {
      "type": "http",
      "url": "https://dynamics-mcp.com/api/mcp",
      "headers": { "Authorization": "Bearer <your API key>" }
    }
  }
}
```

Both halves take the same key. Restart your editor afterwards.

### Optional settings

| Variable | Default | Meaning |
|---|---|---|
| `DYNAMICS_MCP_API_KEY` | — | **Required.** Your subscription key. |
| `DYNAMICS_MCP_URL` | `https://dynamics-mcp.com` | Service base URL. |
| `D365FO_PACKAGE_PATH` | auto-detected | `PackagesLocalDirectory`. Set it if the scan does not find it. |
| `D365FO_MODEL_NAME` | — | Default target model, so calls need not name one. |
| `D365FO_PROJECT_PATH` | — | `.rnrproj` to register new objects in. |
| `EXTENSION_PREFIX` | — | Object-name prefix. |
| `D365FO_SQL_CONNECTION` | local AxDB | Connection string used by database sync. |

The packages root is found by scanning fixed drives, because the letter varies by
VM image — `K:` on cloud-hosted environments, `C:` on the downloadable VHD, `J:`
on newer ones. Set `D365FO_PACKAGE_PATH` if yours is somewhere else.

## Tools

| Tool | What it does |
|---|---|
| `get_workspace_info` | Packages root, target model, project, prefix, custom models |
| `create_model` | New model: package directories and descriptor |
| `write_aot_file` | Write an AOT object as UTF-8 with BOM, register it in the project |
| `write_label` | Add or update a label, per language |
| `verify_objects` | Confirm objects exist on disk *and* in the project |
| `undo_last_write` | Reverse the most recent write from this session |
| `build_model` | Compile with `xppc`, returning the diagnostics |
| `sync_database` | Run a database synchronisation |
| `run_bp_check` | Microsoft best-practice checker |
| `run_systest` | Execute a SysTest class |
| `index_model` | Upload your model's metadata so the service can resolve against it |

## About `index_model`

The hosted index knows Microsoft's models. It does not know yours, so a newly
created `EX_PayerId` is invisible to it until you run this.

What is uploaded: object names, types, and the facts needed to resolve a field's
type — an EDT's base type, what it extends, its enum or string size.

What is **not** uploaded: your method bodies. Your X++ source stays on your
machine.

Run it after creating or changing objects.

## Subscription

Every tool call verifies your subscription against the service, cached for a few
minutes. Without an active subscription the tools refuse and say why.

The client cannot generate anything on its own — it holds no index and no
generation rules. It writes what the hosted service produces.

## Support

<https://dynamics-mcp.com> · support@dynamics-mcp.com
