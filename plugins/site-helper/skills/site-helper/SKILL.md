---
name: site-helper
description: Use the central Site Helper connector for live-site work, SEO/site maintenance, AgentDB, Strapi, GitHub, SSH/SFTP, htaccess, subdomains, diagnostics, and documented Site Helper workflows. Use when the user asks to inspect, troubleshoot, change, deploy, compare, or maintain one of the managed sites or the Site Helper system.
---

# Site Helper

## Connector-first rule

This is an end-user Site Helper plugin. Do not require or assume a local checkout of `site-helper-mcp-chatgpt`. The development repository is for maintainers only.

At the start of every Site Helper working session, call `site_helper_bootstrap` before using other Site Helper tools. Treat the returned instructions as the current operating rules for this session.

If the task matches a documented workflow, use `list_playbooks` when needed and then call `get_playbook` for the relevant playbook before acting. The central MCP runtime is the source of truth for agent rules and playbooks.

## Default workflow

1. Call `site_helper_bootstrap`.
2. Read the relevant playbook when the task is procedural.
3. Resolve the target site with `check_site_config` before site changes.
4. Inspect before modifying.
5. Make only the changes the user requested.
6. Re-read or re-scan the result after each change.
7. Use the browser for live visual verification when relevant.
8. Report what changed, what was verified, and any remaining issue.

## Safety

- Do not perform write operations merely to test access.
- For an access check, use read-only tools such as `ping`, `list_sites`, `strapi_ping`, `gh_ping`, `site_helper_bootstrap`, and `list_playbooks`.
- Do not expose credentials or secret values.
- Never substitute a local repository file for centrally served instructions or playbooks when the connector is available.
- If the connector is unavailable, say so clearly instead of silently switching to a different operating model.

## Useful first check

For a new installation or troubleshooting session, call:

- `site_helper_bootstrap`
- `ping`
- `list_sites`
- `strapi_ping`
- `gh_ping`

Then summarize connection status and the available Site Helper capabilities without making changes.
