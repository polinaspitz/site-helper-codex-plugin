---
name: site-helper
description: Use the central Site Helper connector for live-site work, SEO/site maintenance, AgentDB, Strapi, GitHub, SSH/SFTP, htaccess, subdomains, diagnostics, shared hosting coordination, and documented Site Helper workflows. Use when the user asks to inspect, troubleshoot, change, deploy, compare, or maintain one of the managed sites or the Site Helper system.
---

# Site Helper

## Connector-first rule

This is an end-user Site Helper plugin. Do not require or assume a local checkout of `site-helper-mcp-chatgpt`. The development repository is for maintainers only.

At the start of every Site Helper working session, call `site_helper_bootstrap` before using other Site Helper tools. Treat the returned instructions as the current operating rules for this session.

If the task matches a documented workflow, use `list_playbooks` when needed and then call `get_playbook` for the relevant playbook before acting. The central MCP runtime is the source of truth for agent rules and playbooks.

## Shared hosting traffic light

Site Helper participates in a shared hosting-account lease with Claude and other internal agents. The lease is enforced automatically inside SFTP operations.

- Never bypass, hammer, or retry around a lease refusal. A busy, yellow, or red account is a coordination decision, not a transient error to brute-force.
- When access is refused or when the user asks who is working where, call `traffic_light` and explain the current holder, site, cooldown/block, and remaining wait.
- `traffic_light_release` may be used only to release the current Codex user's own lease when work on that hosting account is genuinely finished. Never use it to release another user or another service.
- If `traffic_light` reports the current holder as `site-helper-mcp-chatgpt:anon`, tell the user that `SITE_HELPER_MCP_USER` is missing or not reaching the connector. Do not treat `anon` as a correctly configured team identity.
- Do not confuse a shared red/cooldown state with a bad SSH password. Wait or switch accounts according to the traffic-light message.

## Default workflow

1. Call `site_helper_bootstrap`.
2. Read the relevant playbook when the task is procedural.
3. Resolve the target site with `check_site_config` before site changes.
4. Use `traffic_light` when account availability is relevant or after a lease refusal.
5. Inspect before modifying.
6. Make only the changes the user requested.
7. Re-read or re-scan the result after each change.
8. Use the browser for live visual verification when relevant.
9. Report what changed, what was verified, and any remaining issue.

## Safety

- Do not perform write operations merely to test access.
- For an access check, use read-only tools such as `ping`, `list_sites`, `strapi_ping`, `gh_ping`, `site_helper_bootstrap`, `list_playbooks`, and `traffic_light`.
- Do not expose credentials or secret values.
- Never substitute a local repository file for centrally served instructions or playbooks when the connector is available.
- If the connector is unavailable, say so clearly instead of silently switching to a different operating model.
- Never attempt to force-release another holder's hosting lease.

## Useful first check

For a new installation or troubleshooting session, call:

- `site_helper_bootstrap`
- `ping`
- `list_sites`
- `strapi_ping`
- `gh_ping`
- `traffic_light`

Then summarize connection status, current holder identity, and the available Site Helper capabilities without making changes.
