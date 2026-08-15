# Working Policy for This Repository

## Repository Boundary

This repository is a public documentation portfolio for the `basement-node` infrastructure lab. It is not the live system configuration, and its documentation may describe historical rather than current state.

- Treat the repository and the live `basement-node` system as separate environments.
- Do not assume a documented service, firewall rule, backup, tunnel, mount, or other component is still active.
- Distinguish clearly between repository state, historical observations, planned work, and verified live state.
- Do not inspect or change the live system unless the user explicitly authorizes that specific work.

## Default Behavior

- Use read-only inspection by default. Do not modify files unless the user explicitly asks for a change.
- Before editing, inspect the relevant files, nearby documentation, Git status, and any history needed to understand the change.
- Ask before changing anything whose scope or intended outcome is ambiguous.
- When editing is authorized, make the smallest coherent change that satisfies the request. Do not include unrelated cleanup or rewrites.
- Preserve unrelated tracked, untracked, and ignored files.

## Sensitive Information

Never expose or commit secrets, tokens, credentials, passwords, private keys, certificates, recovery codes, session data, or authentication material.

- Do not add non-public infrastructure details such as private addresses, internal DNS records, tunnel identifiers, firewall specifics, or sensitive storage paths unless the user explicitly approves them for publication.
- Preserve existing placeholders and redactions. Do not reconstruct redacted values from other files or Git history.
- Treat ignored files, raw notes, command output, and repository history as potentially sensitive.
- If sensitive material is discovered, report its presence and location without reproducing its value.
- Review every proposed diff for accidental disclosure before considering the work complete.

## Documentation and History

- Preserve useful historical documentation rather than silently rewriting it as current fact.
- Keep dates and validation context when they explain when a claim was observed.
- When older information is stale, prefer a dated clarification or explicit status update over erasing the historical record.
- Do not delete historical notes, restore intentionally excluded personal-use material, or substantially rewrite past records unless explicitly asked.
- When documents conflict, identify the conflict and use available dates and evidence to explain the resolution. Ask the user if the authoritative state remains unclear.

## Validation and Diff Review

After an authorized edit:

1. Inspect the complete Git diff and confirm that only intended files changed.
2. Check relevant Markdown links, formatting, dates, status claims, and cross-document consistency.
3. Check the diff for secrets and accidentally unredacted infrastructure details.
4. Show and explain meaningful changes to the user.
5. Report the validation performed, remaining uncertainty, and anything that could not be verified.

Repository checks validate the documentation only; they do not prove the health or current state of the live system.

## Actions Requiring Explicit Authorization

Do not perform any of the following unless the user explicitly authorizes the specific action and scope:

- Destructive filesystem or Git commands, deletion, or discarding user changes
- Git history rewriting, commits, pushes, force-pushes, or publication actions
- Package installation, removal, or upgrades
- Service or container starts, stops, restarts, reloads, or enablement changes
- Reboots or shutdowns
- Firewall, DNS, tunnel, routing, storage, mount, backup, account, or other infrastructure changes
- Changes to Cloudflare, Tailscale, Gmail, domains, or other external services

Prefer read-only inspection when authorization does not clearly cover a state-changing action.
