# Security and reliability foundation

Status: proposed hardening workstream; **not a completed security audit**.
Upstream: https://github.com/noblehacks/frenemy
Baseline: Frenemy v2.0.0 (16 November 2026).

## Intent
Improve general-purpose safety and cross-platform reliability without changing the upstream CLI/MCP contract. Upstreamable fixes must remain independent of Heaventree's internal infrastructure. Keep `main` aligned with upstream and use focused PRs.

## Trust boundaries
1. **User → driver AI**: user authorisation cannot be inferred from another model's output.
2. **Driver → relay → worker CLI**: prompts, tool outputs, and model instructions are untrusted input. Worker permissions must be enforced outside prompts.
3. **Worker → filesystem**: file allowlists and private working copies prevent accidental collisions but are not operating-system isolation. No production secrets or deployment credentials in the worker environment.
4. **Workspace → apply/recover**: validate scope, symlink escapes, replacement races, file modes, content hashes and interrupted writes.
5. **Local logs → users/providers**: prompts, file paths, excerpts and CLI arguments can reveal sensitive material. Do not upload `.fleet`, `.worker-state` or CLI logs.

## First audit gates (P0)
- [ ] Inventory every subprocess spawn, shell invocation, inherited environment variable and command-line argument.
- [ ] Test hostile paths (spaces, Unicode, traversal, symlinks, junctions, Windows drive/UNC paths) in prepare, apply, abandon and recovery.
- [ ] Test that delegated workers cannot apply their own jobs and that overlapping/changed files are rejected.
- [ ] Verify process-tree termination when a worker times out, loses quota or is cancelled; fail closed on unverified termination.
- [ ] Check prompt and output handling for instruction injection, terminal escape sequences, secret leakage and output limits.
- [ ] Review command execution approvals and document exactly which providers can run tools non-interactively.
- [ ] Run release tests on clean Windows, macOS and Linux environments; distinguish stand-in CLI tests from live signed-in tests.
- [ ] Verify release archive checksums and inspect installer changes before enabling auto-connect.

## Acceptance criteria for first upstream PR
1. One independently reproducible failure, or an objectively missing regression test, with a concise threat model.
2. A minimal source change that preserves existing behaviour for valid users.
3. Before/after tests (including a Windows case when applicable).
4. A documented compatibility and rollback plan.
5. No Heaventree-specific service URL, credential, integration or project detail in the upstream patch.

## Heaventree-only extensions (not upstream defaults)
- Optional GitHub PR review/approval bridge, and Nifty status reporting.
- Super Router adapter, kept outside core MCP relays.
- Production deployment approval gates enforced by Heaventree's own control plane.

## Important limitations
Frenemy v2's worker safeguards are expressly **cooperative**, not a security sandbox. A malicious or compromised same-user CLI may bypass file-level scope. We must not describe the platform as securely sandboxed without implementing and testing genuine OS-level isolation. 
